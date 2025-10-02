# OpenAPI спецификация для ForcAD

Этот файл содержит полную OpenAPI 3.0 спецификацию для системы жюри ForcAD.

## Использование

### Просмотр документации

Вы можете просмотреть документацию используя различные инструменты:

#### 1. Swagger UI (онлайн)
Откройте [Swagger Editor](https://editor.swagger.io/) и вставьте содержимое файла `openapi.yaml`.

#### 2. Swagger UI (локально через Docker)
```bash
docker run -p 8080:8080 -e SWAGGER_JSON=/api/openapi.yaml -v $(pwd):/api swaggerapi/swagger-ui
```
Затем откройте http://localhost:8080

#### 3. Redoc (локально через Docker)
```bash
docker run -p 8080:80 -e SPEC_URL=/specs/openapi.yaml -v $(pwd):/usr/share/nginx/html/specs redocly/redoc
```
Затем откройте http://localhost:8080

### Генерация клиентов

Используйте [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator) для генерации клиентов на различных языках:

#### Python клиент
```bash
openapi-generator-cli generate \
  -i openapi.yaml \
  -g python \
  -o ./python-client \
  --additional-properties=packageName=forcad_client
```

#### JavaScript/TypeScript клиент
```bash
openapi-generator-cli generate \
  -i openapi.yaml \
  -g typescript-axios \
  -o ./ts-client
```

#### Go клиент
```bash
openapi-generator-cli generate \
  -i openapi.yaml \
  -g go \
  -o ./go-client \
  --additional-properties=packageName=forcad
```

### Валидация запросов

Используйте схему для валидации запросов и ответов в вашем приложении.

## Структура API

ForcAD предоставляет три основных API:

### 1. Client API (`/api/client/`)
Публичный API для участников соревнования. Не требует аутентификации.

**Основные эндпоинты:**
- `GET /api/client/teams/` - список команд
- `GET /api/client/tasks/` - список задач
- `GET /api/client/config/` - конфигурация игры
- `GET /api/client/attack_data/` - данные об атаках
- `GET /api/client/teams/{team_id}/` - история команды
- `GET /api/client/ctftime/` - таблица результатов для CTFTime

### 2. Admin API (`/api/admin/`)
Административный API для управления игрой. Требует аутентификации через cookie-сессию.

**Аутентификация:**
1. POST запрос на `/api/admin/login/` с credentials
2. Получение session cookie
3. Использование cookie во всех последующих запросах

**Основные эндпоинты:**
- Teams CRUD: `/api/admin/teams/`
- Tasks CRUD: `/api/admin/tasks/`
- TeamTasks: `/api/admin/teamtasks/`

### 3. HTTP Receiver API (`/flags`)
API для отправки флагов командами. Требует токен команды в заголовке `X-Team-Token`.

**Основные эндпоинты:**
- `PUT /flags/` - отправить флаги (до 100 за раз)

## Примеры использования

### Получение списка команд
```bash
curl -X GET http://localhost:8080/api/client/teams/
```

### Логин администратора
```bash
curl -X POST http://localhost:8080/api/admin/login/ \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "admin"}' \
  -c cookies.txt
```

### Создание команды
```bash
curl -X POST http://localhost:8080/api/admin/teams/ \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "name": "HackerTeam",
    "ip": "10.60.1.1",
    "highlighted": false,
    "active": true
  }'
```

### Отправка флагов
```bash
curl -X PUT http://localhost:8080/flags/ \
  -H "Content-Type: application/json" \
  -H "X-Team-Token: a1b2c3d4e5f6g7h8" \
  -d '["31337_abcdef1234567890abcdef1234567890=", "31337_1234567890abcdefabcdef1234567890="]'
```

## Модели данных

### Team (Команда)
- `id`: уникальный идентификатор
- `name`: название команды
- `ip`: IP-адрес команды
- `token`: токен для отправки флагов (только в Admin API)
- `highlighted`: выделение в таблице результатов
- `active`: активность команды

### Task (Задача/Сервис)
- `id`: уникальный идентификатор
- `name`: название задачи
- `checker`: имя чекера
- `gets`, `puts`, `places`: параметры проверок
- `checker_timeout`: таймаут чекера
- `checker_type`: тип чекера (pfr, nfr и т.д.)
- `default_score`: базовые очки
- `active`: активность задачи

### GameConfig (Конфигурация игры)
- `flag_lifetime`: время жизни флага в раундах
- `game_hardness`: сложность игры
- `round_time`: длительность раунда
- `mode`: режим игры (classic/blitz)
- `start_time`: время начала
- `real_round`: текущий раунд

## Примечания

1. **Формат флагов**: Флаги имеют формат `{round}_{hash}=`, например `31337_abcdef1234567890abcdef1234567890=`

2. **Ограничения**:
   - Максимум 100 флагов за один запрос
   - Флаги можно отправлять только во время активной игры

3. **Типы чекеров**:
   - `pfr` - чекер возвращает публичные данные флага
   - `nfr` - чекер не возвращает flag_id

4. **Аутентификация**:
   - Admin API использует cookie-based аутентификацию
   - Flags API использует token-based аутентификацию через заголовок

## Дополнительная информация

Для более подробной информации о ForcAD посетите:
- [GitHub репозиторий](https://github.com/pomo-mondreganto/ForcAD)
- [Документация](https://github.com/pomo-mondreganto/ForcAD/blob/master/README.md)

