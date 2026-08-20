# 08 — Полный аудит MiniMax H3: актуальность вариантов и план установки для Codex Agent

**Дата аудита: 20.08.2026**

## 0. Реальное железо и цель

- GPU: **RTX 3070 Ti Laptop, 8 GB VRAM**
- RAM: **32 GB**
- ОС: Windows
- Интерфейс: ComfyUI
- Цель: локальный I2V для Visual Novel, приоритет **качество / время**, время до ~10 минут за короткий клип допустимо.

### Критическая поправка

Старый план был неверен в двух местах:

1. W4/W8W4 ConvRot варианты DmitryDB были удалены автором как failed experiments.
2. 480×720 не является нативным полотном H3. В коде ComfyUI базовый short edge = 768, размеры кратны 32. Для дешёвого первого теста используем **832×480 (0.4 MP)**, а не 480×720. Для вертикального теста сохраняем кратность 32 и начинаем с близкой площади, например **480×832**. Это эксперимент ниже нативного short edge; качество может уступать 768-short-edge.

---

# 1. Полный список ранее рассматривавшихся направлений

Ниже разделены **реальные актуальные файлы**, **удалённые/неактуальные**, и **экспериментальные ветки**. Название похожего quant не означает совместимость с одним loader.

## A. Официальные / Comfy-Org diffusion checkpoints

### 1. FL2VA BF16 — АКТУАЛЕН, НЕ СТАВИМ
- Автор/источник: MiniMax / Comfy-Org.
- Задача: T2VA + first/last-frame + I2V.
- Статус: существует.
- Для 8 GB VRAM + 32 GB RAM: **нецелесообразен**.
- Причина: огромный host-memory/offload путь.

### 2. FL2VA INT8 ConvRot — АКТУАЛЕН, НЕ ПЕРВЫЙ
- Официальный Comfy-Org вариант.
- Статус: существует.
- Проблема: большой checkpoint и тяжёлый общий комплект.
- Вердикт: тестировать только после более лёгкого baseline.

### 3. FL2VA Pruned INT8 ConvRot — АКТУАЛЕН, КАНДИДАТ
- Официальный рекомендуемый low-memory diffusion baseline в ComfyUI templates.
- Статус: существует.
- Вердикт: **один из главных кандидатов**.
- Но: с 32 GB RAM нет гарантии комфортного запуска. Проверяем фактический peak RAM.

### 4. FL2VA Pruned FP8 Scaled — АКТУАЛЕН, ЭКСПЕРИМЕНТ №2
- В текущем Comfy-Org репозитории есть pruned FP8 scaled.
- Статус: существует.
- Вердикт: **обязательно сравнить с pruned INT8**, если текущий ComfyUI/RTX 30 backend корректно запускает workflow.

### 5–8. Ref2VA BF16 / INT8 / Pruned INT8 / Pruned FP8
- Статус: актуальные варианты Comfy-Org.
- Задача: reference-to-video/audio.
- Вердикт: **не первые**. Сначала FL2VA/I2V, затем лучший diffusion quant переносим на Ref2VA.

---

# B. DmitryDB MiniMax-H3-ComfyUI-Quants — актуальный аудит

Репозиторий прямо помечает себя как community conversions, не официальный MiniMax/Comfy-Org.

## 9. INT8 ConvRot HQ — АКТУАЛЕН, НЕ ДЛЯ НАС
- ~21.9 GiB.
- Автор рекомендует класс 32 GB+ / RTX 30/40.
- Вердикт: не первый тест.

## 10. INT8 ConvRot Balanced — АКТУАЛЕН, НЕ ДЛЯ НАС
- ~20.94 GiB.
- В model card ориентирован на RTX 3090/4090 24 GB.
- Вердикт: не ставить первым.

## 11. INT8 ConvRot Lite — АКТУАЛЕН, РЕЗЕРВ
- ~20.33 GiB.
- Снижает часть BF16 islands.
- Вердикт: возможный поздний тест, но всё ещё тяжёлый для 8 GB/32 GB.

## 12. W8/W4 ConvRot — УДАЛЁН / НЕ СТАВИМ
- Был ~13.565 GiB.
- Автор удалил failed experiments.
- Вердикт: **не устанавливать**.

## 13. W4 ConvRot — УДАЛЁН / НЕ СТАВИМ
- Был ~10.067 GiB.
- Автор удалил как failed experiment.
- Вердикт: **не устанавливать**.

## 14. W4 ConvRot Offload — УДАЛЁН / НЕ СТАВИМ
- Был ~9.708 GiB.
- Ранее ошибочно был выбран как стартовый для 8 GB + CPU offload.
- Автор удалил failed experiment.
- Вердикт: **полностью убрать из гайда и Codex-задачи**.

## 15. NVFP4 HQ — АКТУАЛЕН, НЕ ДЛЯ RTX 3070 Ti
- Требует/ориентирован на Blackwell RTX 50.
- На RTX 30 не является рациональным native ускорением.
- Вердикт: не ставить.

## 16. NVFP4 — АКТУАЛЕН, НЕ ДЛЯ RTX 3070 Ti
- То же ограничение.
- Вердикт: не ставить.

---

# C. Qwen3-VL-32B H3 Encoder

Это не «вторая видеомодель». Это обязательный H3-specific conditioner/text encoder, который даёт hidden states для H3.

## 17. Qwen BF16 — АКТУАЛЕН, НЕ ДЛЯ НАС
- Слишком тяжёлый для 32 GB RAM.
- Не устанавливать.

## 18. Qwen INT8 ConvRot — АКТУАЛЕН, СЛИШКОМ ТЯЖЁЛЫЙ ДЛЯ ПЕРВОГО ТЕСТА
- Официальный Comfy-Org файл порядка 27 GB.
- Один только файл почти съедает весь практический запас 32 GB RAM с Windows и ComfyUI.
- Вердикт: не первый вариант; тест только при доказанной выгрузке/потоковой загрузке.

## 19. Qwen NVFP4 AWQ — АКТУАЛЕН, НО ПРОБЛЕМА RTX 30
- Официальный template default для мощных GPU.
- NVFP4 ориентирован на RTX 50/Blackwell; на RTX 30 native преимущества нет.
- Вердикт: не считать автоматическим лучшим вариантом для 3070 Ti.

### Практический вывод по Qwen

Старый план «сначала Qwen 8-bit, потом 4-bit» был слишком упрощён: в официальном Comfy-Org наборе нет подтверждённого обычного 4-bit Qwen для H3, который можно просто заменить на INT8 без смены backend/loader.

**Нельзя заставлять Codex скачать случайный Qwen GGUF/INT4Q/INT4BQ, пока не доказана его совместимость именно с MiniMax H3 conditioner node.**

---

# D. INT4 / INT4Q / INT4BQ / GGUF

Эти названия фигурировали в предыдущих разговорах, но после актуального аудита:

- я не нашёл подтверждённого официального Comfy-Org набора H3 diffusion + H3 Qwen, где INT4Q/INT4BQ можно просто использовать вместо штатных safetensors;
- GGUF существует как отдельное направление community experimentation, но loader и совместимость должны проверяться отдельно;
- **не добавлять в первую установку Codex**.

Статус: **исследовательские кандидаты, не baseline**.

---

# E. W4A8 / SVDQuant

Эти варианты ранее обсуждались как возможные эксперименты.

Текущий вывод аудита:
- это не обычные drop-in `.safetensors` замены;
- им нужен конкретный backend/toolchain;
- Windows/RTX 30 совместимость зависит от реализации;
- в первую установку не включать.

Статус: **поздняя экспериментальная ветка**.

---

# F. DynTime / DT-sQKV

В актуальном README DmitryDB stock-compatible варианты отделены от patch-required dynamic-time separate-QKV ветки `MiniMax-H3-DynTime-sQKV`.

Вывод:
- это отдельная ветка;
- обычный stock ComfyUI checkpoint ≠ DT-sQKV;
- не смешивать в одной первой установке.

Статус: **эксперимент после baseline**.

---

# G. Turbo LoRA

Turbo LoRA — не замена основной H3 модели. Это ускоряющая адаптация, позволяющая уменьшить число шагов. Community workflows используют 10-step режим, а стандартные качественные workflow обычно используют около 20 steps.

Статус: **ставить после того, как baseline работает**.

Для первого сравнения:
- Quality baseline: 20 steps.
- Turbo comparison: 10 steps.

Один и тот же input image, seed, prompt и resolution.

---

# 2. Реальный стартовый список для RTX 3070 Ti 8 GB + 32 GB

## Первый набор — НЕ «скачать всё»

### Обязательные общие компоненты
1. `minimax_h3_video_vae_fp16.safetensors`
2. `minimax_h3_audio_vae_fp32.safetensors`

Для H3 в ComfyUI это штатные companion VAE, выбирать между несколькими версиями на первом этапе не нужно.

### Diffusion baseline
**Кандидат A: `minimax_h3_fl2va_pruned_int8_convrot.safetensors`**

Это первый официальный baseline для I2V.

### Encoder baseline
**Не выбирать вслепую «8-bit» или «4-bit».** Codex должен сначала проверить, какой из реально доступных H3-compatible encoder путей совместим с RTX 30 и помещается в 32 GB RAM.

Приоритет проверки:
1. официальный/stock ComfyUI путь с корректным offload;
2. проверенный community low-memory H3-compatible encoder, если есть документированная совместимость с native H3 nodes;
3. только затем GGUF/INT4 experiments.

---

# 3. Порядок тестирования

## Тест 0 — проверка среды
- Обновить ComfyUI до версии с native MiniMax H3 nodes.
- Проверить PyTorch/CUDA.
- Не скачивать одновременно FL2VA и Ref2VA.
- Проверить свободное место на NVMe.

## Тест 1 — FL2VA Pruned INT8, I2V, quality baseline

Цель: понять, запускается ли система вообще.

Настройки:
- I2V, один стартовый 3D render.
- 5 секунд: **124 frames** при 24 FPS.
- 20 steps.
- Начать с **832×480** для горизонтального дешёвого теста или **480×832** для вертикального.
- Размеры кратны 32.
- Seed фиксировать.

Важно: 480×720 не кратно 32 и не является корректной точной стартовой сеткой H3. Для вертикального формата используем 480×832 как ближайший практический тест.

## Тест 2 — тот же workflow, Turbo LoRA
- Не менять image, prompt, seed, resolution.
- 10 steps.
- Сравнить качество/скорость.

## Тест 3 — Pruned FP8 Scaled
Если backend RTX 30 запускает его корректно:
- тот же workflow;
- 20 steps;
- сравнить время, peak VRAM, peak RAM и качество.

## Тест 4 — официальный full INT8
Только если есть запас RAM или streaming/offload режим доказан.

## Тест 5 — Ref2VA
После успешного FL2VA:
- сначала pruned INT8;
- затем pruned FP8, если тест 3 успешен.

## Тест 6+ — experiments
Отдельно и по одному:
- GGUF;
- INT4-family только с документированным H3 loader;
- W4A8/SVDQuant;
- DynTime DT-sQKV.

Удалённые W4/W8W4 DmitryDB **никогда не возвращать в план**.

---

# 4. Prompting: что реально важно

Официальная MiniMax документация говорит, что полная система использует H3-Context-IR, а локальный H3-Base получает только локальный этап. Поэтому простой художественный prompt может давать хуже результат, чем структурированное описание.

Для FL2VA/I2V используем структуру:

1. **Shot / scene** — где персонаж.
2. **Subject identity** — кто в кадре, что сохраняем.
3. **Action** — одно главное движение.
4. **Camera** — один простой тип движения.
5. **Timing** — когда начинается/заканчивается действие.
6. **Environment** — что должно оставаться стабильным.
7. **Audio** — только если нужен.

### Стартовый шаблон I2V

```text
[Shot 1]
Subject: preserve the character's identity, face, hairstyle, clothing, body proportions and scene layout from the input image.
Action: the character slowly turns her head toward the camera, blinks naturally, and subtly shifts her posture.
Camera: slow, stable cinematic push-in. No sudden camera movement.
Environment: preserve the background, lighting and composition from the input image.
Timing: gentle continuous motion from the beginning to the end of the clip.
Audio: quiet room ambience only.
```

Для теста модели prompt должен быть максимально простым. Не добавлять одновременно 10 действий, несколько смен камеры и длинный сюжет.

### Для Ref2VA
Использовать теги `<Picture 1>`, `<Picture 2>` и т.д. в том порядке, в котором references подаются в node. Порядок references влияет на conditioning.

---

# 5. Что измерять

После каждого запуска записывать:

| Поле | Значение |
|---|---|
| Diffusion model | |
| Encoder | |
| Workflow | FL2VA / Ref2VA |
| Resolution | |
| Frames | |
| Seconds | |
| Steps | |
| Turbo LoRA | yes/no |
| Seed | |
| Peak VRAM | |
| Peak RAM | |
| Total time | |
| s/step | |
| Face consistency | 1–5 |
| Motion quality | 1–5 |
| Background stability | 1–5 |
| Artifacts | notes |

---

# 6. Прямое задание Codex Agent

## Цель

Установить только актуальный, проверяемый baseline MiniMax H3 для ComfyUI на Windows с RTX 3070 Ti Laptop 8 GB VRAM и 32 GB RAM. Не скачивать удалённые или недоказанные варианты.

## Обязательные действия

1. Проверить текущую версию ComfyUI и наличие native MiniMax H3 support.
2. Создать отдельную папку/manifest `MiniMax-H3` с логом всех файлов и источников.
3. Скачать штатные VAE:
   - `minimax_h3_video_vae_fp16.safetensors`
   - `minimax_h3_audio_vae_fp32.safetensors`
4. Скачать только один diffusion baseline:
   - `minimax_h3_fl2va_pruned_int8_convrot.safetensors`
5. Не скачивать Ref2VA до успешного FL2VA теста.
6. Не скачивать W4 ConvRot, W4 Offload или W8W4 ConvRot из старых ссылок DmitryDB: они удалены автором как failed experiments.
7. Не использовать NVFP4 как «оптимальный» путь на RTX 3070 Ti без явной проверки backend compatibility.
8. Перед загрузкой text encoder провести compatibility check: H3 требует специальный Qwen3-VL-32B conditioner; обычные Qwen LLM не подходят.
9. Выбрать только encoder, который документирован как совместимый с текущими MiniMax H3 ComfyUI nodes и может быть реально размещён/выгружен при 32 GB RAM. Если такого encoder не удаётся подтвердить — остановиться и написать отчёт, а не скачивать случайный GGUF.
10. Использовать native I2V workflow template или точную проверенную копию.
11. Первый тест: 124 frames (~5.17 sec), 24 FPS, 20 steps, fixed seed, 832×480 или 480×832.
12. Записать peak VRAM, peak RAM, время, ошибки и полный console log.
13. Только после успешного baseline установить Turbo LoRA и повторить тест на 10 steps.
14. Затем протестировать pruned FP8 scaled, если текущий RTX 30 backend поддерживает его корректно.
15. Не менять несколько параметров одновременно.

## Стоп-условия

Если RAM превышает ~90–95% и начинается swap/thrashing, не продолжать длинные тесты. Сначала сохранить лог и предложить более лёгкий подтверждённый encoder/backend.

Если Windows/ComfyUI показывает OOM, сначала уменьшить canvas/frames или улучшить offload strategy, а не скачивать случайную другую модель.

---

# 7. Итоговый приоритет

## Ставим сейчас
1. ComfyUI native H3 support.
2. Video VAE.
3. Audio VAE.
4. FL2VA Pruned INT8 ConvRot.
5. Только подтверждённый H3-compatible low-memory encoder.
6. Первый I2V baseline.

## Ставим после baseline
7. Turbo LoRA.
8. Pruned FP8 scaled.
9. Ref2VA Pruned INT8.
10. Ref2VA Pruned FP8.

## Исследуем позже
11. GGUF.
12. INT4-family с подтверждённым loader.
13. W4A8/SVDQuant.
14. DynTime DT-sQKV.

## Не ставим
- удалённые W4 ConvRot;
- W4 ConvRot Offload;
- W8/W4 ConvRot failed experiments;
- NVFP4 как основной путь для RTX 3070 Ti;
- BF16 как baseline на 8 GB/32 GB.

---

# 8. Источники для Codex Agent

Официальный MiniMax H3:
https://huggingface.co/MiniMaxAI/MiniMax-H3

Официальная Comfy-Org repackaging:
https://huggingface.co/Comfy-Org/MiniMax-H3

ComfyUI H3 nodes:
https://github.com/Comfy-Org/ComfyUI/blob/master/comfy_extras/nodes_minimax_h3.py

DmitryDB актуальный quant audit:
https://huggingface.co/DmitryDB/MiniMax-H3-ComfyUI-Quants

DmitryDB commit removing failed W4/W8W4:
https://huggingface.co/DmitryDB/MiniMax-H3-ComfyUI-Quants/commit/06aba4816457400d1dcb12ffd77e37499c8116b1

Diffusers MiniMax H3 documentation:
https://huggingface.co/docs/diffusers/main/en/api/pipelines/minimax_h3

ComfyUI workflow templates:
https://github.com/Comfy-Org/workflow_templates

Community workflow observations are supporting evidence only and must not override official model-card compatibility claims.
