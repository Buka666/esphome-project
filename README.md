# ESPHome Project

Шаблон ESPHome-проекта для **ESP32-C6 SuperMini** с переиспользуемыми пакетами и отдельными конфигами устройств.

## Структура

- `devices/` — конфигурации отдельных устройств.
- `packages/connectivity/` — общие пакеты для Wi‑Fi, API и OTA.

## Что настроено в `device1.yaml`

- Кнопка на `GPIO9`.
- Встроенный status LED (по умолчанию `GPIO15`).
- Встроенный RGB LED (WS2812, по умолчанию `GPIO8`).

> Если у вашей ревизии платы другие пины, измените `status_led_pin` и `rgb_led_pin` в блоке `substitutions`.

## Secrets без проблем с путями

ESPHome ищет `!secret` относительно **основного файла устройства** (`devices/device1.yaml`).
Чтобы исключить ошибки поиска пути при компиляции:

1. Скопируйте шаблон:
   ```bash
   cp devices/secrets.yaml.example devices/secrets.yaml
   ```
2. Заполните `devices/secrets.yaml` своими значениями.

`devices/secrets.yaml` добавлен в `.gitignore`, чтобы реальные учетные данные не попадали в git.

## Как использовать

1. Убедитесь, что заполнен `devices/secrets.yaml`:
   - `wifi_ssid`
   - `wifi_password`
   - `api_encryption_key` (base64 ключ)
   - `ota_password`
2. Проверьте конфиг:
   ```bash
   python -m esphome config devices/device1.yaml
   ```
3. Скомпилируйте прошивку:
   ```bash
   python -m esphome compile devices/device1.yaml
   ```
