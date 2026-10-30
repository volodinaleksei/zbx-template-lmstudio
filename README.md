# Zabbix шаблон: LM Studio API

Мониторинг [LM Studio](https://lmstudio.ai) local server через Zabbix 7.x.

## Эндпоинт

`GET http://{HOST.CONN}:{$LMSTUDIO_PORT}/api/v0/models` — нарочно **v0**-совместимый путь:
он отдаёт плоский список с полем `state: loaded|not-loaded`, тогда как v1 (`/api/v1/models`)
вложены `loaded_instances[]` — парсить хуже.

## ITEM'ы

- `lmstudio.models.count` / `lmstudio.models.list` — всего моделей
- `lmstudio.models.loaded.count` / `lmstudio.models.loaded.list` — только `state == loaded`
- `net.tcp.service[http,,{$LMSTUDIO_PORT}]` — simple check порта

## LLD

Правило discovery по `/api/v0/models` — на каждую модель свои ITEM'ы (состояние, id);
триггер-прототип «модель выгружена» по смене `state`.

## Триггеры

| Событие | Sev | Условие |
|---|---|---|
| LM Studio: порт {$LMSTUDIO_PORT} недоступен | High | TCP закрыт |
| LM Studio: ни одной загруженной модели | Warning | порт живой, loaded пусто |

## Макросы

| Макрос | По умолчанию |
|---|---|
| `{$LMSTUDIO_PORT}` | `1234` |

## Установка

1. Import `template_lmstudio.yaml`.
2. Хост с Agent-interface (`{HOST.CONN}`), привязать шаблон.
3. LM Studio должен быть запущен как сервер (Developer → Start Server).
