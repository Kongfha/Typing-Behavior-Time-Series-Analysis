# Thai Typing Time-Series Mining - Comprehensive Project Context

## Purpose of this file

This file is the authoritative LLM-ready project context for downstream coding, data preparation, feature engineering, quality control, and analysis.

It is intended to be used by an LLM or another analysis agent that will:
- load the collected browser export files
- validate whether each session is usable
- transform raw edit logs into event-aligned and position-aligned analytic tables
- build time-series objects for DTW, Matrix Profile, clustering, visualization, and interpretation
- write analyses and reports without drifting away from the actual project design

This file reflects the final current state of the project after prompt design, website validation, pilot checking, and launch-readiness review.

---

## 1. Project identity

### Project title
Thai Typing Time-Series Mining

### One-sentence summary
This project treats Thai prompt-copying as a correction-driven edit-event time series under append-only typing and pop-only deletion constraints, then analyzes the reconstructed behavior with target alignment, keyboard-aware metadata, DTW, Matrix Profile, clustering, and visual analytics.

### Core methodological stance
This project is **not** a raw physical keystroke biometrics study.

The browser website does not log keydown/keyup timing, IME internals, or true finger trajectories. Instead, it logs timestamped text-diff events under a constrained interface. The project intentionally embraces this and models the session as a structured correction process.

### Core structural idea
Because the interface is constrained to append-only typing and pop-only deletion, the typed state at any event can be interpreted as:

```text
current_text = correct_prefix + wrong_suffix
```

This is the key simplification that makes the browser log mineable as a time series.

---

## 2. Motivation and why this project matters

A weaker version of the project would only compare final accuracy, typing speed, and total correction counts.

Another weak version would incorrectly claim that a browser diff log is equivalent to full keystroke biometrics.

This project avoids both problems.

Instead, it asks a stronger question:

> Can constrained browser edit logs reveal meaningful sequential structure about hesitation, local difficulty, correction behavior, and recovery style in Thai typing?

The project matters because it turns a realistic web-collected dataset into a well-defined time-series mining problem with clear interpretability. It is not only about whether people type fast or slow. It is about:
- where they struggle
- what kinds of prompt regions create friction
- how errors accumulate and get repaired
- whether multiple people struggle at the same locations
- whether different participants exhibit recognizable correction styles

---

## 3. Research questions

### RQ1. Character- and keyboard-demand-level fluency
Do participants show different effective fluency across Thai character groups and keyboard-demand conditions such as Thai script versus English layout, shift-required characters, Arabic versus Thai numerals, punctuation spans, and other keyboard-related burden patterns?

### RQ2. Linguistic friction versus keyboard-layout friction
When hesitation and correction occur, are they more strongly associated with linguistic burden such as formal wording, named entities, expectation-violating phrases, and orthographic complexity, or with keyboard-related burden such as shift usage, layout switching, symbol spans, numeral spans, and high key-distance transitions?

### RQ3. Shared difficulty regions across users
For the same prompt, do multiple users struggle at the same character positions or local spans? Can those shared regions be recovered from prompt-position stability statistics, friction traces, and anomaly-style subsequence analysis?

### RQ4. Recovery style as a behavioral signature
Do users exhibit different correction styles after mistakes, such as pause-then-delete, long wrong-suffix accumulation, repeated burst correction, or smooth quick repair?

---

## 4. Final project framing

### Problem type
The final project is primarily a time-series mining and sequential pattern analysis project.

It includes:
- descriptive statistics
- similarity analysis using DTW
- subsequence analysis with Matrix Profile or Matrix-Profile-style logic
- region-level difficulty analysis
- participant-level profiling and clustering

### Key analysis objects
The project does not rely on a single time series. It creates several linked analytic objects from the same raw event log:

1. **Event-aligned multivariate edit sequence**
2. **Scalar friction waveform**
3. **Prompt-position difficulty / stability sequence**
4. **Participant keyboard-demand profile**

These objects support different methods and different interpretations.

---

## 5. What has already been done

### 5.1 Website design has been finalized
The browser task has already been designed and tested.

Current interaction assumptions:
- one prompt at a time
- append-only typing
- pop-only deletion
- no arbitrary cursor repositioning
- no in-line editing
- no paste
- one practice prompt followed by 14 main prompts

### 5.2 Prompt inventory has been finalized
The final prompt set and display order are already decided and should be treated as locked unless there is a major methodological reason to change them.

### 5.3 Pilot validation has already been performed
The website logs were manually checked and validated against the intended interface assumptions.

The key conclusion from pilot validation is that the current logging design is sufficient for the project.

### 5.4 Data collection has already started
A pooled raw event log file already exists:
- `combined_edit_event_logs_data.csv`

Current pooled snapshot available in the working directory:
- sessions: 26
- participants: 26
- distinct trials: 390
- edit events: 30930

This indicates that the project has moved beyond prototype design and already has real collected data.

### 5.5 Launch readiness has already been checked
The current conclusion is that the website is launch-ready for volunteer collection, subject to standard quality control on collected sessions.

---

## 6. Final study design

### Study mode
Browser-only export, no backend storage.

### Participant scale
Exploratory volunteer collection.

The project is not trying to make large population-level causal claims. The intended contribution is strong methodology, interpretable sequential analysis, and meaningful insight.

### Prompt structure
- 1 practice prompt
- 14 main prompts
- short, medium, and long prompts
- deliberate contrasts between familiar language, formal language, numerals, mixed-layout spans, and long-form texts

---

## 7. Final prompt order and purpose

The final main prompt order is:

| prompt_id   |   prompt_typing_order | length_class   | condition                    | purpose                                                                      | expected_typing_behavior                         | prompt_text                                                                                                         |
|:------------|----------------------:|:---------------|:-----------------------------|:-----------------------------------------------------------------------------|:-------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------|
| P01         |                     1 | short          | familiar_baseline            | baseline familiar Thai                                                       | smooth typing, low hesitation                    | วันนี้เราจะทำการบ้านให้เสร็จ                                                                                              |
| P02         |                     2 | short          | familiar_polite              | baseline polite everyday Thai                                                | mostly smooth typing, low hesitation             | กรุณาตอบกลับภายในวันนี้                                                                                                  |
| P04         |                     3 | short          | orthographic_named_entity    | test long proper noun typing difficulty                                      | pause spikes and possible corrections            | นิสิตจากจุฬาลงกรณ์มหาวิทยาลัย                                                                                             |
| P03         |                     4 | short          | formal_register              | test formal bureaucratic wording                                             | slightly slower rhythm, higher first-key latency | ขอความกรุณาจัดทำเอกสารดังกล่าว                                                                                          |
| P05         |                     5 | short          | twisted_proverb              | test expectation violation in proverb                                        | local hesitation near the altered phrase         | เข้าเมืองตาหลิ่ว ต้องหลิ่วมาตาม                                                                                            |
| P09         |                     6 | medium         | arabic_numerals              | baseline numeral form with Arabic digits                                     | relatively smooth typing for many users          | ชั้นมัธยมศึกษาปีที่ 6 จะส่งรายงานจำนวน 10 หน้า                                                                               |
| P11         |                     7 | medium         | rhythmic_repetition          | test repetition and tongue-twister-like rhythmic structure                   | possible bursty timing and local re-typing       | วันนี้ผมทักผมรักผมรอ วันไหนผมท้อผมพอผมลา                                                                                   |
| P10         |                     8 | medium         | thai_numerals                | test Thai numeral and formal writing difficulty                              | slower rhythm around Thai numerals               | ชั้นมัธยมศึกษาปีที่ ๖ จะส่งรายงานจำนวน ๑๐ หน้า                                                                               |
| P06         |                     9 | medium         | mixed_symbols_identification | test abbreviation, digits, slash, and hyphenated name field                  | local hesitation around ม.5/6 and ชื่อ-สกุล span    | นักเรียน ม.5/6 ช่วยตรวจสอบ ชื่อ-สกุล ของตัวเองด้วยนะ                                                                        |
| P08         |                    10 | medium         | formal_lexical_difficulty    | test formal vocabulary and low-frequency wording                             | hesitation around difficult lexical items        | ขอความร่วมมือจัดเตรียมเอกสารเพื่อเป็นวิทยาทานแก่นักเรียน                                                                       |
| P07         |                    11 | medium         | formal_educational           | compare plain versus formal educational register                             | longer pauses around formal words                | นักเรียนพึงศึกษาเล่าเรียนอย่างสม่ำเสมอเพื่อเตรียมความพร้อมในการสอบ                                                              |
| P12         |                    12 | long           | long_familiar_expository     | long readable baseline for localization analysis                             | mostly smooth with minor pauses at long words    | วันนี้ครูขอให้นักเรียนเขียนสรุปเรื่องสุภาษิตไทยอย่างกระชับและชัดเจน โดยให้ยกตัวอย่างหนึ่งสำนวนและอธิบายความหมายด้วยภาษาของตนเอง            |
| P14         |                    13 | long           | long_mixed_difficulty        | test mixed difficulty including proper noun, formal words, and Thai numerals | multiple local hesitation zones                  | สวัสดีคุณชาคริษฐ์ วันนี้เราขอให้คุณจัดทำหนังสือเรื่องการสอนภาษาไทยสำหรับนักเรียนชั้นมัธยมศึกษาปีที่ ๖ เพื่อเป็นวิทยาทานแก่โรงเรียนในเครือพันธมิตร     |
| P13         |                    14 | long           | long_formal_bureaucratic     | test sustained formal-register friction                                      | overall slower typing and diffuse hesitation     | ทางคณะผู้จัดทำขอความกรุณาให้นักเรียนจัดทำรายงานเรื่องสุภาษิตและคำพังเพย โดยเขียนอธิบายเนื้อหาให้ครบถ้วนเพื่อเป็นประโยชน์ต่อการเรียนรู้ในภายหลัง |

### Prompt design rationale
The prompt set is designed to contain:
- short familiar prompts for baseline behavior
- formal / lexical prompts for linguistic burden
- Arabic vs Thai numeral contrast
- a rhythmic repetition prompt
- a mixed-symbol prompt with English-layout punctuation and digits
- long prompts for subsequence-level difficulty localization and time-series mining

### Important prompt handling rule
Prompt strings must be treated as exact Unicode strings.

Do not:
- trim internal spaces
- normalize away punctuation
- collapse spaces
- replace Thai numerals with Arabic numerals

In particular, preserve the exact spacing in **P06**.

---

## 8. Current data files and schema

Each export contains three main CSV files.

### 8.1 `participant_session*.csv`
One row per session.

Role:
- session metadata
- device/context reporting
- high-level quality control

Expected fields include:
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

### 8.2 `trial_log*.csv`
One row per completed trial, including practice.

Role:
- trial-level descriptive analysis
- duration, speed, and correction summaries
- validation target for event reconstruction

### 8.3 `edit_event_log*.csv`
One row per textarea input event.

Role:
- raw behavioral sequence
- text reconstruction source
- main foundation for time-series feature extraction

### 8.4 Pooled raw event file
The working directory also contains a pooled file:
- `combined_edit_event_logs_data.csv`

This should be treated as an already-combined multi-session event table. It is useful for bulk transformation once the per-session logic is stable.

---

## 9. Data interpretation assumptions

### 9.1 The project interprets the log as a correction process
At each event, the participant is doing one of the following:
- advancing correctly along the prompt
- inserting an incorrect continuation
- deleting a wrong suffix
- recovering to the correct path

### 9.2 No claim of physical finger tracking
The project must not claim:
- direct finger motion
- biomechanically exact effort
- raw keystroke biometrics

All keyboard-related quantities are **proxies** based on key identity, layout state, and supplied distance information.

### 9.3 Space handling
Space should be treated as a valid prompt position and a valid typed character, but it usually should not create its own forced layout switch burden.

---

## 10. Keyboard modeling decision for the final version

### Earlier idea that is now removed
The project previously discussed using explicit `(x, y)` key coordinates.

### Final decision
The final version will **not** use `(x, y)` coordinates.

Instead, keyboard geometry will be represented through:
- a **character map table**
- a **keyboard distance matrix / adjacency matrix** provided separately

This is now the authoritative design.

### Why this is good
This keeps the project simpler and more robust. It also avoids pretending that coarse hand-made geometry coordinates are more precise than they really are.

The final geometry logic should therefore operate through:
- physical key identity
- layout state (TH / EN / COMMON)
- shift requirement
- supplied pairwise key distance / adjacency values

---

## 11. What the final transformation pipeline is supposed to produce

The downstream transformation and analysis pipeline should produce at least four major outputs:

1. **Event-level enriched sequence table**
2. **Prompt-position feature table**
3. **Trial-level summary feature table**
4. **Participant keyboard-demand profile table**

From these tables, the project will construct:
- event-aligned multivariate sequences
- friction waveforms
- prompt-position difficulty sequences
- participant profile vectors

---

## 12. What we are doing next

The next major stage is not website redesign anymore.

The next stage is:
- cleaning and validating collected sessions
- transforming raw event logs into analytic tables
- attaching character map metadata and distance-matrix-based burden features
- building time-series objects
- running descriptive, similarity, anomaly-style, and visualization analyses

This means the project has entered the **data preparation and time-series transformation phase**.

---

## 13. Intended downstream analyses

The project intends to run analyses such as:
- prompt-level speed and correction comparison
- event-level friction waveform visualization
- DTW comparison between trials or participants
- prompt-position stability heatmaps
- subsequence discord / motif exploration on long prompts
- participant clustering using keyboard-demand profiles
- interpretation of whether difficulty appears more linguistic, more keyboard-driven, or mixed

---

## 14. What counts as a successful final project result

A successful final result does **not** require claiming full biometrics or building a complex black-box model.

A successful result should show convincingly that:
- the browser diff log can be reconstructed and validated
- the edit stream can be converted into meaningful sequential representations
- the resulting time-series objects support real time-series mining methods
- the discovered difficult regions and recovery behaviors are interpretable
- the analysis says something useful about Thai typing behavior

---

## 15. Important limitations to state clearly

1. The data are not raw keystroke biometrics.
2. Keyboard burden is modeled through proxies, not direct finger measurement.
3. Layout-switch assumptions are rule-based.
4. Some characters may exist on more than one layout/key-state combination, so the project must use explicit mapping rules.
5. The project is exploratory and interpretation-heavy rather than population-causal.

---

## 16. Non-negotiable implementation rules for downstream LLMs

Any downstream transformation or analysis code should obey the following rules:

1. Treat prompt text as exact Unicode text.
2. Preserve the deployed prompt order and prompt metadata.
3. Respect the append-only / pop-only interpretation unless a session is flagged as invalid.
4. Use the character map table plus the supplied distance matrix for keyboard burden.
5. Do not reintroduce `(x, y)` geometry.
6. Do not claim raw keystroke biometrics.
7. Separate **planned prompt-demand burden** from **realized error proximity burden** whenever possible.
8. Keep intermediate tables, not only final plots or aggregate results.

---

## 17. Companion files that should be read together with this one

This project context should be paired with:
- `data_preparation_and_time_series_transformation.md`
- `character_map_table_comprehensive.csv`
- the final prompt CSV
- the collected session / trial / edit log CSV exports
- the future keyboard distance matrix file that will be provided separately

---

## 18. Final summary

This is a constrained browser-based Thai typing time-series mining project.

The website and prompt set are already finalized. Pilot validation has already shown that the log is sufficient and the interface assumptions are usable. The project now moves into pooled-data preparation, target-aligned event reconstruction, keyboard-demand feature engineering using a supplied distance matrix, and downstream time-series analysis.

The central intellectual move of the project is not a fancy model. It is the transformation of browser diff logs into well-defined, interpretable time-series objects.
