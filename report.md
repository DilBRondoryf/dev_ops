# CI practice report

## Запуски GitHub Actions

Ссылки будут добавлены после создания и публикации репозитория `cicd-first-run`:

- Красный запуск: `https://github.com/<owner>/cicd-first-run/actions/runs/<red-run-id>`; commit `d9a638f` (`break stylesheet link`). После push замените `<owner>` и `<red-run-id>`.
- Исправленный зелёный запуск: `https://github.com/<owner>/cicd-first-run/actions/runs/<green-run-id>`; commit `4dc376e` (`restore stylesheet link`). После push замените `<owner>` и `<green-run-id>`.

## Прогноз до поломки

После замены `href="style.css"` на `href="missing.css"` упадёт тест `test_stylesheet_exists`. Шаг `Run tests` завершится с ошибкой `Missing stylesheet: missing.css`, поэтому из-за последовательного выполнения шаги `Build site` и `Save site artifact` не выполнятся.

## Фактический результат поломки

Локальный эквивалент GitHub Actions также дал 3 успешных теста и 1 ошибку:

```text
test_stylesheet_exists ... FAIL
AssertionError: False is not true : Missing stylesheet: missing.css
Ran 4 tests in 0.001s
FAILED (failures=1)
```

На GitHub это должен быть красный запуск job `check-build`. Runner выполняет команды workflow, а обычные шаги идут последовательно; ненулевой код `Run tests` останавливает job. Поэтому `Build site` и `Save site artifact` после ошибки не стартуют, и артефакт `lesson-site` не сохраняется.

## Исправление

В коммите `restore stylesheet link` (`4dc376e`) ссылка возвращена к `href="style.css"`. После push исправленного коммита ожидаются 4 успешных теста, сборка `dist/` и сохранённый артефакт `lesson-site`.

## Где выполнялись проверки

Проверки выполняются на GitHub-hosted runner `ubuntu-latest`, потому что это указано в `runs-on` workflow. Локальный запуск нужен для быстрой проверки воспроизводимости, но только GitHub Actions показывает результат именно настроенного workflow, его журнал и артефакт в интерфейсе GitHub.

## Ограничение проверки аудитории

После отдельного коммита `change classroom number` (`773e5f8`) с заменой `Аудитория: 301` на `Аудитория: 302` тесты остаются зелёными: текущий набор проверяет заголовок, тему, наличие стилей и title, но не номер аудитории. Поэтому зелёный результат после изменения аудитории не доказывает правильность номера: это непроверяемое содержимое, отсутствующее в контракте тестов.
