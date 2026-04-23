# Thai Typing Time-Series Mining Project Context

## Purpose of this file

This file is a **comprehensive LLM-friendly project context** for downstream data transformation, feature engineering, validation, and analysis.

Use this file as the authoritative context for:
- loading and validating the browser export
- reconstructing text state from edit diffs
- deriving event-level and position-level features
- building time-series objects for DTW, Matrix Profile, clustering, and visualization
- writing analysis code and reports that stay aligned with the actual project design

This file reflects the **final deployed website structure, final prompt order, and the latest validated pilot exports**.

---

## 1. Project identity

### Project title
Thai Typing Time-Series Mining

### One-sentence summary
This project models Thai prompt-copying as a **correction-driven edit-event time series** under **append-only typing and pop-only deletion** constraints, then uses target alignment, Kedmanee-aware metadata, DTW, Matrix Profile, clustering, and visual analytics to discover where people hesitate and how they recover.

### Core methodological stance
This project is **not** raw physical keystroke biometrics.

The website does **not** log keydown/keyup timing or true finger movement. Instead, it logs **timestamped text-diff events** from a constrained browser typing interface. The project intentionally treats those events as a structured correction process rather than pretending they are full keystroke biometrics.

### Core structural idea
Because the interface is constrained to **append-only typing** and **pop-only backward deletion**, the current text can be interpreted as:

```text
current_text = correct_prefix + wrong_suffix
```

This makes the diff log highly suitable for target-aligned time-series mining.

---

## 2. What the project is trying to answer

### RQ1. Character- and zone-level fluency
Do participants show different effective fluency across Thai character classes and Kedmanee zones such as left versus right, row position, Thai numerals, tone marks, punctuation, and mixed-symbol spans?

### RQ2. Linguistic friction versus keyboard-layout friction
When hesitation and correction happen, are they more strongly associated with linguistic burden such as formal wording, orthographic difficulty, named entities, and expectation-violating phrases, or with keyboard-layout burden such as symbol use, Thai numerals, row jumps, and non-home positions?

### RQ3. Shared difficulty regions across users
For the same prompt, do multiple users struggle at the same text region? Can those shared regions be detected through prompt-position stability statistics, friction traces, anomaly scores, or discord windows?

### RQ4. Recovery style as a behavioral signature
Do users exhibit different recovery styles after errors, such as pause-then-delete, long wrong-suffix accumulation, smooth quick recovery, or repeated burst-like correction behavior?

---

## 3. Final deployed website design assumptions

### Interaction constraints
The website is designed as a simplified typing-practice interface with the following rules:

- one prompt at a time
- append-only typing
- pop-only deletion
- no arbitrary cursor repositioning
- no in-line replacement
- no paste
- fixed prompt order for the deployed single-order version
- one practice prompt, then 14 main prompts

### Important interpretation rule
If a user makes an error, they cannot repair it in the middle. They must delete backward until the wrong suffix is removed and then continue typing forward again.

This gives the project a clean observable recovery process.

### Current implementation status
The latest validated pilot exports confirm that the current website behavior is consistent with the intended constraint design:

- no mid-string insertions detected
- no mid-string deletions detected
- no replacement events detected
- edit logs are fully reconstructable
- final reconstructed text matches both the logged final text and the target prompt

Therefore, the current website is considered **launch-ready for volunteer data collection**.

---

## 4. Final prompt inventory and final typing order

The deployed study contains **1 practice prompt** and **14 main prompts**.

### Practice prompt
- `prompt_id`: `PRACTICE`
- `trial_type`: `practice`
- text: `วันนี้อากาศดีและฉันพร้อมเริ่มการทดลอง`
- role: interface familiarization only
- not part of the main analysis

### Final main prompt order
The website should show main prompts in this exact order using `prompt_typing_order`.

| prompt_typing_order | prompt_id | length_class | condition | purpose | expected_typing_behavior | prompt_text |
|---:|---|---|---|---|---|---|
| 1 | P01 | short | familiar_baseline | baseline familiar Thai | smooth typing, low hesitation | วันนี้เราจะทำการบ้านให้เสร็จ |
| 2 | P02 | short | familiar_polite | baseline polite everyday Thai | mostly smooth typing, low hesitation | กรุณาตอบกลับภายในวันนี้ |
| 3 | P04 | short | orthographic_named_entity | test long proper noun typing difficulty | pause spikes and possible corrections | นิสิตจากจุฬาลงกรณ์มหาวิทยาลัย |
| 4 | P03 | short | formal_register | test formal bureaucratic wording | slightly slower rhythm, higher first-key latency | ขอความกรุณาจัดทำเอกสารดังกล่าว |
| 5 | P05 | short | twisted_proverb | test expectation violation in proverb | local hesitation near the altered phrase | เข้าเมืองตาหลิ่ว ต้องหลิ่วมาตาม |
| 6 | P09 | medium | arabic_numerals | baseline numeral form with Arabic digits | relatively smooth typing for many users | ชั้นมัธยมศึกษาปีที่ 6 จะส่งรายงานจำนวน 10 หน้า |
| 7 | P11 | medium | rhythmic_repetition | test repetition and tongue-twister-like rhythmic structure | possible bursty timing and local re-typing | วันนี้ผมทักผมรักผมรอ วันไหนผมท้อผมพอผมลา |
| 8 | P10 | medium | thai_numerals | test Thai numeral and formal writing difficulty | slower rhythm around Thai numerals | ชั้นมัธยมศึกษาปีที่ ๖ จะส่งรายงานจำนวน ๑๐ หน้า |
| 9 | P06 | medium | mixed_symbols_identification | test abbreviation, digits, slash, and hyphenated name field | local hesitation around `ม.5/6` and `ชื่อ-สกุล` span | นักเรียน ม.5/6 ช่วยตรวจสอบ ชื่อ-สกุล ของตัวเองด้วยนะ |
| 10 | P08 | medium | formal_lexical_difficulty | test formal vocabulary and low-frequency wording | hesitation around difficult lexical items | ขอความร่วมมือจัดเตรียมเอกสารเพื่อเป็นวิทยาทานแก่นักเรียน |
| 11 | P07 | medium | formal_educational | compare plain versus formal educational register | longer pauses around formal words | นักเรียนพึงศึกษาเล่าเรียนอย่างสม่ำเสมอเพื่อเตรียมความพร้อมในการสอบ |
| 12 | P12 | long | long_familiar_expository | long readable baseline for localization analysis | mostly smooth with minor pauses at long words | วันนี้ครูขอให้นักเรียนเขียนสรุปเรื่องสุภาษิตไทยอย่างกระชับและชัดเจน โดยให้ยกตัวอย่างหนึ่งสำนวนและอธิบายความหมายด้วยภาษาของตนเอง |
| 13 | P14 | long | long_mixed_difficulty | test mixed difficulty including proper noun, formal words, and Thai numerals | multiple local hesitation zones | สวัสดีคุณชาคริษฐ์ วันนี้เราขอให้คุณจัดทำหนังสือเรื่องการสอนภาษาไทยสำหรับนักเรียนชั้นมัธยมศึกษาปีที่ ๖ เพื่อเป็นวิทยาทานแก่โรงเรียนในเครือพันธมิตร |
| 14 | P13 | long | long_formal_bureaucratic | test sustained formal-register friction | overall slower typing and diffuse hesitation | ทางคณะผู้จัดทำขอความกรุณาให้นักเรียนจัดทำรายงานเรื่องสุภาษิตและคำพังเพย โดยเขียนอธิบายเนื้อหาให้ครบถ้วนเพื่อเป็นประโยชน์ต่อการเรียนรู้ในภายหลัง |

### Notes about prompt text handling
- Prompt text must be treated as exact Unicode text from the CSV.
- Do not trim spaces inside prompt strings.
- For `P06`, preserve the spaces around `ชื่อ-สกุล` exactly as deployed.
- Position-based analysis should operate on the exact stored prompt text.

---

## 5. Current data sources and expected file structure

Each browser export contains three core CSV files and a small readme.

### 5.1 `participant_session*.csv`
One row per session.

Expected fields:
- `session_id`
- `participant_code`
- `started_at_iso`
- `completed_at_iso`
- `total_duration_ms`
- `user_agent`
- `platform`
- `browser_language`
- `screen_width`
- `screen_height`
- `typing_skill_rating`
- `preferred_input_note`
- `consent_acknowledged`
- `main_prompt_count_expected`
- `main_prompt_count_completed`
- `app_version`

Role:
- session metadata
- device/context validation
- high-level quality control

### 5.2 `trial_log*.csv`
One row per completed trial, including practice.

Expected fields:
- `session_id`
- `participant_code`
- `trial_index`
- `trial_type`
- `prompt_id`
- `length_class`
- `condition`
- `purpose`
- `expected_typing_behavior`
- `prompt_text`
- `typed_text_final`
- `prompt_char_count`
- `typed_char_count_final`
- `exact_match`
- `levenshtein_distance`
- `normalized_edit_distance`
- `prompt_started_at_ms_from_session_start`
- `first_input_at_ms_from_session_start`
- `submitted_at_ms_from_session_start`
- `first_key_latency_ms`
- `trial_duration_ms`
- `edit_event_count`
- `insertion_event_count`
- `deletion_event_count`
- `replacement_event_count`
- `backspace_like_event_count`
- `visibility_hidden_count`
- `visibility_hidden_total_ms`

Role:
- prompt-level descriptive analysis
- speed and correction summaries
- validation against reconstructed event logs

### 5.3 `edit_event_log*.csv`
One row per textarea input event.

Expected fields:
- `session_id`
- `participant_code`
- `trial_index`
- `trial_type`
- `prompt_id`
- `event_index`
- `event_time_ms_from_prompt_start`
- `action_type`
- `diff_start_index`
- `deleted_text`
- `inserted_text`
- `deleted_length`
- `inserted_length`
- `previous_text_length`
- `resulting_text_length`
- `selection_start_before`
- `selection_end_before`
- `selection_start_after`
- `selection_end_after`

Role:
- raw sequential behavior
- source for reconstructing text state
- source for derived time-series features

---

## 6. Latest validated pilot exports

Two recent pilot exports have already been validated and should be treated as proof that the collection pipeline is working.

### Pilot A
- session id: `20260419141512-jt34p4`
- participant code: `ก้องฟ้า`
- main prompts completed: `14/14`
- total trials: `15` including practice
- total edit events: `1073`
- total main-prompt backspace-like events: `87`
- total main-prompt duration: `214023 ms`
- all 14 main prompts ended with exact match and zero normalized edit distance

### Pilot B
- session id: `20260419144848-x70p47`
- participant code: `Test`
- main prompts completed: `14/14`
- total trials: `15` including practice
- total edit events: `1291`
- total main-prompt backspace-like events: `191`
- total main-prompt duration: `237487 ms`
- all 14 main prompts ended with exact match and zero normalized edit distance

### Structural validation results from the latest pilots
Both pilot exports passed the following checks:

- all main prompts completed
- event indices contiguous within each trial
- event timestamps monotonic within each trial
- final text reconstructable from diffs
- reconstructed final text equals `typed_text_final`
- reconstructed final text equals `prompt_text` for all main trials
- no mid-string insertions
- no mid-string deletions
- no replacement events

Conclusion:
The browser export is **sufficient for the planned transformation and analysis pipeline**.

---

## 7. What the downstream analysis should and should not claim

### Safe claims
The project can safely claim:
- browser diff logs can be transformed into meaningful sequential objects
- append/pop constrained typing creates an analyzable correction process
- prompt-position stability and friction traces can localize difficulty
- different prompts and users can exhibit different correction and hesitation profiles
- Kedmanee metadata can be used as a proxy for layout burden

### Claims that should not be made
Do **not** claim:
- true physical keystroke biometrics
- true finger tracking
- exact hand or finger assignment
- exact IME composition timing
- causal proof that a region is difficult solely because of geometry or solely because of linguistics

The project should consistently describe Kedmanee geometry as a **proxy**, not direct measurement.

---

## 8. Required transformation pipeline

This is the intended pipeline for any LLM or code agent that processes the dataset.

### Step 1. Load all three files for one session
- read `participant_session`
- read `trial_log`
- read `edit_event_log`
- ensure `session_id` is consistent across files

### Step 2. Exclude the practice trial from the main analysis
- keep the practice trial only for sanity checks or interface debugging
- all main analysis should use `trial_type == "main"`

### Step 3. Validate trial-level integrity
For each main trial, check:
- `exact_match == True`
- `normalized_edit_distance == 0`
- `edit_event_count == insertion_event_count + deletion_event_count + replacement_event_count`
- `backspace_like_event_count == deletion_event_count` in the current implementation
- event count in `trial_log` matches the number of rows in `edit_event_log`

### Step 4. Validate event-level integrity
Within each trial:
- `event_index` should start at 1 and increase by 1
- `event_time_ms_from_prompt_start` should be nondecreasing
- `previous_text_length` and `resulting_text_length` should be consistent with the diff
- `selection_start_before == selection_end_before` and `selection_start_after == selection_end_after` for simple caret typing
- `diff_start_index` should be at the end for insertions under append-only behavior
- deletions should remove only the terminal suffix under pop-only behavior

### Step 5. Reconstruct full text state after each event
For each event row, reconstruct:
- `text_before`
- `text_after`

Use the diff fields:
- remove `deleted_text` starting at `diff_start_index`
- insert `inserted_text` at `diff_start_index`

Then verify:
- reconstructed `text_after` length matches `resulting_text_length`
- final reconstructed text matches `typed_text_final`

### Step 6. Align current text to the target prompt
For each event, compare `text_after` to `prompt_text` and derive:

- `prefix_match_len`
- `prefix_gain`
- `error_suffix_len`
- `remaining_len`
- `is_progress`
- `is_regress`

Definitions:
- `prefix_match_len`: length of the longest common prefix between `text_after` and the target prompt
- `prefix_gain`: current `prefix_match_len` minus previous event's `prefix_match_len`
- `error_suffix_len`: `len(text_after) - prefix_match_len`
- `remaining_len`: `len(prompt_text) - prefix_match_len`
- `is_progress`: 1 if `prefix_gain > 0`, else 0
- `is_regress`: 1 if `action_type == "delete"` or `prefix_gain < 0`, else 0

### Step 7. Derive timing features
For each event, derive:
- `dt_ms`
- `log_dt`
- `pause_flag`
- optional standardized timing measures within participant or within trial

Definitions:
- `dt_ms`: difference in `event_time_ms_from_prompt_start` from the previous event; use first event time directly for event 1
- `log_dt = log(1 + dt_ms)`
- `pause_flag`: indicator for large local pauses using a threshold or quantile rule

### Step 8. Attach prompt-position and character metadata
For each prompt character position, attach:
- character itself
- row
- column
- zone
- layer
- character class
- whether it is Thai numeral
- whether it is punctuation
- whether it is tone mark or upper-layer mark

This requires a manually prepared Thai Kedmanee metadata table.

### Step 9. Build event-level and prompt-level derived tables
Produce at least two clean derived datasets:

#### A. Event-level derived table
One row per edit event with original columns plus derived features such as:
- `text_before`
- `text_after`
- `dt_ms`
- `log_dt`
- `prefix_match_len`
- `prefix_gain`
- `error_suffix_len`
- `remaining_len`
- `is_progress`
- `is_regress`
- `geo_burden`
- `row_jump`
- `home_distance`
- `char_class`
- `zone`
- `layer`

#### B. Prompt-position derived table
One row per `(session_id, prompt_id, char_position)` with features such as:
- `target_char`
- `time_to_first_correct`
- `time_to_stability`
- `revision_count`
- `wrong_first_flag`
- `first_correct_event_index`
- `stability_event_index`
- character metadata fields

### Step 10. Save transformation outputs
Recommended output files:
- `derived_event_features.csv`
- `derived_position_features.csv`
- `trial_summary_enriched.csv`
- optional `participant_profile.csv`

---

## 9. Recommended core derived variables

### 9.1 Event-level variables
Minimum recommended variables:
- `dt_ms`
- `log_dt`
- `action_type`
- `is_delete`
- `prefix_match_len`
- `prefix_gain`
- `error_suffix_len`
- `remaining_len`
- `current_text_len`
- `progress_ratio`
- `geo_burden`
- `zone`
- `row`
- `layer`
- `char_class`
- `is_thai_numeral`
- `is_punctuation`
- `is_tone_mark`

Useful additional variables:
- `delete_run_length`
- `insert_run_length`
- `local_backtrack_depth`
- `recovery_span_events`
- `recovery_span_ms`
- `overshoot_flag`
- `burst_id`

### 9.2 Prompt-position variables
Minimum recommended variables:
- `char_position`
- `target_char`
- `time_to_first_correct`
- `time_to_stability`
- `revision_count`
- `wrong_first_flag`
- `stabilized_flag`
- `final_correct_flag`
- character metadata columns

### 9.3 Trial-level variables
Minimum recommended variables:
- `trial_duration_ms`
- `first_key_latency_ms`
- `edit_event_count`
- `backspace_like_event_count`
- `characters_per_second`
- `backspaces_per_char`
- `mean_dt_ms`
- `median_dt_ms`
- `max_dt_ms`
- `mean_error_suffix_len`
- `max_error_suffix_len`
- `mean_prefix_gain_nonzero`
- `n_pause_events`

### 9.4 Participant-level variables
Useful participant profile variables:
- average speed
- average correction burden
- average friction by prompt type
- average time-to-stability by zone
- average revision rate by character class
- Thai numeral difficulty contrast
- long-prompt burden summary

---

## 10. Core time-series objects

These are the intended sequence objects for downstream mining.

### TS Object A. Event-aligned multivariate edit sequence
For each event `i`, define a feature vector such as:

```text
x_i = [log(1 + dt_i), isDelete_i, prefixGain_i, errorSuffixLen_i, geoBurden_i]
```

Main use cases:
- DTW similarity
- DTW clustering
- recovery-style comparison
- same-prompt comparison across users

### TS Object B. Friction waveform
Collapse event-level behavior into one scalar sequence:

```text
F_i = w1 * log(1 + dt_i) + w2 * isDelete_i + w3 * errorSuffixLen_i + w4 * geoBurden_i
```

Notes:
- standardize component scales before combining
- freeze the weighting scheme before large-scale analysis
- if uncertain, use equal weights after z-normalization as a default baseline

Main use cases:
- Euclidean baseline
- DTW on scalar signal
- Matrix Profile
- motif and discord discovery

### TS Object C. Prompt-position stability sequence
For each target character position `j`, derive:
- `time_to_first_correct_j`
- `time_to_stability_j`
- `revisions_j`
- `wrong_first_j`

Main use cases:
- shared difficulty localization
- prompt-position heatmaps
- region-level comparisons against manual spans

### TS Object D. Participant keyboard-zone fluency profile
Aggregate by metadata categories such as:
- left / center / right zone
- top / home / bottom row
- consonant / vowel / tone mark / numeral / punctuation / mixed-symbol span

Main use cases:
- participant clustering
- descriptive profiling
- RQ1 interpretation

---

## 11. Which analysis methods should be used on which objects

### Euclidean distance
Use mainly on:
- friction waveform
- prompt-position stability sequence
- participant profile vectors

Role:
- simple baseline
- fixed-length comparison

### DTW
Use mainly on:
- multivariate event sequences
- friction waveforms
- prompt-position sequences when shifted but structurally similar

Role:
- core similarity method for misaligned correction behavior

### Matrix Profile
Use mainly on:
- long-prompt friction waveform
- one-dimensional local burden signals
- long-prompt position-level difficulty curves

Role:
- motif discovery
- discord discovery
- subsequence localization

### Clustering
Use mainly on:
- DTW distance matrix between trials or participants
- participant profile vectors
- prompt summary vectors

Role:
- discover prompt families and user styles

### Optional SAX or symbolic summaries
Possible on:
- discretized friction waveform
- discretized pause/correction states

Role:
- compact motif summaries if time allows

---

## 12. Suggested annotation strategy for evaluation only

Manual or rule-based annotation spans should be created **after final prompt text is frozen** and should be used **only for evaluation**, not as model input.

Suggested annotation categories:
- named entity span
- twisted or expectation-violating span
- Thai numeral span
- Arabic numeral span
- mixed-symbol span
- formal lexical span
- repetitive span
- likely difficult span

### Important anti-leakage rule
Do not feed these labels into anomaly detection or sequence mining.
Use them only after discovery for interpretation and evaluation.

### Likely candidate prompts for manual spans
- `P04`: named entity
- `P05`: twisted proverb region
- `P06`: `ม.5/6` and `ชื่อ-สกุล`
- `P09`: Arabic numeral region
- `P10`: Thai numeral region
- `P11`: repetitive phrase structure
- `P14`: proper noun + Thai numeral + formal phrase mixture

---

## 13. Recommended evaluation and validation rules

### Structural quality checks per session
Exclude or flag a session if:
- main prompt count is not complete
- event reconstruction fails
- final reconstructed text does not match final typed text
- large numbers of non-terminal edits occur
- visibility-hidden time is excessive, if that matters for your study protocol

### Behavioral checks
Useful quality metrics:
- total completion time
- mean speed
- unusually high or low event counts
- unusually high correction rate
- whether the practice trial suggests wrong input mode at the beginning

### Analytical evaluation ideas
- compare Arabic numerals versus Thai numerals
- compare long formal prompt versus long familiar prompt
- compare mixed-symbol prompt versus ordinary medium prompts
- use top-k discord overlap with manually annotated difficult spans
- compare participants on prompt-position heatmaps

---

## 14. Recommended visualizations

### Prompt-level descriptive charts
- duration by prompt
- backspaces by prompt
- characters per second by prompt
- edit count by prompt

### Event-level process visualizations
For selected trials, especially long prompts:
- `dt_ms` trace
- `prefix_match_len` trace
- `error_suffix_len` trace
- friction waveform

### Prompt-position visualizations
- prompt-position heatmap of `time_to_stability`
- prompt-position heatmap of revision count
- prompt-position anomaly scores for long prompts

### Metadata visualizations
- Kedmanee keyboard heatmap
- zone-level fluency bar charts
- character-class burden summary

### Similarity and clustering visualizations
- DTW distance heatmap
- dendrogram or clustering scatter plot
- motif and discord marking on long friction sequences

---

## 15. What the current pilot evidence already suggests

Based on the two validated pilots:
- the logging schema is sufficient
- the append/pop interpretation is working
- long prompts are long enough for subsequence mining
- Thai numerals and mixed-symbol prompts are likely useful difficulty probes
- there is meaningful variation in correction burden across prompts and sessions

Notable pilot tendencies that should be checked again on the volunteer dataset:
- `P10` often induces more hesitation than `P09`
- `P06` is intentionally symbol-heavy and may create localized friction
- long prompts such as `P12`, `P13`, and `P14` provide enough events for friction and discord analysis

These are hypotheses, not final claims.

---

## 16. Practical coding guidance for downstream LLM or agent

### Preferred style of analysis
Use simple, robust, interpretable processing first.

Preferred order:
1. validation
2. reconstruction
3. event-level feature engineering
4. prompt-position feature engineering
5. descriptive statistics
6. one or two clean time-series mining analyses
7. visualization
8. interpretation

### Do not overcomplicate early
Avoid jumping immediately into:
- deep learning
- overparameterized models
- too many composite metrics at once
- unsupported causal claims

The project is strongest when it shows that the browser log can be transformed into meaningful time-series objects and interpreted well.

### Recommended analysis priority
Highest priority:
- reconstruction and validation
- prompt summary statistics
- position-level stability heatmaps
- friction waveform for long prompts
- one clear DTW example
- one clear Matrix Profile or discord example

Medium priority:
- participant clustering
- zone-level fluency profile
- contrast analysis across prompt categories

Optional:
- SAX
- lower-bound demonstrations
- more advanced clustering variants

---

## 17. Expected outputs for a full analysis run

A good downstream analysis pipeline should ideally produce:

### Clean tables
- validated session summary
- enriched trial summary
- event-level derived feature table
- prompt-position derived feature table
- participant profile table

### Figures
- duration by prompt
- speed by prompt
- backspace burden by prompt
- event trace on a long prompt
- position-level heatmap
- keyboard-zone heatmap
- DTW example
- Matrix Profile or discord example

### Interpretation
The write-up should explain:
- which prompts appear to create more friction
- where difficulty localizes within prompts
- whether difficulty seems more linguistic, geometric, or mixed
- whether users show distinct recovery styles
- what the method can and cannot claim

---

## 18. Final project status

### Status summary
- prompt set finalized
- prompt order finalized
- website constraint behavior validated
- pilot exports validated
- current logging schema sufficient
- ready for volunteer launch
- ready for transformation and downstream analysis

### Final bottom line
This project is ready to move from pilot testing to real data collection and analysis.

The central intellectual move is not complexity for its own sake. It is the careful transformation of constrained browser edit logs into interpretable time-series objects.

