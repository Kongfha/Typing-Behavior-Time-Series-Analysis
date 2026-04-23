# Data Preparation and Time Series Data Transformation

## Purpose of this file

This file defines the detailed data-preparation and transformation plan for the Thai Typing Time-Series Mining project.

It is the implementation-oriented companion to `project_context_comprehensive.md`.

Its job is to tell an LLM, analyst, or engineer exactly how the raw browser logs should be turned into:
- validated session-level data
- enriched event-level sequence data
- prompt-position feature data
- trial-level summary features
- participant-level keyboard-demand profiles
- time-series objects for DTW, Matrix Profile, clustering, and visualization

This file reflects the **final current design**, including the decision to use a **character map table + keyboard distance matrix** rather than explicit `(x, y)` geometry.

---

## 1. High-level transformation philosophy

The transformation pipeline should be organized in layers, not as one direct jump from raw event log to final model input.

The recommended architecture is:

```text
Raw session/trial/event logs
-> validation and QC
-> reconstructed text states
-> target-aligned enriched event table
-> keyboard-demand augmentation via character map + distance matrix
-> prompt-position feature table
-> trial summary table
-> participant keyboard-demand profile table
-> final time-series objects
```

The most important idea is to keep **intermediate tables**. Do not collapse everything too early.

---

## 2. Input files

The transformation stage should expect the following inputs.

### 2.1 Raw browser export tables
- `participant_session*.csv`
- `trial_log*.csv`
- `edit_event_log*.csv`

### 2.2 Pooled raw event file
Optional bulk input:
- `combined_edit_event_logs_data.csv`

### 2.3 Prompt metadata table
The final prompt CSV with prompt order and prompt metadata.

### 2.4 Character map table
- `character_map_table_comprehensive.csv`

This table maps each printable character to:
- layout requirement
- shift requirement
- physical key identity
- row / column proxies
- zone / hand proxies
- same-key cross-layout information
- default preference rules for prompt-demand joins

### 2.5 Keyboard distance matrix
This file will be supplied separately by the user.

The preferred concept is a distance or adjacency representation that can be used to answer questions such as:
- how far is key A from key B?
- are two characters physically close on the keyboard?
- is a mistyped character adjacent to the intended character?

---

## 3. Final geometry decision

### 3.1 What is not used anymore
The final project does **not** use explicit `(x, y)` coordinates.

### 3.2 What is used instead
Keyboard geometry and local typo proximity should be computed through:
- `physical_key_id`
- `layout_required`
- `shift_required`
- the supplied keyboard distance / adjacency matrix

### 3.3 Why this is the correct final choice
This is simpler, more explicit, and less fragile than invented coordinate geometry. It also matches the actual needs of the project more directly:
- planned key-transition burden
- layout-switch burden
- shift burden
- near-key typo detection

---

## 4. Character map design and usage rules

### 4.1 Why the character map exists
A single character is not enough by itself to infer keyboard burden. The project needs to know:
- whether the character belongs to Thai layout or English layout
- whether shift is required
- which physical key node is involved
- whether the character belongs to a particular script or typing category

### 4.2 Important ambiguity rule
Some visible characters can appear in more than one layout / key-state combination.

Examples include punctuation or symbols that can exist both:
- on the English keyboard
- and as outputs from the Thai Kedmanee layout

Therefore, joins must not assume that `char` is globally unique.

### 4.3 Preferred join rule for prompt-demand features
For **prompt-demand** modeling, use the project policy:
- Thai script characters -> prefer `layout_required = TH`
- English letters -> prefer `layout_required = EN`
- Arabic numerals -> default to `layout_required = EN`
- English punctuation / symbols -> default to `layout_required = EN`
- space -> treat as `COMMON`

This is why the character map contains `preferred_for_prompt_demand`.

### 4.4 Preferred join rule for typed wrong characters
For **realized mistype analysis**, use the actual typed character and then resolve ambiguity with the following order:
1. exact row in the map that matches the project layout rule if the character category is unambiguous
2. otherwise the preferred row in the character map
3. if still ambiguous, record an ambiguity flag and keep the event but mark the geometry subfeatures as uncertain

---

## 5. Recommended keyboard distance matrix formats

The best format is a **long-form edge table** rather than a wide square CSV.

Recommended columns:
- `source_node_id`
- `target_node_id`
- `distance`
- `is_adjacent`
- optional `distance_type`

Where `source_node_id` and `target_node_id` match `distance_node_id` or `physical_key_id` from the character map.

If the user instead provides a **character-to-character** adjacency matrix, the pipeline should convert it into a long form and document the conversion clearly.

### Important recommendation
If possible, the distance matrix should be defined over **physical keys** rather than visible characters. That is cleaner, because multiple visible characters can share the same physical key under different layouts or shift states.

---

## 6. Raw-log validation and quality control

Before any feature engineering, validate each session and each trial.

### 6.1 Session-level checks
Recommended checks:
- `main_prompt_count_completed == main_prompt_count_expected`
- consent acknowledged
- non-missing `session_id`
- plausible `total_duration_ms`

### 6.2 Trial-level checks
Recommended checks:
- expected prompt exists
- `prompt_text` matches the deployed prompt table for the given `prompt_id`
- `trial_index` is unique within the session
- main trials cover all expected prompt IDs in the expected order if using the single-order deployment

### 6.3 Event-level checks
Recommended checks:
- events are sorted by `event_index`
- `event_index` is contiguous within the trial
- `event_time_ms_from_prompt_start` is non-decreasing
- `previous_text_length`, `inserted_length`, `deleted_length`, and `resulting_text_length` are internally consistent
- selection positions behave like end-caret typing

### 6.4 Constraint checks
Recommended checks:
- insertions occur only at the end of the current text
- deletions behave like pop-only backward deletion
- no mid-string replacements

### 6.5 Reconstruction checks
After reconstruction:
- final reconstructed text should match `typed_text_final`
- if `exact_match == True`, reconstructed final text should equal `prompt_text`

### 6.6 QC flag columns
Create explicit flags such as:
- `qc_session_complete_ok`
- `qc_trial_prompt_ok`
- `qc_event_index_ok`
- `qc_time_monotonic_ok`
- `qc_append_pop_ok`
- `qc_reconstruction_ok`
- `qc_exact_match_ok`

Do not silently drop sessions without recording why.

---

## 7. Reconstructing text state from the event log

This is the foundation of the whole project.

### 7.1 Reconstruction principle
Initialize each trial with:
- `text_before = ""`

Then process events in order.

### 7.2 For each event
Use:
- `action_type`
- `diff_start_index`
- `deleted_text`
- `inserted_text`
- `deleted_length`
- `inserted_length`

Under the append-only / pop-only interface, the normal cases are:
- insert: append `inserted_text` to the end
- delete: remove the trailing substring corresponding to `deleted_text`

### 7.3 Save both sides of the event
For each event row, store:
- `text_before`
- `text_after`

Never only store `text_after`. Keeping both is extremely useful for debugging and for defining event-local context.

---

## 8. Target alignment

Once `text_after` is reconstructed, align it to the target prompt.

Let:
- `target = prompt_text`
- `current = text_after`

### 8.1 Prefix match length
Compute the longest prefix length such that:

```text
current[:k] == target[:k]
```

Store:
- `prefix_match_len_after`

Also keep the previous value:
- `prefix_match_len_before`

### 8.2 Prefix gain

```text
prefix_gain = prefix_match_len_after - prefix_match_len_before
```

Interpretation:
- positive -> forward progress
- zero -> stall / no net progress
- negative -> regression

### 8.3 Error suffix length

```text
error_suffix_len = len(text_after) - prefix_match_len_after
```

This is the key observable of how far the user has drifted away from the correct path.

### 8.4 Remaining length

```text
remaining_len = len(target) - prefix_match_len_after
```

### 8.5 Event state flags
Store:
- `is_progress`
- `is_regress`
- `is_stall`
- `in_error_state = 1 if error_suffix_len > 0 else 0`

---

## 9. Timing features

Use `event_time_ms_from_prompt_start` to derive timing-based features.

### 9.1 Inter-event gap
For each event after the first:

```text
dt_ms = time_t - time_{t-1}
```

For the first event, use either:
- `first_key_latency_ms` from the trial log
- or leave `dt_ms` null and handle separately

### 9.2 Log-transformed gap
Because timing is skewed, create:

```text
log_dt = log1p(dt_ms)
```

### 9.3 Pause indicators
Optional but useful:
- `is_pause_500`
- `is_pause_1000`
- `is_pause_2000`

These are helpful for visual explanations and recovery-style profiling.

---

## 10. Edit behavior features

From the raw event log, derive:
- `is_insert`
- `is_delete`
- `insert_len`
- `delete_len`
- `net_len_change`
- `is_backspace_like`

These are not enough by themselves, but they are important ingredients.

---

## 11. Recovery-process features

The project should explicitly model error episodes and recovery behavior.

### 11.1 Error state dynamics
Derive:
- `error_suffix_delta`
- `delete_after_error`
- `is_recovery_step`

Suggested definitions:
- `delete_after_error = 1` if the event is a delete and the prior event had `error_suffix_len > 0`
- `is_recovery_step = 1` if the event improves alignment while the trial is recovering from a wrong suffix

### 11.2 Error episode segmentation
Define an error episode as a contiguous segment where:
- `error_suffix_len` becomes positive
- stays positive for one or more events
- then returns to zero

Store:
- `error_episode_id`

Later derive episode summaries such as:
- maximum wrong-suffix depth
- episode duration
- number of delete events
- pause before first delete

These are especially useful for RQ4.

---

## 12. Target-character context

At each event, attach the target context.

Using `prefix_match_len_before`, define:
- `prev_correct_pos`
- `expected_pos`
- `prev_correct_char`
- `expected_char`

Interpretation:
- `prev_correct_char` is the last prompt character already matched before the event
- `expected_char` is the next correct prompt character the participant needs to produce

This context is required for keyboard-demand features.

---

## 13. Keyboard-demand features using character map + distance matrix

This is the final authoritative replacement for old `(x, y)` geometry ideas.

### 13.1 Prompt-demand features
These describe what the prompt is demanding at that moment.

For `prev_correct_char` and `expected_char`, attach from the character map:
- `layout_required`
- `shift_required`
- `physical_key_id`
- `distance_node_id`
- `key_row`
- `zone_proxy`
- `hand_proxy`
- `char_class`

Then derive:
- `prev_layout`
- `expected_layout`
- `layout_switch_needed`
- `expected_shift_required`
- `shift_transition`
- `prev_node_id`
- `expected_node_id`
- `target_key_distance`
- `same_key_transition`
- `same_hand_transition`
- `zone_transition`

### 13.2 Layout-switch rule
Use:

```text
layout_switch_needed = 1 if prev_layout != expected_layout else 0
```

Special cases:
- if either side is `COMMON` space, apply a neutral rule rather than forcing a switch
- document the exact rule in code

### 13.3 Key distance
Use the distance matrix to compute:

```text
target_key_distance = distance(prev_node_id, expected_node_id)
```

This is the main planned movement burden feature.

### 13.4 Shift burden
At minimum store:
- `expected_shift_required`

Optional:
- `shift_transition = 1` when the expected character changes shift state relative to the prior correct character

---

## 14. Realized error proximity features

These features describe what kind of wrong character was actually typed.

Apply them only when there is an inserted wrong character.

### 14.1 Identify candidate wrong character
Use cases such as:
- `is_insert == 1`
- `inserted_text` length is 1
- event does not produce positive prefix gain
- expected target character exists

Then define:
- `wrong_char = inserted_text`

### 14.2 Join wrong character to map
Resolve `wrong_char` through the character map using the project ambiguity rules.

### 14.3 Error-to-target distance
If both sides are resolved:

```text
error_to_target_distance = distance(wrong_node_id, expected_node_id)
```

This measures whether the mistype is physically close to the intended key.

### 14.4 Suggested error labels
Create labels such as:
- `is_near_key_typo`
- `is_layout_confusion`
- `is_shift_confusion`
- `is_far_error`

These should be rule-based and documented.

Example intuition:
- near-key typo -> small key distance
- layout confusion -> same physical key family but mismatched layout expectation
- shift confusion -> same physical key family but wrong shift state
- far error -> large key distance and no obvious local explanation

These features are very useful for separating geometry-driven mistakes from more conceptual or linguistic mistakes.

---

## 15. Geometry burden definitions

The project should separate two concepts.

### 15.1 Planned geometry burden
This reflects the burden of producing the **next correct character**.

Suggested components:
- `target_key_distance`
- `layout_switch_needed`
- `expected_shift_required`
- optional row / zone / same-hand penalties

A practical formula is:

```text
geo_burden_planned =
    wd * z(target_key_distance)
  + wl * layout_switch_needed
  + ws * expected_shift_required
```

Optional extensions:
- `wr * row_change_penalty`
- `wh * same_hand_penalty`

### 15.2 Realized error geometry burden
This reflects the geometry of the actual mistake.

Suggested components:
- `error_to_target_distance`
- `is_layout_confusion`
- `is_shift_confusion`

Example:

```text
geo_burden_error =
    ue * z(error_to_target_distance)
  + ul * is_layout_confusion
  + us * is_shift_confusion
```

### 15.3 Why both should be kept
Do not collapse these too early.

They answer different questions:
- planned burden -> how demanding the prompt was
- realized error burden -> what kind of mistake actually happened

---

## 16. Friction waveform construction

The project should build a scalar event-by-event friction signal.

### 16.1 Why friction exists
A single scalar sequence is very useful for:
- visualization
- Euclidean baseline comparison
- DTW on a compact signal
- Matrix Profile motif and discord analysis

### 16.2 Recommended ingredients
Use standardized ingredients such as:
- `z_log_dt`
- `is_delete`
- `z_error_suffix_len`
- `z_geo_burden_planned`
- `layout_switch_needed`

The scalar friction signal should now be intentionally simplified. `geo_burden_error` may still exist as a separate diagnostic feature, but it should **not** be included in the canonical friction calculation.

### 16.3 Practical formula
Use the following fixed weighted formula for the canonical event-level friction column `friction`:

```text
friction_t =
    0.30 * z_log_dt_ms
  + 0.25 * is_delete
  + 0.30 * z_error_suffix_len_after
  + 0.08 * z_geo_burden_planned
  + 0.02 * layout_switch_needed
```

This change was made because the older friction definition was too noisy for downstream use. Sparse error-side terms, especially `geo_burden_error`, produced undesirable spikes in plots and scalar waveforms.

For visualization only, also create:

```text
friction_plot_smooth =
    centered rolling mean of friction
    with window = 5 and min_periods = 1
    computed within each trial only
```

Use:
- canonical `friction` for downstream modeling and exported analysis tables
- `friction_plot_smooth` for event-trace and waveform plotting

### 16.4 Important rule
Freeze the friction definition before running the full final analysis.

---

## 17. Output table 1: enriched event table

This is the main master table.

Recommended filename:
- `event_sequence_enriched.csv`

Recommended columns include:
- identifiers: `session_id`, `participant_code`, `trial_index`, `prompt_id`, `event_index`
- raw timing and action fields
- `text_before`, `text_after`
- alignment features
- recovery features
- target-character context
- character-map metadata for expected / previous chars
- distance-matrix-based burden features
- realized error proximity features
- final friction value
- smoothed visualization-only friction trace
- QC flags

This table should remain event-granular.

---

## 18. Output table 2: prompt-position feature table

Recommended filename:
- `prompt_position_features.csv`

This table has one row per `(session_id, trial_index, prompt_position)`.

### 18.1 Core position features
For each prompt position `j`, derive:
- `time_to_first_correct_j`
- `time_to_stability_j`
- `revisions_j`
- `wrong_first_j`

### 18.2 Optional local timing features
- `local_mean_dt_j`
- `local_max_dt_j`
- `local_mean_friction_j`

### 18.3 Position metadata
Attach:
- `char_j`
- `char_class_j`
- `layout_required_j`
- `shift_required_j`
- `distance_from_prev_correct_j`
- `layout_switch_into_j`
- `zone_proxy_j`
- `hand_proxy_j`

This table is the main source for shared difficulty region analysis.

---

## 19. Output table 3: trial summary table

Recommended filename:
- `trial_features.csv`

One row per trial.

Suggested contents:
- duration
- first-key latency
- event count
- insert count
- delete count
- mean / median `dt_ms`
- mean / max friction
- total time spent in error state
- number of error episodes
- maximum wrong-suffix depth
- mean planned geometry burden
- layout-switch count
- shift-required count
- near-key typo count
- layout confusion count

This table is useful for prompt-level comparisons and descriptive reporting.

---

## 20. Output table 4: participant keyboard-demand profile table

Recommended filename:
- `participant_keyboard_profile.csv`

This table aggregates behavior across sessions or across the final accepted session per participant.

### 20.1 Why this table matters
It supports participant-level profiling and clustering.

### 20.2 Suggested grouping dimensions
Aggregate by conditions such as:
- Thai vs English layout
- shifted vs non-shifted characters
- layout-switch vs non-switch positions
- low / medium / high target-distance bins
- low / medium / high error-distance bins
- tone marks
- punctuation / symbol spans
- Arabic numerals vs Thai numerals
- zone proxy
- hand proxy

### 20.3 Suggested aggregated metrics
For each participant x subgroup:
- mean `time_to_stability`
- mean `wrong_first`
- mean `revisions`
- mean `log_dt`
- mean friction
- delete rate
- near-key typo rate
- layout confusion rate
- shift confusion rate

---

## 21. Final time-series objects

### TS Object A. Event-aligned multivariate edit sequence
Suggested base dimensions:

```text
x_t = [
  log_dt,
  is_delete,
  prefix_gain,
  error_suffix_len,
  geo_burden_planned,
  layout_switch_needed
]
```

Optional extra dimension:
- `geo_burden_error`

Use for:
- DTW similarity
- recovery-style comparison
- clustering with a DTW distance matrix

### TS Object B. Scalar friction waveform
Use canonical `friction_t` as the 1D modeling sequence. Use `friction_plot_smooth` only when plotting or presenting the waveform visually.

Use for:
- Euclidean baseline
- DTW on a compact representation
- Matrix Profile motif / discord exploration
- clear visual narratives in the report

### TS Object C. Prompt-position stability sequence
For each position `j`, use one or more of:
- `time_to_stability_j`
- `time_to_first_correct_j`
- `revisions_j`
- `wrong_first_j`

Use for:
- shared difficulty heatmaps
- region-level comparison across users
- linking difficulty spans to prompt structure

### TS Object D. Participant keyboard-demand profile
A participant-level structured vector aggregated across keyboard and burden categories.

Use for:
- participant clustering
- style profiling
- interpretation of individual typing tendencies

---

## 22. Recommended normalization strategy

### 22.1 Event sequence features
For continuous event features such as:
- `log_dt`
- `error_suffix_len`
- `geo_burden_planned`
- `geo_burden_error`

Use global or training-set-level normalization, then keep binary flags as 0/1.

### 22.2 Friction waveform
If friction is built from standardized ingredients, optionally z-normalize the final waveform per trial for methods that assume local comparability.

### 22.3 Prompt-position timing
Keep raw milliseconds for interpretability, but also consider log transforms for robust modeling.

---

## 23. Suggested implementation outputs

The transformation notebook or pipeline should ideally produce:
- `event_sequence_enriched.csv`
- `error_episode_summary.csv`
- `prompt_position_features.csv`
- `trial_features.csv`
- `participant_keyboard_profile.csv`
- sequence-friendly files such as parquet for long per-trial sequences if necessary

---

## 24. Suggested notebook / module breakdown

A clean implementation split would be:

1. `01_load_and_validate.ipynb`
2. `02_reconstruct_text_state.ipynb`
3. `03_target_alignment_and_event_features.ipynb`
4. `04_character_map_and_distance_features.ipynb`
5. `05_prompt_position_features.ipynb`
6. `06_trial_and_participant_profiles.ipynb`
7. `07_time_series_objects_and_exports.ipynb`

This makes the project easier to debug and easier to explain.

---

## 25. Common mistakes to avoid

Do not:
- use raw text itself as the main time series
- reduce the whole project to `dt_ms` only
- collapse everything into one final score too early
- assume `char` uniquely identifies a key-state without checking the character map
- silently reintroduce `(x, y)` geometry
- claim biometrics that were never collected

---

## 26. Final concise summary

The final transformation plan is:

1. validate raw logs
2. reconstruct text before/after each event
3. align each event state to the target prompt
4. derive timing, correction, and recovery features
5. attach character-map metadata to prompt-demand context
6. use the supplied distance matrix to derive planned and realized geometry features
7. build the enriched event table
8. derive prompt-position features
9. derive trial summaries and participant keyboard-demand profiles
10. construct the final time-series objects for DTW, Matrix Profile, clustering, and visualization

This is the final current data-preparation and time-series transformation blueprint for the project.
