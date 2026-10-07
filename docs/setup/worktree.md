# Тесты в tests/Unit/ (ветка easy-hardcover)

- `ApplicationValidatorTest.php` — валидация заявки: нормализация VIN, отказ при годе из будущего, суммирование всех ошибок разом.
- `AssessmentServiceTest.php` — сквозная оценка заявки: approve при низком LTV, review при среднем, reject при высоком.
- `DecisionEngineTest.php` — решение по LTV: approve ≤ 60, review ≤ 85, reject выше, включая границу 85.01.
- `LtvCalculatorTest.php` — расчёт LTV в процентах и InvalidArgumentException при нулевой стоимости или неположительной сумме.
- `VinValidatorTest.php` — формат VIN: 17 символов, регистронезависимость, запрещённые I/O/Q, спецсимволы, пустая строка.

Результат получен в worktree `C:\AI\carmoney-lab-KateStatkevich\.kilo\worktrees\easy-hardcover`, ветка `easy-hardcover`.

C:\AI\carmoney-lab-KateStatkevich>git worktree list
C:/AI/carmoney-lab-KateStatkevich                                     ae61628 [d1/1.2.1-1.2.3-KateStatkevich]
C:/AI/carmoney-lab-KateStatkevich/.kilo/worktrees/easy-hardcover      ae61628 [easy-hardcover]
C:/AI/carmoney-lab-KateStatkevich/.kilo/worktrees/worried-basketball  ae61628 (detached HEAD)

Ответ на вопрос "Почему два агента в одной папке на одной ветке — плохая идея ?":

Потому что оба агента будут одновременно менять одни и те же файлы и Git-состояние, из-за чего легко получить конфликты или перезапись изменений.
