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
- OTA-обновление из GitHub Releases через `update` (`platform: http_request`).

> В `substitutions` задайте `github_repo` в формате `owner/repo` для OTA из GitHub Releases.

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

## GitHub Actions CI, версии и релизы

Workflow: `.github/workflows/build.yml`.

Перед запуском CI добавьте Secrets в репозитории (`Settings` -> `Secrets and variables` -> `Actions`):

- `WIFI_SSID`
- `WIFI_PASSWORD`
- `API_ENCRYPTION_KEY`
- `OTA_PASSWORD`

Что делает workflow:

1. Формирует версию сборки:
   - для тегов `v*` версия = имя тега;
   - для остальных запусков версия вида `0.0.<run_number>-<sha7>`.
2. Создает `devices/secrets.yaml` из GitHub Secrets.
3. Выполняет `esphome config` и `esphome compile` для всех файлов устройств в `devices/*.yaml` (кроме secrets).
4. Рекурсивно собирает firmware-бинарники (`firmware*.bin`) по workspace в `dist/` (с исключением `dist/` и `.git/`) и формирует JSON-манифесты для OTA.
5. Публикует artifacts во всех запусках.
6. На тегах `v*` создает GitHub Release и прикладывает `dist/*.bin` + `dist/*.json`.
7. В конце удаляет временный `devices/secrets.yaml`.

Для OTA через GitHub Releases создавайте релиз тегом вида `v1.0.0`.
