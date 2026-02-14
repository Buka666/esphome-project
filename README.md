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
- OTA-обновление из GitHub Releases через `update` (`platform: http_request`) по JSON-манифесту `${device_name}.json`.
- Для стабильной загрузки манифеста с GitHub увеличены HTTP-буферы (`buffer_size_rx: 4096`, `buffer_size_tx: 1024`) и таймаут `10s`, чтобы избежать ошибки `Out of buffer` на ESP32-C3.
- Кнопки для OTA в Home Assistant:
  - `Check OTA Update` — вручную проверяет наличие новой прошивки.
  - `Install OTA Update` — запускает установку найденного обновления.

> По умолчанию в `substitutions` указан `github_repo: Buka666/esphome-project`. Если у вас другой репозиторий, обязательно замените на `owner/repo`, иначе OTA будет давать 404.

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


## Прошивка устройства

### Первый запуск (через USB)

1. Подключите ESP32-C3 SuperMini по USB.
2. Узнайте порт устройства:
   ```bash
   python -m esphome logs devices/device1.yaml
   ```
   В выводе будет порт вида `/dev/ttyUSB0` или `/dev/ttyACM0` (Linux) / `COMx` (Windows).
3. Прошейте устройство по USB:
   ```bash
   python -m esphome run devices/device1.yaml --device <PORT>
   ```

### Дальнейшие обновления

- По USB (локально):
  ```bash
  python -m esphome upload devices/device1.yaml --device <PORT>
  ```
- По OTA из Home Assistant:
  1. Нажмите кнопку `Check OTA Update`.
  2. Если обновление найдено, нажмите `Install OTA Update`.

## GitHub Actions CI, версии и релизы

Workflow: `.github/workflows/build.yml`.

Workflow запускается на всех push в ветки и на тегах `v*`; для Pull Request запускается при изменениях в `devices/**`, `packages/**` и `.github/workflows/build.yml`.


Перед запуском CI добавьте Secrets в репозитории (`Settings` -> `Secrets and variables` -> `Actions`):

- `WIFI_SSID`
- `WIFI_PASSWORD`
- `API_ENCRYPTION_KEY`
- `OTA_PASSWORD`

Что делает workflow:

1. Формирует версию сборки:
   - для тегов `v*` версия = имя тега;
   - для остальных запусков версия вида `v0.0.<run_number>-<sha7>`.
2. Создает `devices/secrets.yaml` из GitHub Secrets.
3. Выполняет `esphome config` и `esphome compile` для всех файлов устройств в `devices/*.yaml` (кроме secrets).
4. Рекурсивно собирает firmware-бинарники (`firmware*.bin`) по workspace в `dist/` (с исключением `dist/` и `.git/`), сохраняя короткие имена (`<env>.bin`), и формирует JSON-манифесты для OTA в формате ESPHome (`chipFamily` + `ota.path`/`ota.md5`), где `ota.path` указывает на конкретный tag (`releases/download/<version>/...`) для исключения MD5 mismatch.
5. Публикует artifacts во всех запусках.
6. После успешной компиляции (кроме PR) автоматически создает/обновляет git-тег версии и публикует GitHub Release с `dist/*.bin` + `dist/*.json`.
7. В конце удаляет временный `devices/secrets.yaml`.

Для OTA через GitHub Releases можно использовать как ручные теги (например `v1.0.0`), так и автоматически созданные теги сборки (`v0.0.<run>-<sha7>`).
