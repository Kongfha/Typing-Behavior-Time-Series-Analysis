# Transformed CSV Outputs and TS Object Guide

This document explains:

- which CSV files were created by `01_time_series_transformation_and_pilot_visualization.ipynb`
- what each CSV represents
- how each CSV was derived from the raw browser diff logs
- how to construct the project's time-series objects from those files

The notebook path is:

- [01_time_series_transformation_and_pilot_visualization.ipynb](/Users/kongfha/Desktop/Time_Series_Mining/Final Project/Typing-Behavior-Time-Series-Analysis/01_time_series_transformation_and_pilot_visualization.ipynb)

The output directory is:

- [outputs/01_time_series_transformation_and_pilot_visualization](/Users/kongfha/Desktop/Time_Series_Mining/Final Project/Typing-Behavior-Time-Series-Analysis/outputs/01_time_series_transformation_and_pilot_visualization)

## 1. Input Files Used

The transformation notebook is path-driven. In the current workspace snapshot, it used:

- raw pooled edit-event log: `data/combined_edit_event_logs_data.csv`
- prompt metadata: `thai_typing_prompts.csv`
- character map: `character_map_table_comprehensive.csv`
- keyboard distance matrix: `Key_Distance_Matrix/dist_mat.pkl`
- keyboard adjacency matrix: `Key_Distance_Matrix/adj_mat.pkl`
- design references:
  - `project_context_comprehensive.md`
  - `data_preparation_and_time_series_transformation.md`

Important current-state note:

- `trial_log*.csv` and `participant_session*.csv` were not present under `data/`
- the notebook still ran successfully from the pooled event log
- trial/session QC therefore comes from what can be verified from the pooled event table and prompt table

## 2. High-Level Derivation Flow

The full derivation chain is:

```text
raw pooled edit-event log
-> prompt table loading
-> practice prompt recovery
-> character-map ambiguity resolution
-> keyboard distance lookup construction
-> event reconstruction (text_before / text_after)
-> target alignment to prompt_text
-> timing, edit, recovery, and QC features
-> keyboard-demand joins and typo-proximity features
-> enriched event table
-> error episode summaries
-> prompt-position features
-> trial QC and trial summary features
-> session QC
-> participant keyboard-demand profile
-> sequence-ready narrow tables for later time-series mining
```

Conceptually, the key modeling rule is:

```text
current_text = correct_prefix + wrong_suffix
```

## 3. What Each CSV Means

### 3.1 `char_map_resolution_audit.csv`

Grain:

- one row per visible character after prompt-demand resolution

Purpose:

- audit how ambiguous characters were resolved before they were used in prompt-demand or wrong-character joins

How it was derived:

- start from `character_map_table_comprehensive.csv`
- compute a project layout preference:
  - Thai script -> `TH`
  - English letters / digits / punctuation -> `EN`
  - space -> `COMMON`
- if a character has multiple candidate map rows, choose the best row by:
  - layout preference match
  - `preferred_for_prompt_demand`
  - deterministic fallback if still tied

Typical columns:

- `char`
- `project_layout_preference`
- `char_map_candidate_count`
- `char_map_top_rule_candidate_count`
- `chosen_map_id`
- `chosen_physical_key_id`
- `resolution_note`

Use it for:

- validating ambiguity handling
- documenting punctuation / mixed-layout assumptions

### 3.2 `keyboard_distance_lookup.csv`

Grain:

- one row per source-node x target-node key pair

Purpose:

- convert the pickle distance / adjacency assets into a transparent long-form table

How it was derived:

- map each `physical_key_id` / `distance_node_id` to a standard keycode
- read `dist_mat.pkl` and `adj_mat.pkl`
- expand all key-to-key combinations into a long-form lookup

Typical columns:

- `source_node_id`
- `target_node_id`
- `source_physical_key_id`
- `target_physical_key_id`
- `source_keycode`
- `target_keycode`
- `distance`
- `is_adjacent`

Use it for:

- sanity-checking key geometry
- joining planned and realized key distances outside the notebook

### 3.3 `prompt_character_reference.csv`

Grain:

- one row per prompt position

Purpose:

- build a clean prompt-position reference table before looking at any participant behavior

How it was derived:

- expand each prompt string into one row per character position
- attach current-character metadata from the resolved character map
- attach previous-character metadata
- compute prompt-internal demand features such as:
  - `distance_from_prev_correct_j`
  - `layout_switch_into_j`
  - `distance_from_prev_bin_j`

Typical columns:

- `prompt_id`
- `prompt_position`
- `char_j`
- `prev_char_j`
- `char_class_j`
- `layout_required_j`
- `shift_required_j`
- `distance_from_prev_correct_j`
- `layout_switch_into_j`

Use it for:

- pure prompt-demand analysis
- merging prompt structure into prompt-position behavioral tables

### 3.4 `event_sequence_enriched.csv`

Grain:

- one row per raw textarea diff event

Purpose:

- this is the master transformation table

How it was derived:

1. start from the pooled raw edit-event log
2. sort within each trial by `event_index`
3. reconstruct `text_before` and `text_after`
4. verify append-only insert and pop-only delete behavior
5. align each `text_before` / `text_after` to the target prompt
6. derive timing, edit, recovery, and QC features
7. attach target-character metadata:
   - `prev_correct_char`
   - `expected_char`
8. attach keyboard-demand features:
   - `target_key_distance`
   - `layout_switch_needed`
   - `shift_transition`
9. detect realized wrong-character features:
   - `wrong_char`
   - `error_to_target_distance`
   - `is_near_key_typo`
   - `is_layout_confusion`
   - `is_shift_confusion`
10. compute geometry burden and scalar `friction`

Canonical friction definition now used in this notebook:

```text
friction =
    0.30 * z_log_dt_ms
  + 0.25 * is_delete
  + 0.30 * z_error_suffix_len_after
  + 0.08 * z_geo_burden_planned
  + 0.02 * layout_switch_needed
```

Important notes:

- `geo_burden_error` is no longer included in canonical friction
- the older friction behavior was too noisy because sparse error-side terms, especially `geo_burden_error`, created undesirable spikes
- `friction_plot_smooth` is a visualization-only centered rolling mean of canonical `friction`, computed within trial with `window=5` and `min_periods=1`

Major feature families inside this file:

- raw event fields
- reconstruction fields:
  - `text_before`
  - `text_after`
- alignment fields:
  - `prefix_match_len_before`
  - `prefix_match_len_after`
  - `prefix_gain`
  - `error_suffix_len_after`
  - `remaining_len_after`
- timing fields:
  - `dt_ms`
  - `dt_ms_filled`
  - `log_dt_ms`
  - `is_pause_500`
  - `is_pause_1000`
  - `is_pause_2000`
- recovery fields:
  - `delete_after_error`
  - `is_recovery_step`
  - `error_episode_id`
- target-demand metadata:
  - `prev_*`
  - `expected_*`
- realized mistype metadata:
  - `wrong_*`
- geometry / burden fields:
  - `target_key_distance`
  - `error_to_target_distance`
  - `geo_burden_planned`
  - `geo_burden_error`
- scalar summary field:
  - `friction`
  - `friction_plot_smooth`
- QC fields:
  - `qc_event_index_ok`
  - `qc_time_monotonic_ok`
  - `qc_append_pop_ok`
  - `qc_reconstruction_ok`
  - `qc_exact_match_ok`

Use it for:

- full event-level analysis
- any later feature engineering
- building multivariate trial sequences

### 3.5 `event_sequence_core.csv`

Grain:

- one row per event, but narrowed to the core time-series fields

Purpose:

- lightweight sequence table for later time-series mining

How it was derived:

- subset selected columns from `event_sequence_enriched.csv`

Columns:

- identifiers:
  - `trial_uid`
  - `session_id`
  - `participant_code`
  - `trial_index`
  - `trial_type`
  - `prompt_id`
  - `event_index`
- core sequence features:
  - `dt_ms_filled`
  - `log_dt_ms`
  - `is_delete`
  - `prefix_gain`
  - `error_suffix_len_after`
  - `geo_burden_planned`
  - `layout_switch_needed`
- `friction`
- `friction_plot_smooth`

Use it for:

- DTW-ready event sequences
- friction-waveform experiments
- fast prototyping without carrying 188 columns

### 3.6 `error_episode_summary.csv`

Grain:

- one row per error episode

Purpose:

- summarize contiguous wrong-suffix episodes for recovery-style analysis

How it was derived:

- take `event_sequence_enriched.csv`
- isolate contiguous spans where `error_suffix_len_after > 0`
- assign `error_episode_id`
- aggregate each episode

Typical columns:

- `trial_uid`
- `error_episode_id`
- `episode_start_event_index`
- `episode_end_event_index`
- `episode_duration_ms`
- `episode_event_count`
- `episode_delete_count`
- `episode_recovery_step_count`
- `max_error_suffix_depth`
- `pause_before_first_delete_ms`

Use it for:

- RQ4 recovery-style profiling
- pause-then-delete vs immediate repair comparisons

### 3.7 `prompt_position_features.csv`

Grain:

- one row per trial x prompt position

Purpose:

- convert the event stream into position-aligned behavioral features

How it was derived:

- for each trial and each prompt position:
  - find `time_to_first_correct_ms`
  - find `time_to_stability_ms`
  - count `revisions`
  - compute `wrong_first`
  - summarize local event timing and friction
- then merge prompt-position metadata from `prompt_character_reference.csv`

Typical columns:

- identifiers:
  - `trial_uid`
  - `session_id`
  - `participant_code`
  - `prompt_id`
  - `prompt_position`
- behavioral position features:
  - `time_to_first_correct_ms`
  - `time_to_stability_ms`
  - `revisions`
  - `wrong_first`
  - `local_mean_dt_ms`
  - `local_max_dt_ms`
  - `local_mean_friction`
  - `final_position_correct`
- prompt metadata:
  - `char_j`
  - `char_class_j`
  - `layout_required_j`
  - `shift_required_j`
  - `distance_from_prev_correct_j`
  - `layout_switch_into_j`

Use it for:

- shared difficulty heatmaps
- prompt-region difficulty analysis
- position-based TS Object construction

### 3.8 `prompt_position_sequence_ready.csv`

Grain:

- one row per trial x prompt position

Purpose:

- narrow prompt-position sequence table for direct time-series modeling

How it was derived:

- subset selected columns from `prompt_position_features.csv`

Columns:

- `trial_uid`
- `session_id`
- `participant_code`
- `trial_index`
- `trial_type`
- `prompt_id`
- `prompt_position`
- `prompt_position_1based`
- `time_to_first_correct_ms`
- `time_to_stability_ms`
- `revisions`
- `wrong_first`

Use it for:

- TS Object C
- prompt-position sequence clustering or region comparison

### 3.9 `trial_qc.csv`

Grain:

- one row per trial

Purpose:

- compact trial-level QC checkpoint

How it was derived:

- aggregate from `event_sequence_enriched.csv` after reconstruction and alignment

Typical columns:

- prompt identity fields
- `event_count`
- `final_reconstructed_text`
- `final_prefix_match_len`
- `final_resulting_text_length`
- QC flags:
  - `qc_prompt_available_ok`
  - `qc_event_index_ok`
  - `qc_time_monotonic_ok`
  - `qc_append_pop_ok`
  - `qc_reconstruction_ok`
  - `qc_exact_match_ok`
  - `all_core_qc_ok`

Use it for:

- filtering valid trials before analysis
- explaining why a trial should be kept or excluded

### 3.10 `session_qc.csv`

Grain:

- one row per session

Purpose:

- session-level completion and ordering QC

How it was derived:

- aggregate `trial_qc.csv` by session
- verify:
  - number of main trials
  - presence of practice
  - completeness of prompt IDs
  - deployed prompt order
  - whether all trials in the session passed core QC

Typical columns:

- `session_id`
- `participant_code`
- `n_trials_observed`
- `n_main_trials_observed`
- `has_practice_trial`
- `qc_main_prompt_count_ok`
- `qc_main_prompt_ids_complete`
- `qc_main_prompt_order_ok`
- `qc_all_trial_core_ok`

Use it for:

- session-level inclusion decisions

### 3.11 `trial_features.csv`

Grain:

- one row per trial

Purpose:

- main trial summary table for descriptive and comparative analysis

How it was derived:

- aggregate `event_sequence_enriched.csv` to trial level
- merge non-duplicative QC information from `trial_qc.csv`

Typical columns:

- timing / size:
  - `duration_ms`
  - `first_key_latency_ms`
  - `event_count`
  - `insert_count`
  - `delete_count`
  - `mean_dt_ms`
  - `median_dt_ms`
- friction / correction:
  - `mean_friction`
  - `max_friction`
  - `total_time_in_error_ms`
  - `number_of_error_episodes`
  - `maximum_wrong_suffix_depth`
- burden / typo:
  - `mean_planned_geometry_burden`
  - `layout_switch_count`
  - `shift_required_count`
  - `near_key_typo_count`
  - `layout_confusion_count`
  - `shift_confusion_count`
- QC summary:
  - `qc_exact_match_ok`
  - `all_core_qc_ok`

Use it for:

- prompt-level comparison
- pilot descriptive statistics
- selecting interesting trials for visualization

### 3.12 `participant_keyboard_profile.csv`

Grain:

- one row per participant x subgroup

Purpose:

- participant-level keyboard-demand profile table

How it was derived:

- aggregate event and position features by participant over different profile dimensions:
  - `expected_layout`
  - `expected_shift_required`
  - `layout_switch_needed`
  - `target_distance_bin`
  - `expected_char_class`
  - `expected_zone_proxy`
  - `expected_hand_proxy`

Typical columns:

- `participant_code`
- `profile_dimension`
- `profile_value`
- event-side aggregates:
  - `mean_log_dt_ms`
  - `mean_friction`
  - `delete_rate`
  - `near_key_typo_rate`
  - `layout_confusion_rate`
  - `shift_confusion_rate`
- position-side aggregates:
  - `mean_time_to_first_correct_ms`
  - `mean_time_to_stability_ms`
  - `mean_revisions`
  - `wrong_first_rate`

Use it for:

- participant profiling
- participant clustering
- keyboard-demand interpretation

## 4. Which Files Are “Core” vs “Support”

Recommended core analytic files:

- `event_sequence_enriched.csv`
- `event_sequence_core.csv`
- `prompt_position_features.csv`
- `prompt_position_sequence_ready.csv`
- `trial_features.csv`
- `participant_keyboard_profile.csv`

Recommended support / audit files:

- `trial_qc.csv`
- `session_qc.csv`
- `error_episode_summary.csv`
- `prompt_character_reference.csv`
- `keyboard_distance_lookup.csv`
- `char_map_resolution_audit.csv`

## 5. Recommended Filtering Before Building TS Objects

Before final time-series analysis, usually filter to:

- `trial_type == "main"` if you want only main prompts
- `all_core_qc_ok == 1`
- `qc_exact_match_ok == 1` if you want only successfully completed exact-copy trials

Possible Python pattern:

```python
import pandas as pd

trial_qc = pd.read_csv("outputs/01_time_series_transformation_and_pilot_visualization/trial_qc.csv")

keep_trials = trial_qc.loc[
    (trial_qc["trial_type"] == "main")
    & (trial_qc["all_core_qc_ok"] == 1)
    & (trial_qc["qc_exact_match_ok"] == 1),
    "trial_uid",
]
```

## 6. How To Build TS Objects From These Files

The project design defines four main TS objects.

### 6.1 TS Object A: Event-Aligned Multivariate Edit Sequence

Best source file:

- `event_sequence_core.csv`

Optional richer source:

- `event_sequence_enriched.csv`

Recommended feature vector:

```text
x_t = [
  log_dt_ms,
  is_delete,
  prefix_gain,
  error_suffix_len_after,
  geo_burden_planned,
  layout_switch_needed
]
```

How to build it:

```python
import pandas as pd

event_core = pd.read_csv("outputs/01_time_series_transformation_and_pilot_visualization/event_sequence_core.csv")
trial_qc = pd.read_csv("outputs/01_time_series_transformation_and_pilot_visualization/trial_qc.csv")

keep_trials = trial_qc.loc[
    (trial_qc["trial_type"] == "main")
    & (trial_qc["all_core_qc_ok"] == 1)
    & (trial_qc["qc_exact_match_ok"] == 1),
    "trial_uid",
]

event_core = event_core[event_core["trial_uid"].isin(keep_trials)].copy()
event_core = event_core.sort_values(["trial_uid", "event_index"])

feature_cols = [
    "log_dt_ms",
    "is_delete",
    "prefix_gain",
    "error_suffix_len_after",
    "geo_burden_planned",
    "layout_switch_needed",
]

ts_object_a = {
    trial_uid: grp[feature_cols].to_numpy()
    for trial_uid, grp in event_core.groupby("trial_uid", sort=False)
}
```

Interpretation:

- one multivariate event sequence per trial
- useful for DTW, recovery-style similarity, and event-level clustering

### 6.2 TS Object B: Scalar Friction Waveform

Best source file:

- `event_sequence_core.csv`

Required column:

- `friction`

Visualization-only companion column:

- `friction_plot_smooth`

How to build it:

```python
friction_waveforms = {
    trial_uid: grp.sort_values("event_index")["friction"].to_numpy()
    for trial_uid, grp in event_core.groupby("trial_uid", sort=False)
}
```

For plotting only:

```python
friction_waveforms_plot = {
    trial_uid: grp.sort_values("event_index")["friction_plot_smooth"].to_numpy()
    for trial_uid, grp in event_core.groupby("trial_uid", sort=False)
}
```

Interpretation:

- one 1D waveform per trial
- use canonical `friction` for downstream analysis, Euclidean baselines, DTW, and Matrix Profile work
- use `friction_plot_smooth` for event-trace plotting and visual waveform presentation

### 6.3 TS Object C: Prompt-Position Stability Sequence

Best source file:

- `prompt_position_sequence_ready.csv`

Optional richer source:

- `prompt_position_features.csv`

Recommended feature candidates:

- `time_to_stability_ms`
- `time_to_first_correct_ms`
- `revisions`
- `wrong_first`

How to build a multivariate position sequence:

```python
position_seq = pd.read_csv(
    "outputs/01_time_series_transformation_and_pilot_visualization/prompt_position_sequence_ready.csv"
)

position_seq = position_seq[position_seq["trial_uid"].isin(keep_trials)].copy()
position_seq = position_seq.sort_values(["trial_uid", "prompt_position"])

position_feature_cols = [
    "time_to_stability_ms",
    "time_to_first_correct_ms",
    "revisions",
    "wrong_first",
]

ts_object_c = {
    trial_uid: grp[position_feature_cols].to_numpy()
    for trial_uid, grp in position_seq.groupby("trial_uid", sort=False)
}
```

Interpretation:

- one prompt-position sequence per trial
- useful for shared difficulty region analysis and prompt-region comparison across users

### 6.4 TS Object D: Participant Keyboard-Demand Profile

Best source file:

- `participant_keyboard_profile.csv`

How to build it:

This file is already a participant-level structured vector table. You usually pivot it to a wide feature matrix.

```python
profile = pd.read_csv(
    "outputs/01_time_series_transformation_and_pilot_visualization/participant_keyboard_profile.csv"
)

wide_profile = profile.pivot_table(
    index="participant_code",
    columns=["profile_dimension", "profile_value"],
    values="mean_friction",
)
```

You can repeat the same pivot for:

- `mean_time_to_stability_ms`
- `delete_rate`
- `near_key_typo_rate`
- `layout_confusion_rate`
- `shift_confusion_rate`

Interpretation:

- one structured vector per participant
- useful for participant clustering and style profiling

## 7. Which File To Use For Which Task

If the task is:

- debug raw reconstruction -> use `event_sequence_enriched.csv`
- check whether interface constraints held -> use `trial_qc.csv`, `session_qc.csv`, and event QC columns in `event_sequence_enriched.csv`
- inspect keyboard-demand definitions -> use `prompt_character_reference.csv`, `keyboard_distance_lookup.csv`, `char_map_resolution_audit.csv`
- analyze shared difficult prompt regions -> use `prompt_position_features.csv`
- compare trials at a descriptive level -> use `trial_features.csv`
- analyze recovery episodes -> use `error_episode_summary.csv`
- build event-level time-series models -> use `event_sequence_core.csv`
- build prompt-position time-series models -> use `prompt_position_sequence_ready.csv`
- build participant-level profile vectors -> use `participant_keyboard_profile.csv`

## 8. Practical Notes

- `event_sequence_enriched.csv` is the authoritative master file.
- `event_sequence_core.csv` and `prompt_position_sequence_ready.csv` are convenience files for modeling.
- `trial_features.csv` should usually be the first table for descriptive reporting.
- `trial_qc.csv` should usually be the first table for inclusion filtering.
- `participant_keyboard_profile.csv` is already close to TS Object D, but it is long-form and normally needs pivoting.
- `prompt_character_reference.csv` is not behavior by itself; it is a prompt-demand scaffold.

## 9. Recommended Next Step

For the next notebook or script that creates the final time-series mining inputs, a good pattern is:

1. load `trial_qc.csv`
2. define the final trial inclusion rule
3. load `event_sequence_core.csv` for TS Objects A and B
4. load `prompt_position_sequence_ready.csv` for TS Object C
5. load `participant_keyboard_profile.csv` for TS Object D
6. normalize or standardize features according to the final modeling plan
7. save the final Python objects or NumPy arrays for DTW / Matrix Profile / clustering

That keeps transformation and modeling cleanly separated.
