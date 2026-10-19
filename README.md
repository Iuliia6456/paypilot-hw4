# L05 · Multi-turn Test Plan — PayPilot


## Run

Copy `l05_dialogs.jsonl` into:
`C:\Users\YJukova\paypilot-l05-eval\sets`

Run from `C:\Users\YJukova\paypilot-l05-eval`:

```powershell
docker compose run --rm eval --set l05_dialogs --dry-run
docker compose run --rm eval --set l05_dialogs --profile clean --traces
docker compose run --rm eval --set l05_dialogs --profile lesson-05 --traces
```

Final set hash: `6526568fcdaa`


## Dialogue cases

Each case runs five times on each profile.
All cases check the amount and reason code.

| ID | Role | Turns | Tracked fields | Unrelated topic blocks |
|---|---|---:|---:|---:|
| CONV-H01 | Main scenario | 7 | 4 | 2 |
| CONV-H02 | Shorter version | 5 | 4 | 2 |
| CONV-H03 | Two-field draft | 7 | 2 | 2 |
| CONV-H04 | Longer version | 10 | 4 | 3 |
| CONV-H05 | Control without summarization | 7 | 4 | 2 |


## Final results
| Profile | Cases passed | Individual runs passed | Report |
|---|---:|---:|---|
| clean | 5/5 | 25/25 | `reports/l05_dialogs-clean-20261018T232124.json` |
| lesson-05 | 0/5 | 5/25 | `reports/l05_dialogs-lesson-05-20261018T233426.json` |

H05 had one false alarm: the wording check matched a sentence
about review times. The bot still knew the correct transaction,
amount and reason. The automatic results above are unchanged.

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


## Comparison with the prediction

Original prediction commit: `00f1361`.

The updated clean set passed as expected.
H01–H04 showed failures on lesson-05, as expected.
H05's automatic failure was caused by our check, not demonstrated memory loss.

