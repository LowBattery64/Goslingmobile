# Скрипты запуска Goslingmobile

Скрипты запускают сервер управления и видеосервер на Raspberry Pi. В текущем репозитории основной вариант — USB-камера и скрипты `start_usb_camera_robot.sh` / `stop_usb_camera_robot.sh`.

> **Важно о каталоге.** Текущие скрипты используют рабочий каталог `$HOME/labirinth_robot`. Поэтому клонируй новый репозиторий Goslingmobile именно в эту локальную папку, пока путь в скриптах не изменён.

## 1. Получить репозиторий на Raspberry Pi

Если репозиторий ещё не склонирован:

```bash
git clone https://github.com/LowBattery64/Goslingmobile.git ~/labirinth_robot
cd ~/labirinth_robot
```

Если папка уже существует, перейди в неё и обнови файлы:

```bash
cd ~/labirinth_robot
git remote -v
git pull
```

Убедись, что `origin` указывает на `https://github.com/LowBattery64/Goslingmobile.git`. Если ранее был настроен старый репозиторий, замени URL:

```bash
git remote set-url origin https://github.com/LowBattery64/Goslingmobile.git
git pull
```

## 2. Установить зависимости

```bash
sudo apt update
sudo apt install -y g++ pkg-config libopencv-dev
```

Проверь подключение Arduino и USB-камеры:

```bash
ls /dev/ttyUSB*
ls /dev/video*
```

По умолчанию используются Arduino `/dev/ttyUSB0` и камера `/dev/video0`.

## 3. Запустить систему с USB-камерой

```bash
chmod +x scripts/start_usb_camera_robot.sh scripts/stop_usb_camera_robot.sh
./scripts/start_usb_camera_robot.sh
```

Скрипт проверяет наличие каталога репозитория, Arduino и камеры, компилирует `labirinth_server.cpp` и `usb_camera_server.cpp`, затем запускает оба сервера в фоне.

### Параметры

| Переменная | Значение по умолчанию | Назначение |
|---|---|---|
| `SERIAL_DEVICE` | `/dev/ttyUSB0` | Последовательный порт Arduino |
| `USB_CAMERA_INDEX` | `0` | Индекс камеры: `0` означает `/dev/video0` |
| `VIDEO_PORT` | `5001` | TCP-порт видеопотока |
| `VIDEO_CONTROL_PORT` | `5002` | TCP-порт управления качеством и записью |
| `RECORD_DIR` | `/media/raspberry/76E8-CACF/omegabot_recordings` | Каталог видеозаписей |
| `RECORD_SEGMENT_SECONDS` | `600` | Длительность сегмента записи в секундах |

Пример запуска со второй камерой:

```bash
USB_CAMERA_INDEX=1 ./scripts/start_usb_camera_robot.sh
```

Каталог записи создаётся автоматически, но скрипт не проверяет, смонтирован ли USB-накопитель в этот путь.

## 4. Остановить серверы

```bash
./scripts/stop_usb_camera_robot.sh
```

## 5. Логи и проверка портов

Файлы создаются в корне локального каталога репозитория:

| Файл | Назначение |
|---|---|
| `server_console.log` | Вывод сервера управления |
| `usb_camera_console.log` | Вывод видеосервера |
| `.labirinth_server.pid` | PID сервера управления |
| `.usb_camera_server.pid` | PID видеосервера |

Команды диагностики:

```bash
tail -n 50 ~/labirinth_robot/server_console.log
tail -n 50 ~/labirinth_robot/usb_camera_console.log
ps aux | grep -E 'labirinth_server|usb_camera_server'
ss -lntp | grep -E ':5000|:5001|:5002'
```

## Сеть

| Назначение | Порт |
|---|---:|
| Управление и телеметрия | TCP `5000` |
| Видеопоток | TCP `5001` |
| Управление качеством и записью | TCP `5002` |

IP Raspberry Pi зависит от текущей сети. Адрес, который выводится скриптом, является параметром конкретной конфигурации, а не универсальным адресом. Если IP изменился, проверь настройки в `operator_ui/MainWindow.cpp`.

## Старый запуск через сетевую камеру

`start_robot.sh` и `stop_robot.sh` относятся к прежней конфигурации с `camera_bridge.py` и внешним источником видеопотока. Файл `camera_bridge.py` не находится в корне текущего репозитория; архивная копия лежит в `archive/`. Этот сценарий не является рекомендуемым запуском. Для актуальной USB-камеры используй `start_usb_camera_robot.sh`.

## Ограничения

- Скрипты не прошивают Arduino: прошивку `str02-controller.ino` нужно загрузить отдельно.
- Перед запуском Arduino должна быть подключена к Raspberry Pi и доступна через указанный Serial-порт.
- Камера должна определяться как `/dev/videoN`.
- Для записи на внешний накопитель он должен быть смонтирован в `RECORD_DIR`.
- После клонирования не переименовывай локальный каталог без изменения `REPO_DIR` в скриптах: сейчас там явно указан `$HOME/labirinth_robot`.
