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

## Как использовать

1. Создайте файл `secrets.yaml` рядом с конфигом устройства и добавьте секреты:
   - `wifi_ssid`
   - `wifi_password`
   - `api_encryption_key`
   - `ota_password`
2. Проверьте конфиг:
   ```bash
   esphome config devices/device1.yaml
   ```
3. Скомпилируйте прошивку:
   ```bash
   esphome compile devices/device1.yaml
   ```
