# ESPHome Project

Небольшой шаблон ESPHome-проекта с переиспользуемыми пакетами и отдельными конфигами устройств.

## Структура

- `devices/` — конфигурации отдельных устройств.
- `packages/connectivity/` — общие пакеты для Wi‑Fi, API и OTA.

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
