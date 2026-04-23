# Thai Typing Time-Series Mining — Evaluation Plan

## 1. Purpose of evaluation

This project is an **exploratory time-series mining** study rather than a pure supervised prediction task.  
Therefore, evaluation should not focus only on one accuracy number. Instead, it should show that the transformed signals and mining methods are:

1. **trustworthy after preprocessing and reconstruction**,  
2. **able to reveal interpretable structure**,  
3. **better than simpler baselines when the task requires it**, and  
4. **reasonably stable under sensible analysis choices**.

Accordingly, the evaluation design for this project is organized around:

- data validity,
- descriptive difficulty metrics,
- Euclidean vs DTW comparison,
- clustering quality and stability,
- region-level validity of expected difficult spans,
- Matrix Profile sensitivity and interpretability,
- regression fit and uncertainty.

This section should appear **after Methodology Overview** and **before Dataset at a Glance**, so the reader understands how each method will be judged before seeing the results.

---

## 2. Evaluation design overview

| Component | Evaluation question | Metrics / evidence |
|---|---|---|
| QC + preprocessing | Are the reconstructed time series trustworthy? | exact-match reconstruction rate, retained/excluded trials, event constraint violations, core QC pass rate |
| Descriptive difficulty analysis | Are difficulty differences meaningful and interpretable? | median time-to-stability, wrong-first rate, revision count, prompt-level summaries |
| Euclidean baseline | What structure is visible without temporal warping? | top-1 / top-3 same-prompt retrieval accuracy, within-prompt vs between-prompt distance ratio, same-prompt heatmap coherence |
| DTW | Does temporal warping reveal more meaningful behavioral similarity than Euclidean? | top-1 / top-3 same-prompt retrieval accuracy under DTW, Euclidean-vs-DTW neighbor overlap, constrained-band sensitivity |
| Clustering | Are clusters meaningful and stable? | silhouette score where appropriate, cophenetic correlation, cluster-size balance, cluster feature summaries, bootstrap stability |
| Region-level validity | Do transformed signals become high at predefined difficult spans? | inside-vs-outside friction difference, inside-vs-outside time-to-stability difference, span rank among same-length windows |
| Matrix Profile | Do motifs and discords identify interpretable local regions? | overlap with local difficulty peaks and predefined spans, top-k motif/discord interpretability, window-size sensitivity |
| RQ2 modeling | Are keyboard-demand and linguistic effects distinguishable under cautious inference? | coefficient direction, effect size, confidence intervals, model fit statistics, clustered or mixed-effects uncertainty |

---

## 3. Preprocessing validity

### 3.1 Goal

Before any time-series mining is performed, we must verify that the browser logs can be transformed into reliable sequential objects.

### 3.2 Main validity checks

The preprocessing and transformation pipeline should report:

- number of sessions collected,
- number of main trials retained for analysis,
- number of practice trials excluded from main-trial analysis,
- exact-match reconstruction rate,
- append/pop constraint validity,
- monotonic timestamp validity,
- final reconstruction validity,
- overall core QC pass rate.

### 3.3 Interpretation

Only trials that pass the core QC checks are used for downstream analysis.  
This ensures that the final time-series objects are built from valid completed prompt-copying behavior rather than corrupted or structurally inconsistent logs.

This is the first evaluation layer because every later metric depends on the integrity of the event reconstruction.

---

## 4. Descriptive difficulty metrics

### 4.1 Goal

For RQ1 and RQ3, the project needs interpretable measures of difficulty at both the trial level and the prompt-position level.

### 4.2 Main descriptive metrics

The main position-level metrics are:

- **`time_to_stability_ms`**  
  How long it takes until a prompt position becomes correct and remains correct until the end of the trial.

- **`wrong_first`**  
  Whether the first attempt at that position was incorrect.

- **`revisions`**  
  How many later edits affected that prompt position before the final correct outcome.

The main trial-level metrics are:

- trial duration,
- first-key latency,
- delete count,
- number of error episodes,
- maximum wrong-suffix depth,
- mean friction.

### 4.3 Why these metrics matter

These metrics separate different kinds of difficulty:

- **slow but accurate** behavior,
- **fast but error-prone** behavior,
- repeated revision behavior,
- deep overshoot versus shallow correction.

This is especially important for contrasts such as:

- Thai numerals,
- named entities,
- punctuation or layout-switch regions,
- long formal prompts.

---

## 5. Euclidean vs DTW evaluation

### 5.1 Goal

Because the project uses both **Euclidean distance** and **DTW**, evaluation should show when temporal warping provides a meaningful advantage over the lockstep baseline.

### 5.2 Main comparison metrics

For the selected time-series objects, report:

- **Top-1 same-prompt retrieval accuracy**  
  For each trial, find its nearest neighbor under Euclidean distance and under DTW.  
  Check whether the nearest neighbor comes from the same prompt.

- **Top-3 same-prompt retrieval accuracy**  
  Check whether at least one of the three nearest neighbors comes from the same prompt.

- **Within-prompt vs between-prompt distance ratio**  
  Compare the mean distance between trials of the same prompt against the mean distance between trials of different prompts.

- **Euclidean-vs-DTW neighbor overlap**  
  Measure how often Euclidean and DTW produce the same nearest neighbors.

- **Constrained-band sensitivity**  
  Check whether DTW conclusions remain similar when the Sakoe-Chiba band is changed within a reasonable range.

### 5.3 Interpretation

If DTW improves same-prompt retrieval or produces more interpretable neighbors than Euclidean, that supports using DTW for event-aligned behavioral traces and recovery-style similarity.

If Euclidean performs similarly on some fixed-length objects, that supports keeping Euclidean as a simpler baseline where lockstep comparison is already adequate.

---

## 6. Clustering quality and stability

### 6.1 Goal

When participant groups, prompt groups, or recovery-style groups are reported, the project should show that these clusters are not arbitrary or unstable artifacts.

### 6.2 Main clustering metrics

Depending on the representation and distance type, report:

- **Silhouette score**  
  For fixed-length vector representations where silhouette is appropriate.

- **Cophenetic correlation coefficient**  
  For hierarchical clustering, to measure how well the dendrogram preserves the original distance structure.

- **Cluster-size balance**  
  To detect degenerate solutions such as one-point or nearly empty clusters.

- **Feature summaries by cluster**  
  To confirm that clusters differ in behaviorally meaningful ways.

- **Bootstrap stability**  
  Resample trials or participants, rerun clustering, and check whether cluster memberships remain similar.

### 6.3 Interpretation

A good clustering result should satisfy two conditions:

1. clusters should differ in interpretable behavioral features, and  
2. the broad grouping should remain stable under reasonable re-sampling or linkage variation.

### 6.4 RQ4 caveat

If recovery-style clusters are imbalanced or dominated by one style, that should be stated explicitly.  
This kind of honesty improves credibility and is preferable to overstating a weaker clustering result.

---

## 7. Region-level validity of predefined difficult spans

### 7.1 Goal

One of the most project-specific evaluation questions is whether the transformed signals become high at prompt regions that were **expected in advance** to be difficult.

Examples of predefined difficult spans may include:

- Thai numeral spans such as `ปีที่ ๖`,
- named-entity spans such as `ชาคริษฐ์`,
- formal bureaucratic spans such as `เพื่อเป็นประโยชน์`.

These spans must be treated as **evaluation annotations only**.  
They should not be used as model input or as part of the feature-engineering pipeline.

### 7.2 Main region-level metrics

For each annotated span, compute:

#### A. Inside vs outside mean friction

\[
\Delta_f = \overline{f}_{inside} - \overline{f}_{outside}
\]

where:

- \( \overline{f}_{inside} \) = mean friction inside the annotated span,
- \( \overline{f}_{outside} \) = mean friction outside the span.

#### B. Inside vs outside mean time-to-stability

\[
\Delta_s = \overline{t}_{inside} - \overline{t}_{outside}
\]

where:

- \( \overline{t}_{inside} \) = mean time-to-stability inside the span,
- \( \overline{t}_{outside} \) = mean time-to-stability outside the span.

#### C. Rank of the annotated span among same-length windows

Within the same prompt, slide a window of equal length across all positions and rank all windows by:

- mean friction,
- or mean time-to-stability.

Record where the annotated difficult span ranks among all windows of the same length.

### 7.3 Interpretation

These metrics answer a direct construct-validity question:

> Do the transformed behavioral signals become unusually high at the regions that were expected to be difficult?

This is a necessary evaluation block because the project is not only about similarity or clustering.  
It is also about whether the pipeline can **localize interpretable difficulty regions**.

---

## 8. Matrix Profile evaluation

### 8.1 Goal

Matrix Profile results should be evaluated by **interpretability** and **stability**, not by generic prediction accuracy.

### 8.2 Main evaluation checks

For long-prompt Matrix Profile analyses, report:

- whether top discords overlap with:
  - high `wrong_first` regions,
  - high revision regions,
  - high local time-to-stability regions,
  - predefined difficult spans;

- whether motifs correspond to ordinary repeated typing or recovery patterns;

- whether the main discord spans remain similar under nearby subsequence window lengths.

### 8.3 Window-size sensitivity

Because Matrix Profile depends on subsequence length \(m\), evaluate sensitivity using:

- a default window,
- a smaller window (for example, \(-20\%\)),
- a larger window (for example, \(+20\%\)).

Then compare:

- top-k discord overlap,
- rank stability of the main discord regions,
- whether the same annotated difficult spans remain near the top.

### 8.4 Interpretation

If the same regions remain prominent across nearby window sizes, then the Matrix Profile findings are more credible.  
If the discords change completely under small changes in \(m\), conclusions should be reported more cautiously.

---

## 9. Modeling metrics for RQ2

### 9.1 Goal

The RQ2 models are intended as **transparent association models**, not causal models.

The purpose is to compare whether keyboard-demand and linguistic factors are associated with higher difficulty, not to claim that one factor directly causes the other.

### 9.2 Main reporting metrics

For each model, report:

- coefficient estimates,
- confidence intervals,
- p-values where appropriate,
- model fit statistics such as \(R^2\) or pseudo-\(R^2\),
- clustered standard errors or mixed-effects uncertainty if feasible.

### 9.3 Statistical caution

Prompt positions are nested within both:

- participants,
- and prompts.

Therefore, rows are not independent.  
Interpretation should use cautious wording such as:

- “associated with higher time-to-stability,”
- “associated with higher wrong-first rate,”
- “shows a larger coefficient under the current model.”

Avoid causal language such as:

- “explains,”
- “causes,”
- “drives.”

### 9.4 Interpretation

The goal of these models is not maximum predictive power.  
The goal is to show, under a transparent specification, whether keyboard-demand and linguistic effects remain distinguishable and directionally consistent.

---

## 10. Mapping research questions to metrics

| RQ | Main metrics |
|---|---|
| **RQ1** Character- and zone-level fluency | median time-to-stability, wrong-first rate, revision count by character class and keyboard-demand group |
| **RQ2** Linguistic vs keyboard friction | regression coefficients, confidence intervals, p-values, model fit, clustered or mixed-effects uncertainty |
| **RQ3** Shared difficulty regions across users | same-prompt heatmap agreement, local difficulty peaks, region-level validity of predefined difficult spans, Matrix Profile discord overlap |
| **RQ4** Recovery style as a behavioral signature | recovery-style counts, DTW cluster summaries, cluster balance, bootstrap stability, representative motif/discord examples |

---

## 11. Summary of evaluation logic

In summary, the project is evaluated along six main dimensions:

1. **Preprocessing validity**  
   Are the reconstructed event sequences trustworthy?

2. **Descriptive interpretability**  
   Do the core trial- and position-level metrics reveal meaningful difficulty patterns?

3. **Method comparison**  
   Does DTW reveal useful similarity structure beyond Euclidean lockstep distance?

4. **Cluster robustness**  
   Are participant and recovery-style clusters behaviorally meaningful and reasonably stable?

5. **Region-level validity**  
   Do transformed signals become high at predefined difficult spans?

6. **Model uncertainty and caution**  
   Are reported linguistic and keyboard-demand effects stated as associations with appropriate uncertainty?

This evaluation design fits the nature of the project: it does not treat the work as a single-number benchmark, but as a structured attempt to show that the transformed time-series objects produce **trustworthy, interpretable, and reasonably stable discoveries**.
