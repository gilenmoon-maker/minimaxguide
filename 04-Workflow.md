# 04 — Workflow: I2V, T2V, First/Last Frame и Ref2VA

## 1. Выбор режима

| Режим | Что даём | Контроль персонажа | Приоритет |
|---|---|---|---|
| T2VA | текст | низкий | 4 |
| I2V / FL2VA | картинка + текст | высокий | **1** |
| First + Last Frame | 2 картинки + текст | очень высокий для начала/конца | **2** |
| Ref2VA | изображения/видео/аудио + текст | высокий/очень высокий | **3** |

Официальные workflow: I2V, T2V и R2V. citeturn0search0turn0search1turn0search2

---

# 2. I2V — основной pipeline

```text
3D Render
   ↓
Load Image
   ↓
H3 FL2VA
   ↑
Qwen H3 Encoder
   ↑
Prompt
   ↓
Video + Audio Latent
   ↓
Video VAE + Audio VAE
   ↓
MP4
```

Для VN это основной путь, потому что персонаж уже создан в 3D.

---

# 3. Как подготовить исходный render

Лучший стартовый render:

- финальное лицо уже готово;
- глаза хорошо видны;
- нет сильного motion blur;
- нет экстремально мелких деталей;
- персонаж занимает разумную часть кадра;
- освещение стабильное;
- фон не перегружен.

Не начинай с групповой сцены.

---

# 4. Prompt для I2V

Структура:

```text
[SHOT]
Cinematic medium shot of [subject] in [location].

[ACTION]
The subject slowly [one main action].

[CAMERA]
The camera [one camera movement].

[SECONDARY MOTION]
Hair and clothing move subtly and naturally.

[AUDIO]
Describe simple environmental sound.
```

Главное правило первого теста:

> **одно главное действие + одно движение камеры.**

---

# 5. Хороший первый пример

```text
Cinematic medium shot of a woman standing in an elegant modern bedroom at night.
She slowly turns her head toward the camera.
Her hair and clothing move subtly and naturally.
The camera performs a very slow cinematic push-in.
Warm practical lighting with a soft cool rim light from the window.
Quiet bedroom ambience and subtle fabric movement.
```

---

# 6. Почему не надо писать слишком длинный prompt

Слишком много требований одновременно увеличивает вероятность конфликтов.

Плохой первый тест:

```text
turns + walks + sits + touches hair + changes expression + camera orbit + zoom + lighting change + rain + wind
```

Хороший:

```text
slowly turns her head toward the camera
+ slow camera push-in
```

После стабильного результата усложняем.

---

# 7. First Frame

First Frame задаёт стартовое состояние.

Для VN:

`готовый 3D render → First Frame → движение`.

Это наш базовый production-подход.

---

# 8. Last Frame

Last Frame задаёт желаемый финал.

Например:

```text
Frame A: персонаж стоит
Frame B: персонаж уже сидит
```

H3 строит переход.

Начинаем с небольшого различия между кадрами. Большой скачок позы может быть сложнее.

---

# 9. First + Last Frame

Очень интересный режим для VN.

Используем, когда хотим получить не случайное движение, а переход между двумя заранее подготовленными состояниями.

---

# 10. T2VA

```text
Prompt
 ↓
H3 FL2VA
 ↓
Video + Audio
```

Без изображения FL2VA workflow может использоваться как text-to-video/text-to-audio. citeturn0search0

T2V полезен для:

- establishing shots;
- интерьеров;
- природы;
- фонов;
- второстепенных кадров.

Для постоянных персонажей предпочитаем I2V/Ref2VA.

---

# 11. Ref2VA

Ref2VA — отдельный checkpoint и отдельный workflow.

Актуальная документация позволяет использовать несколько типов референсов. В workflow описаны изображения, видео и отдельные аудиореференсы. citeturn0search1

### Пример логики

```text
Image 1 = identity
Image 2 = clothing
Image 3 = environment
Video 1 = motion/camera reference
Audio 1 = voice/sound reference
                 ↓
              Ref2VA
                 ↓
              new shot
```

---

# 12. Роли референсов

В prompt явно описывай роль каждого reference.

Пример:

```text
Use <Picture 1> as the main character identity.
Use <Picture 2> as the clothing reference.
Use <Picture 3> as the environment reference.
Create a cinematic medium shot of the character slowly turning toward the camera.
```

Не заставляй модель угадывать, зачем ты дал картинку.

---

# 13. ref_image_size

Если конкретный Ref2VA workflow предлагает:

`match` — начинаем с него на 8 GB.

`max` — тестируем потом, если нужна дополнительная детализация/идентичность.

В документации Ref2VA больший reference size может увеличить вычислительную стоимость. citeturn0search1

---

# 14. Sampler / Scheduler

Не переносим настройки одного workflow в другой.

Сначала оставляем sampler/scheduler из официального шаблона.

В Ref2VA документация отдельно отмечает `res_multistep` и предпочтение `beta`/`normal` scheduler для некоторых reference-heavy prompts. citeturn0search1

---

# 15. Аудио

H3 — видео+аудио модель.

Поэтому prompt может содержать звук:

```text
quiet bedroom ambience
soft fabric movement
distant city noise
subtle footsteps
```

Не обязательно добавлять звук в каждый prompt, но для production-клипов лучше сразу учитывать audio design.

---

# 16. Разрешение

Официальные workflow задают свои допустимые размеры и сетку. Не надо вручную вводить произвольное разрешение, если workflow его округляет.

На 3070 Ti:

1. стартовый размер из workflow;
2. проверить VRAM;
3. затем поднять разрешение;
4. потом отдельно увеличить длительность.

---

# 17. Длительность

H3 работает с допустимой сеткой кадров, поэтому введённые секунды могут преобразовываться workflow в ближайшее допустимое число кадров.

Не пугайся небольшого расхождения между введённой длительностью и фактическим числом кадров.

---

# 18. Настройки каждой quant разные

Это принципиально.

```text
Official INT8 → официальный loader/settings
INT4 → настройки конкретного INT4
GGUF Q4 → GGUF loader/settings
W4A8 → W4A8 loader/settings
SVDQuant → SVDQuant loader/settings
```

Нельзя сделать один универсальный preset и считать его оптимальным для всех.

---

# 19. Production workflow №1 — персонаж

```text
3D render
 ↓
I2V / FL2VA
 ↓
4–8 sec
 ↓
24 fps
 ↓
H3 audio
 ↓
MP4
```

---

# 20. Production workflow №2 — identity-heavy

```text
Character references
        +
Environment references
        +
Optional motion/video
        ↓
      Ref2VA
        ↓
    Video + Audio
```

---

# 21. Production workflow №3 — переход

```text
First Frame
     +
Last Frame
     ↓
   FL2VA
     ↓
transition shot
```

---

# 22. Production workflow №4 — фон

```text
Text
 ↓
T2VA
 ↓
background / establishing shot
```

---

# 23. Рекомендуемая библиотека preset

После тестов сохраняем:

```text
VN_I2V_FAST.json
VN_I2V_BALANCED.json
VN_I2V_QUALITY.json
VN_FIRST_LAST_FRAME.json
VN_REF2VA_CHARACTER.json
VN_REF2VA_SCENE.json
VN_T2VA_BACKGROUND.json
```

В каждом preset записываем H3/Qwen/commit и дату.
