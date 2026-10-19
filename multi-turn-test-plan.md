# Multi-turn Test Plan + Degradation Profile 

## 1. Context and Forecast

| Field | Value |
|---|---|
| System | PayPilot |
| Provider | Anthropic |
| Model | claude-haiku-5-5 |
| Profiles | `clean`; `lesson-05` with D06 and D07 |
| Main scenario | C-07, seven steps, client `CUS-0004`, transaction `TX-0401` |
| Lab run dates | 17–18 October 2026 |
| Simulated business time | `2026-09-15T10:00:00+00:00` |
| Dataset | `l05_dialogs.jsonl`; 5 cases; 5 runs per case per profile; `set_hash: 6526568fcdaa` |
| HW run date | 19 October 2026, Europe/Warsaw (report filenames use 18 October UTC) |
| HW results | `clean`: 5/5 cases, 25/25 runs passed. `lesson-05`: 0/5 cases, 5/25 runs passed automatically. One failed run was a false alarm in H05. |


Pre-lab prediction: Not recorded before the first run.

HW prediction was saved in commit `00f1361` before the first HW run: clean should pass; H01–H04 may fail after summarization; H05 should pass without summarization. The first clean run exposed weak questions: the tests required details that the last question did not request. H01, H04 and H05 were then updated to ask for those details and clearly prohibit filing. The results below use the updated set only. The original report is kept.

Final reports:
- `l05_dialogs-clean-20261018T232124.json`
- `l05_dialogs-lesson-05-20261018T233426.json`

---

## 2. Degradation profile

### 2.1. Record entry: test run on `clean`

Profile `clean` · Client `CUS-0004` ("as customer") · new conversation.

| Step | Customer message (shortened) | Agent response (shortened) | Entities in history | Retrieval context for the step |
| --- | --- | --- | --- | --- |
| 1 | I'm CUS-0004. Transaction TX-0401, 240.00 EUR: the goods never arrived. Which reason code applies? | `goods_not_received`; 90-day window. Transaction eligibility not checked yet. | User message contains TX-0401 and EUR 240. Actual model-input history not yet inspected. | kb_clean, top_k 4: `disputes.md#s1` - “goods_not_received \| 90”; `reason-codes.md#s6`; `disputes-process.md#s6`; `glossary.md#s4`. |
| 2 | Can I dispute it? | TX-0401 is eligible under goods_not_received; EUR 240; transaction date 20 July 2026; 90-day window; deadline 18 October 2026. Requests confirmation before filing. | Actual input history not yet inspected. Tool arguments correctly retain TX-0401 and goods_not_received; the response repeats EUR 240. | No knowledge-base search. check_dispute_eligibility returns eligible=true, window_days=90, tx_date=2026-07-20, deadline=2026-10-18; all checks pass. |
| 3 | What is my daily card limit? | Daily transfer limit EUR 5,000; cannot confirm a separate card-specific limit. Repeats TX-0401, EUR 240, goods_not_received, and deadline 18 October 2026. | Summarized history (agent.summarize, 8 messages replaced) retains EUR 240.00, transaction date 20 July 2026, goods_not_received, and the 90-day window. TX-0401 and deadline 18 October 2026 are also retained. No loss of the tracked entities at this step. | No search shown in supplied details. get_account confirms Tier 1; check_limits returns daily limit and remainder EUR 5,000. |
| 4 | And the FX spread if I pay in USD? | Cannot confirm the Tier 1 spread; explains conversion components and offers a quote. | Step-3 summary preserved all four entities. Step-4 input history not yet inspected. | kb_clean, top_k 4: `fx-guide.md#s1`, `fx-operations.md#s1`, `fees-detail.md#s2`, `fx-guide.md#s2`. No Tier 1 spread percentage in the returned fragments. |
| 5 | Back to that charge—what do I do next? | Requests yes/no confirmation for TX-0401, EUR 240, dated 20 July 2026, goods not received; repeats deadline 18 October 2026.| Step-5 input history not inspected. The answer correctly repeats the case details preserved in the step-3 summary. | No search or other tool calls; only llm.call. No new summarization span.|
| 6 | 240 euro, as I said at the start. | Dispute not filed; requests explicit yes/no confirmation. Repeats TX-0401, EUR 240, 20 July 2026, goods not received, and deadline 18 October 2026. | Input history not inspected. The answer retains the amount, transaction date, reason and deadline. | No retrieval or other tool calls; only llm.call. No summarization span.|
| 7 | And I still have time to file, right? | Confirms deadline 18 October 2026 and earlier eligibility check. Retains TX-0401, EUR 240 and goods-not-received reason; awaits filing confirmation. | Input history not inspected. The answer retains the amount, reason and deadline, consistent with step 2. | No search or other tool calls; only llm.call. No summarization span.|


### 2.2. Record entry: test run on `lesson-05`

| Step | Agent response (shortened) | Entities in history | Retrieval context for the step | Is loss visible in the response? |
| --- | --- | --- | --- | --- |
| 1 | Identifies `goods_not_received`; repeats TX-0401 and EUR 240. Eligibility not checked; offers to check and file. | User message supplies TX-0401 and EUR 240. Input history not inspected. Transaction date and window have not been established in this conversation. | No search or other tool calls; only `llm.call`. No summarization span. | No. The answer retains the transaction and amount. |
| 2 | Confirms TX-0401 is eligible under goods_not_received; EUR 240; transaction date 20 July 2026; 90-day window; deadline 18 October 2026. Requests filing confirmation. | Input history not inspected. Tool arguments retain TX-0401 and reason code. Tool result establishes date and window; the answer repeats EUR 240. | No knowledge-base search. check_dispute_eligibility: eligible=true, window_days=90, tx_date=2026-07-20, deadline=2026-10-18; all checks pass. | No. The answer retains the amount and reason and states the tool’s date and window. |
| 3 | Daily transfer limit EUR 5,000. Says the goods-not-received dispute passed eligibility and awaits confirmation for the full transaction amount. | Summary replaces 6 messages. Amount, transaction ID, date, window and deadline are absent. Goods-not-received reason, eligibility outcome and pending confirmation remain. | get_account confirms Tier 1; check_limits returns daily limit EUR 5,000. Neither supplied tool result restores the missing dispute details. | The current question concerns limits. “Full transaction amount” does not demonstrate recall of EUR 240. |
| 4 | States Tier 1 FX spread 1.5%, monthly allowance EUR 500, and card fee 1.2%. Says the dispute awaits confirmation. | Step-3 summary omitted the dispute amount, ID, date, window and deadline; retained the goods-not-received reason. Step-4 input history not inspected. | kb_clean, top_k 4: `tariffs.md#s2` - “Tier 1,  1.5%  EUR 500”; `fx-guide.md#s1`; `fees-detail.md#s2`; `tariffs.md#s3` - card fee 1.2%. No case-specific dispute details. | No clear symptom: the answer addresses FX and does not need the dispute amount. |
| 5 | Asks which TechMart charge is disputed: TX-0401 or TX-0402, both EUR 240. Retains goods-not-received as a possible reason. | Step-3 summary omitted the selected transaction ID and amount. Step-5 input history not inspected; the tool now supplies both candidate transactions and their amounts/dates. | get_transactions returns TX-0401: EUR 240, 20 July 2026; TX-0402: EUR 240, 14 July 2026. These records do not identify which transaction the customer selected earlier. | Yes: asks the customer to identify the transaction again. The amount appears through a fresh tool lookup. |
| 6 | Both charges are EUR 240; asks for transaction ID or date before checking eligibility. Keeps goods-not-received as the intended reason. | Current user message supplies EUR 240 again. Step-5 tool results supplied both transaction IDs and dates, but the earlier selection of TX-0401 remains unresolved in the answer. Input history not inspected. | No search or other tool calls; only llm.call. No new summarization span. | Yes: still cannot identify the previously selected transaction. |
| 7 | Both charges are eligible under goods_not_received, with a 90-day window. Deadlines: TX-0401—18 October; TX-0402—12 October. Still asks which transaction to dispute. | Input history not inspected. Fresh tool results restore dates, windows and deadlines for both charges. The earlier selection of TX-0401 remains unresolved. |No knowledge-base search. Two check_dispute_eligibility calls confirm both transactions are eligible and return their respective deadlines. | Yes: says the customer had not identified the transaction, although TX-0401 was specified at step 1. No incorrect deadline. |

### 2.3. Loss steps across five runs

Profile `lesson-05`, each run in a new conversation.

| Run | Loss step: amount disappears from history | Manifestation step (loss becomes visible in the response) |
| --- | --- | --- |
| 1 (from 2.2) | 3 | 5 - asks which transaction was selected |
| 2 | 3 | Not observed through step 7; later tool calls and the repeated amount supplied missing details |
| 3 | 3 | 5 - asks which transaction was selected |
| 4 | 3 | 6 - asks for the previously supplied transaction ID |
| 5 | 3 | 3 - repeats the wrong reason code; transaction confusion appears at step 5 |

**Sorted loss steps**: 3, 3, 3, 3, 3.<br>
**Range**: 3-3.<br>
**Median**: 3.<br>
**Operational boundary**: Starting from step 3 - after the first summarization - do not rely solely on the dialogue history to retain details of the dispute. This limitation applies to the tested configuration, in which summarization occurs after two steps.<br>
**Comparison with the prediction**: No pre-run prediction was recorded; a comparison is not applicable.
All five documented summaries lost the amount at step 3. Visible consequences varied: steps 5, not observed through 7, 5, 6, and 3 respectively. These are separate observations from the loss-step distribution.

### 2.4. Degradation profile of the baseline scenario

| Step | What is retained | What is lost | Evidence available for the step | Mechanism | Visible in response? |
|---|---|---|---|---|---|
| 1 | Customer supplies TX-0401 and EUR 240; agent identifies `goods_not_received`. | No loss established. Date and window are not yet established. | Customer message; no tools. | None established. | No. |
| 2 | TX-0401, EUR 240 and reason; tool establishes 20 July 2026, 90 days and deadline 18 October. | No loss established. | Eligibility-call arguments and result. | None established. | No. |
| 3 | Summary retains goods-not-received reason, successful eligibility and pending confirmation. | Amount, selected transaction ID, date, window and deadline disappear from the summary. | `summary.text`, replacing six messages; account and limit tools do not restore dispute details. | D06: loss during summarization. | No clear symptom; answer concerns limits. |
| 4 | FX information is retrieved correctly. Previous summary retained the dispute reason. | Missing dispute details are not restored by the FX fragments. | Tariff and FX fragments. | Earlier D06 loss; no new loss established. | No clear symptom; answer concerns FX. |
| 5 | Transaction search supplies amounts, dates and IDs for both TechMart charges. | The customer’s original selection of TX-0401 remains unresolved. | `get_transactions` results; these identify transactions but not the customer’s earlier selection. | Visible consequence consistent with the step-3 loss. | **Yes:** asks which transaction. |
| 6 | Customer supplies EUR 240 again; both candidate transactions were returned at step 5. | Original transaction selection remains unresolved. | Current customer message; no new tools. | Earlier loss remains unresolved. | **Yes:** asks for transaction ID or date. |
| 7 | Fresh eligibility checks restore both transactions’ windows and deadlines. | Original selection of TX-0401 remains unresolved. | Two eligibility results, both under `goods_not_received`. | Tools recover facts, but not the customer’s selection. | **Yes:** still asks which transaction; no deadline contradiction. |

**Loss point**: Step 3.<br>
**First visible manifestation**: Step 5.<br>
**Steps without a visible failure**: Four-steps 1-4. Among them, steps 3-4 conceal a loss that has already occurred.


### 2.5. Dialogue length: shorter and longer scenarios.

| Scenario version | Profile | Did the amount disappear? At which step? | What else was lost? |
|---|---|---|---|
| 5 steps | `lesson-05` | Yes, at step 3: EUR 240 is absent from the summary. | Selected transaction ID, transaction date, 90-day window and deadline. The goods-not-received reason survives. |
| 7 steps (main) | `lesson-05` | Yes, at step 3 in all five runs with summary evidence in Section 2.3. | Date, window and deadline; selected transaction ID in some runs. Two summaries substitute `service_not_rendered` for the original reason. |
| 10 steps (+1 topic switch) | `lesson-05` | Yes, at step 3: EUR 240 is absent from the summary. | Selected transaction ID, date, window and deadline. The reason changes to `service_not_rendered`; this becomes visible at step 8 and is used in eligibility calls at step 9. |

**Conclusion about the dialogue length**: With summarization after two steps, all tested lengths lost the amount at step 3; the ten-step run additionally propagated a corrupted reason into tool calls, but these runs do not establish that greater length caused more loss.

---

## 3. Failure mechanisms

| What happened | Evidence needed |
|---|---|
| Forgot or changed case details (D06) | A value was present before summarization and is missing or changed in the summary. |
| Contradicted itself (D07) | The needed inputs remain correct, but the later answer conflicts with the earlier answer. |
| Contradiction after D06 | A needed value was lost or changed in the summary before the conflicting answer. |
| Retrieval failure | The current search did not return the information needed for that answer. No search call alone does not prove this. |

### 3.1. Loss Step and Proof of Cause

Loss step: 3.

**Evidence before loss**:
Step 1 identified TX-0401 and EUR 240.
Step 2's tool result supplied the transaction date, 90-day
window and deadline.

**Evidence after loss**:
The step-3 summary omitted the transaction ID, amount,
date, window and deadline. It retained the goods-not-received
reason and the fact that eligibility had passed.

**Manifestation step**: 5.
The agent asked which of two transactions the customer meant.

**Context at manifestation**:
get_transactions returned both EUR 240 charges correctly.
It did not restore the customer's earlier selection of TX-0401.
No knowledge-base search occurred at this step.

**Mechanism**:
D06 - summarization removed case details, including the
selected transaction. Fresh tool calls recovered some facts
but did not recover the customer's selection.

D07 deadline contradiction:
Not observed in this run. Step 7 used fresh eligibility checks
and returned the correct 90-day windows.

**Conclusion**: The step-3 summary removed the selected transaction and its details; the transaction search at step 5 returned the correct records but could not reconstruct which one the customer had selected.
The absence of a search in the knowledge base at step 5 does not, in itself, imply a failure to retrieve information. The selected transaction came from the customer’s earlier message, and its disappearance is demonstrated in the summary.


### 3.2. Self-contradiction

Source: Main scenario, run 5 in Section 2.3.

**Deadline**: No conflict found. At steps 2 and 7, the deadline for TX-0401 was 18 October 2026.<br>
**Reason code**: At step 2, check_dispute_eligibility used goods_not_received. At step 3, the conversation summary changed it to service_not_rendered. The agent then repeated this wrong code in its step-3 answer.<br>
**Cause**: The wrong code first appeared in the summary. The tools used at step 3 returned account and limit information; they did not supply a dispute reason code.<br>
**Conclusion**: The evidence shows that summarization changed the reason code, and the agent repeated the mistake. This supports D06. It does not prove a separate D07 failure just because D07 was enabled.

### 3.3. Control on `clean`

**Entities retained through**: The answers retain the case details through the final scenario step 7. The inspected step-3 summary explicitly preserves EUR 240, TX-0401, the transaction date, goods_not_received, the 90-day window and the deadline.<br>
**At step 7**: The agent states the deadline 18 October 2026, consistent with the 90-day window established at step 2. It does not repeat the number “90” in that final answer.<br>
**What this demonstrates**: In this clean run, summarization preserved the tracked case details, and later answers remained consistent with them. 

### 3.4. Failures in the HW scenarios

| Case | What the step-3 summary did | Mechanism | Verified loss/change step | First failed check | Evidence at the later step |
|---|---|---|---:|---|---|
| H01 | Removed EUR 240 and the selected ID in all five runs. Run 4 changed the reason to `service_not_rendered`. | D06 | 3 in 5/5 runs | 5 in 5/5 runs | At step 5, four runs search transactions again and ask which charge. Run 4 makes no tool call and repeats the wrong reason without the amount. |
| H02 | Removed EUR 240 in all five runs. Kept TX-0401 in runs 2 and 5. Changed the reason in runs 1, 3 and 5. | D06 | 3 in 5/5 runs | 5 in 4/5 runs | Run 2 checks eligibility, but that result does not supply the amount. Run 5 searches the KB, checks eligibility and retrieves transactions; it passes despite the earlier summary loss. |
| H03 | Removed the amount in four runs; changed EUR 240 to EUR 180 in run 2. Changed the reason in runs 2, 3 and 5. | D06 | 3 in 5/5 runs | 5 in 5/5 runs | No tools are called at step 5. The bot cannot state the amount or repeats EUR 180 from the summary. |
| H04 | Removed the amount in runs 1-3; changed it to EUR 180 in runs 4-5. Runs 4-5 also contain the wrong reason. | D06 | 3 in 5/5 runs | 8 in 5/5 runs | Runs 1-4 retrieve the correct EUR 240 transaction records. Run 5 makes no tool call at step 8 and repeats EUR 180 and the wrong reason. |
| H05 | No summary in any of the five runs. | Test false alarm; no memory loss shown by that match | None established | 5 in run 3 only | The answer correctly names TX-0401, EUR 240 and the reason. “Which applies to this payment” is about review times, not choosing a transaction. |

**Clear example - H03, run 2:**
- `CONV-H03-run2-turn2.json`: the answer confirms EUR 240 and `goods_not_received`.
- `CONV-H03-run2-turn3.json`: `summary.text` says “EUR 180.00” and “service_not_rendered”; it replaces four messages.
- `CONV-H03-run2-turn5.json`: no tool call; the answer repeats EUR 180. The reason check still passes because the answer also mentions the correct code while discussing a correction.

The original facts were lost or changed at step 3. The first failed automated check is later: step 5 or 8. These are different measures. Some errors may be visible before the first checked step; this table does not claim a full first-visible-error scan of every answer. A separate D07 cause is not proven: the wrong facts already appear in the summaries.

### 3.5. Final automated comparison

| Case | clean: runs passed | lesson-05: runs passed |
|---|---:|---:|
| CONV-H01 | 5/5 | 0/5 |
| CONV-H02 | 5/5 | 1/5 |
| CONV-H03 | 5/5 | 0/5 |
| CONV-H04 | 5/5 | 0/5 |
| CONV-H05 | 5/5 | 4/5 |

H05's remaining run is a false alarm on manual review. The clean prediction held for the updated set. The lesson-05 results show the expected problems in H01–H04. The control result differs from the prediction only because of the broad wording check.

---

## 4. Invariants and Oracles

### 4.1. Two checks for the main scenario

Results below are for CONV-H01 in the final HW set, at probe steps 5 and 7. A run counts as failed for an invariant if either probe fails.

| Check | Entity | From step | Level | clean | lesson-05: failed runs |
|---|---|---:|---|---|---:|
| 1 | Dispute amount: EUR 240 | 1 | 2 - number with tolerance 0.01 | Passed 5/5 | 1/5 |
| 2 | Reason: goods_not_received | 2 | 3 - text pattern | Passed 5/5 | 1/5 |

Both failures occur in run 4. H01 fails in all five runs overall because the extra transaction-selection check catches other failures. Invariant results and whole-case results are different counts.

### 4.2. Invariants and oracles for all cases

All amount checks use EUR 240 with tolerance 0.01. All reason checks expect `goods_not_received`. Both checks pass in all five clean runs for every case.

| Case | Invariants and probe steps | Oracle: first source and correctness check | lesson-05: invariant failures |
|---|---|---|---|
| CONV-H01 | Amount from 1, level 2; reason from 2, level 3; probes 5 and 7. | Step 1: human checks EUR 240 against TX-0401; checks the goods complaint against the reason-code guide. | Amount 1/5; reason 1/5. |
| CONV-H02 | Amount from 1, level 2; reason from 2, level 3; probe 5. | Step 1: same transaction and reason sources as H01. | Amount 1/5; reason 0/5. |
| CONV-H03 | Amount from 1, level 2; reason from 2, level 3; probes 5 and 7. | Step 1: customer supplies EUR 240 for a draft; human checks the complaint against the reason-code guide. No transaction verification is claimed. | Amount 5/5; reason 0/5. |
| CONV-H04 | Amount from 1, level 2; reason from 2, level 3; probes 8 and 10. | Step 1: same transaction and reason sources as H01. | Amount 1/5; reason 2/5. |
| CONV-H05 | Amount from 1, level 2; reason from 2, level 3; probes 5 and 7. | Step 1: same transaction and reason sources as H01. | Amount 0/5; reason 0/5. |

**Consistent error:** A bot could repeat the wrong amount or code throughout a conversation. Independent transaction records and the reason-code guide are needed to check the starting value. Eligibility alone does not prove that a reason fits the complaint.

**Limits of the implemented checks:** A numeric check passes if EUR 240 appears anywhere, even beside EUR 180 or in a list of other charges. A reason check passes if the correct phrase appears, even beside a wrong code. The transaction-selection regex can also produce false alarms, as H05 showed. These checks flag answers for reviewy.

---

## 5. Test Plan and Price

### 5.1. Axes and Full Combinatorics

| Axis | Values | What it tests |
|---|---|---|
| Dialogue length | 5, 7, 10 turns | Whether more turns increase forgetting or delay its visible effects. |
| Tracked entities | 2 or 4 | Whether more case details increase loss or confusion. |
| Unrelated topic blocks | 0, 2, 3 | Whether moving away from the dispute affects recall. |

Two entities: amount and reason code.<br>
Four entities: amount, reason code, transaction date and filing window.

**Full combinations**: 3 lengths × 2 entity counts × 3 topic settings = 18 scenarios.

Each length occurs in six combinations:<br>
**One repetition of all combinations**:
6 × (5 + 7 + 10) = 132 turns.
Five repetitions per scenario:
132 × 5 = 660 turns per profile, or 1,320 turns for both profiles.

### 5.2. Selected Scenarios

| ID | Length | Entities | Topic blocks | Role | Question it answers |
|---|---:|---:|---:|---|---|
| CONV-H01 | 7 | 4 | 2 | Main | Does C-07, with an explicit final recall question, retain case details after summarization? |
| CONV-H02 | 5 | 4 | 2 | Isolate length | Does a shorter version retain the details when the tracked fields and topic blocks stay the same? |
| CONV-H03 | 7 | 2 | 2 | Isolate entity count | Can the agent retain the amount and reason in a simple draft complaint after two unrelated topics? |
| CONV-H04 | 10 | 4 | 3 | Stress case | What happens with a longer conversation and an extra SWIFT topic block? |
| CONV-H05 | 7 | 4 | 2 | Control | Does the main scenario retain details when summarization is disabled for the conversation? |


Plan limits: H02 uses the first five turns of H01, so it tests a shorter endpoint, not a different history before turn 5. H03 also changes the task to a draft and asks directly for recall; it does not isolate entity count perfectly. H04 changes both length and topics. The four tracked fields do not include the selected transaction ID, which is checked separately. 

### 5.3. Cost and schedule

**Measured final run time:** clean 656.4 seconds (about 10.9 minutes); lesson-05 784.7 seconds (about 13.1 minutes). 

**Selected plan**: 5 scenarios with lengths 7 + 5 + 7 + 10 + 7 = 36 turns per repetition.<br>
36 × 5 repetitions = 180 turns per profile.<br>
Running both clean and lesson-05 costs 360 turns.<br>
**Comparison with golden.jsonl**: The latest HW3 daily runs executed 44 single-turn cases. The selected dialogue plan uses about 4.1 times as many customer turns per profile: 180 ÷ 44 ≈ 4.1. Actual model usage can be higher because dialogue history and tool calls add work.<br>
**Schedule**: Before release (gate: release), and after changes to summarization or memory handling.<br>
**Reason**: The dialogue set is much larger than the daily single-turn set. Running it before release keeps the cost manageable while checking memory-related changes.<br>
**Trade-off**: We select five scenarios instead of all 18 combinations. We cover the main failure, shorter length, fewer tracked details, combined stress and a control without summarization. Other combinations remain untested.

---

## 6. Residual Risk and Decisions

### 6.1. Uncovered combinations

| What is not covered | What could happen | Why we leave it out for now | Decision |
|---|---|---|---|
| 5 turns, 2 tracked entities, 0 unrelated topic blocks | The agent could lose the amount or reason even in a short, focused conversation. | We give priority to conversations with topic changes, where the lab showed failures. | **Accept** this coverage gap for the first test set. |
| 10 turns, 2 tracked entities, 2 unrelated topic blocks | The agent could forget details because of length, even with fewer entities. | The selected ten-turn case tests combined stress. A separate case would help explain the effect of length but adds cost. | **Defer** until long-dialogue failures need closer study. |
| 7 turns, 4 tracked entities, 0 unrelated topic blocks, with summarization enabled | The agent could lose details even without leaving the dispute topic. | Our control disables summarization. It does not separately test whether staying on topic prevents loss. | **Defer**; add this case if focused dispute conversations show failures. |


### 6.2. Blind spots

| Blind spot | How we address it, or why we accept it |
|---|---|
| The profile shows when a value disappeared, but does not fully explain why that particular summary dropped it. | Save the history, summary and prompt settings. Compare runs with and without summarization. Do not claim a deeper cause than the evidence shows. |
| The bot repeats the same wrong value throughout the chat. | Check the value against a trusted source, such as the transaction record or reason-code guide. Repeating a value does not make it correct. |
| The expected value appears in an answer, but the agent has forgotten which transaction it belongs to. | Check whether it asks “Which transaction?” again. Review these matches manually because the check can flag harmless wording, as it did in H05. |
| Language changes, returning the next day, and two disputes in one conversation are outside this plan. | Record these gaps. Add separate scenarios before relying on the results for those uses. |


### 6.3. Recommended D06 fix

| Option | Choose? | Estimated cost | Trade-off |
|---|---|---|---|
| Store case details in structured fields outside the summary | **Yes** | About **2-4 developer days** for a small implementation and **1–2 QA days** for checks. Store the selected transaction, amount, date, reason and window; include them in later model inputs. | More code and state to maintain. Incorrect or outdated fields could preserve the wrong information, so updates must be checked. |
| Increase `SUMMARIZE_AFTER` | No, as the main fix | About **15-30 minutes** to change the setting, plus test runs. Longer histories also increase ongoing token use. | Delays summarization but does not ensure that details survive when it eventually happens. |
| Add “preserve numbers” to the summarizer prompt | No, as the main fix | About **30-60 minutes** to edit and review, plus test runs. | The model may still omit or change details. Preserving numbers alone does not protect the reason code or selected transaction. |

**Why this option under Ship it**: The lab showed that summaries can drop the selected transaction and change the reason code. Keeping these details outside the summary directly addresses that failure. Start with the dispute flow and verify that the stored values match trusted sources and explicit customer corrections.

---

## 7. Final decision

- **What we trade off:** Cover five chosen scenarios instead of all 18 combinations to limit cost. Use structured case fields to reduce dependence on summaries, accepting extra development and maintenance work.
- **What we do not do:** We do not treat every failed test as a memory problem, or every passed test as proof of good memory. We review failures before blocking a release. We have not tested language changes, returning the next day, or several disputes in one chat.
- **When this decision changes:** Add tests when new uses or problems appear. Run the tests again after fixing memory. Fix the false alarm before letting tests block releases automatically.

