# ESPHome Project

Шаблон ESPHome-проекта для **ESP32-C3 SuperMini** с переиспользуемыми пакетами и отдельными конфигами устройств.

## Структура

- `devices/` — конфигурации отдельных устройств.
- `packages/connectivity/` — общие пакеты для Wi‑Fi, API и OTA.

## Что настроено в `device1.yaml`

- Плата: `ESP32-C3 SuperMini` (`esp32-c3-devkitm-1`).
- Кнопка на `GPIO9`.
- Индикатор статуса (`status_led`) на встроенном LED `GPIO8`.
- Системный датчик статуса (`binary_sensor` platform `status`) для Home Assistant.
- Веб-интерфейс ESPHome `web_server` версии 3.

> Если у вашей ревизии платы другие пины, измените `status_led_pin` в блоке `substitutions`.

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



## GitHub Actions CI

Для автоматической проверки и компиляции добавлен workflow: `.github/workflows/build.yml`.

Перед запуском CI добавьте Secrets в репозитории (`Settings` -> `Secrets and variables` -> `Actions`):

- `WIFI_SSID`
- `WIFI_PASSWORD`
- `API_ENCRYPTION_KEY`
- `OTA_PASSWORD`

Workflow создает `devices/secrets.yaml` из этих значений и выполняет `esphome config` + `esphome compile` для всех файлов устройств в `devices/*.yaml` (кроме файлов secrets).\n\nWorkflow запускается при изменениях в `devices/**`, `packages/**` и в самом `.github/workflows/build.yml`. В конце сборки временный `devices/secrets.yaml` удаляется.

