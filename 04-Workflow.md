# 04 — Workflow: I2V, T2V и Ref2VA

## 1. Какой режим использовать

| Режим | Вход | Контроль персонажа | Для VN |
|---|---|---|---|
| T2VA | текст | низкий | эксперимент |
| I2V / FL2VA | картинка + текст | высокий | **основной** |
| First + Last Frame | 2 кадра + текст | очень высокий по началу/концу | очень интересен |
| Ref2VA | изображения/видео/аудио + текст | высокий/очень высокий | **важен для персонажей** |

Официальные шаблоны Comfy-Org предоставляют отдельные I2V, T2V и R2V JSON. citeturn0search0turn0search1turn0search2

---

# 2. I2V — наш основной pipeline

Схема:

```text
3D Render
   ↓
Load Image
   ↓
MiniMax H3 FL2VA
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

Официальный I2V workflow уже содержит эту логику и использует `minimax_h3_fl2va_pruned_int8_convrot.safetensors`. citeturn0search0

---

# 3. Как писать prompt для I2V

Не нужно писать роман.

Лучше описывать:

1. кто в кадре;
2. где находится;
3. что делает;
4. как движется камера;
5. какие мелкие движения происходят;
6. что происходит со звуком.

### Шаблон

```text
[SHOT]
Cinematic medium shot of [subject] in [location].

[ACTION]
The subject slowly [main movement].

[CAMERA]
The camera [camera movement].

[SECONDARY MOTION]
Hair and clothing move subtly and naturally.

[AUDIO]
Quiet room ambience, subtle fabric movement, soft environmental sound.
```

---

# 4. Пример для 3D Visual Novel

```text
Cinematic medium shot of a woman standing in an elegant modern bedroom at night.
She slowly turns her head toward the camera and gives a subtle natural smile.
Her hair moves slightly as she turns.
The camera performs a very slow cinematic push-in.
Warm practical lighting with a soft cool rim light from the window.
Quiet bedroom ambience, subtle fabric movement and distant city noise.
```

Для первого теста намеренно используем **одно главное действие**.

---

# 5. First Frame

First Frame задаёт стартовую картинку.

Использование:

```text
Render
 ↓
First Frame
 ↓
H3
 ↓
движение
```

Это наиболее естественный способ анимировать уже готовый рендер.

---

# 6. Last Frame

Last Frame задаёт желаемый конечный кадр.

Можно использовать:

```text
First Frame
      +
Last Frame
      ↓
    H3
      ↓
движение между ними
```

Это интересно для сцен, где важно получить определённую позу или композицию в конце.

---

# 7. First + Last Frame

Это один из самых интересных режимов для VN.

Например:

**Кадр 1:** персонаж стоит у окна.

**Кадр 2:** персонаж уже сидит на кровати.

H3 должна построить переход.

Не стоит начинать с очень большого изменения. Сначала проверяем простые переходы.

---

# 8. T2VA

T2VA:

```text
Prompt
 ↓
H3
 ↓
Video + Audio
```

Официальный T2V workflow использует тот же FL2VA diffusion checkpoint и H3 Qwen encoder. Если изображения не подключены, I2V workflow также может работать как text-to-video. citeturn0search0turn0search2

T2V полезен для:

- тестирования H3;
- фонов;
- establishing shots;
- природы;
- интерьеров;
- сцен, где персонаж не должен быть строго идентичен исходному render.

---

# 9. Ref2VA

Ref2VA интереснее, когда нужно использовать несколько источников.

Официальный workflow позволяет подключать до:

- 9 reference images;
- 3 reference videos;
- 3 standalone audio clips.

Можно использовать референсы для персонажа, стиля, движения, камеры или голоса. citeturn0search1

---

# 10. Как мыслить о Ref2VA

Например:

```text
Picture 1 = персонаж
Picture 2 = одежда
Picture 3 = интерьер
Video 1 = движение камеры
Audio 1 = голос

             ↓
          Ref2VA
             ↓
         новый shot
```

В prompt нужно явно указывать, какой референс за что отвечает.

Пример:

```text
Use <Picture 1> as the main character identity.
Use <Picture 2> as the clothing reference.
Use <Picture 3> as the environment reference.
Create a cinematic medium shot of the character slowly turning toward the camera.
```

Официальная документация workflow подчёркивает, что точные теги референсов и явное описание их роли особенно важны. citeturn0search1

---

# 11. ref_image_size

В Ref2VA есть концепция размера референса.

`match` — уменьшает референс до размера генерации и работает быстрее.

`max` — позволяет сохранить до 2048 px по короткой стороне, потенциально лучше для идентичности, но дороже по скорости, потому что reference tokens участвуют в каждом sampling step. citeturn0search1

На 8 GB начинаем с:

**match**

Потом сравниваем с:

**max**

---

# 12. Sampler

В официальном Ref2VA workflow используется `res_multistep`.

Для reference-heavy prompts там отмечено, что `beta` или `normal` scheduler обычно лучше `simple`. citeturn0search1

Не переносим это автоматически на каждый другой workflow. Сначала используем настройки официального шаблона.

---

# 13. Аудио

H3 создаёт видео и аудио совместно.

Официальный workflow декодирует видео и аудио из общего packed latent и затем объединяет их в видеофайл. citeturn0search1

Это значит, что prompt должен учитывать не только изображение.

Можно явно писать:

```text
quiet room ambience
soft fabric movement
subtle footsteps
city ambience outside the window
```

---

# 14. Разрешение

Официальный I2V/T2V шаблон использует native canvas с короткой стороной около 768 px и ограничением 768×1344, с округлением к размеру, кратному 32. citeturn0search0turn0search2

На нашей 3070 Ti не надо сразу пытаться максимизировать размер.

Первый тест — минимально разумный размер, который предлагает сам workflow.

После успешного запуска увеличиваем разрешение.

---

# 15. Длительность

H3 работает с дискретной сеткой кадров. Официальный workflow преобразует секунды в допустимый `length` и округляет до сетки 17 кадров на блок. citeturn0search0

Поэтому не удивляйся, если введённые 5 секунд будут преобразованы в немного другое количество кадров.

---

# 16. Настройки не должны быть одинаковыми для всех моделей

Это принципиально.

Например:

```text
INT8
→ одна конфигурация

INT4
→ другая

GGUF Q4
→ другой loader и настройки

W4A8
→ отдельный backend
```

Мы будем хранить настройки каждого варианта отдельно.

---

# 17. Наш первый production-пайплайн

Когда baseline будет работать:

```text
3D render
   ↓
I2V / FL2VA
   ↓
4–8 sec shot
   ↓
24 fps
   ↓
H3 native audio
   ↓
MP4
```

После этого можно строить библиотеку сцен для VN.

---

# 18. Второй production-пайплайн

Для сцен, где важнее идентичность:

```text
Character reference
       +
Environment reference
       +
Optional motion reference
       ↓
     Ref2VA
       ↓
   Video + Audio
```

---

# 19. Не пытаться решить всё одним workflow

Для реального проекта будет лучше иметь несколько preset-файлов:

```text
VN_I2V_CHARACTER.json
VN_I2V_CAMERA.json
VN_FIRST_LAST_FRAME.json
VN_REF2VA_CHARACTER.json
VN_REF2VA_SCENE.json
VN_T2V_BACKGROUND.json
```

После тестирования их можно сохранить в отдельную папку проекта.
