# Краткие инструкции

## Что это
- Одностраничный ASCII-UI (`index.html`) для генерации обложек через kie.ai (модель `nano-banana-pro`).

## Как пользоваться
- Открыть `index.html` в браузере, ввести API key, задать промпт, выбрать размер/аспект/разрешение, загрузить до 8 фото, нажать «Сгенерировать».
- В списке файлов можно кликнуть по имени, чтобы показать его в превью; перетаскивать для смены порядка; удалять крестиком.
- После успешного ответа кнопка «Скачать результат» сохраняет PNG (итоговое изображение из модели).

## Как работает интеграция
- Каждое загруженное фото конвертируется в PNG data URL, загружается на `https://kieai.redpandaai.co/api/file-base64-upload`, берётся `downloadUrl`.
- Создаётся задача: `POST https://api.kie.ai/api/v1/jobs/createTask` с `model: nano-banana-pro`, `image_input: [downloadUrl1, downloadUrl2, ...]` (порядок — как в списке), `aspect_ratio`, `resolution`, `output_format`, `subject_position`, `subject_scale`, `prompt`.
- Опрос статуса: `GET https://api.kie.ai/api/v1/jobs/recordInfo?taskId=...` каждые 3 c, до 80 попыток (~4 мин). Если state=success без URL — продолжаем опрашивать, пока не появится ссылка.

## Парсинг результата
- Пытаемся вытащить ссылку из полей: `image_urls`, `images`, `data`, `output`, `output_urls`, `urls`, `result_urls`, `resultUrls`, `image_url`, `imageUrl`, `b64_json`.
- Если не нашли — смотрим верхний уровень `data` с теми же полями.
- Финальный fallback: первая `https://` ссылка в тексте ответа.
- При отсутствии ссылки после тайм-аута выводится полный `data` для диагностики.

## Настройки UI
- Пресеты размера: YouTube/Instagram/Stories. Отдельный выбор `aspect_ratio` (все варианты из API) и `resolution` (1K/2K/4K).
- Позиция и масштаб фото управляют подсказками в payload (`subject_position`, `subject_scale`).

## Тайм-ауты/поведение
- Тайм-аут генерации ~4 минуты (80 попыток по 3 c).  
- При state=success без URL — статус «Ответ success, жду ссылку...».

## Быстрые проверки
- Мини-ручной тест загрузки: `curl -s -X POST https://kieai.redpandaai.co/api/file-base64-upload -H "Authorization: Bearer <KEY>" -H "Content-Type: application/json" -d '{"base64Data":"data:image/png;base64,<B64>","uploadPath":"covers","fileName":"cover.png"}'`
- Мини-ручной createTask: `curl -s -X POST https://api.kie.ai/api/v1/jobs/createTask -H "Authorization: Bearer <KEY>" -H "Content-Type: application/json" -d '{"model":"nano-banana-pro","input":{"prompt":"test","image_input":["<URL>"],"aspect_ratio":"16:9","resolution":"1K","output_format":"png"}}'`
