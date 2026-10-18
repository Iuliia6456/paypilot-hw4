# L05 · Multi-turn Test Plan — PayPilot


## Запуск
Покладіть `l05_dialogs.jsonl` у `~/paypilot/l03/sets/` і виконайте:


    docker compose run --rm eval --set l05_dialogs --profile clean


`set_hash`: <12 символів>


## Діалогові кейси
| id | Роль у плані | Довжина · сутності · перемикання | Інваріанти |
| --- | --- | --- | --- |
| | | | |


## Прогони
| Прогін | Профіль | Результат | Звіт |
| --- | --- | --- | --- |
| Baseline | `clean` | <N/M> | `reports/…` |
| Профіль заняття | `lesson-05` | <N/M> | `reports/…` |

## Prediction before the HW test runs

Set hash: `ebed831d6530`

Each case will run five times on each profile.

On `clean`, I expect all five cases to pass.

On `lesson-05`, I expect:

| Case | Prediction |
| --- | --- |
| CONV-H01 | Some runs fail at step 5 after case details are lost during summarization at step 3. |
| CONV-H02 | Some runs fail at step 5. The five-step lab test already showed that a short conversation can lose details. |
| CONV-H03 | Some runs fail at step 5 or 7 because the draft amount or reason is lost or changed. |
| CONV-H04 | Some runs fail at step 8 or 10 after the extra SWIFT topic. |
| CONV-H05 | Expected to pass because summarization should not run during its seven steps. Other errors are still possible. |

These are predictions, not measured results.

## Test changes before the second run

The first clean run passed 2/5 cases. Some checks failed because
the final question did not ask the bot to repeat the amount and reason.

In H01, H04 and H05, the final question now asks for all checked details.
The amount-confirmation message now clearly says not to file anything.
The expected values and checks are unchanged.

Prediction: fewer failures caused by omitted details on clean.
On lesson-05, memory failures may still appear after summarization.
The first report is kept as evidence.


Прогноз до прогону `lesson-05`: <які кейси впадуть і на якому кроці>
Коміт із прогнозом: `<hash>`
Що впало: <id>. Розбіжність із прогнозом: <чим пояснюється або «немає»>
