# Project 1 - Finding Similar arXiv Abstracts

This project uses the Kaggle dataset:

`Cornell-University/arxiv`

Each paper abstract is represented as a set of three-word shingles.
The paper ID and title are used to identify and display matching papers.

The main techniques are MinHash and Locality-Sensitive Hashing (LSH).
They select candidate pairs, which are then checked using exact Jaccard
similarity.

## Project files

- `find_similar_items.ipynb`: the executed Colab notebook.
- `project_1_find similar.pitems.pdf`: the project report.
- `results/simple/`: result tables and figures.
- `.gitignore`: excludes credentials, raw data, and local environment files.

## Notebook

The main notebook is:

- `find_similar_items.ipynb`

It is designed to run on Google Colab or Jupyter. To run it in Colab:

1. Upload the notebook to Google Colab and choose a CPU runtime.
2. Run the cells from top to bottom. The first code cell installs dependencies.
3. When prompted, upload the Kaggle API credential JSON file from your own
   Kaggle account.
4. Wait for the dataset download, similarity search, evaluation, and experiments.
5. Download the results from the Colab Files panel before closing the session.

Each user must provide their own credentials; none are included in this project.
The credential JSON file must contain the `username` and `key` fields.
There is no need to paste them into the code. Do not commit the credentials
file to GitHub.

The full metadata file is downloaded before sampling. If the file already exists
under `data/`, the notebook reuses it.

## Setup in VS Code

1. Open the project folder in VS Code and create a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

2. Install a notebook kernel:

```bash
python -m pip install ipykernel
```

3. For local execution, create a `.kaggle` folder in your home directory
   if it does not exist. Save your own Kaggle API credential JSON file inside
   it with the filename `kaggle.json`:
   - macOS/Linux: `~/.kaggle/kaggle.json`

   This local setup is not needed in Colab, which uses the upload step above.

4. Open the notebook with the VS Code Python and Jupyter extensions installed.
5. Select the `.venv` Python environment as the notebook kernel and run all cells.
   The installation cell installs the remaining dependencies.

## Useful options

Change the variables in Section 1 of the notebook, then rerun the cells
from the top.

Run the main search on a smaller sample and skip the parameter experiments:

```python
USE_SAMPLE = True
SAMPLE_SIZE = 1000
RUN_EXPERIMENTS = False
```

Change the similarity threshold and LSH settings:

```python
THRESHOLD = 0.5
NUM_HASHES = 120
NUM_BANDS = 30
```

`NUM_HASHES` must be divisible by `NUM_BANDS`.

Use all eligible papers in the selected category:

```python
USE_SAMPLE = False
CATEGORY = "cs.IR"
```

Set `CATEGORY = ""` to include all categories. Larger selections require
more memory and time. `EVALUATION_SIZE` limits the independent exact
check to 5,000 papers by default.

Set `RUN_EXPERIMENTS = True` to run the three experiment groups.
The scalability experiment uses its own sample of 10,000 papers, so the
selected category must contain at least that many eligible abstracts.

## Outputs

The notebook writes the following files:

- `results/similar_pairs.csv`
- `results/scalability.csv`
- `results/minhash.csv`
- `results/lsh.csv`
- `results/scalability.png`
- `results/minhash.png`
- `results/lsh.png`

The included experiment CSVs reproduce the saved Colab tables, rounded as
shown in the notebook. The PNG figures are extracted from that run. The
matching-pairs CSV has been checked against the five pairs shown in the notebook.
Rerunning the notebook writes fresh CSVs with the calculated precision.

The experiment CSVs and figures are generated when `RUN_EXPERIMENTS = True`.
The independent evaluation prints the exact match count, missed pairs,
and recall in the notebook.

The main settings used in the recorded Colab run are:

- category: `cs.IR`
- sample size: `5000`
- shingle size: `3`
- MinHash signature length: `120`
- LSH bands: `40`, with `3` rows per band
- Jaccard threshold: `0.50`
- random seed (`SEED`): `42`

The main run found 5 similar pairs from 16 candidates. The independent
exact check found the same 5 pairs, giving 100% recall on this sample.
Detection took 58.82 seconds, excluding downloading, loading, and evaluation.
The exact evaluation took another 94.64 seconds. Times can vary between runs.

The LSH comparison includes:

| Bands | Rows | Candidates | Similar pairs | Missed pairs | Recall |
|---|---:|---:|---:|---:|---:|
| 10 | 12 | 3 | 3 | 2 | 60% |
| 20 | 6 | 4 | 4 | 1 | 80% |
| 30 | 4 | 5 | 5 | 0 | 100% |
| 40 | 3 | 16 | 5 | 0 | 100% |
| 60 | 2 | 7946 | 5 | 0 | 100% |

## Experiments

1. **Scalability:** compare 1,000, 2,000, 5,000, and 10,000 papers.
   Smaller inputs are prefixes of a separate shuffled 10,000-paper sample.
   Its 5,000-paper subset differs from the main sample.
2. **MinHash:** compare signature lengths 50, 100, and 150 on the same
   evaluation sample. Measure signature-building time and estimation error
   on 10,000 random pairs and all exact-positive pairs with the default settings.
3. **LSH:** keep 120 signature values and compare five bands/rows settings
   using the same evaluation sample and exact reference matches.

## Method

The implementation follows these steps:

1. Read the dataset one record at a time and select papers using reservoir sampling.
2. Lowercase each abstract and split it into words and numbers.
3. Build a set of consecutive three-word shingles.
4. Create MinHash signatures using the same hash functions for every paper.
5. Divide signatures into bands and use matching bands to find candidate pairs.
6. Calculate exact Jaccard similarity for candidates and keep scores at least 0.50.
7. Independently check the evaluation sample to measure missed pairs and recall.

Shingles are stored as Python sets, signatures as tuples, and LSH buckets
in a dictionary. Candidate pairs are yielded once even if they share several
bands. LSH reduces comparisons on the tested samples, but its worst case can
still approach all pairs. The selected data and signatures remain in memory.

Jaccard measures shared wording, not semantic meaning. The reported recall
applies to the evaluated sample; it does not guarantee perfect recall on all
arXiv papers. If there are no exact-positive pairs, recall is undefined.
