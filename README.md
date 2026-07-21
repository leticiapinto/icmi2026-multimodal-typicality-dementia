# Multimodal Behavioral Typicality as a Training-Free Screening Signal for Dementia

Code and derived scores for the ICMI 2026 paper.

> Pinto-Alva, L., Lucas, G., Matarić, M., and Thomason, J. (2026).
> *Multimodal Behavioral Typicality as a Training-Free Screening Signal for Dementia.*
> ICMI '26, Naples, Italy. https://doi.org/10.1145/3776574.3831111

---

## What this is

We score gaze, text, and audio by negative log-likelihood (NLL) under pretrained
models, and ask whether a participant's behavior looks typical to a model that has
never seen dementia data.

**No model is trained or fine-tuned at any stage.** Every model is frozen and
publicly available. There is no train/test split because there is no training.
The only sample-derived quantity is the per-modality mean and standard deviation
used to place modalities on a common scale before fusion (Eq. 3), computed over
the pooled sample without reference to group labels.

---

## What is and is not released

| Item | Released | Why |
|---|---|---|
| Analysis code (all modalities, fusion, statistics) | ✅ | |
| Figure and table generation scripts | ✅ | |
| CLIP golden-target maps for the stimulus | ✅ | derived from the image, not participants |
| Per-participant NLL scores and fusion z-scores | ✅ | derived, non-identifiable |
| Raw gaze, audio, video, transcripts | ❌ | identifiable; IRB does not permit redistribution |
| Participant demographics at individual level | ❌ | re-identification risk at n = 39 |
| Cookie Theft stimulus image | ❌ | third-party copyright (PRO-ED, Inc.) |

Group-level demographics are reported in Section 3 of the paper.

The **DementiaBank Pitt Corpus** used for external validation is not redistributed
here. It is available to researchers under DementiaBank's data use agreement:
https://dementia.talkbank.org/

---

## Reproducing the results

### Environment

```bash
conda env create -f environment.yml
conda activate eyegaze_env
```

Or with pip:

```bash
pip install -r requirements.txt
```

Model weights download from HuggingFace on first run. Set `HF_HOME` if you want
them somewhere other than the default cache.

### From derived scores (no private data needed)

Every table and figure in the paper can be regenerated from the released
per-participant scores:

```bash
python scripts/make_table1_cross_modal.py
python scripts/make_table2_gaze_strategies.py
python scripts/make_table3_text_models.py
python scripts/make_table4_audio.py
python scripts/make_table5_fusion.py
python scripts/make_figure3_scatter.py
```

### From raw data (requires the private dataset)

```bash
python -m src.gaze.run_gaze_nll     --config configs/gaze_clip.yaml
python -m src.text.run_text_nll     --config configs/text_models.yaml
python -m src.audio.run_audio_nll   --config configs/audio_models.yaml
python -m src.fusion.run_fusion     --config configs/fusion.yaml
```

---

## Repository layout

```
configs/          run configurations, one per experiment
src/
  gaze/           CLIP golden targets, GMM fitting, gaze NLL
  text/           eleven language models, three transcript conditions
  audio/          wav2vec2 / HuBERT / Whisper NLL, EnCodec reconstruction MSE
  fusion/         z-normalization and late fusion
  stats/          Hedges' g, permutation tests, Benjamini-Hochberg FDR
scripts/          one script per table and figure in the paper
results/          canonical statistical outputs (the CSVs the paper reports)
data/
  derived/        per-participant NLL and fusion z-scores
  targets/        CLIP golden-target maps
```

**Convention:** figure scripts read the same CSVs in `results/` that the tables
read. Statistics are never recomputed inside a plotting script.

---

## Key results

| Modality | Semantic model | *g* | Mechanical baseline | *g* |
|---|---|---|---|---|
| Gaze | CLIP | +1.04 | GradCAM | −0.28 |
| | | | EyeFormer (temporal) | −0.02 |
| Text | Mistral-7B | +1.24 | — | — |
| Audio | wav2vec2 | +0.66 | EnCodec | −0.03 |

Three-way fusion: *g* = 1.79, AUC = 0.94 (n = 38).

Grid search over all 1,320 configurations: no mechanical baseline appears in the
top 10. The configuration reported in the paper ranks #3 and was not selected by
grid search.

---

## Scope and intended use

This is a **screening signal, not a diagnostic tool.** Classification thresholds
reported in the paper are descriptive and derived in-sample; they are not proposed
for use. Any deployed threshold would require derivation on an independent
reference cohort and prospective validation with clinician oversight.

The pipeline is English-only and inherits the priors of its pretrained components.
Speakers of under-represented dialects may register as more atypical. See Section 8
of the paper.

---

## License

Code: MIT. Derived score files: CC-BY 4.0.
The Cookie Theft stimulus is © PRO-ED, Inc. and is not included in this repository.

---

## Acknowledgments

Supported by the National Institute on Aging of the National Institutes of Health
under Award P30AG073105, via UPenn's Penn Artificial Intelligence and Technology
Collaboratory for Healthy Aging, and by the Army Research Office under Award
W911NF-25-2-0040. The content is solely the responsibility of the authors and does
not necessarily represent the official views of the National Institutes of Health.
