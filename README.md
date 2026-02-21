# ESPHome Project

Шаблон ESPHome-проекта для **ESP32-C3 SuperMini** с переиспользуемыми пакетами и отдельными конфигами устройств.

## Структура

- `devices/` — конфигурации отдельных устройств (`device1.yaml` для ESP32-C3 и `device2.yaml` для ESP32-C6 SuperMini).
- `packages/connectivity/` — общие пакеты для Wi‑Fi, API и OTA.

## Что настроено в `device1.yaml`

- Плата: `ESP32-C3 SuperMini` (`esp32-c3-devkitm-1`).
- Кнопка на `GPIO9`.
- Индикатор статуса (`status_led`) на встроенном LED `GPIO8`.
- Системный датчик статуса (`binary_sensor` platform `status`) для Home Assistant.
- Веб-интерфейс ESPHome `web_server` версии 3.
- OTA-обновление из GitHub Releases через `update` (`platform: http_request`) по JSON-манифесту `${device_name}.json`.
- Для стабильной загрузки манифеста с GitHub увеличены HTTP-буферы (`buffer_size_rx: 4096`, `buffer_size_tx: 1024`) и таймаут `10s`, чтобы избежать ошибки `Out of buffer` на ESP32-C3.
- Индикатор версии прошивки (`text_sensor`): `${friendly_name} Firmware Version` (отображается в web-интерфейсе ESPHome).
- Индикатор уровня Wi‑Fi (`sensor`): `${friendly_name} WiFi Signal` в dBm.
- Кнопки для OTA в Home Assistant:
  - `Check OTA Update` — вручную проверяет наличие новой прошивки.
  - `Install OTA Update` — запускает установку найденного обновления.

> По умолчанию в `substitutions` указан `github_repo: Buka666/esphome-project`. Если у вас другой репозиторий, обязательно замените на `owner/repo`, иначе OTA будет давать 404.


## Что настроено в `device2.yaml`

- Плата: `ESP32-C6 SuperMini` (`esp32-c6-devkitc-1`).
- Имя устройства для OTA/артефактов: `esp32c6-supermini` (без суффикса `-1`).
- Кнопка на `GPIO9`, индикатор статуса (`status_led`) на `GPIO15`, RGB LED (`ws2812`) на `GPIO8`.
- OTA через GitHub Releases (`update` + кнопки `Check/Install OTA Update`).
- Индикаторы в web/HA: версия прошивки и уровень Wi‑Fi.

## Secrets без проблем с путями

ESPHome ищет `!secret` относительно основного файла конкретного устройства (например, `devices/device1.yaml` или `devices/device2.yaml`).
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
   python -m esphome config devices/device2.yaml
   ```
3. Версия прошивки по умолчанию задана как `v1.0.0` в `devices/device1.yaml` и `devices/device2.yaml` -> `substitutions.firmware_version` (эта версия отображается в `Firmware Version`).
4. Для локальной сборки при необходимости переопределите версию через `-s firmware_version <version>`.
5. Скомпилируйте прошивку:
   ```bash
   python -m esphome compile devices/device1.yaml
   python -m esphome compile devices/device2.yaml
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

Workflow запускается на push только в `main`/`master` и только по релевантным путям (`devices/**`, `packages/**`, `.github/workflows/build.yml`, `README.md`), без отдельного запуска на auto-tag. Для Pull Request — только на события `opened/synchronize/reopened`, только в `main`/`master`, и по тем же релевантным путям.


Это убирает лишние прогоны при PR+merge: нет запуска на `pull_request.closed` и нет отдельного запуска по push тега. Нумерация авто-версий для релизных push считается от последнего тега `v0.0.*` (+1), поэтому PR-прогоны больше не съедают номера билдов.

Перед запуском CI добавьте Secrets в репозитории (`Settings` -> `Secrets and variables` -> `Actions`):

- `WIFI_SSID`
- `WIFI_PASSWORD`
- `API_ENCRYPTION_KEY`
- `OTA_PASSWORD`

Что делает workflow:

1. Формирует версию сборки и передает её в `firmware_version` при `config/compile` (`-s firmware_version <version>`):
   - для тегов `v*` версия = имя тега;
   - для `push` в `main/master` версия = следующий тег формата `v0.0.N` (инкремент от последнего `v0.0.*`);
   - для `pull_request` используется preview-версия `pr-<number>-<sha7>` (без публикации релиза).
2. Создает `devices/secrets.yaml` из GitHub Secrets.
3. Выполняет `esphome config` и `esphome compile` для всех файлов устройств в `devices/*.yaml` (кроме secrets).
4. Рекурсивно собирает firmware-бинарники (`firmware*.bin`) по workspace в `dist/`, сохраняя исторический формат имен (`<env>-firmware.factory.bin` / `<env>-firmware.bin`), и формирует JSON-манифесты `<env>.json` для OTA в формате ESPHome (`chipFamily` + `ota.path`/`ota.md5`).
5. Публикует artifacts во всех запусках.
6. После успешной компиляции (кроме PR) автоматически создает/обновляет git-тег версии и публикует GitHub Release с `dist/*.bin` + `dist/*.json`.
7. В конце удаляет временный `devices/secrets.yaml`.

Для OTA через GitHub Releases можно использовать как ручные теги (например `v1.0.0`), так и автоматически созданные теги сборки (`v0.0.N`).
