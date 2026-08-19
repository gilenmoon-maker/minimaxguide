# MiniMax H3 + ComfyUI — практический гайд

**Цель:** локальная генерация видео для 3D Visual Novel.

**ПК:** RTX 3070 Ti 8 GB VRAM + 64 GB RAM.

> Статус документа: рабочая база, обновляется по мере проверки моделей и реальных тестов на RTX 3070 Ti.

## Содержание

1. [База](01-База.md)
2. [Установка](02-Установка.md)
3. [Модели и квантизации](03-Модели.md)
4. [I2V / T2V / Ref2VA](04-Workflow.md)
5. [Эксперименты](05-Эксперименты.md)
6. [Тестирование на RTX 3070 Ti](06-Тестирование.md)

## Что используем

- **MiniMax H3** — основная видео+аудио модель.
- **Qwen3-VL-32B** — текстовый/визуальный encoder для H3.
- **Video VAE** — декодирование видео.
- **Audio VAE** — декодирование аудио.
- **ComfyUI** — интерфейс и workflow.

## Наш подход

Не ищем самую большую модель. Ищем **лучшее качество за приемлемое время** на 8 GB VRAM.

Порядок тестов:

`Qwen 8-bit → Qwen 4-bit → H3 INT8/W4/INT4 → Ref2VA → экспериментальные варианты → GGUF`

## Актуальные официальные файлы

Comfy-Org сейчас публикует для H3 FL2VA и Ref2VA в BF16, INT8 ConvRot и pruned INT8 ConvRot, а также Qwen3-VL-32B в BF16, INT8 ConvRot и NVFP4/AWQ. Также опубликованы два VAE. См. [официальный репозиторий Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3).

## Важно

BF16-версии огромны и не являются нашей первой целью на 8 GB. Community INT4/W4 варианты тестируем отдельно. GGUF — отдельный формат/экосистема и не равен обычному safetensors workflow.

## Основные ссылки

- [ComfyUI](https://github.com/Comfy-Org/ComfyUI)
- [Comfy-Org MiniMax H3](https://huggingface.co/Comfy-Org/MiniMax-H3)
- [MiniMaxAI MiniMax H3](https://huggingface.co/MiniMaxAI/MiniMax-H3)
- [DmitryDB H3 ComfyUI Quants](https://huggingface.co/DmitryDB/MiniMax-H3-ComfyUI-Quants)
- [Merserk INT4 ConvRot](https://huggingface.co/Merserk/MiniMax-H3-INT4-ConvRot)
- [molbal H3 GGUF](https://huggingface.co/molbal/MiniMax-H3-GGUF)
- [leejet H3 GGUF](https://huggingface.co/leejet/MiniMax-H3-GGUF)

## Workflow templates

- [I2V](https://github.com/Comfy-Org/workflow_templates/blob/main/templates/video_minimax_h3_i2v.json)
- [T2V](https://github.com/Comfy-Org/workflow_templates/blob/main/templates/video_minimax_h3_t2v.json)
- [R2V](https://github.com/Comfy-Org/workflow_templates/blob/main/templates/video_minimax_h3_r2v.json)
