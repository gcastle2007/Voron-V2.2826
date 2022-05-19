## Макрос PRINT_START

Идеология данного макроса заключается в том, чтобы разбить процесс подготовки печати на этапы, нагреть стол, убедиться, что средний PID стола стал ниже 25% (а это значит, что стол прогрелся):
- Prepare - самая первая стадия. До этого идёт задание параметров для необходимых переменных.
- Heating bed - стадия запуска прогрева стола. Также запускается предварительный нагрев экструдера.
- HeatSoak - стадия догрева стола. В данном макросе применена идеология того, что когда стол прогревается, то средняя величина PID прогрева стола уменьшается. В моём случае ждём, когда PID станет 25%, после этого можно переходить на следующую стадию.
- QGL - делаем Quad Gantry Level для гантри.
- BedMesh - в засимости от необходимости, либо перестраиваем полностью Bed Mesh, либо подгружаем уже заранее сохранённую карту в зависимости от температуры.
- Final - последняя стадия, в которой догревается экструдер, чистится сопло, калибркуется Z-Offset через CALIBRATE_Z, прогоняется PRIME_LINE и далее уже идёт печать.

###Разберём подробно все этапы и действия.

```
[gcode_macro PRINT_START]
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
variable_soak: 0.25                   # ожидаемое значание среднеро PWM стола. Идея состоит м втом, чтобы снять некоторое кол-во показаний PWD и высчитать среднее.
variable_pwm: 1                       # начальное значение PWM
variable_avgpwm: 1                    # среднее накопительное значение PWM
variable_tests: 0                     # счётчик опросов PWM
variable_left: 60                     # кол-во опросов PWM. Опросы будут проводиться 1 раз в секунду. Соответственно 60 раз.
variable_clean2: False
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
  # get actual temp from extra sensor or heater sensor
  {% if 'temperature_sensor bed' in printer %}
    {% set actBed = printer['temperature_sensor bed'].temperature|int %}
  {% else %}
    {% set actBed = printer.heater_bed.temperature|int %}
  {% endif %}
  # get max allow bed temp from config. Lower it by 5C to avoid shutdown
  {% set cfg_bed_max = printer.configfile.settings.heater_bed.max_temp|int - 5 %}
```