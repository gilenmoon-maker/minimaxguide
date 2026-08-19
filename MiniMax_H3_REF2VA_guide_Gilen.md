# MiniMax H3 REF2VA — практический ориентир

**Для ComfyUI, reference-to-video и запуска на RTX 3070 Ti 8 GB + 64 GB RAM**

## 1. Что установлено точно

- Источник: Civitai image/video ID **139970013**.
- Тип: **Video**.
- Опубликованный размер: **1920 × 2944**.
- Дата создания: **17 августа 2026**.
- Base model в metadata: **MiniMax H3**.
- Model version IDs: не указаны.
- Используются **Picture 1, Picture 2, Picture 3**.
- Prompt построен как reference-driven video generation.
- Sampler, scheduler, CFG, steps и seed в metadata не указаны.

## 2. Архитектура prompt

Prompt разделён на шесть смысловых блоков:

1. `subject_definitions`
2. `summary`
3. `retention_analysis`
4. `detailed_description`
5. `overall_soundscape`
6. `non_diegetic_music`

Главная идея — создать переменную `<Subject 1>` и привязать к ней несколько референсов.

### 2.1 Три референса

| Референс | Роль | Что закрепляет |
|---|---|---|
| Picture 1 | Основной character reference | Внешность, одежда, волосы, лицо, пропорции и общий дизайн |
| Picture 2 | Body reference | Форма тела и пропорции |
| Picture 3 | Head/face reference | Лицо, глаза, губы, волосы и выражение |

### 2.2 Сохранение идентичности

В `retention_analysis` автор требует `fully_preserved` для Subject 1 и Picture 1 и перечисляет appearance, clothing, pose, art style, color palette и proportion.

Это разумный способ сформулировать приоритет сохранения персонажа. Но нет доказательства, что `retention_analysis` является отдельным техническим API-полем MiniMax; вероятнее, это структурированный естественный язык внутри prompt.

### 2.3 Таймлайн и камера

| Время | Содержание |
|---|---|
| 00:00:000 | Стартовая поза на троне; камера сверху; начало движения |
| 00:04:700 | Новая поза/выражение; поворот тела; действие с одеждой |
| 00:07:900 | Финальная ключевая поза; изменение положения головы; сохранение harness |

Камера описана как последовательность состояний: высокий ракурс → быстрое движение вниз → low-angle.

## 3. Почему это REF2VA

Metadata указывает MiniMax H3, а prompt использует несколько изображений как Picture 1/2/3. В экосистеме H3 Ref2VA предназначен для reference-to-video. FL2VA используется для first/last-frame сценариев.

**Схема:**

`Picture 1 + Picture 2 + Picture 3 → Subject Definition → prompt/timeline → MiniMax H3 Ref2VA → video`

## 4. Компоненты локального workflow

- **Diffusion model:** `minimax_h3_ref2va_pruned_int8_convrot.safetensors`
- **Text encoder:** Qwen3-VL-32B, подходящая квантизация.
- **Video VAE:** `minimax_h3_video_vae_fp16.safetensors`
- **Audio VAE:** `minimax_h3_audio_vae_fp32.safetensors`
- **ComfyUI:** интерфейс и workflow.

Для задачи с несколькими reference images нужен именно **REF2VA/R2V workflow**, а не FL2VA.

## 5. Квантизация и VRAM

| Вариант | Примерный размер | Класс GPU | RTX 3070 Ti 8 GB |
|---|---:|---|---|
| INT8 ConvRot | ~20.94 GiB | 24 GB | Нет |
| W8/W4 ConvRot | ~13.57 GiB | 16 GB | Нет/нецелесообразно |
| W4 ConvRot | ~10.07 GiB | 12 GB | Сам по себе больше VRAM |
| W4 ConvRot Offload | ~9.71 GiB | 8 GB + CPU offload | **Да, экспериментально** |
| NVFP4 | ~10.86 GiB | в первую очередь RTX 50/Blackwell | Не целевой вариант |

**Для RTX 3070 Ti:** целевой вариант — **W4 ConvRot Offload + CPU offload**.

Это не означает высокую скорость: часть модели будет находиться в системной RAM и обмениваться с VRAM.

## 6. Твой ПК

- GPU: **RTX 3070 Ti, 8 GB VRAM** — подходит только для агрессивного W4/offload режима.
- RAM: **64 GB** — хороший запас для экспериментов и CPU offload.
- Reference images: начать с 1, затем перейти к 3.
- INT8 checkpoint 20+ GiB — не подходит.
- W4 Offload — основной кандидат.

## 7. Наиболее вероятный workflow автора

```text
Picture 1 — полный персонаж / одежда
Picture 2 — body reference
Picture 3 — face reference
        ↓
REF2VA reference conditioning
        ↓
<Subject 1> + structured prompt
        ↓
timeline + camera movement + style
        ↓
MiniMax H3
        ↓
base video
        ↓
неизвестный post-process / upscale
        ↓
опубликованный 1920×2944 MP4
```

**Важно:** 1920×2944 — размер опубликованного файла. Из metadata не следует, что H3 генерировал его нативно в таком разрешении. Точный upscale pipeline неизвестен.

## 8. Что пока неизвестно

- Точный JSON workflow автора.
- Точный sampler/scheduler/seed/steps/CFG.
- Точное базовое разрешение и число кадров на этапе генерации.
- Чем сделан upscale/post-processing до 1920×2944.
- Были ли Picture 1–3 заранее сделаны в ComfyUI, Virt-A-Mate или другом инструменте.
- Является ли `retention_analysis` специальной технической командой MiniMax или просто структурированным prompt.

## 9. Практический план запуска на RTX 3070 Ti

1. Установить/обновить совместимый ComfyUI.
2. Установить H3 REF2VA workflow.
3. Взять W4 ConvRot Offload для 8 GB GPU.
4. Настроить CPU offload.
5. Начать с одного reference image и короткого теста.
6. Добавить Picture 2 и Picture 3.
7. Повторить структуру Subject Definition + retention + timeline.
8. Только после стабильности увеличивать длительность и разрешение.
9. Отдельно тестировать upscale/post-processing.

**Не начинать сразу с 1920×2944.** Сначала нужно проверить стабильность идентичности и движения на небольшом базовом разрешении.

## 10. Reference pack для 3D-персонажей

| Файл | Содержание | Цель |
|---|---|---|
| REF_01_FULL.png | Полный рост, фронт/3⁄4, полный костюм | Основной identity/outfit |
| REF_02_BODY.png | Body reference | Стабильность силуэта и пропорций |
| REF_03_FACE.png | Крупный портрет | Стабильность лица/глаз/волос |

Для Virt-A-Mate удобно заранее получить контролируемые 3D-рендеры одного персонажа и использовать AI преимущественно для движения и камеры.

## 11. Итог

**RTX 3070 Ti 8 GB + 64 GB RAM → MiniMax H3 REF2VA W4 ConvRot Offload.** Это режим для экспериментов и проверки технологии, а не оптимальный production-вариант.

Если позже менять GPU, 24 GB становится значительно комфортнее для H3, а 32 GB даёт ещё больший запас. Но сначала разумно проверить качество W4 Offload на имеющейся карте.

## 12. Источники

- Civitai — MiniMax H3 INT8/INT4 ConvRot, modelVersionId 3193385.
- Civitai — DaSiWa MiniMax H3 Workflows, modelVersionId 3195699.
- Comfy-Org / MiniMax-H3 — актуальные модели и workflow templates.
- DmitryDB / MiniMax-H3-ComfyUI-Quants — квантизации и memory classes.
- MiniMax — официальный анонс открытого H3.

> Данные о конкретном ролике основаны на metadata, предоставленной из Civitai API. Где точного подтверждения нет, это прямо отмечено.
