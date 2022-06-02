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