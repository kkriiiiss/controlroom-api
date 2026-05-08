# ControlRoom API

**ControlRoom API** — учебный headless REST API для дисциплины «Программная инженерия управляющих систем».

Проект моделирует небольшую систему управления помещениями: аудитории, датчики, исполнительные устройства, правила автоматики, команды управления и аварийные уведомления.

---

## Возможности

- ведение справочника помещений;
- подключение датчиков и исполнительных устройств;
- прием показаний датчиков;
- хранение правил автоматики;
- автоматическое создание команд устройствам при срабатывании правил;
- автоматическое создание уведомлений `alerts`;
- просмотр сводки по системе;
- CRUD и search-эндпоинты;
- запуск локально или через Docker.

Пример логики:

> Если датчик температуры показывает значение выше 26 °C, API создает alert и команду кондиционеру перейти в режим охлаждения.

---

## Стек

- Python 3.11+
- FastAPI
- Pydantic
- SQLite
- Pytest
- Docker / Docker Compose
- GitHub Actions CI

---

## Быстрый запуск

### 1. Создание виртуального окружения

```bash
python -m venv .venv
```

### 2. Активация окружения

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Linux/macOS:

```bash
source .venv/bin/activate
```

### 3. Установка зависимостей

```bash
pip install -r requirements.txt
```

### 4. Запуск API

```bash
uvicorn app.main:app --reload
```

После запуска доступны:

- Swagger UI: <http://127.0.0.1:8000/docs>
- OpenAPI JSON: <http://127.0.0.1:8000/openapi.json>
- Healthcheck: <http://127.0.0.1:8000/health>

---

## Заполнение демо-данными

```bash
python -m app.seed
```

Команда создает тестовые помещения, устройства, датчики, правила автоматики и стартовые показания.

---

## Запуск через Docker

```bash
docker compose up --build
```

Swagger будет доступен по адресу:

```text
http://127.0.0.1:8000/docs
```

---

## Тесты

```bash
pytest -q
```

Тестами проверяется:

- доступность healthcheck;
- создание помещения и устройства;
- поиск помещений;
- автоматическое срабатывание правила;
- создание команды и alert.

---

## Основные сущности

### Room

Помещение, которым управляет система.

Поля: `id`, `name`, `floor`, `purpose`, `target_temperature`, `created_at`.

### Device

Исполнительное устройство: кондиционер, вентиляция, освещение и так далее.

Поля: `id`, `room_id`, `name`, `device_type`, `status`, `mode`, `power_kw`, `last_command_at`.

### Sensor

Датчик, привязанный к помещению.

Поля: `id`, `room_id`, `name`, `sensor_type`, `unit`, `is_active`.

### SensorReading

Показание датчика.

Поля: `id`, `sensor_id`, `value`, `measured_at`.

### ControlRule

Правило автоматики.

Пример:

```json
{
  "room_id": 1,
  "name": "Охлаждение лаборатории при перегреве",
  "metric_type": "temperature",
  "operator": ">",
  "threshold_value": 26,
  "target_device_type": "climate",
  "command_type": "set_mode",
  "command_payload": {
    "mode": "cooling",
    "target_temperature": 22
  },
  "is_enabled": true
}
```

### ControlCommand

Команда устройству. Может быть создана вручную оператором или автоматически правилом.

### Alert

Уведомление о нарушении контрольного условия.

---

## Формат ответа API

API использует единый формат ответа:

```json
{
  "data": {},
  "errors": [],
  "meta": {}
}
```

---

## Основные эндпоинты

### Помещения

```text
GET    /api/v1/rooms
POST   /api/v1/rooms
GET    /api/v1/rooms/{room_id}
PATCH  /api/v1/rooms/{room_id}
DELETE /api/v1/rooms/{room_id}
POST   /api/v1/rooms:search
```

### Устройства

```text
GET    /api/v1/devices
POST   /api/v1/devices
GET    /api/v1/devices/{device_id}
PATCH  /api/v1/devices/{device_id}
DELETE /api/v1/devices/{device_id}
POST   /api/v1/devices:search
POST   /api/v1/devices/{device_id}/commands
GET    /api/v1/commands
```

### Датчики и показания

```text
GET    /api/v1/sensors
POST   /api/v1/sensors
GET    /api/v1/sensors/{sensor_id}
PATCH  /api/v1/sensors/{sensor_id}
DELETE /api/v1/sensors/{sensor_id}
POST   /api/v1/sensors:search
POST   /api/v1/sensors/{sensor_id}/readings
GET    /api/v1/sensors/{sensor_id}/readings
```

### Правила автоматики

```text
GET    /api/v1/rules
POST   /api/v1/rules
GET    /api/v1/rules/{rule_id}
PATCH  /api/v1/rules/{rule_id}
DELETE /api/v1/rules/{rule_id}
POST   /api/v1/rules:search
```

### Уведомления и сводка

```text
GET    /api/v1/alerts
GET    /api/v1/alerts/{alert_id}
PATCH  /api/v1/alerts/{alert_id}
POST   /api/v1/alerts:search
GET    /api/v1/overview
```

---

## Структура проекта

```text
controlroom_api/
├── app/
│   ├── main.py
│   ├── database.py
│   ├── responses.py
│   ├── schemas.py
│   ├── seed.py
│   ├── services.py
│   └── routers/
├── tests/
│   └── test_api.py
├── docs/
│   └── api_examples.md
├── Dockerfile
├── docker-compose.yml
├── Makefile
├── pyproject.toml
├── requirements.txt
└── README.md
```

---

## Возможные доработки

- авторизация операторов;
- роли администратора, инженера и наблюдателя;
- журналирование действий;
- интеграция с MQTT;
- экспорт отчетов;
- веб-интерфейс диспетчера.
