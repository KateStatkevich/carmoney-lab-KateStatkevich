готов

1) По README: учебный сервис предварительной оценки заявки на заём под ПТС (carmoney-lab, практикум М3) — принимает заявку (VIN, год, пробег, оценочная стоимость, сумма, срок), считает LTV и возвращает решение approve/review/reject; все данные синтетические.
2) Makefile: up, down, ps, logs, install, test, lint, seed, help; docker-compose.yml описывает сервисы backend (php -S на 8080, порт через APP_PORT) и db (mysql:8.0, порт через DB_PORT, том db-data, initdb из db/schema.sql и db/seed.sql) — отдельных команд запуска/проверки в нём нет, только описание сервисов.
3) Решение approve/review/reject считается в backend/src/Domain (правила) при значениях порогов из backend/config/rules.php.

модель: MiniMax-M3
