# Translation Data Processing Pipeline

## Overview

This project provides a modular pipeline for preparing parallel corpora for machine translation research.\
It standardises preprocessing across datasets, making it easier to train and evaluate translation models on consistent, high-quality data.

The pipeline is:

- **Modular** – enable or disable steps as needed.
- **Configurable** – control all behaviour via a YAML config file.
- **Reproducible** – consistent outputs for large-scale experiments.
- **Scalable** – simple to adapt for parallelisation in an HPC environment.

---

## Features

- **Input handling**: Read corpora in plain text (`.txt`), tab-separated (`.tsv`) or translation memory (`.tmx`) formats.
- **Embeddings**: Compute multilingual sentence embeddings (e.g. LaBSE, SONAR).
- **Language ID**: Calculate probability that segments are in the desired language.
- **Filtering**: Filter by embedding scores and language probability.
- **Deduplication**: Remove duplicate sentence pairs and fuzzy matches across corpora.
- **Bifixer**: Apply any of bifixer's functionality. Default is to ignore deduplication and segmentation.
- **Normalisation**: Standardise punctuation, spacing, and casing. Includes easy-to extend language specific normalisation.

---

## Prerequisites

- Python ≥3.8
- Sufficient disk space (embeddings can be large).
- (Optional) HPC cluster or multi-core machine for parallel processing.

---

## Installation

Clone the repository and set up a virtual environment:

```bash
git clone https://github.com/paraclean023-coder/ParaCLEAN
cd ParaClean
python -m venv venv
source venv/bin/activate   # or venv\Scripts\activate on Windows
pip install -r requirements.txt
```
## Usage
Run the pipeline with a configuration file:

```bash
python pipeline.py --config config_multi.yaml
```

## Configuration

The pipeline supports both single-corpus and multi-corpus runs.

- In single runs, one input corpus passes through all selected steps sequentially.
- In multi-corpus runs, several corpora are at first processed individually through the scoring steps, then **merged** for filtering, deduplication and normalisation.
The pipeline is configured via a YAML file. Example config files are provided for both single and multi file inputs. Below is a single file example:

```yaml
# ===========================
# Pipeline configuration file
# Example: single corpus run
# ===========================

# Output directory (will be created if missing)
output: "data/Europarl"

# Languages
l1: "es"
l2: "de"

# Embedding model (choices: labse, sonar if downloaded)
model: "labse"
model_path: null   # optional, if you want to point to a local model path.

# Filtering thresholds
alignment_score: 0.75
langid_l1_prob: 0.5
langid_l2_prob: 0.5

# Pipeline steps to run (in order)
steps:
  - input
  - embeddings
  - langid
  - filter
  - dedup
  - bifixer
  - normalise

#Optional Bifixer flags. More info can be found at  https://github.com/bitextor/bifixer
bifixer_flags: ["--ignore_segmentation", "--ignore_duplicates"]

# Input corpus (single)
input: ["data-storage/Europarl.es-de.es", "data-storage/Europarl.es-de.de"]
format: "plain_text"  # or "tsv" or "tmx"
```
### Multi-corpus configuration

In a multi-corpus setup, each dataset runs its own *per-corpus steps* (input, embeddings, langid) before being merged for the later *merged steps* (filter, dedup, bifixer, normalise).  
This allows consistent filtering and deduplication across corpora.

Example:
```yaml
# ===========================
# Pipeline configuration file
# Example: multi-corpus run
# ===========================

output: "testing/multi"

l1: "Catalan"
l2: "cmn_Hani"

model: "labse"
model_path: null

alignment_score: 0.75
langid_l1_prob: 0.5
langid_l2_prob: 0.5

steps:
  - filter
  - dedup
  - bifixer
  - normalise

bifixer_flags: ["--ignore_segmentation", "--ignore_duplicates"]

inputs:
  - name: "TED2020"
    type: "plain_text"
    start_from: "testing/multi/TED2020.embeddings.tsv"
    steps: ["langid"]

  - name: "QED"
    type: "plain_text"
    start_from: "testing/multi/QED.embeddings.tsv"
    steps: ["langid"]
```

## Pipeline Steps
**input** Preprocesses and normalises the raw input format.

**embeddings** Computes sentence embeddings for later filtering.

**langid** Runs language identification to remove sentences in the wrong language.

**filter** Applies thresholds (embedding similarity, language ID confidence).

**dedup** Removes duplicate or near-duplicate sentence pairs.

**bifixer** Integrates bifixer functionality. Recommended: no deduplication/segmentation. Requires an existing installation of bifixer

**normalise** Applies lightweight normalisation to punctuation, text encoding, etc.

## Outputs

Each processing step writes its output as a `.tsv` file in the designated output directory.  
Intermediate files are named according to their step (e.g. `Europarl.embeddings.tsv`, `Europarl.langid.tsv`, etc.).

The **final output** (from the `normalise` step) contains four columns:
1. Source language (original)
2. Target language (original)
3. Source language (normalised)
4. Target language (normalised)

## Language Identifiers

Language inputs can be specified in multiple formats. The pipeline automatically resolves these to the correct internal representation.  
The following examples are all equivalent for Catalan:
  - Catalan
  - ca
  - ca_Latn
  - cat


## Notes & Caveats

SONAR embeddings: inputs longer than 514 tokens are truncated.

Disk space: embeddings can be large; ensure sufficient space.

Extensibility: filtering and normalisation rules can be customised by editing the corresponding modules in steps/ or adding language-specific rules in `normalisation`

## Development

To add a new processing step:

1. Create a module in steps/.
2. Define a function with a consistent interface.
3. Register it in pipeline.py.

## License


## Citation

```bibtex
@software{translation_pipeline,
  author = {Your Name},
  title = {Translation Data Processing Pipeline},
  year = {2025},
  url = {https://github.com/<your-repo>}
}
```
