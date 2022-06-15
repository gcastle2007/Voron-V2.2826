## Exclude Object

В конфиг Klpper добавляется из https://github.com/Klipper3d/klipper/blob/master/config/sample-macros.cfg:

[exclude_object]

[gcode_macro M486]
… <до конца>

В SS в PostProcessing добавляем:
c:\Python38\python.exe d:\portable\SuperSlicer_2.4.58.2_win64_220401\preprocess_cancellation.py;

В SS обязательна галка:

Итого:
картинка