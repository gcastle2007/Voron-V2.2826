## Макрос PRINT_START

Идеология данного макроса заключается в том, чтобы разбить процесс подготовки печати на этапы, нагреть стол, убедиться, что средний PID стола стал ниже 25% (а это значит, что стол прогрелся):
- Prepare - самая первая стадия. До этого идёт задание параметров для необходимых переменных.
- Heating bed - стадия запуска прогрева стола. Также запускается предварительный нагрев экструдера.
- HeatSoak - стадия догрева стола. В данном макросе применена идеология того, что когда стол прогревается, то средняя величина PID прогрева стола уменьшается. В моём случае ждём, когда PID станет 25%, после этого можно переходить на следующую стадию.
- QGL - делаем Quad Gantry Level для гантри.
- BedMesh - в засимости от необходимости, либо перестраиваем полностью Bed Mesh, либо подгружаем уже заранее сохранённую карту в зависимости от температуры.
- Final - последняя стадия, в которой догревается экструдер, чистится сопло, калибркуется Z-Offset через CALIBRATE_Z, прогоняется PRIME_LINE и далее уже идёт печать.

Скрипт периодически может корректироваться и меняться, в зависимости от моих потребностей.

###Разберём подробно все этапы и действия.

```
[gcode_macro PRINT_START]
```
### Инициализация необходимых переменных при запуске Klipper:
```
variable_retract: 0                   # задание ретракта
##  User Paramaters
##  BED_TEMP      : Target temperature for the Bed. Is also used to decide 
##                  if heatsoak is needed
##  EXTRUDER_TEMP : Target temperature for the Extruder
##  RETRACT_LEN   : Retract length for firmware retracts
##  RETRACT_SPD   : Retract speed for firmware retracts
##  System Varables
# change this to define the wait time per call
variable_extruder: 245                # температура экструдера по умолчанию
variable_extruder_pre: 171            # температура преднагрева экструдера по умолчанию
variable_bed: 100                     # температура стола по умолчанию
variable_print_info: 'true'
## Valid state 
##   Prepare : decision if heat Soak is needed 
##   HeatSoak: loop the time specified with SOAK
##   Final   : all what needs to be done after wait timet
variable_state: 'Prepare'             # переменная state - задаёт этап выполнения подготовки к печати. По умолчанию стадия Prepare
variable_first: 'true'
variable_soak: 0.35                   # ожидаемое значание среднеро PWM стола. Идея состоит в том, чтобы снять некоторое кол-во показаний PWM и высчитать среднее.
variable_pwm: 1                       # начальное значение PWM
variable_avgpwm: 1                    # среднее накопительное значение PWM
variable_tests: 0                     # счётчик опросов PWM
variable_left: 30                     # кол-во опросов PWM. Опросы будут проводиться 1 раз в секунду. Соответственно 30 раз.
variable_clean2: False
variable_bmeshv: False                # перепенная, отвечающая за необходимость выполнения BED_MESH_CALIBRATE (True/False)
gcode:
  {% set extruder_temp = params.EXTRUDER_TEMP|default(240)|float %}   # Получание значения температуры экструдера из слайсера
  {% set bed_temp = params.BED_TEMP|default(80)|float %}              # Получание значения температуры стола из слайсера
  {% set E = printer["gcode_macro PRINT_START"].retract|float %}      # Retract - variable from this macros
#  {% set soak = params.SOAK|default(0.25) %}                         # 
  {% set soak = printer["gcode_macro PRINT_START"].soak %}            # Значение ожидаемого PID
  {% set actPwm = printer.heater_bed.power|float %}                   # Получение актуального PWD для стола
  {% set cham_temp = params.CHAMBER|default(40)|float %}              # температура нагрева камеры. В настоящее время мной не используется
  {% set filament_name = params.FILAMENT%}                            # поределение типа филамента из слайсера. исходя из этого я запускаю Nevermore или нет
  {% set bmesh = params.BMESH|default("no") %}                        # Нужно ли собирать новый bed mesh, yes/no задаётся через слайсер

#############  BED temp values  #############
  # get actual temp from extra sensor or heater sensor / получение актуальной температуры стола с дополнительного сенсора или с термистора в столе
  {% if 'temperature_sensor bed' in printer %}                             # проверка на наличие дополнительного сенсора
    {% set actBed = printer['temperature_sensor bed'].temperature|int %}   # если дополнительный сенсор есть, то actBed (актуальная температура стола) равен данным с температуры этого сенсора
  {% else %}
    {% set actBed = printer.heater_bed.temperature|int %}                  # иначе actBed равна температуры с термистора стола по умолчанию
  {% endif %}
  # get max allow bed temp from config. Lower it by 5C to avoid shutdown
  {% set cfg_bed_max = printer.configfile.settings.heater_bed.max_temp|int - 5 %} # получение максимальной температуры стола, которую можно использовать. Это максимальная температура, указанная в конфиге минус 5 градусов.
```

### Подготовительная стадия, называется Prepare:
```
#############  Prepare Stage #############
  {% if state == 'Prepare' %}  
#    _PRINT_AR T="Preparation stage"
    M117 Preparation stage
    {% set bed_temp = params.BED_TEMP|default(80)|int %}
    {% set extruder_temp = params.EXTRUDER_TEMP|default(240)|int %}
    {% set extruder_temp_pre = (extruder_temp|float * 0.7)|int %}
#    {% set soak = params.SOAK|default(0.35) %}
  {% set soak = printer["gcode_macro PRINT_START"].soak %}  
```

### Стадия сохранения значений глобальных переменных:
```
#############  Variable setup  #############
    CLEAR_PAUSE
    SET_GCODE_VARIABLE MACRO=CANCEL_PRINT VARIABLE=execute VALUE='"false"'                # macros CANCEL_PRINT must have variable "execute" with value "false"
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=extruder VALUE={extruder_temp}          # extruder temperature
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=extruder_pre VALUE={extruder_temp_pre}  # extruder pre-heat
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=bed VALUE={bed_temp}                    # target bed temperature
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=bmeshv VALUE={bmesh}                    # bed mesh new or not new
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=left VALUE=30                           # counter for PWM calculate
```

### Стадия нагрева стола:
```
#############  Heating bed  #############
    LED_ON                                                                       # включение освещения внутри принтера
    # start Nevermore if ABS
    {% if "ABS" in filament_name %}                                              # если печатается SBD, то включается Nevermore, чтобы фильтровать воздух в режиме рециркуляции внутри камеры
    #      NEVERMORE_ON
       SET_FAN_SPEED FAN=Nevermore SPEED=0.95
    {% else %}
      NEVERMORE_OFF                                                              # иначе не включается
    {% endif %}
    _PRINT_AR T="Heating Bed"                                                    # выводим сообщение от стади на грева стола
    {action_respond_info("Heating Bed")}
    M140 S{bed_temp|int}                                                         # запускаем нагрев стола и не ждём
    M104 S{extruder_pre|int}  ; heat bed and wait                                # запускаем преднагрев экструдера и ждём
    G28                                                                          # хомимся
    G90                                                                          # координаты абсолютные
    G0 Z30 F1500                                                                 # поднимем голову на 30мм
    G0 X175 Y175 F24000                                                          # переместим её в центр стола
    M106 S90              ; switch part cooling ~35% to move air in chamber      # включим, примерно, на 35% фаны охлаждения модели, чтобы перепешивать воздух в камере
    M190 S{bed_temp|int}  ; heat bed and wait                                    # нагреваем стол, но уже ждём
    PAUSE_BASE                                                                   # PAUSE!!!! важный момент, если этого не сделать, то получим начало печати раньше времени
    {% set pwm = printer['heater_bed'].power | float %}                          # получаем PWM от нагревателя стола
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=pwm VALUE={pwm}                # сохраняем глобально значения переменных
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=avgpwm VALUE={pwm}             # ...
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=tests VALUE=1                  # ...
    _PRINT_AR T="{"T:%02d/30 P:%.3f/%.3f" % (left|int,pwm|float,soak|float)}"    # выводим сообщение
    # Call the wait macro the first time
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=state VALUE='"HeatSoak"'       # устанавливаем стадию догрева стола - HeatSoak
    UPDATE_DELAYED_GCODE ID=START_PRINT_WAIT DURATION=0.1                        # вывваем скрипт START_PRINT_WAIT на выполнение, но с задержкой старта с 0.1 секунды
```

### Стадия догрева стола до PID = 35%:
```
#############  HeatSoak #############
  {% elif state == 'HeatSoak' %}
    M117 HeatSoak                                                                # Вывод на экран сообщения о догреве стола
    {action_respond_info("HeatSoak")}
    {% if left == 0 %}
      {% if avgpwm >= soak|float %}
        SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=pwm VALUE=0                # сброс значения PWM в ноль
        SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=tests VALUE=0              # сброс счётчика тестовых запросов в ноль
        SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=left VALUE=30              # кол-во опросов ставим 30. Ставятся с задержкой 30 секунд.
      {% else %}
        {action_respond_info("Done. Mean PWM: %f" % (avgpwm|float))}
        SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=state VALUE='"QGL"'        # Переходим на стадию Quad Gantry Level (выравнивание портала)
        {action_respond_info("Must made Bed Mesh")}
      {% endif %}
    {% endif %}
    UPDATE_DELAYED_GCODE ID=START_PRINT_WAIT DURATION=1
```

### Стадия выравнивания портала:
```
#############  QGL #############
  {% elif state == 'QGL' %}
    M106 S0                                                                      # выключаем вентиляторы обдува модели
    M117 Quad Gantry Level                                                       # сообщение на экран
    QUAD_GANTRY_LEVEL                                                            # выравниваем портал
    G28 Z                                                                        # хомим снова Z, т.к. если у нас портал был искривлён, то предыдущий хоминг не корректен
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=state VALUE='"BedMesh"'        # переводим стадию на сборку Bed Mesh
    UPDATE_DELAYED_GCODE ID=START_PRINT_WAIT DURATION=0.1
```

### Стадия калибровки меша (BED_MESH_CALIBRATE):
```
#############  Bed Mesh calibrate #############
  {% elif state == 'BedMesh' %}
    _PRINT_AR T="Bed mesh stage"
    {action_respond_info("Bed Mesh Calibrate")}
    {% if bmeshv %}
      {action_respond_info("New bed mesh create")}
      _PRINT_AR T="New bed mesh create"
      BED_MESH_CALIBRATE
    {% else %}
      {% if bed_temp>=110 %}
        BED_MESH_PROFILE LOAD=PEI-110T
      {% endif %}
      {% if bed_temp>=100 and bed_temp<110 %}
        BED_MESH_PROFILE LOAD=PEI-100T
      {% endif %}
      {% if bed_temp>=90 and bed_temp<100 %}
        BED_MESH_PROFILE LOAD=PEI-90T
      {% endif %}
      {% if bed_temp>=80 and bed_temp<90 %}
        BED_MESH_PROFILE LOAD=PEI-80T
      {% endif %}
      {% if bed_temp>=70 and bed_temp<80 %}
        BED_MESH_PROFILE LOAD=PEI-70T
      {% endif %}
      {% if bed_temp>=60 and bed_temp<70 %}
        BED_MESH_PROFILE LOAD=PEI-60T
      {% endif %}
    {% endif %}
    G90
    G0 Z30 F1500                                                                 # Z to low 30
    G1 X175 Y175 F30000                                                            # head to center of bed
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=state VALUE='"Final"'
    UPDATE_DELAYED_GCODE ID=START_PRINT_WAIT DURATION=0.1
```

### Финальная стадия:
```
#############  Final Stage #############
  {% elif state == 'Final' %}
    {action_respond_info("Final Stage")}
    M117 Final Stage
    # set staus back to prepare for the next run 
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=state VALUE='"Prepare"'       # Восстанавливаем будущий статус на Prepare для следущей печати
    M109 S{extruder|int} ; heat extruder and wait                               # догреваем экструдер и ждём
    M117 Clean nozzle.                                                          # сообщение об очистке сопла на экран
    CLEAN_NOZZLE                                                                # едем чистить сопло
    M83                                                                         # перевод мотора экструдера в режим относительных координат
    G1 E-2                                                                      # детаем откат филамента на 2мм, чтобы сопло не текло
    M117 Caibration Z-offset.                                                   # сообщение о калибровке Z-offset на экран
    RESUME_BASE                                                                 # Важно! Просыпаемся от PAUSE, чтобы значения калибровки запомнились
    CALIBRATE_Z                                                                 # Калибруем Z-offset
    PAUSE_BASE                                                                  # Важно! Снова уходим в PAUSE
    G92 E0                                                                      # Сброс значеня экструдера в ноль
    M117 Prime Line Print                                                       # сообщение о печати Prime Line
    PRIME_LINE                                                                  # Печатаем Prime Line
    M117                                                                        # Очищаем экран
    RESUME_BASE                                                                 # Просыпаемся из паузы и... уходим в печать
  {% endif %}
#############  Begin of ptinting #############
```

### Скрипт PRINT_START_WAIT - запускается на выполнение с задержкой. Также позволяет просчитать средний PWM на базе выборки из N значений.
```
[delayed_gcode START_PRINT_WAIT]
gcode:
 # Print remaining time
  {% if printer["gcode_macro PRINT_START"].state == 'HeatSoak' %}
      {% set pwm = printer['heater_bed'].power | float %}
      {% set totalpwm = printer["gcode_macro PRINT_START"].pwm|float %}
      {% set tests = printer["gcode_macro PRINT_START"].tests|int + 1 %}
      {% set left = printer["gcode_macro PRINT_START"].left|int - 1 %}
      {% set soak = printer["gcode_macro PRINT_START"].soak | float %}
      {% set avgpwm = ((totalpwm+pwm)/tests)|float %}
      SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=avgpwm VALUE={avgpwm}
      SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=pwm VALUE={(totalpwm+pwm)|float}
      SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=tests VALUE={tests}
      SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=left VALUE={left}
      M117 S{'T:%02d' % left|int}{'/30 P:%.2f' % (avgpwm|float)}{'/%.2f' % (soak|float)}
  {% endif %}
  # Check CANCLE_PRINT was executed
  {% if printer["gcode_macro CANCEL_PRINT"].execute == 'false' %}
    # Jump back to PRINT_START
    PRINT_START
  {% else %}
    # break loop
    # insure state is correct for the next print start
    SET_GCODE_VARIABLE MACRO=CANCEL_PRINT VARIABLE=execute VALUE='"false"'
    SET_GCODE_VARIABLE MACRO=PRINT_START VARIABLE=state VALUE='"Prepare"'
#    UPDATE_DELAYED_GCODE ID=_CLEAR_DISPLAY DURATION=1
  {% endif %}
```