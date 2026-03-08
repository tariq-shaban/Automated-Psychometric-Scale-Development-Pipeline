# Automated Psychometric Scale Development Pipeline

**Author:** Tariq Shaban — Inubilum Ltd  
**Version:** Revised March 2026  
**Origin:** Originally developed under the name Project Majdal  
**Contact:** [tariq@inubilum.io](mailto:tariq@inubilum.io) · [GitHub](https://github.com/tariq-shaban) · [LinkedIn](https://www.linkedin.com/in/tariq-shaban-0090ba2/)
---

## Table of Contents

1. [What This Pipeline Does](#1-what-this-pipeline-does)
2. [Background and Academic Basis](#2-background-and-academic-basis)
3. [Pipeline Architecture](#3-pipeline-architecture)
4. [Human Review at Every Stage](#4-human-review-at-every-stage)
5. [What You Need to Install](#5-what-you-need-to-install)
6. [Setting Up Your Environment](#6-setting-up-your-environment)
7. [Configuring Your Scales](#7-configuring-your-scales)
8. [Running the Pipeline](#8-running-the-pipeline)
9. [Phase-by-Phase Guide](#9-phase-by-phase-guide)
10. [Settings Reference](#10-settings-reference)
11. [Troubleshooting](#11-troubleshooting)
12. [Glossary](#12-glossary)
13. [References](#13-references)

---

## 1. What This Pipeline Does

This pipeline is an automated system for the early stages of psychometric scale development. It takes a set of psychological construct definitions and produces a curated, human-review-ready pool of assessment items — questionnaire statements suitable for use in workplace personality assessments.

### The problem it solves

Traditional scale development is slow and expensive. Subject matter experts manually write hundreds of items, psychologists review each one, and items are piloted on large groups before any assessment can be used operationally. The full process can take months and cost tens of thousands of pounds.

This pipeline automates the generation and initial screening of items using Large Language Models (LLMs) and a technique called Pseudo-Factor Analysis (PFA), which simulates how items would behave statistically before any real human data is collected.

### What it produces

At the end of the pipeline you receive a structured Excel file containing a pool of assessment items, each annotated with quality scores, statistical fit estimates, readability grades, bias flags, and recommendations for human review. A qualified psychologist then makes the final decisions on which items to retain, modify, or discard.

### Important caveat

This pipeline produces a **pre-trial item pool**, not a finished assessment. Real human response data and full psychometric validation are required before any items are used operationally. The pipeline replaces weeks of manual drafting and initial screening — it does not replace empirical validation.

---

## 2. Background and Academic Basis

This pipeline was originally conceived and developed as **Project Majdal** by Tariq Shaban (Inubilum Ltd) as a practical implementation of emerging methods in AI-assisted psychometric scale development.

The architecture draws directly on the following published work:

> Guenole, N., Samo, A., & Sun, T. (2024). *Pseudo-discrimination parameters from language embeddings*. Open Science Framework. https://doi.org/10.31234/osf.io/9a4qx

> Guenole, N., D'Urso, E. D., Samo, A., Sun, T., & Haslbeck, J. M. B. (2025). *Enhancing scale development: Pseudo factor analysis of language embedding similarity matrices*. Manuscript in preparation.

> Suárez-Álvarez, J., Fernández-Alonso, R., & Muñiz, J. (2026). Evaluating pseudo-factor analysis as a tool for item selection in psychological test development. *Psicothema*, *38*(1), 1–12.

> Milano, N., Luongo, M., Ponticorvo, M., & Marocco, D. (2025). Semantic analysis of test items through large language model embeddings predicts a-priori factorial structure of personality tests. *Current Research in Behavioral Sciences*, *8*, 100168.

> Lee, J. Y., Li, Y., & Rounds, J. (2025). *AI-powered item generation for psychological tests*. Manuscript submitted for publication.

> Lee, J. Y., Joo, S. H., Antonakis, J., & Rounds, J. (2025). The journey of forced-choice measurement over 80 years: Past, present, and future. *Annual Review of Organizational Psychology and Organizational Behavior*, *12*, 1–30.

---

## 3. Pipeline Architecture

The pipeline is a single Jupyter Notebook. Each phase reads the output of the previous phase and writes a richer Excel file, creating a complete auditable chain from construct definition to human-review-ready item pool.

```
scales.json (input)
    │
    ├── PHASE 1 │ Behavioural Domain Generation   → 01_behavioral_domains.xlsx
    ├── PHASE 2 │ Domain Validation               → 02_domain_validation.xlsx
    ├── PHASE 3 │ Item Generation                 → 03_generated_items.xlsx
    ├── PHASE 4 │ Readability & Bias Analysis     → 04_readability_bias.xlsx
    ├── PHASE 5 │ Content Validity (LLM SMEs)     → 05_content_validity.xlsx
    ├── PHASE 6 │ Pseudo-Factor Analysis          → 06_pseudo_factor_analysis.xlsx
    └── PHASE 7 │ Item Pool Assembly              → 07_item_pool_for_review.xlsx
```

**Phase 1** reads your construct definitions from `scales.json` and generates observable behavioural examples — recurring patterns of behaviour that someone with this trait would exhibit in the target environment. The `occurrence_likelihood` field (low / moderate / high) reflects how frequently the behaviour appears in context, not item difficulty.

**Phase 2** passes each behavioural example to a second LLM for validation against three criteria: construct validity, face validity, and discriminant validity. Failed examples are flagged and excluded from item writing by default.

**Phase 3** turns validated behavioural examples into survey items — first-person "I" statements following Goldberg item-writing principles: maximum 9 words, present tense, plain vocabulary, single idea, no negations. Each item is tightly anchored to its source behavioural domain to prevent the generic cross-domain duplication that occurs when items reflect the construct in general rather than a specific behaviour.

**Phase 4** computes six readability metrics per item. Items above the reading level hard cap are simplified automatically by LLM. Bias pattern detection flags gender, age, cultural, and socioeconomic language for human review.

**Phase 5** has five LLM models acting as independent subject matter experts. Each rates the item's fit to the target construct (1–10) and maps it to the best-matching scale from the full list. Items only proceed if they clear both a minimum mean rating and a minimum mapping agreement threshold.

**Phase 6** is the statistical core. Sentence transformer embeddings serve as a semantic proxy for item inter-correlations, and factor analysis is applied to the resulting similarity matrices — the Pseudo-Factor Analysis (PFA) technique. Two transformer models are run in parallel and their similarity matrices averaged to improve reliability.

**Phase 7** assembles the final review pool. Items that passed content validity and PFA are compiled into a structured Excel file with quality flags, scale-level recommendations, and blank columns for reviewer decisions.

---

## 4. Human Review at Every Stage

Every phase that applies automated pass/fail logic also writes two columns to its output Excel file:

| Column | Default | Purpose |
|---|---|---|
| `human_review_pass` | `True` | Set to `False` to exclude an item before the next phase runs. The next phase reads this as its primary filter. |
| `human_comments` | *(blank)* | Free-text rationale for any override decision. |

The rule is: **the next phase always reads `human_review_pass` from the previous phase's output.** If no human has intervened, all values remain `True` and nothing changes. Setting a value to `False` excludes that item from all subsequent phases. Setting it to `True` on an item the system failed reinstates it.

After running any phase, open the output Excel, review the results, make changes in `human_review_pass`, save the file, and run the next phase — which will automatically respect your decisions.

---

## 5. What You Need to Install

### 5.1 Anaconda

Installs Python and Jupyter Notebook.

**Download:** https://www.anaconda.com/download

1. Download the Windows 64-bit installer and run it
2. Accept all defaults; tick "Add Anaconda to PATH" when asked
3. Verify: open Anaconda Prompt and type `python --version` — expect `Python 3.11.x`

### 5.2 Visual Studio Build Tools

Required by the `factor-analyzer` package, which compiles C++ code during installation. Without this you will see `Microsoft Visual C++ 14.0 or greater is required`.

**Download:** https://visualstudio.microsoft.com/visual-cpp-build-tools/

1. Run the installer and select **Desktop development with C++**
2. Ensure MSVC v143, Windows SDK, and C++ CMake tools are ticked
3. Install (~4–6 GB, 10–20 minutes), then restart your computer

### 5.3 Visual Studio Code

A text editor with JSON error detection. Do not use Notepad for `pipeline_settings.json` — it will not warn you about syntax errors that break the pipeline.

**Download:** https://code.visualstudio.com/

Install the **JSON Language Features** extension from the Extensions panel.

### 5.4 Git (optional)

Only needed if downloading the project from GitHub.

**Download:** https://git-scm.com/download/win — accept all defaults.

---

## 6. Setting Up Your Environment

### 6.1 Install Python packages

Open Anaconda Prompt and run these commands in order:

```bash
pip install jupyter notebook pandas openpyxl tqdm python-dotenv
pip install sentence-transformers
pip install factor-analyzer
pip install openai
pip install scikit-learn scipy numpy
pip install textstat
```

If any command fails with a Visual C++ error, complete Section 5.2, restart Anaconda Prompt, and try again.

### 6.2 Install PyTorch with GPU support

Phase 6 runs significantly faster on a GPU. Check your CUDA version first:

```
nvidia-smi
```

Look for "CUDA Version" in the output, then run the matching command:

```bash
# CUDA 11.8
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118 --force-reinstall

# CUDA 12.1 or 12.2
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121 --force-reinstall

# CUDA 12.4–12.7
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124 --force-reinstall

# No NVIDIA GPU
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu --force-reinstall
```

> A CUDA version mismatch produces no error during installation. It only fails when Phase 6 tries to load PyTorch, producing a `WinError 1114` DLL error. Getting this right upfront avoids the most common Phase 6 failure.

### 6.3 Set up your API key

The pipeline uses OpenRouter to access multiple LLM providers through a single API.

1. Create an account at https://openrouter.ai
2. Go to your profile → API Keys → Create new key
3. Copy the key (begins with `sk-or-`)
4. Copy `env_template` to `.env` in your project folder and add:

```
OPENROUTER_API_KEY=sk-or-your-key-here
```

Keep `.env` private. Do not commit it to version control or share it.

### 6.4 Opening the notebook correctly

Open VS Code with **File → Open Folder** pointing at your project folder — the one containing `majdal_pipeline.ipynb`. This ensures the notebook finds `pipeline_settings.json` and `scales.json` in the same directory.

Do not open the notebook by double-clicking it directly. The working directory will be wrong and the pipeline will fail to find its settings file.

---

## 7. Configuring Your Scales

The pipeline reads construct definitions from `scales.json`. This is the most important configuration step — the quality of your construct definitions determines the quality of everything the pipeline produces.

### Required fields

```json
{
  "scale_name": "Conscientiousness",
  "construct_definition": "The tendency to be organised, diligent, and self-disciplined...",
  "high_anchor": "Works in an organised and systematic way...",
  "low_anchor": "Approaches tasks in an unsystematic way...",
  "measure_type": "Trait",
  "discriminant_description": "Conscientiousness concerns self-regulation, not moral conduct...",
  "target_population": {
    "age_range": "18-65",
    "education_level": "High school graduates or above",
    "environment_context": "Workplace and social environments",
    "cultural_context": "Global business context"
  },
  "target_environment": "General working population",
  "reading_level_target": 8,
  "response_format": "likert_5"
}
```

### Writing effective construct definitions

The `construct_definition` and `discriminant_description` fields are the two most critical inputs. A vague definition produces generic items. A definition that overlaps with another scale produces items that fail content validity and discriminant validity.

Good construct definitions are behaviourally grounded — they describe what someone with this trait actually does, not just what they are. They are specific enough to be distinguished from adjacent constructs, and consistent in level of abstraction across all scales.

The `discriminant_description` tells the LLM what this construct is NOT. For Conscientiousness this would be: "Conscientiousness concerns self-regulation and task performance, not moral conduct (Honesty-Humility) and not intellectual curiosity (Openness to Experience)." This single field does more to prevent cross-construct item contamination than any other configuration setting.

### A validated starting point: HEXACO

A `hexaco_scales.json` file is provided with academically validated definitions for the six HEXACO personality factors — Honesty-Humility, Emotionality, Extraversion, Agreeableness, Conscientiousness, and Openness to Experience — drawn directly from Ashton and Lee (2007). These can be used as-is or as a template for custom scale development.

---

## 8. Running the Pipeline

### The golden rule: always run Section 0 first

Section 0 loads all libraries, detects the working directory, reads settings, initialises the OpenRouter client, and loads scales. Every subsequent phase depends on variables set in Section 0. Run it every time you open the notebook or restart the kernel.

**Symptoms of a missing Section 0:** `NameError: name 'CFG' is not defined`, `NameError: name 'BASE_OUTPUT_DIR' is not defined`.

### Settings reload

Each phase cell begins by reloading `pipeline_settings.json`:

```python
with open(SETTINGS_FILE, "r", encoding="utf-8") as f:
    CFG = json.load(f)
```

This means you can edit settings and re-run a phase without restarting the kernel. Always save the settings file before running a phase.

### Phase order

Run phases strictly in order: 1 → 2 → 3 → 4 → 5 → 6 → 7. Each phase reads the previous phase's output. Phases can be re-run individually when needed.

### Approximate run times

For approximately 24 scales, 5 behavioural examples per scale, 5 items per difficulty level (600 items entering Phase 5), with a GPU:

| Phase | Approximate time | Bottleneck |
|---|---|---|
| Phase 1 | 5–10 minutes | LLM API calls |
| Phase 2 | 10–20 minutes | LLM API calls |
| Phase 3 | 20–60 minutes | LLM API calls |
| Phase 4 | 2–5 minutes | Local computation + LLM simplification |
| Phase 5 | 30–90 minutes | 5 parallel LLM reviewers × all items |
| Phase 6 | 15–40 minutes | Transformer encoding |
| Phase 7 | Under 1 minute | Assembly only |

### Preventing computer sleep

Phases 3 and 5 can run for over an hour. Computer sleep stalls them silently. Phase 5 has a checkpoint/resume system — if it does stall, simply re-run the cell and it resumes. For other phases prevention is the only option.

On Windows: Settings → System → Power & Sleep → set both "turn off" and "sleep" to **Never**.

---

## 9. Phase-by-Phase Guide

### Section 0 — Setup

Section 0 detects the notebook's location using VS Code's `__vsc_ipynb_file__` variable, with fallback to `ipynbname` and then `Path.cwd()`. No hardcoded file paths are needed. It also loads `.env`, validates `pipeline_settings.json`, initialises the OpenRouter client, loads and normalises `scales.json`, and defines shared utilities including `llm_call` and `clean_text`.

---

### Phase 1 — Behavioural Domain Generation

**Input:** `scales.json`  
**Output:** `01_behavioral_domains.xlsx`

For each construct, the LLM generates observable behavioural examples. These describe recurring patterns of behaviour — not one-off scenarios. The `occurrence_likelihood` field (low / moderate / high) reflects how commonly the behaviour occurs in the target environment, not psychometric difficulty.

**What to review:** Open `Behavioral_Domains` and read the examples per scale. They should sound like descriptions of actual workplace behaviour, not restatements of the definition. Generic or theoretical examples will produce generic items downstream.

**Human review columns added:** `human_review_pass` (default `True`), `human_comments`. Phase 2 reads `human_review_pass`.

**Key settings:** `examples_per_construct` (default 5), `temperature` (default 0.7).

---

### Phase 2 — Domain Validation

**Input:** `01_behavioral_domains.xlsx` (respects `human_review_pass`)  
**Output:** `02_domain_validation.xlsx`

A second LLM evaluates each example against construct validity, face validity, and discriminant validity. Failed examples are flagged `process_flag = False`.

**What to review:** Check `Validation_Summary` for pass rates per scale. If a scale has fewer than 3 valid examples, Phase 3 will produce very few items — revise the construct definition or anchors in `scales.json`. Check `validation_issues` on failed examples — a pattern of discriminant validity failures points to a construct boundary problem.

**Human review columns added:** `human_review_pass` (default `True`), `human_comments`. Phase 3 reads `human_review_pass`.

---

### Phase 3 — Item Generation

**Input:** `02_domain_validation.xlsx` (respects `human_review_pass`)  
**Output:** `03_generated_items.xlsx`

Survey items are generated from validated behavioural examples. Each item must be tightly anchored to its specific source domain — not to the construct in general. This domain specificity requirement is enforced explicitly in the prompt with a concrete example contrasting a domain-specific item against a generic one.

Items are generated only for difficulty levels with a count greater than zero. Near-duplicate items are detected by TF-IDF cosine similarity and flagged (not removed) so a reviewer can choose which to keep.

**What to review:** Check `Generation_Summary` for item counts. Review items flagged `duplicate_flag = True` as pairs — keep the one more tightly anchored to its domain. Items that seem too generic or applicable to multiple domains should be excluded via `human_review_pass = False`.

**Human review columns added:** `duplicate_flag`, `duplicate_of`, `duplicate_similarity` (system-set), `human_review_pass` (default `True`), `human_comments`. Phase 4 reads `human_review_pass`.

**Key settings:**

| Setting | Default | Notes |
|---|---|---|
| `items_per_difficulty` | `{"high": 5, "moderate": 5, "low": 5}` | Set to 0 to skip a level |
| `duplicate_removal_threshold` | 0.85 | Cosine similarity threshold for flagging |
| `temperature` | 0.8 | Higher = more lexical variety |

---

### Phase 4 — Readability and Bias Screening

**Input:** `03_generated_items.xlsx` (respects `human_review_pass`)  
**Output:** `04_readability_bias.xlsx`

Six readability metrics are computed per item. Items above `reading_level_hard_cap` are simplified by LLM (up to `max_simplification_attempts` times). The original wording is preserved in `original_item_text`. Bias pattern matching flags gender, age, cultural, and socioeconomic language — items are flagged, not excluded.

**What to review:** Open `Bias_Summary` for scale-level flag counts, then filter `Items_With_Readability` by `bias_flags` to review each flagged item. Most flags on short "I" statements are false positives. Use `human_comments` to record reasoning when accepting a flagged item. Check `above_reading_target = True` items for scales targeting lower-literacy populations.

**Human review columns added:** `system_readability_pass` (system-set, `False` only if item remains above hard cap after all simplification attempts), `human_review_pass` (default `True`), `human_comments`. Phase 5 reads `human_review_pass`.

---

### Phase 5 — Content Validity Review

**Input:** `04_readability_bias.xlsx` (respects `human_review_pass`)  
**Output:** `05_content_validity.xlsx`

Five LLM models act as independent subject matter experts. Each rates the item's target fit (1–10) and maps it to the best-matching scale from the full list. An item passes if both thresholds are met:

- `min_mean_target_rating` ≥ 6.0 — average rating across all five SMEs
- `min_scale_mapping_agreement` ≥ 0.6 — proportion mapping to the correct scale

Progress is checkpointed every 100 items. If Phase 5 is interrupted, re-run the cell — it resumes automatically.

**What to review:** Start with `CV_Scale_Summary` for pass rates per scale. A low pass rate (below 50%) signals a construct definition or discriminant description problem. Check `cv_majority_mapped_scale` on failing items — if they consistently map to a different scale, address the construct boundary before re-running. For borderline passers (rating 6–7), examine `cv_sme_rationale` to understand reviewer reasoning.

**Human review columns added:** `cv_pass` (system-set), `human_review_pass` (default `True`), `human_comments`. Phase 6 reads `human_review_pass`.

**Thresholds:** Default 6.0 / 0.6. Raise to 7.0 / 0.8 for published instruments.

---

### Phase 6 — Pseudo-Factor Analysis

**Input:** `05_content_validity.xlsx` (respects `human_review_pass`)  
**Output:** `06_pseudo_factor_analysis.xlsx`

Phase 6 is the statistical core. Sentence transformer embeddings serve as a semantic proxy for item inter-correlations, and factor analysis is applied to the resulting similarity matrices — Pseudo-Factor Analysis (Guenole et al., 2025). Two transformer models (`all-MiniLM-L6-v2` and `all-mpnet-base-v2`) are used and their cosine similarity matrices averaged before factor analysis, following Guenole et al. (2025).

**Negative item reversal for embedding**

Items describing the absence or avoidance of the construct (e.g., "I avoid planning ahead") are detected via heuristic pattern matching followed by an LLM call for borderline cases, and rewritten to their positive pole for embedding only. The original `item_text` is always preserved. Reversed items are flagged in `item_reversed_for_embedding`.

**Six pass/fail rules**

| Rule | Criterion | Default |
|---|---|---|
| Rule 1 — Strong Single Factor | Atomic loading ≥ threshold | 0.40 |
| Rule 2 — Adequate Single Factor | Atomic loading ≥ threshold | 0.30 |
| Rule 3 — Adequate Communality | Communality ≥ threshold | 0.15 |
| Rule 4 — Clean Factor Structure | Max secondary loading < threshold | 0.35 |
| Rule 5 — Cross-Difficulty Viable | All-items loading ≥ threshold | 0.20 |
| Rule 6 — Within-Difficulty Viable | Within-difficulty loading ≥ threshold | 0.25 |

**HIGH_PASS** requires Rules 1, 3, and 4. **STANDARD_PASS** requires at least `min_rules_to_pass` rules (default 3).

**DAAL factor identity**

In the joint multi-scale factor analysis, factors are labelled using the Dominant Average Absolute Loading (DAAL) approach (Guenole et al., 2025). For each extracted factor, the mean absolute loading of each scale's items on that factor is computed. The scale with the highest such value — the dominant average absolute loading — is assigned as the factor's label. A scale's DAAL identity is confirmed if its items load most strongly on the factor labelled with its name. Where two scales compete for the same factor, it is labelled `Unassigned`.

**Tucker's congruence**

Tucker's congruence coefficient is computed between the atomic and macro loading solutions per scale. Values ≥ 0.95 indicate excellent encoding invariance; ≥ 0.85 indicates fair similarity; < 0.85 suggests the two encoding strategies disagree and the construct definition warrants closer examination (Lorenzo-Seva & ten Berge, 2006).

**Model fit**

RMSR (Root Mean Square Residual) is the only reported fit metric. CFI, TLI, and RMSEA are not computed — they require a known sample size and are unreliable in PFA contexts (Guenole et al., 2025; Suárez-Álvarez et al., 2026). Values below 0.08 indicate acceptable fit; below 0.05 indicates good fit.

**Output sheets**

| Sheet | Contents | When to use |
|---|---|---|
| `PFA_Item_Results` | Full per-item metrics — loadings, rules, discriminant validity, DAAL | Primary item-level review |
| `PFA_Scale_Summary` | Scale-level aggregates, DAAL identity, Tucker's congruence | Start here |
| `Discriminant_Matrix` | Raw FA loadings — all items, all factors in rotation order including Unassigned | Examining precise cross-loading values |
| `Item_Scale_Heatmap` | Cosine similarity of each item vs each scale in `scales.json` order — all items, no DAAL dependency | Visual inspection of construct specificity |
| `Loading_Matrix_All` | FA loadings as item × scale — all Phase 5 items; non-PFA items have projected loadings | FA-based values across the full item pool |
| `Loading_Matrix_CV_Pass` | Same, CV-pass items only, all loadings exact | Preferred over Loading_Matrix_All |
| `Fit_Statistics` | RMSR and residual stats per scale | Check `fit_acceptable` first |
| `Eigenvalues` | Eigenvalue tables per scale | Assessing dimensionality |
| `Tucker_Congruence` | Congruence coefficients per scale | Identifying encoding-unstable scales |
| `High_Pass_Loadings` | HIGH_PASS items only | Quick review of strongest candidates |
| `All_Pass_Loadings` | All trial-ready items | Overview of passing pool |
| `Pass_Rules_Definition` | Threshold documentation | Reference |

**Recommended review sequence:**

1. `Tucker_Congruence` — scales with poor congruence (< 0.85) need attention first
2. `PFA_Scale_Summary` — scales where `daal_identity_confirmed = False` or high `disc_flag_count`
3. `Item_Scale_Heatmap` — visual scan for items whose peak similarity falls on the wrong column
4. `PFA_Item_Results` — filter `discriminant_flag_review = True` and `item_reversed_for_embedding = True` first

**Human review columns added:** `item_reversed_for_embedding` (system-set), `pfa_pass_level` (system-set), `human_review_pass` (default `True`), `human_comments`. Phase 7 reads `human_review_pass`.

---

### Phase 7 — Item Pool Assembly

**Input:** `06_pseudo_factor_analysis.xlsx` (respects `human_review_pass`)  
**Output:** `07_item_pool_for_review.xlsx`

Assembles the final review-ready pool. Items where `human_review_pass` is not explicitly `False` are included in `Review_Pool`. Rejected items are preserved in `Rejected_Items` for audit purposes.

**Output sheets:**

- **`Review_Pool`** — all passing items with full quality metadata. Two blank columns for reviewer decisions.
- **`Rejected_Items`** — excluded items, preserved for transparency.
- **`Pool_Statistics`** — one row per scale: item counts by difficulty, mean CV rating, mean atomic loading, bias flag counts, reversal counts.
- **`Quality_Flags`** — every item with at least one automated concern: readability, bias, low CV rating, mapping failure, standard pass only, discriminant flag, DAAL failure, or reversal. Review these first.
- **`Recommendations`** — scale-level guidance with HIGH / MEDIUM priority: scales with too few items, high mean FK grade, multiple discriminant flags, DAAL failures, and reversal counts.

**Reviewer workflow:**

1. Start with **Recommendations** — resolve HIGH priority items before trialling
2. Work through **Quality_Flags** — review flagged items as batches by flag type
3. Open **Review_Pool** scale by scale:
   - `human_decision`: type `accept`, `modify`, or `reject`
   - `human_notes`: free-text rationale, especially for modifications and reversals
4. Check **Pool_Statistics** — each scale should have sufficient items at each active difficulty level

**Review tips:**

- Sort by `pfa_pass_level` descending to review HIGH_PASS items first
- Filter by `discriminant_flag_review = True` and compare flagged items within the same scale
- Items where `cv_majority_mapped_scale` differs from the target scale often need rewording
- Items where `item_reversed_for_embedding = True` — confirm the semantic direction is positive before accepting
- If a scale has fewer than 15 passing items, re-run Phase 3 for that scale before completing review

---

## 10. Settings Reference

All pipeline behaviour is controlled by `pipeline_settings.json`. Key settings are shown below; the file itself contains full annotated documentation.

### Phase 1

```json
"phase_01_behavioral_domains": {
  "examples_per_construct": 5,
  "model": "anthropic/claude-haiku-4.5",
  "temperature": 0.7,
  "retry_attempts": 3
}
```

### Phase 2

```json
"phase_02_domain_validation": {
  "min_valid_examples_threshold": 5,
  "validation_criteria": ["construct_validity", "face_validity", "content_coverage"],
  "model": "anthropic/claude-haiku-4.5",
  "temperature": 0.3
}
```

### Phase 3

```json
"phase_03_item_generation": {
  "items_per_difficulty": {"high": 5, "moderate": 5, "low": 5},
  "duplicate_removal_threshold": 0.85,
  "model": "anthropic/claude-haiku-4.5",
  "temperature": 0.8,
  "inter_example_delay_seconds": 1
}
```

Set any difficulty level to 0 to skip it entirely.

### Phase 4

```json
"phase_04_readability_bias": {
  "reading_level_target": 8,
  "reading_level_hard_cap": 12,
  "max_simplification_attempts": 3,
  "bias_categories": ["gender", "age", "cultural", "socioeconomic"]
}
```

### Phase 5

```json
"phase_05_content_validity": {
  "pass_thresholds": {
    "min_mean_target_rating": 6,
    "min_scale_mapping_agreement": 0.6
  },
  "sme_max_workers": 5
}
```

Raise to 7.0 / 0.8 for published instruments.

### Phase 6

```json
"phase_06_pseudo_factor_analysis": {
  "transformer_models": ["all-MiniLM-L6-v2", "all-mpnet-base-v2"],
  "min_rules_to_pass": 3,
  "model_fit_thresholds": {"rmsr_max": 0.08},
  "discriminant_validity": {
    "target_to_max_ratio_min": 1.5,
    "max_cross_loading_ceiling": 0.45,
    "ceiling_enabled": true
  },
  "reversal_detection": {
    "model": "anthropic/claude-haiku-4.5",
    "temperature": 0.0
  }
}
```

### Phase 7

```json
"phase_07_item_pool_assembly": {
  "final_pool_target_per_difficulty": 20
}
```

---

## 11. Troubleshooting

### Section 0

**`NameError: name 'CFG' is not defined`**  
Section 0 has not been run, or the kernel was restarted. Always run Section 0 first.

**`AssertionError: Cannot find pipeline_settings.json`**  
The notebook is running from the wrong directory. Open VS Code using File → Open Folder and point it at the project folder directly.

**`AssertionError: OPENROUTER_API_KEY is not set`**  
The `.env` file is missing or empty. Check it exists in the same folder as the notebook and contains `OPENROUTER_API_KEY=sk-or-...`.

---

### Phases 1–3 (LLM generation)

**`401 Unauthorized`**  
API key is wrong or expired. Check https://openrouter.ai → API Keys and update `.env`.

**`429 Too Many Requests`**  
Rate limit hit. The pipeline retries automatically with exponential backoff. Wait a few minutes and re-run if it persists.

**`Model not found`**  
Model names change when providers update their APIs. Check https://openrouter.ai/models for current strings.

**Items generating in the wrong language**  
Your `construct_definition` contains non-English text. All `scales.json` fields must be in English.

**Items are too generic and not domain-specific**  
The behavioural examples in Phase 1 may be too abstract. Review `01_behavioral_domains.xlsx` and ensure examples describe specific, observable behaviours. If examples are concrete but items remain generic, the `CRITICAL RULE — DOMAIN SPECIFICITY` block in the Phase 3 prompt is the lever — it uses the actual domain text as a concrete example of what specific vs. generic looks like.

**Brackets appearing in `example_behavior` column**  
The Phase 1 LLM wrapped its output in `[]`. The `clean_text` function in Section 0 strips these — ensure the current version of Section 0 is running.

---

### Phase 4

**All items showing FK grade 0**  
`textstat` is not installed. Run `pip install textstat`, restart the kernel, re-run Section 0.

**Phase 4 appears frozen at 0%**  
It is not frozen. The `tqdm` progress bar is being overwritten by LLM simplification log messages. The API calls visible in the output confirm it is running. Let it complete.

---

### Phase 5

**Phase 5 stalls partway through**  
Most likely cause is computer sleep. Re-run the Phase 5 cell — it detects `05_checkpoint.json` and resumes. You will see `Already done: N / Remaining: N`.

**Progress bar not updating**  
HTTP log messages are flooding the output. Add to the top of the Phase 5 cell:

```python
import logging
logging.getLogger("httpx").setLevel(logging.WARNING)
logging.getLogger("httpcore").setLevel(logging.WARNING)
```

**Many items failing CV for one scale**  
Check `cv_majority_mapped_scale` — consistent mapping to a different scale indicates genuine semantic overlap. Revise `discriminant_description` fields for both scales in `scales.json` and re-run.

**Reset the checkpoint to start Phase 5 from scratch:**

```python
checkpoint = BASE_OUTPUT_DIR / "05_checkpoint.json"
if checkpoint.exists():
    checkpoint.unlink()
    print("✓ Checkpoint deleted")
```

---

### Phase 6

**`OSError: [WinError 1114] A dynamic link library (DLL) initialization routine failed`**  
PyTorch and CUDA versions are mismatched. This is the most common Phase 6 error.

1. Run `nvidia-smi` to check your CUDA version
2. Reinstall PyTorch with the matching command from Section 6.2
3. **Restart the Jupyter kernel** (Kernel → Restart) — this step is essential
4. Re-run Section 0, then Phase 6

**`ValueError: matmul: Input operand 1 has a mismatch in its core dimension`**  
The two transformer models produce different embedding dimensions and cannot be averaged as raw vectors. Ensure `encode_macro_aggregate` averages item-to-scale cosine similarities per model (scalars) rather than raw embedding vectors.

**All loadings very low (below 0.2)**  
Too few items per scale entered PFA. Check `Pool_Statistics` from Phase 5. Scales with fewer than 4 passing items are skipped entirely. Regenerate more items for those scales.

**`Joint analysis skipped: only N scales`**  
DAAL and discriminant validity require at least 3 scales. If running fewer scales this is expected and per-scale PFA still runs normally.

**Phase 6 running very slowly**  
Run `torch.cuda.is_available()` in a new cell. If `False`, the transformer is on CPU. Reinstall PyTorch with CUDA support (Section 6.2).

---

### Phase 7

**`Review_Pool` is empty**  
Every item failed PFA or CV, or `human_review_pass` was set to `False` for all items. Check the Phase 6 output to diagnose. Most common cause is pass thresholds set too high relative to item count.

**`KeyError` referencing a column name**  
Phase 6 was run with an older version of the code. Re-run Phase 6 with the current code.

**Pool_Statistics shows very uneven difficulty distribution**  
Behavioural examples from Phase 1 may have been concentrated at one occurrence level. Re-run Phase 1 with a higher `examples_per_construct` to get more balanced coverage.

---

## 12. Glossary

**API (Application Programming Interface)** — a communication interface between software systems. The pipeline uses OpenRouter's API to send text to LLMs and receive responses.

**API key** — a private credential identifying you to an API service. Stored in `.env`. Never share it.

**Atomic encoding** — an embedding strategy where each item is encoded individually. The pipeline averages atomic cosine similarity matrices across two transformer models before factor analysis.

**Checkpoint** — a progress save file created by Phase 5 every 100 items (`output/05_checkpoint.json`). Enables automatic resume after interruption.

**Communality** — in factor analysis, the proportion of an item's variance explained by the common factor. Used in Rule 3 of the PFA pass criteria.

**Construct** — a psychological attribute being measured (e.g., Conscientiousness, Resilience, Openness to Experience).

**Content validity** — the degree to which items genuinely represent the intended construct, as judged by subject matter experts.

**CUDA** — NVIDIA's GPU computing framework. The CUDA version must match between your GPU driver and PyTorch installation.

**DAAL (Dominant Average Absolute Loading)** — the method used to confirm factor identity in the joint multi-scale PFA. For each extracted factor, the mean absolute loading of each scale's items on that factor is computed. The scale with the highest such value — the dominant average absolute loading — is assigned as the factor's label. A scale's DAAL identity is confirmed if the factor labelled with its name is also the factor on which that scale's items load most strongly. Where two scales produce equal dominant values on the same factor, neither can be uniquely assigned and the factor is labelled Unassigned.

**Discriminant validity** — evidence that a measure of construct A is sufficiently different from a measure of construct B. Assessed in Phase 6 by comparing each item's target factor loading to its maximum cross-loading.

**Embedding** — a numerical vector representation of text produced by a language model. The pipeline uses embeddings to compute semantic similarity between items and between items and scales.

**Factor analysis** — a statistical technique identifying underlying patterns in a correlation matrix. Phase 6 applies it to AI-generated similarity matrices rather than human response data.

**Flesch-Kincaid grade** — a readability score expressed as a US school grade level. Grade 8 means a typical 13–14-year-old can read the text. The pipeline's primary readability metric.

**GPU (Graphics Processing Unit)** — used for AI computation in Phase 6. Significantly faster than CPU for transformer encoding.

**HEXACO** — a six-factor personality model: Honesty-Humility, Emotionality, Extraversion, Agreeableness, Conscientiousness, and Openness to Experience (Ashton & Lee, 2007).

**JSON (JavaScript Object Notation)** — the text format used for `pipeline_settings.json` and `scales.json`.

**Kernel** — the Python process running the Jupyter Notebook. Restarting it clears all variables. Always re-run Section 0 after a restart.

**LLM (Large Language Model)** — an AI system trained on large amounts of text. The pipeline uses Claude, GPT-4o, Gemini, Grok, and DeepSeek via OpenRouter.

**Loading** — a value (typically −1 to 1) representing how strongly an item is associated with a particular factor in factor analysis.

**Macro encoding** — an embedding strategy where all items in a scale are concatenated before encoding. The pipeline averages macro item-to-scale cosine similarities across two transformer models.

**OpenRouter** — a service providing unified API access to multiple LLM providers through a single key.

**PFA (Pseudo-Factor Analysis)** — factor analysis applied to a matrix of AI-generated embedding similarity scores between items, rather than to a matrix of empirical correlations from human responses. Theoretical basis is the substitutability assumption.

**RMSR (Root Mean Square Residual)** — the only model fit metric computed in Phase 6. Measures the average size of unexplained correlations after the factor model is applied. Values below 0.08 indicate acceptable fit; below 0.05 indicates good fit. CFI, TLI, and RMSEA are not computed as they require sample size.

**Scale mapping agreement** — the proportion of Phase 5 SMEs who correctly identified the target scale for an item. Values below 0.6 cause an item to fail content validity.

**Sentence transformer** — a class of language model producing semantically meaningful sentence embeddings. The pipeline uses `all-MiniLM-L6-v2` (primary) and `all-mpnet-base-v2`.

**SME (Subject Matter Expert)** — in Phase 5, five LLM models play the role of independent SMEs rating item quality.

**Substitutability assumption** — the theoretical basis for PFA: that an item's embedding vector can substitute for an empirical response vector during early-stage scale development (Guenole et al., 2025).

**Tucker's congruence coefficient** — a measure of similarity between two factor loading vectors. Values ≥ 0.95 indicate excellent equivalence; ≥ 0.85 indicates fair similarity (Lorenzo-Seva & ten Berge, 2006). Used in Phase 6 to compare atomic and macro encoding solutions per scale.

---

## 13. References

Guenole, N., Samo, A., Campion, J. K., Meade, A., Sun, T., & Oswald, F. (2024, February). *Pseudo-discrimination parameters from language embeddings*. Presented February 2024.

Guenole, N., D’Urso, E. D., Samo, A., Sun, T., & Haslbeck, J. M. (2025, March). Enhancing scale development: Pseudo factor analysis of language embedding similarity matrices.

Suárez-Álvarez, J., Fernández-Alonso, R., & Muñiz, J. (2026). Evaluating pseudo-factor analysis as a tool for item selection in psychological test development. *Psicothema*, *38*(1), 1–12.

Milano, N., Luongo, M., Ponticorvo, M., & Marocco, D. (2025). Semantic analysis of test items through large language model embeddings predicts a-priori factorial structure of personality tests. *Current Research in Behavioral Sciences*, *8*, 100168. https://doi.org/10.1016/j.crbeha.2025.100168

Lee, P., Son, M., & Jia, Z. (2025). AI-powered automatic item generation for psychological tests: A conceptual framework for an LLM-based multi-agent AIG system. *Journal of Business and Psychology*, 1–29.

Lee, J. Y., Joo, S. H., Antonakis, J., & Rounds, J. (2025). The journey of forced-choice measurement over 80 years: Past, present, and future. *Annual Review of Organizational Psychology and Organizational Behavior*, *12*, 1–30. https://doi.org/10.1146/annurev-orgpsych-021124-055748

Lorenzo-Seva, U., & ten Berge, J. M. F. (2006). Tucker's congruence coefficient as a meaningful index of factor similarity. *Methodology: European Journal of Research Methods for the Behavioral and Social Sciences*, *2*(2), 57–64. https://doi.org/10.1027/1614-2241.2.2.57

---

*Automated Psychometric Scale Development Pipeline — Inubilum Ltd.*  
*Pipeline version: Revised March 2026*  
*Author: Tariq Shaban*
