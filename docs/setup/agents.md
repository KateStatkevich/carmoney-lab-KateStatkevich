## Агенты

### Planner

`planner` — основной агент для планирования изменений.

Может:

- читать код и документацию проекта;
- анализировать задачу;
- создавать планы только в `docs/plan/`.

Не может:

- изменять исходный код;
- выполнять bash-команды.

### Scout

`scout` — вспомогательный агент-разведчик для поиска по коду.

Может:

- читать файлы проекта;
- находить нужные места в коде;
- возвращать путь к файлу, строку и краткое описание найденного места.

Не может:

- изменять файлы;
- выполнять bash-команды;
- предлагать исправления.

### Результат Scout

> ### Разведка: где читается пробег (mileage)
>
> Корень репозитория: `C:\AI\carmoney-lab-KateStatkevich`. Искал `mileage`, `mileage_km`, `odometer`, геттеры и русские варианты.
>
> ### Активные чтения пробега в runtime-коде
>
>
> | # | Файл                                           | Строка | Что делается                                                                                                                                     |
> | - | -------------------------------------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | 1 | `frontend/app.js`                                  | 8            | `NUMERIC_FIELDS` объявляет `mileage` как числовое поле формы.                                                                  |
> | 2 | `frontend/app.js`                                  | 13–14       | `new FormData(form).forEach(...)` читает значение `input[name=mileage]` и кладёт `payload.mileage = Number(value)`.                    |
> | 3 | `backend/src/Domain/ApplicationValidator.php`      | 43           | `$mileage = (int) ($payload['mileage'] ?? -1);` — чтение из DTO/payload.                                                                           |
> | 4 | `backend/src/Domain/ApplicationValidator.php`      | 44           | `if ($mileage < 0                                                                                                                                           |
> | 5 | `backend/src/Domain/ApplicationValidator.php`      | 45           | `sprintf('Пробег от 0 до %d км', $this->rules['vehicle']['max_mileage_km'])` — чтение порога для текста ошибки.     |
> | 6 | `backend/src/Repository/ApplicationRepository.php` | 68           | `SELECT ... v.mileage_km, ...` в `find($id)` — чтение колонки `mileage_km` из БД; попадает в JSON `GET /api/applications/{id}`. |
>
> ### Источники/определения значения (для контекста, не чтения)
>
> * `backend/config/rules.php:23` — порог `'max_mileage_km' => 500000`.
> * `db/schema.sql:22` — колонка `vehicles.mileage_km INT UNSIGNED NOT NULL`.
> * `db/seed.sql:31` — синтетические значения `mileage_km`.
> * `frontend/index.html:30–31` — поле `<input id="mileage" name="mileage" type="number" required>`.
> * `backend/src/Repository/ApplicationRepository.php:38–39, 45` — `INSERT ... mileage_km` + биндинг `':mileage'`.
> * `backend/src/Domain/ApplicationValidator.php:78` — проброс `'mileage' => $mileage` в результат (не самостоятельное чтение).
> * `tests/Unit/ApplicationValidatorTest.php:34`, `tests/Unit/AssessmentServiceTest.php:38` — фикстуры `validPayload()`/`payload()`.
>
> ### Куда пробег НЕ течёт (важно)
>
> * `AssessmentService` — работает с `requested_amount`, `market_value`, `year`; `mileage` приходит в `$input` и уходит в репозиторий как часть `$input`, явных операций нет.
> * `ApplicationController::create/ltv` — только пробрасывают payload в `AssessmentService`.
> * `ApplicationController::index` через `listApplications()` выбирает `vin, production_year` — **пробег в списке не возвращается**.
> * `ApplicationController::show` отдаёт результат `find()`, где `mileage_km` уже присутствует (см. пункт 6).
> * `DecisionEngine` принимает `float $ltv` — пробег в сигнатуре отсутствует (по `docs/setup/code_map.md` это будущая точка расширения для правила «>400 000 → review»).
> * В `LtvCalculator`, `VehicleAge`, `VinValidator`, `Database`, `Support/Json`, `AppFactory`, `public/index.php`, `public/router.php` упоминаний `mileage`/`odometer` нет.
>
> ### Только упоминания в тексте (исключено)
>
> `README.md` (L6, L47), `docs/setup/code_map.md`, `docs/sources/LOAN-12.md`, `docs/sources/client_note.md`, `docs/sources/CASE-08.md`, `docs/README.md`, `docs/hw1/README.md`, `docs/plan/README.md`, `docs/spec/README.md`, `docs/intent/README.md`, `docs/setup/kilo_hello.md`.
>
> ### Резюме
>
> Реальных чтений **три точки**:
>
> 1. **Из формы → payload** — `frontend/app.js` L8, L13–14.
> 2. **Из payload в домене** — `backend/src/Domain/ApplicationValidator.php` L43, L44, L45.
> 3. **Из БД** — `backend/src/Repository/ApplicationRepository.php` L68 (через `show()` попадает в JSON ответа).
>
> Никаких файлов не менял. Если нужно — могу на основе этой разведки собрать план под конкретную задачу (например, добавить правило «>400 000 → review» в `DecisionEngine`).

### Вызов Scout из Planner

Planner смог вызвать `@scout` как subagent. Scout самостоятельно выполнил поиск по коду и вернул найденные файлы и строки, после чего Planner использовал результат в своей работе.
