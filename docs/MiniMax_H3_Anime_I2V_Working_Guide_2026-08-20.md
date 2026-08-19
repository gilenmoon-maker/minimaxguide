# MiniMax H3 / Anime I2V — Working Guide

**Актуальность:** 20 августа 2026

**Задача:** локальный Anime / 2D Image-to-Video pipeline в ComfyUI.

**Железо:** RTX 3070 Ti 8 GB VRAM + 64 GB RAM.

## 1. Главный вывод

Главный кандидат для твоего проекта — **MiniMax H3 Ref2VA**. Это не утверждение, что H3 гарантированно лучше всех конкурентов: честного публичного benchmark-а на одинаковых anime-кадрах против DaSiWa/Wan 2.2 я не нашёл. Поэтому H3 ставится первым в A/B-тест, а не объявляется безусловным победителем.

### Приоритет тестов

1. **H3 Ref2VA INT4** — основной кандидат для 8 GB.
2. **H3 Ref2VA W4 / ConvRot / CPU offload** — fallback при проблемах с VRAM.
3. **H3 Ref2VA INT8** — контроль качества.
4. **H3 FL2VA INT4** — controlled transition / first-last frame.
5. **H3 Ref2VA + актуальный Turbo LoRA** — speed test.
6. **H3 FL2VA + Turbo** — быстрый альтернативный режим.
7. **DaSiWa / Wan 2.2 I2V** — главный внешний контроль.
8. **HunyuanVideo 1.5** — резервный более лёгкий baseline.

## 2. Что именно сравнивать у H3

Не путать название модели с конкретным workflow.

- **Ref2VA** — Reference-to-Audio-Video, приоритет для «этот персонаж/этот кадр → оживить».
- **FL2VA** — First/Last-Frame-to-Audio-Video, приоритет для заданного перехода A → B.
- **INT8 / INT4 / W4** — precision/quantization.
- **pruned** — уменьшенный вариант.
- **ConvRot** — конкретная community quantization/optimization ветка.
- **offload** — часть вычислений/весов размещается за пределами VRAM.
- **Turbo LoRA** — уменьшение количества шагов; проверять только после baseline.

## 3. Почему Ref2VA — главный кандидат

Reference conditioning соответствует visual-novel production: есть готовый anime keyframe, character references и prompt движения.

```text
ANIME KEYFRAME
      +
CHARACTER / STYLE REFERENCES
      +
MOTION PROMPT
      ↓
MINIMAX H3 REF2VA
      ↓
SHORT I2V CLIP
```

Главные критерии: сохранение лица, волос, одежды, силуэта, палитры, line-art/style, фона и композиции.

## 4. FL2VA

Использовать там, где нужно управлять состоянием начала и конца.

```text
FRAME A → controlled transition → FRAME B
```

Проверить, даёт ли FL2VA меньше drift на переходах, чем Ref2VA.

## 5. 3070 Ti 8 GB — практический выбор

| Вариант | Что делать |
|---|---|
| BF16 | Не ставить первым: слишком тяжёлый класс для 8 GB |
| INT8 | Использовать как quality reference / контроль |
| INT4 | Главный тест |
| W4 / aggressive quant | Fallback при OOM / слишком большом memory footprint |
| pruned INT4 | Очень интересный вариант для 8 GB |
| NVFP4 | Не считать автоматически лучшим для 3070 Ti; проверять конкретную реализацию |

64 GB RAM полезны для CPU offload, но сильный offload может сделать генерацию слишком медленной. Поэтому нужно измерять и VRAM, и RAM, и время.

## 6. Turbo

Turbo LoRA — второй этап. Сначала получить quality baseline на обычном H3.

Порядок:

1. Зафиксировать input.
2. Зафиксировать prompt.
3. Зафиксировать resolution, frames и sampler.
4. Получить baseline.
5. Добавить Turbo.
6. Сравнить скорость, identity drift и artifacts.

Если скорость выигрывает без заметной потери качества — Turbo можно использовать для bulk generation.

## 7. Конкуренты

### Wan 2.2

Зрелая open-source video family с I2V и широкой ComfyUI-экосистемой. Нужен как базовый контроль.

### DaSiWa

Специализированная Wan 2.2 derived/fine-tuned ветка. Её смысл для тебя — проверить, даёт ли anime-oriented tuning меньший style/identity drift. Если да, она может оказаться практичнее базового Wan на конкретных сценах.

### HunyuanVideo 1.5

8.3B class, интересен как более компактная альтернатива и baseline для ограниченной памяти.

## 8. Единый A/B тест

Все модели должны получить одинаковые входные условия.

| Параметр | Правило |
|---|---|
| Input | один и тот же anime keyframe |
| Prompt | один и тот же motion prompt |
| Seed | фиксировать, где поддерживается |
| Resolution | одинаковая |
| Frames | одинаковое количество |
| Steps | записывать |
| Sampler | записывать |
| VRAM peak | записывать |
| RAM peak | записывать |
| Generation time | записывать |
| Identity | 0–10 |
| Style | 0–10 |
| Motion | 0–10 |
| Artifacts | 0–10, где 10 = почти нет |
| Camera control | 0–10 |
| Overall | 0–10 |

### Минимальный тестовый набор

1. Поворот головы / глаза.
2. Волосы и одежда.
3. Full-body movement.
4. Медленный camera push-in.
5. Более сложное движение.

## 9. Production pipeline

```text
ILLUSTRIOUS / ANIME IMAGE
          ↓
      FINAL KEYFRAME
          ↓
 CHARACTER / STYLE REFERENCE
          ↓
     MINIMAX H3 REF2VA
          ↓
     QUALITY CHECK
          ↓
   REGENERATE / SELECT
          ↓
     VIDEO UPSCALE
          ↓
     FINAL AVN CLIP
```

Главное правило: сначала зафиксировать хороший keyframe, затем использовать I2V как слой движения. Не просить одну модель одновременно менять персонажа, интерьер, композицию и движение.

## 10. Character consistency

Сделать canonical character sheet: лицо, волосы, одежда, палитра, аксессуары.

Для каждой сцены:

1. Получить стабильный keyframe.
2. Передать reference, если режим его поддерживает.
3. Описывать только необходимое движение.
4. Проверять identity drift.
5. Сохранять удачный кадр как следующий reference.

## 11. Prompting

Для I2V prompt должен в первую очередь описывать изменение во времени, а не повторять всю картинку.

Шаблон:

```text
subject, action, motion intensity, facial motion, hair/clothing motion,
camera movement, environment stability, lighting stability, temporal coherence
```

Пример:

```text
the character remains in the same place and keeps the same appearance,
she slowly turns her head toward the camera, her hair moves subtly and naturally,
her clothing moves slightly, the background remains stable,
the camera performs a very slow push-in, smooth cinematic motion, consistent lighting
```

## 12. Решение после тестов

```text
IF H3 Ref2VA INT4 fits + quality is excellent
    → use H3 Ref2VA INT4 as primary

ELSE IF H3 W4 fits and quality is acceptable
    → use H3 W4 as primary

IF FL2VA is better for controlled transitions
    → keep FL2VA as secondary

IF DaSiWa has materially lower identity/style drift
    → keep DaSiWa as specialized fallback

IF Turbo quality loss is small
    → use Turbo for bulk generation

IF all H3 variants are too slow/heavy
    → switch baseline to Wan/DaSiWa or HunyuanVideo 1.5
```

## 13. Что считать доказанным

**Высокая уверенность:** Wan 2.2 имеет I2V family; HunyuanVideo 1.5 — 8.3B class с I2V/step-distilled режимами; H3 имеет разные conditioning и quantized workflows.

**Требует собственного теста:** H3 Ref2VA лучше DaSiWa на твоём anime стиле; INT4 точно оптимален на конкретном workflow; Turbo лучше обычного режима; W4 будет достаточно быстрым на твоём ПК.

## 14. Что скачать первым

1. H3 Ref2VA INT4/pruned.
2. H3 W4/ConvRot/offload.
3. H3 FL2VA INT4.
4. H3 INT8.
5. H3 Turbo LoRA.
6. Wan 2.2 I2V.
7. DaSiWa.
8. HunyuanVideo 1.5.

Не скачивать всё одновременно. Сначала добиться запуска одного H3 workflow, затем сравнивать варианты.

## 15. Финальная рабочая рекомендация

**Основная ставка:** MiniMax H3 Ref2VA INT4.

**Fallback по памяти:** H3 W4 / ConvRot / offload.

**Quality reference:** H3 Ref2VA INT8.

**Controlled transitions:** H3 FL2VA INT4.

**Speed:** H3 Ref2VA + Turbo LoRA.

**Главный конкурент:** DaSiWa / Wan 2.2.

Цель — не самый красивый одиночный ролик, а стабильная production-модель, которая сможет генерировать десятки клипов одного персонажа с минимальным drift, приемлемым временем и воспроизводимым результатом.

## Sources

- Comfy-Org MiniMax H3: https://huggingface.co/Comfy-Org/MiniMax-H3
- MiniMax H3: https://huggingface.co/MiniMaxAI/MiniMax-H3
- H3 ComfyUI quantizations: https://huggingface.co/DmitryDB/MiniMax-H3-ComfyUI-Quants
- H3 INT4 ConvRot: https://huggingface.co/Merserk/MiniMax-H3-INT4-ConvRot
- Wan 2.2: https://github.com/Wan-Video/Wan2.2
- HunyuanVideo 1.5: https://github.com/Tencent-Hunyuan/HunyuanVideo-1.5
