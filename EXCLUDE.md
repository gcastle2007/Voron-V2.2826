## Exclude Object

В конфиг Klpper добавляется из https://github.com/Klipper3d/klipper/blob/master/config/sample-macros.cfg:

```
[exclude_object]

[gcode_macro M486]
… <до конца>
```

В SS в PostProcessing добавляем:
```
preprocess_cancellation.py;
```

В SS обязательна галка:

![SS_objects](./imgs/SS_obj.png)

Итого:

![Final](./imgs/exclobjfin.png)
