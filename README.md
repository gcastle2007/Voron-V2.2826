## Configuration of Voron 2.2826

В данном репозитории находится мой актуальный конфиг от 3D принетра Voron 2.4 (V2.2826).

Даный конфиг написан полностью самостоятельно с нуля.
Какие-то дополнительные фичи из макросов были подсмотрены в конфигах нижепредставленных товарищей, разобраны для понимания и перепилены под себя:
- Zellner Alex (очень лютый конфиг, не для слабонервных): https://github.com/zellneralex/klipper_config
- ZZToP: https://github.com/zztopper/voron_v2.4_1030
- Vassssko: https://github.com/Vassssko/voron_v2.4_982

[Раздел макросов ](./MACROS/)

Запуск печати осуществляется через
```
PRINT_START EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature] CHAMBER=[chamber_temperature] FILAMENT=[filament_type] BMESH=True

Где:
    [first_layer_temperatire] - температура печати первого слоя
    [first_layer_bed_temperature] - температура стола для печати первого слоя
    [chamber_temperature] - минимальная необходима температура камеры для начала печати (сейчас не использую)
    [filament_type] - тип филамента используемого при печати. В зависимости от этого включается Nevermore и вытяжной вентилятор
    BMESH=True - сообщение принтеру, что нужно будет собрать Bed Mesh. Если не нужно, то False

```

## Что существует полезного?

- [Print Tuning Guide](https://github.com/AndrewEllis93/Print-Tuning-Guide)
- [PIF profile](https://github.com/AndrewEllis93/Ellis-PIF-Profile)
- [Exclude Object импортировано в Klipper](https://github.com/Klipper3d/klipper/blob/master/docs/Exclude_Object.md) - [Что для этого нужно?](EXCLUDE.md)

## Полезные, с моей точки зрения, модицифкации:
- [Корзина и щётка для сопла](https://github.com/VoronDesign/VoronUsers/tree/master/abandoned_mods/printer_mods/edwardyeeks/Decontaminator_Purge_Bucket_&_Nozzle_Scrubber)
- [Pinmod, замена винтов на стальные или карбоновые валы, а заодно и шпуль на подшипники](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/hartk1213/Voron2.4_Trident_Pins_Mod)
- [Перенос Y-end_stop на A drive](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/hartk1213/Voron2.4_Y_Endstop_Relocation)
- [Голова Mantis с отличным обдувом на двух 5015 турбинках](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/Long/Mantis_Dual_5015)
- [Убираем "цепи" (Umbilical Mod)](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/Minsekt/Rear_Umbilical)
- [Ретрактор для Umbilical Mod](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/Ellis/Badge_Retractor_Mount)
- [Удобные крепления для панелей](https://github.com/Annex-Engineering/Other_Printer_Mods/tree/master/All_Printers/Annex_Panel_2020_Clips_and_Hinges)
- [Установка верхней панели на магнитах (пока сам не тестировал)](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/Printopal/Magnetic_top_panel)
- [Vefach, угольный фильтр + Hepa на выдув](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/KevinAkaSam/VEFACH)
- [Nevermore, система рециркуляции воздуха в камере. Позволяет быстрее прогреть камеру, а также фильтрует воздух через активированный уголь](https://github.com/nevermore3d/Nevermore_Micro)
- [Klicky Probe](https://github.com/VoronDesign/VoronUsers/tree/master/printer_mods/JosAr/Klicky-Probe)
- [Unklicky, альтернатива Klicky Probe](https://github.com/majarspeed/Unklicky)
- [Кинематическое крепление стола](https://github.com/tanaes/whopping_Voron_mods/tree/main/kinematic_bed)

## На что смотреть после Voron 2.4?
- [Voron 0](https://github.com/VoronDesign/Voron-0)
- [Gasherbrum-K3](https://github.com/Annex-Engineering/Gasherbrum-K3)