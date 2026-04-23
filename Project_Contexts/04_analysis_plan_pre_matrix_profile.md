# Thai Typing Time-Series Mining Project
## Detailed Analysis Plan for Descriptive Statistics, Euclidean Baselines, DTW, and Hierarchical Clustering

## 1. Purpose of this document

This document specifies the **next analysis notebook** after the transformation notebook.

The transformation stage is already complete. The project now has:
- event-level enriched sequential data
- prompt-position feature tables
- trial-level summaries
- participant-level keyboard-demand profiles
- quality-control tables
- sequence-ready tables for downstream time-series mining

This analysis plan covers the following stages only:

1. **Descriptive and sanity analysis**
2. **Euclidean baseline analysis**
3. **DTW-based similarity analysis**
4. **Hierarchical clustering**

This document **does not** include:
- Matrix Profile analysis
- lower-bound acceleration experiments
- SAX or symbolic analysis

Matrix Profile will be handled in a **separate notebook** because it is conceptually cleaner and is better treated as its own focused subsequence-mining stage.

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

### 3.2 Explicitly excluded from this notebook

Do **not** implement here:
- Stage 5 Matrix Profile
- lower-bound pruning experiments such as LB_Keogh
- SAX or symbolic analysis

Those should remain outside this notebook.

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
The strongest answer in this notebook is likely to come from:
- prompt-position heatmaps
- aggregated per-position difficulty curves

Matrix Profile on the aggregated curves will be handled later in the separate notebook.

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

Do **not** use it for Matrix Profile in this notebook.

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
- shared difficulty heatmaps
- within-prompt user comparison
- regional difficulty curves
- Euclidean baseline or DTW within the same prompt if needed

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

## 11. Stage 5 reserved for another notebook

Matrix Profile analysis should be implemented separately.

That later notebook should focus on:
- long-prompt friction waveforms
- long-prompt wrong-suffix sequences
- aggregated prompt-position difficulty curves
- motif and discord discovery
- comparison of discord windows with manual difficult-span annotations

This notebook should only prepare the stage by:
- identifying which prompts and regions look strongest
- saving clean inputs for later use

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

## 14. Suggested notebook output structure

Create a clean output directory such as:

```text
outputs/
  02_analysis_descriptive_dtw_clustering/
    tables/
    figures/
    distance_matrices/
    cluster_outputs/
    logs/
```

### Suggested saved tables
- prompt summary table
- participant summary table
- same-prompt aggregated position curves
- Euclidean distance matrices
- DTW distance matrices
- cluster assignment tables
- cluster summary tables

### Suggested saved objects
- serialized dictionaries or arrays for DTW-ready objects if helpful

---

## 15. Suggested notebook architecture

The notebook should have these sections:

1. title and purpose
2. imports and configuration
3. Thai font setup and verification
4. load transformed inputs
5. QC-based filtering
6. descriptive prompt-level analysis
7. descriptive participant-level analysis
8. prompt-position heatmaps and regional curves
9. keyboard-demand summaries
10. recovery-episode summaries
11. Euclidean baseline analyses
12. DTW sequence construction
13. DTW analyses by research question
14. hierarchical clustering analyses
15. small transparent modeling block for RQ2
16. export tables, figures, and distance matrices
17. conclusions and handoff to Matrix Profile notebook

---

## 16. Acceptance criteria for the notebook

The notebook should be considered complete only if it:

- runs top-to-bottom after configuration
- uses QC filtering explicitly
- generates descriptive summaries for prompts and participants
- produces prompt-position heatmaps
- produces participant keyboard-demand profiles or heatmaps
- computes Euclidean baseline distance matrices
- computes DTW distance matrices for at least:
  - same-prompt user comparison
  - error-episode comparison
- performs hierarchical clustering on:
  - participant profile vectors
  - error-episode DTW distances
- saves all major figures to disk
- successfully renders Thai text in the saved plots
- does **not** include Matrix Profile, lower bounds, or SAX

---

## 17. Final implementation philosophy

This analysis notebook should not be treated as a random collection of experiments.

It should be the **main project analysis notebook before Matrix Profile**, with a clear structure:

- first establish descriptive validity
- then establish simple Euclidean baselines
- then use DTW where alignment flexibility is actually needed
- then use hierarchical clustering to summarize behavioral families
- finally prepare the cleanest outputs for the separate Matrix Profile notebook

That design is:
- aligned with the transformed data you already have
- aligned with the project research questions
- aligned with the course emphasis on thoughtful use of similarity, clustering, and interpretation
- cleaner and more defendable than mixing every method into one giant notebook
