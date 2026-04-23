# Thai Typing Time-Series Mining Project
## Detailed Analysis Plan for Descriptive Statistics, Euclidean Baselines, DTW, Hierarchical Clustering, and Matrix Profile

## 1. Purpose of this document

This document specifies the **main analysis notebook** (`02_analysis_descriptive_euclidean_dtw_clustering.ipynb`) after the transformation notebook.

The transformation stage is already complete. The project now has:
- event-level enriched sequential data
- prompt-position feature tables
- trial-level summaries
- participant-level keyboard-demand profiles
- quality-control tables
- sequence-ready tables for downstream time-series mining

This analysis plan covers **all five analysis stages** in a single notebook:

1. **Descriptive and sanity analysis**
2. **Euclidean baseline analysis**
3. **DTW-based similarity analysis**
4. **Hierarchical clustering**
5. **Matrix Profile on long 1D sequences**

This document **does not** include:
- lower-bound acceleration experiments (LB_Keogh etc.)
- SAX or symbolic analysis

> **Note on Stage 5.** Matrix Profile was originally planned for a separate notebook.
> It has been integrated directly into this notebook because the long-prompt data
> (`P12`, `P13`, `P14`) is already loaded, filtered, and aggregated by Stage 1, and
> running MP immediately after clustering gives the most economical bridge between
> user-level and prompt-level signals. All Stage 5 outputs land in
> `outputs/02_analysis_descriptive_euclidean_dtw_clustering/stage5_matrix_profile/`.

---

## 2. Current project state

The project has already completed the browser-log transformation pipeline. The current transformed outputs include:
- `event_sequence_enriched.csv`
- `event_sequence_core.csv`
- `prompt_position_features.csv`
- `prompt_position_sequence_ready.csv`
- `trial_features.csv`
- `participant_keyboard_profile.csv`
- `trial_qc.csv`
- `session_qc.csv`
- `error_episode_summary.csv`
- several support and audit tables

The transformation guide also defines how these outputs map to the project’s time-series objects:
- **TS Object A**: event-aligned multivariate sequence
- **TS Object B**: scalar friction waveform
- **TS Object C**: prompt-position stability sequence
- **TS Object D**: participant keyboard-demand profile

This means the project is now ready to move from transformation into structured analysis.

---

## 3. Scope of the analysis notebook

### 3.1 Included stages

This notebook should implement:

### Stage 1. Descriptive and sanity layer
Goal:
- understand prompt-level and participant-level behavior
- verify that the transformed features behave sensibly
- identify the strongest candidate prompts and regions for downstream time-series analysis

### Stage 2. Euclidean baselines
Goal:
- create fixed-length baseline similarity analyses where lockstep comparison is appropriate
- establish simple reference results before using DTW

### Stage 3. DTW analysis
Goal:
- compare warped event-level recovery sequences
- compare prompt-level friction or position sequences when temporal misalignment matters
- answer recovery-style questions and same-prompt cross-user similarity questions

### Stage 4. Hierarchical clustering
Goal:
- group participants, prompts, and recovery episodes
- obtain interpretable behavioral families
- use Euclidean or DTW distance matrices depending on the object type

### Stage 5. Matrix Profile on long 1D sequences
Goal:
- discover recurring local patterns (motifs) and anomalous subsequences (discords)
  within the long-prompt friction waveforms, wrong-suffix-depth sequences, and
  aggregated per-position difficulty curves
- use results to answer *where* users share difficulty (RQ3) and *how* recurring
  recovery shapes appear within a single user's trial (RQ4)
- restrict MP to P12/P13/P14 only (127–145 characters, 148–317 events) where
  sequences are long enough to make subsequence mining meaningful

### 3.2 Explicitly excluded from this notebook

Do **not** implement here:
- lower-bound pruning experiments such as LB_Keogh
- SAX or symbolic analysis

Those remain outside this notebook.

---

## 4. Main research questions and how this notebook addresses them

## RQ1. Character- and zone-level fluency
Do participants show different levels of fluency across keyboard-demand categories such as:
- left / center / right
- row groups
- shifted vs non-shifted
- Thai vs English layout
- layout-switch positions
- numerals, punctuation, tone marks, consonants, vowels

### Best data sources
- `participant_keyboard_profile.csv`
- `prompt_position_features.csv`
- `trial_features.csv`

### Main methods in this notebook
- descriptive summaries
- Euclidean baseline clustering on participant profile vectors
- participant profile heatmaps
- groupwise comparison plots

---

## RQ2. Linguistic friction versus keyboard-layout friction
When difficulty appears, is it better explained by:
- keyboard-demand structure
- or linguistic / lexical / orthographic burden

### Best data sources
- `prompt_position_features.csv`
- `prompt_character_reference.csv`
- `trial_features.csv`
- optionally `event_sequence_enriched.csv`

### Main methods in this notebook
- descriptive contrasts
- prompt-level paired comparisons
- transparent regression-style models
- region heatmaps and burden overlays

### Important note
This question is only partly a time-series question. The best answer is likely to combine:
- prompt-position summaries
- transparent statistical modeling
- region interpretation

Do not force this question into DTW or clustering if those methods are not the most interpretable choice.

---

## RQ3. Shared difficulty regions across users
For the same prompt, do multiple users struggle at the same prompt region?

### Best data sources
- `prompt_position_features.csv`
- `prompt_position_sequence_ready.csv`

### Main methods in this notebook
- prompt-position heatmaps
- median or trimmed-mean difficulty curves by prompt position
- Euclidean comparison within prompt
- optional DTW comparison within prompt when difficulty signatures are temporally shifted

### Important note
The strongest answer in this notebook comes from three converging signals:
- prompt-position heatmaps (same-prompt Euclidean distances, Stage 2)
- aggregated per-position difficulty curves (local-difficulty peaks, Stage 1)
- **Matrix Profile discords on the aggregated curves** (Stage 5) — the most
  economical summary of *which exact character spans* drive shared difficulty

All three signals are produced in this notebook. Stage 5 discords on P14 converge
on the Thai numeral `๖` and the named entities `ชาคริษฐ์` / `เครือพันธ์`; Stage 5
discords on P13 converge on formal connective phrases (`เพื่อเป็นประโยชน์`, `เนื้อหา`).

---

## RQ4. Recovery style as a behavioral signature
Do users exhibit different recovery styles after making mistakes, such as:
- pause then delete
- immediate short correction
- deep wrong-suffix overshoot before repair
- burst-like delete/insert routines

### Best data sources
- `event_sequence_core.csv`
- `event_sequence_enriched.csv`
- `error_episode_summary.csv`

### Main methods in this notebook
- descriptive episode summaries
- DTW on event-level windows around error episodes
- hierarchical clustering of episode patterns
- participant-level recovery-style summaries

This is the research question where DTW is most central.

---

## 5. Core analysis objects to use

The notebook should use exactly these objects.

## 5.1 Object A: Event-aligned multivariate sequence
Source:
- `event_sequence_core.csv`

Recommended feature vector per event:
- `log_dt_ms`
- `is_delete`
- `prefix_gain`
- `error_suffix_len_after`
- `geo_burden_planned`
- `layout_switch_needed`

Use cases:
- DTW between trials
- DTW between recovery windows
- same-prompt cross-user comparison
- prompt similarity at the event level

---

## 5.2 Object B: Scalar friction waveform
Source:
- `event_sequence_core.csv`

Feature:
- `friction`

Use cases in this notebook:
- descriptive plots
- Euclidean baseline on normalized waveforms
- optional DTW on scalar waveforms for prompt-level comparison
- **Stage 5A**: per-trial Matrix Profile on `friction` for P12/P13/P14
  (window m=16) to find recurring friction patterns and anomalous friction spikes

For Stage 5A, compute MP separately for each trial; aggregate discord
positions across users using normalized event position (0–1 scale) to
identify *where in the trial* friction anomalies concentrate.

---

## 5.3 Object C: Prompt-position stability sequence
Source:
- `prompt_position_sequence_ready.csv`

Recommended per-position features:
- `time_to_stability_ms`
- `time_to_first_correct_ms`
- `revisions`
- `wrong_first`

Use cases:
- shared difficulty heatmaps (Stage 1)
- within-prompt user comparison (Stage 2, Euclidean)
- regional difficulty curves (Stage 1)
- Euclidean baseline or DTW within the same prompt (Stage 2/3)
- **Stage 5C**: MP on the *aggregated* per-position `median_time_to_stability_ms`
  curve (log-transformed, window m=8) for P12/P13/P14 — the clearest source of
  interpretable discord results in the project

For Stage 5C, first aggregate across users to get one curve per prompt (median
stability at each position), then run `stumpy.stump`. The per-position alignment
is exact (all users share the same prompt), so no warping is needed before MP.

---

## 5.4 Object D: Participant keyboard-demand profile
Source:
- `participant_keyboard_profile.csv`

Use cases:
- RQ1 fluency analysis
- participant clustering
- participant heatmaps
- style profiling

This object is already close to a feature matrix after pivoting.

---

## 6. Inclusion and filtering rules

Before any serious analysis, filter trials using the QC tables.

### Recommended rule for main analysis
Keep only trials satisfying:
- `trial_type == "main"`
- `all_core_qc_ok == 1`
- `qc_exact_match_ok == 1`

### Why
This keeps the analysis aligned with the intended interface assumptions:
- valid event ordering
- valid reconstruction
- append/pop constraint preserved
- correct final prompt completion

### Secondary optional analysis
You may keep practice trials only for:
- smoke-test plotting
- sanity checks
- debugging plots

But they should not be part of the main project analysis.

---

## 7. Stage 1: Descriptive and sanity analysis

## 7.1 Goals
- verify that transformed features behave sensibly
- identify the hardest prompts
- identify the hardest character classes and keyboard-demand conditions
- locate visually interesting prompt-position regions
- identify recovery behavior patterns before running DTW

## 7.2 Tables to load
- `trial_qc.csv`
- `session_qc.csv`
- `trial_features.csv`
- `prompt_position_features.csv`
- `participant_keyboard_profile.csv`
- `error_episode_summary.csv`
- optionally `event_sequence_enriched.csv`

## 7.3 Prompt-level descriptive outputs
Create prompt-level summaries such as:
- mean duration
- duration per character
- mean event count
- mean delete count
- mean number of error episodes
- mean maximum wrong-suffix depth
- mean friction
- mean layout-switch count
- near-key typo count
- layout-confusion count
- shift-confusion count

### Recommended plots
1. bar chart of mean duration by prompt
2. bar chart of duration-per-character by prompt
3. bar chart of mean delete count by prompt
4. bar chart of mean number of error episodes by prompt
5. bar chart of mean friction by prompt

## 7.4 Participant-level descriptive outputs
Create participant summaries such as:
- mean trial duration
- mean delete count
- mean number of error episodes
- mean friction
- mean time in error state
- typo-type rates

### Recommended plots
1. participant summary bar charts
2. participant x prompt heatmap of duration
3. participant x prompt heatmap of mean friction

## 7.5 Prompt-position descriptive outputs
This is one of the strongest parts of the notebook.

For each prompt:
- build a participant x prompt-position heatmap for `time_to_stability_ms`
- also optionally build:
  - revisions heatmap
  - wrong_first heatmap
  - local friction heatmap

Then compute aggregated per-position curves:
- median `time_to_stability_ms`
- median revisions
- mean wrong_first
- mean local friction

### Why this matters
This is the most interpretable route to answering RQ3 before Matrix Profile.

## 7.6 Keyboard-demand descriptive outputs
Group over:
- expected layout
- expected shift_required
- layout_switch_needed
- char class
- distance bins
- zone
- hand

Compare:
- mean time_to_stability
- wrong_first rate
- revisions
- mean friction
- delete rate
- typo rates

### Recommended plots
1. class-level bar chart of mean time_to_stability
2. shift vs non-shift boxplots or bar charts
3. layout-switch vs non-switch comparison
4. Thai vs English comparison
5. left / center / right zone comparison

## 7.7 Recovery episode descriptive outputs
From `error_episode_summary.csv`:
- episode duration distribution
- max wrong-suffix depth distribution
- pause-before-first-delete distribution
- delete count distribution

Compare across:
- prompt
- participant
- prompt family

This prepares the ground for DTW and clustering in RQ4.

---

## 8. Stage 2: Euclidean baseline analysis

## 8.1 Why Euclidean baseline is needed
The course strongly emphasizes similarity choice. Euclidean distance is the clean baseline when:
- sequences are fixed-length
- lockstep comparison is acceptable
- we want a simple reference before moving to DTW

Use Euclidean only where it makes conceptual sense.

## 8.2 Best uses of Euclidean in this project

### A. Participant profile vectors
Pivot `participant_keyboard_profile.csv` into wide format.
Possible value choices:
- `mean_time_to_stability_ms`
- `mean_friction`
- `wrong_first_rate`
- `delete_rate`

Then compute Euclidean distances between participants.

This is the strongest Euclidean baseline in the project because participant profiles are naturally fixed-length vectors.

### B. Prompt-position curves within the same prompt
For participants on the same prompt:
- compare their `time_to_stability_ms` curves using Euclidean distance
- compare their revisions curves using Euclidean distance

This works because all users on the same prompt share the same prompt length.

### C. Friction waveforms after normalization
You may optionally compare normalized friction waveforms with Euclidean distance as a baseline, but only after:
- within-trial z-normalization or another documented normalization choice
- careful interpretation

This is less reliable than participant-profile Euclidean comparison, but still useful as a baseline before DTW.

## 8.3 Outputs for Euclidean baseline
- participant distance heatmap
- same-prompt participant distance heatmaps
- dendrograms based on Euclidean distances
- nearest-neighbor tables under Euclidean baseline

## 8.4 Interpretation goals
- establish what simple lockstep similarity can already capture
- identify where DTW meaningfully improves over Euclidean
- demonstrate course understanding through baseline-vs-elastic comparison

---

## 9. Stage 3: DTW analysis

## 9.1 Why DTW is needed here
The course notes emphasize that DTW is appropriate when similar behavior occurs at slightly different speeds or alignments.

That is exactly the case in this project:
- one user pauses longer before deleting
- another user deletes immediately
- one user overshoots farther before recovering
- one user reaches a difficult region later in event order

If we compare such sequences point-by-point, Euclidean distance will be overly sensitive to timing offsets.

DTW is therefore the main tool for:
- event-level recovery similarity
- recovery-episode similarity
- cross-user comparison on the same prompt

## 9.2 Recommended DTW objects in this notebook

### A. Event multivariate sequence
Use:
- `log_dt_ms`
- `is_delete`
- `prefix_gain`
- `error_suffix_len_after`
- `geo_burden_planned`
- `layout_switch_needed`

This is the main DTW object for RQ4.

### B. Scalar friction waveform
Use:
- `friction`

Use only as a secondary DTW object, for:
- prompt-level similarity
- illustrative comparisons
- compact single-channel comparisons

### C. Prompt-position curves
Use DTW only when comparing within the same prompt and when there is reason to believe difficulty patterns are shifted rather than strictly aligned.

This should be a secondary option, not the primary use.

## 9.3 DTW design choices

### Use a constrained DTW
Do not start with fully unconstrained DTW.
Use a Sakoe-Chiba band.

### Recommended starting band
Use a radius around:
- 10% to 15% of sequence length for event sequences
- possibly smaller for prompt-position curves

Then optionally test sensitivity with a small grid of band sizes.

### Feature scaling
Before multivariate DTW:
- standardize continuous dimensions
- leave binary variables as 0/1
- document the scaling rule in the notebook

### Trial grouping strategy
Use DTW in these settings:
1. same prompt across different participants
2. same participant across different prompts
3. error-episode windows across all participants
4. selected prompt families or conditions

## 9.4 Main DTW analyses to run

### A. Same-prompt cross-user similarity
For a selected prompt, compare all participants’ event sequences using DTW.

Questions:
- do certain users behave similarly on the same prompt?
- do Thai numeral prompts induce shared recovery styles?

### B. Same-participant cross-prompt similarity
Compare different prompts for the same participant.

Questions:
- does a participant have a stable recovery style across prompts?
- which prompts elicit similar recovery dynamics?

### C. Error-episode DTW
Extract local event windows around error episodes and compare them with DTW.

This is probably the best DTW application in the project.

Window idea:
- a small number of events before error onset
- all events during the episode
- a small number of events after recovery

This lets you compare shapes such as:
- pause then delete
- immediate delete
- deep overshoot then correction
- long hesitation before final recovery

### D. Prompt-family similarity using friction waveforms
Use DTW on normalized friction waveforms to compare prompts or trials at a coarser level.

This is useful for:
- quick illustrative figures
- compact prompt-level clustering

---

## 10. Stage 4: Hierarchical clustering

## 10.1 Why hierarchical clustering
The course presents hierarchical clustering as a strong exploratory tool and a good way to visualize similarity structure through dendrograms.

This is especially suitable here because:
- you want interpretable behavioral families
- you may not know the true number of clusters in advance
- different objects in the project call for different distance matrices

## 10.2 What to cluster

### A. Participants via Euclidean profile vectors
Input:
- pivoted `participant_keyboard_profile.csv`

Distance:
- Euclidean

Goal:
- discover participant fluency styles
- identify groups such as:
  - stable typists
  - shift/layout-sensitive typists
  - typo-prone typists

### B. Trials via DTW event sequences
Input:
- TS Object A

Distance:
- DTW

Goal:
- discover trial-level behavioral families
- compare whether certain prompts cluster together

### C. Error episodes via DTW windows
Input:
- event windows around error episodes

Distance:
- DTW

Goal:
- cluster recovery styles

This is probably the most interpretable clustering result in the notebook.

### D. Same-prompt users via prompt-position curves
Input:
- TS Object C within a fixed prompt

Distance:
- Euclidean or DTW depending on the analysis

Goal:
- identify subgroups of users who struggle in similar regions

## 10.3 Clustering settings
Use hierarchical clustering with:
- average linkage as a robust default
- optionally complete linkage as sensitivity analysis

Do not use too many clustering variants unless they materially improve interpretation.

## 10.4 Outputs
- dendrograms
- clustered heatmaps or reordered heatmaps
- cluster membership tables
- cluster-level summary tables

## 10.5 Cluster interpretation
For each important cluster, summarize:
- prompt composition
- mean duration
- mean friction
- mean wrong-suffix depth
- mean typo rates
- mean pause-before-delete
- dominant keyboard-demand categories

This is essential. Clustering alone is not enough. The report must interpret what the clusters mean.

---

## 11. Stage 5: Matrix Profile on Long 1D Sequences

Stage 5 is implemented inside this notebook. Use `stumpy.stump` only on sequences
long enough to make subsequence mining meaningful — in practice, the long prompts
P12/P13/P14 (127–145 characters, 148–317 events per trial).

### 11.1 Three MP targets

**5A — per-trial friction waveform** (TS Object B, window m=16)
- Run `stump` on each user's `friction` series for a given long prompt.
- Extract top-k motifs (lowest MP distance) and top-k discords (highest MP distance).
- Save per-trial tables; plot raw series + MP profile for three exemplar users
  (fastest, median, slowest by trial duration).
- Aggregate discord event-positions across users using normalized position (0–1)
  to identify hotspots shared across the cohort.

**5B — per-trial `error_suffix_len_after` waveform** (window m=12)
- Run `stump` on each user's `error_suffix_len_after` series.
- Skip trials where `max(error_suffix_len_after) == 0` (no corrections at all).
- Motifs = recurring overshoot-then-repair patterns; discords = unusually deep
  or unusually short wrong-suffix episodes.
- Plot exemplar users chosen by maximum suffix depth (deepest, median, shallowest).

**5C — aggregated prompt-position difficulty curve** (TS Object C aggregated, window m=8)
- For each long prompt, group by `prompt_position_1based` and compute
  `median_time_to_stability_ms`, `mean_revisions`, `mean_wrong_first`.
- Log-transform the median stability curve before running `stump`.
- One MP per prompt (not per user) — this is the *shared* difficulty curve.
- Top discords correspond directly to interpretable character spans: index back
  into the `char_j` column to label each discord with its Thai text.

**5D — discord position histogram**
- For friction (5A), aggregate all per-user discord positions by normalized
  event position and plot a histogram per long prompt to show *where in the trial*
  friction anomalies concentrate.

### 11.2 Window-size rationale

| target | window | reasoning |
|--------|--------|-----------|
| friction (5A) | m=16 | ~16 events ≈ one syllable-level hesitation+repair cycle |
| error_suffix (5B) | m=12 | ~12 events captures onset+peak+recovery of one overshoot |
| agg. curve (5C) | m=8 | ~8 positions ≈ one Thai word boundary segment |

### 11.3 Motif / discord extraction

Use an exclusion zone equal to the window size around any already-selected index.
Extract `top_k=3` motifs and `top_k=3` discords per series.

```python
def extract_motifs_discords(mp_values, k, exclusion_radius):
    # copy mp; iteratively pick argmin/argmax and blank exclusion zone
    ...
```

### 11.4 Participant anonymization

Before Stage 5, map all participant nicknames to `U01..U26` ordered by mean
trial duration (U01 = fastest). Persist the map to:
- `tables/participant_nickname_to_anon_id_map.csv`
- `stage5_matrix_profile/tables/participant_nickname_to_anon_id_map.csv`

All Stage 5 figure filenames, plot labels, and summary tables use `U##` IDs.

### 11.5 Expected outputs

- `stage5_matrix_profile/figures/mp_friction_{P12,P13,P14}_{anon_id}.png` — 9 figs (3 prompts × 3 exemplars)
- `stage5_matrix_profile/figures/mp_error_suffix_{P12,P13,P14}_{anon_id}.png` — up to 9 figs
- `stage5_matrix_profile/figures/mp_aggregated_curve_{P12,P13,P14}.png` — 3 figs
- `stage5_matrix_profile/figures/mp_friction_discord_hist_{P12,P13,P14}.png` — 3 figs
- `stage5_matrix_profile/tables/stage5_friction_mp_motifs_discords.csv`
- `stage5_matrix_profile/tables/stage5_error_suffix_mp_motifs_discords.csv`
- `stage5_matrix_profile/tables/stage5_aggregated_curve_mp_motifs_discords.csv`
- `stage5_matrix_profile/tables/stage5_aggregated_curve_{P12,P13,P14}.csv`

### 11.6 Key results observed

The Stage 5 discords converge tightly with Stage 1 local-difficulty peaks and
Stage 2 same-prompt Euclidean distances, validating the earlier analysis:

| prompt | top discord span | MP distance | dominant cause |
|--------|-----------------|-------------|----------------|
| P14 | `าปีที่ ๖` (pos 94–101) | 0.730 | Thai numeral |
| P14 | `ครือพันธ` (pos 134–141) | 0.568 | named entity |
| P13 | `นเพื่อเป` (pos 103–110) | 0.604 | formal connective |
| P12 | distributed (no single peak) | ≤0.39 | expository prose |

---

## 12. Statistical / modeling layer for RQ2

Although the notebook is mainly time-series focused, include a small transparent modeling block for RQ2.

Recommended approach:
- use `prompt_position_features.csv`
- fit simple interpretable models such as OLS or mixed-effects approximations if feasible

Example targets:
- `time_to_stability_ms`
- `revisions`
- `wrong_first`

Example predictors:
- `distance_from_prev_correct_j`
- `layout_switch_into_j`
- `shift_required_j`
- `char_class_j`
- prompt fixed effects

Purpose:
- quantify whether keyboard-demand variables explain difficulty
- compare against manual linguistic span annotations later
- support narrative interpretation

This should remain transparent and secondary, not over-engineered.

---

## 13. Visualization requirements

## 13.1 General rules
- use **matplotlib**, not seaborn
- save every figure to disk
- also display selected figures inline in the notebook
- use clear filenames
- use consistent figure sizes and font sizes
- make publication-ready labels and titles

## 13.2 Thai text rendering
The notebook must ensure Thai text renders correctly in plots.

### Required behavior
- detect whether a Thai-capable font is available
- if not, install one at runtime where possible
- set matplotlib to use the installed Thai font
- rebuild font cache if necessary
- verify with a small Thai-render test plot

### Recommended fonts
Preferred options:
- Noto Sans Thai
- Sarabun
- TH Sarabun New
- Tahoma
- other Thai-capable system fonts as fallbacks

### Important note
This is necessary because previous notebooks had Thai rendering problems.

## 13.3 Save both image and metadata
For each important figure:
- save PNG
- optionally save PDF if convenient
- keep a structured output directory

Suggested folder:
- `outputs/02_analysis_descriptive_dtw_clustering/figures/`

Suggested filenames:
- `prompt_duration_bar.png`
- `prompt_delete_bar.png`
- `p14_time_to_stability_heatmap.png`
- `participant_profile_mean_friction_heatmap.png`
- `error_episode_dtw_dendrogram.png`

---

## 14. Notebook output structure

```text
outputs/
  02_analysis_descriptive_euclidean_dtw_clustering/
    tables/
      prompt_summary.csv
      participant_summary.csv
      participant_nickname_to_anon_id_map.csv       ← anonymization map
      all_prompt_position_curve_summaries.csv
      keyboard_demand_summary.csv
      same_prompt_euclidean_summary.csv
      same_prompt_dtw_summary.csv
      ...
    figures/
      prompt_mean_duration_ms.png
      participant_prompt_duration_heatmap.png
      keyboard_demand_*.png
      prompt_position_heatmaps_P*.png
      dtw_same_prompt_event_heatmap_P*.png
      euclidean_prompt_position_heatmap_P*.png
      error_episode_dendrogram.png
      participant_profile_euclidean_dendrogram.png
      ...
    distance_matrices/
      euclidean_*.csv
      dtw_*.csv
    cluster_outputs/
      *_cluster_membership.csv
      *_cluster_summary.csv
    logs/
    anonymized_figures/                             ← U## labelled summary figs
      anon_participant_mean_duration.png
      anon_participant_prompt_duration_heatmap.png
      character_class_difficulty.png
      keyboard_demand_compare.png
      long_prompt_aggregated_difficulty_curves.png
      prompt_level_summary.png
      recovery_style_stacked.png
    stage5_matrix_profile/                          ← Stage 5 outputs
      figures/
        mp_friction_P{12,13,14}_{U##}.png
        mp_error_suffix_P{12,13,14}_{U##}.png
        mp_aggregated_curve_P{12,13,14}.png
        mp_friction_discord_hist_P{12,13,14}.png
      tables/
        participant_nickname_to_anon_id_map.csv
        stage5_friction_mp_motifs_discords.csv
        stage5_error_suffix_mp_motifs_discords.csv
        stage5_aggregated_curve_mp_motifs_discords.csv
        stage5_aggregated_curve_P{12,13,14}.csv
      stage5_summary.json
```

---

## 15. Notebook architecture

The notebook (`02_analysis_descriptive_euclidean_dtw_clustering.ipynb`) has 46 cells organized as:

1. title and purpose (markdown)
2. imports and configuration
3. helper functions (save, load, plot utilities)
4. Thai font setup and verification
5. load transformed inputs
6. QC-based filtering
7. **Stage 1** — descriptive prompt-level analysis
8. **Stage 1** — descriptive participant-level analysis
9. **Stage 1** — prompt-position heatmaps and regional curves
10. **Stage 1** — keyboard-demand summaries
11. **Stage 1** — recovery-episode summaries
12. **Stage 2** — Euclidean baseline analyses (participant profiles + same-prompt curves + friction)
13. **Stage 3** — DTW sequence construction
14. **Stage 3** — DTW analyses (same-prompt, same-participant cross-prompt, error-episode windows, friction)
15. **Stage 4** — hierarchical clustering (participants, trials, episodes, prompt-position curves)
16. **RQ2** — transparent OLS modeling block
17. **Stage 5** — participant anonymization + MP helpers + stumpy import
18. **Stage 5A** — per-trial friction waveform MP (P12/P13/P14)
19. **Stage 5B** — per-trial error_suffix_len_after MP (P12/P13/P14)
20. **Stage 5C** — aggregated prompt-position difficulty curve MP (P12/P13/P14)
21. **Stage 5D** — discord position histograms
22. Stage 5 takeaways (markdown)
23. export manifest and run summary
24. conclusions and handoff

---

## 16. Acceptance criteria for the notebook

The notebook is complete only if it:

**Stages 1–4 (existing)**
- [ ] runs top-to-bottom after configuration
- [ ] uses QC filtering explicitly (main trials, `all_core_qc_ok`, `qc_exact_match_ok`)
- [ ] generates descriptive summaries for prompts and participants
- [ ] produces prompt-position heatmaps for all 14 prompts
- [ ] produces participant keyboard-demand profiles or heatmaps
- [ ] computes Euclidean baseline distance matrices (participant profiles + same-prompt curves)
- [ ] computes DTW distance matrices for same-prompt user comparison and error-episode comparison
- [ ] performs hierarchical clustering on participant profile vectors and error-episode DTW distances
- [ ] saves all major figures to disk and renders Thai text correctly

**Stage 5 (Matrix Profile)**
- [ ] anonymizes all participants to `U01..U26` and saves the map CSV
- [ ] runs friction waveform MP (stumpy.stump, m=16) for each trial on P12/P13/P14
- [ ] runs error_suffix_len_after MP (m=12) for each trial on P12/P13/P14
- [ ] runs aggregated-curve MP (m=8) for P12/P13/P14 and saves motif/discord tables
- [ ] plots discord position histograms (normalized event position across users)
- [ ] all Stage 5 figures display inline (via `display_and_save_figure`) and save to disk
- [ ] does **not** implement lower bounds (LB_Keogh) or SAX

---

## 17. Final implementation philosophy

This analysis notebook is the **single main analysis notebook** for the project.
It is not a random collection of experiments.

The progression is deliberate:

1. **Descriptive validity first** — verify that the transformed features behave sensibly
   before running any similarity or mining algorithm.
2. **Euclidean baselines next** — establish simple lockstep reference results; identify
   where DTW meaningfully improves over Euclidean.
3. **DTW where alignment flexibility is needed** — event-level recovery sequences,
   cross-user same-prompt comparison, error-episode windows.
4. **Hierarchical clustering to summarize** — participant fluency groups, recovery-style
   families; always interpret clusters, never just show dendrograms.
5. **Matrix Profile last, on the longest sequences only** — motif and discord discovery
   is the right tool for P12/P13/P14 friction and difficulty curves; applying it to
   short or medium prompts would give unreliable results with too few potential matches.

That design is:
- **method-appropriate**: each stage uses the tool that fits the data and question
- **interpretable at each step**: every distance matrix, cluster, and MP discord gets
  a narrative explanation, not just a figure
- **progressive**: each stage builds on the previous one rather than branching independently
- **honest about scope**: lower-bound pruning and SAX are not implemented because they do
  not add interpretability for a 26-participant study at this scale

The companion document `Analysis_and_Discussion.md` provides the full narrative
answering RQ1–RQ4, with anonymized (`U##`) plots and tables.
