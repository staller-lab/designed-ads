# Designed ADs analysis code

Code and processed data for the designed activation domain study.

Sequences were generated with Markov chains trained on plant activation
domains, filtered through activation-domain predictors, and measured by
sort-seq in yeast. Selected designs were retested in plant leaves and in
HEK293T cells.

## Repository layout

| Folder | Contents |
|---|---|
| `designed AD generation notebooks/` | Markov chain sequence generation and predictor filtering used to choose the 18,750 tested designs |
| `sequencing data processing notebooks/` | Sort-seq read processing and activity calculation |
| `flow notebooks/` | Flow cytometry gating and per-plate processing for the HEK293T experiments |
| `LevenshteinDistance/` | Minimum edit distance of designed sequences to reference sequence sets |
| `notebooks/` | Sequence feature calculation, MMseqs2 clustering, predictor merging |
| `figures_notebooks/` | `Figures.ipynb` produces every main and Extended Data panel |
| `data/` | Inputs read by `Figures.ipynb` |
| `data_predictions/` | Predictor scores and ROC/PRC curve data |
| `supplementary_tables/` | Supplementary tables |
| `envs/` | Conda environment files, one per analysis stage |

## Order of analysis

1. **Design.** `designed AD generation notebooks/4. Markov_Chain_Maker_Original.ipynb`
   generates sequences; `ADHunter, Paddle, TADA Predictor Analysis.ipynb` and
   `250 Combination Cleaning.ipynb` apply the predictors and select designs.
2. **Sequencing.** `labtools_SynADs_*.ipynb` convert reads to per-barcode
   activities; `CollapseADs_*.ipynb` average barcodes, harmonize against the
   controls library, and write per-sequence activities.
3. **Flow.** `Set_Gates_*.ipynb` define gate polygons, saved to a pickle.
   `Process_Plate_*.ipynb` apply them and write
   `data/HEK293T14_{EpCAM,ICAM1}_AD_ID.csv`. `GatingFigureS15*.ipynb` redraw
   the gating series for the supplement.
4. **Sequence analysis.** `notebooks/Calculate_Sequence_Features_500k.ipynb`
   (localCIDER), `Cluster_MMSeqs2_50pctidentity.ipynb`,
   `CombinePredictions.ipynb`, and `LevenshteinDistance/`.
5. **Figures.** `figures_notebooks/Figures.ipynb`.

## Software environments

The analyses ran across three machines, each with its own environment: a
Windows 11 desktop, WSL2 (Ubuntu) on that same desktop, and the Berkeley Savio
cluster. The environment files are in `envs/`, exported in two forms —
`*_from-history.yml` lists only the packages that were explicitly requested and
is the portable file to install from, while `*_full.yml` records the complete
solved environment and is the provenance record.

Create any of them with, for example:

```
conda env create -f envs/environment-figures.yml
```

| Environment | Machine | Used for |
|---|---|---|
| `environment-figures.yml` | Windows 11 | `figures_notebooks/Figures.ipynb` |
| `flow_*.yml`, `flow_v2_*.yml` | Windows 11 | `flow notebooks/` |
| `markovml_*.yml` | Windows 11 | `designed AD generation notebooks/` |
| `multi_*.yml`, `labtools_*.yml` | Windows 11 | General local analysis |
| `labtools2407_*.yml` | Savio | `sequencing data processing notebooks/` |
| `editdistance_*.yml` | Savio | `LevenshteinDistance/` |
| `wsl_localcider_env_*.yml`, `wsl_sparrow_*.yml` | WSL2 Ubuntu | `notebooks/Calculate_Sequence_Features_500k.ipynb` |
| `wsl_mmseqs_env_*.yml` | WSL2 Ubuntu | `notebooks/Cluster_MMSeqs2_50pctidentity.ipynb` |
| *(system Python module)* | Savio | `notebooks/` predictor comparison and composition analyses |

### Figures and statistics — `environment-figures.yml` (Windows 11)

Everything needed to reproduce every main-text and Extended Data panel.

| Package | Version |
|---|---|
| python | 3.12.3 |
| numpy | 2.3.0 |
| pandas | 2.3.0 |
| scipy | 1.15.2 |
| scikit-learn | 1.7.0 |
| statsmodels | 0.14.4 |
| matplotlib | 3.10.3 |
| seaborn | 0.13.2 |
| openpyxl | 3.1.5 |

### Flow cytometry — `flow`, `flow_v2` (Windows 11)

| Package | Version |
|---|---|
| python | 3.11.11 |
| FlowCal | 1.3.1 |
| numpy | 2.3.5 |
| pandas | 2.3.3 |
| statsmodels | 0.14.6 |
| matplotlib | 3.11.2 |
| seaborn | 0.13.2 |

### Sequence generation — `markovml` (Windows 11)

| Package | Version |
|---|---|
| python | 3.12.4 |
| markovify | 0.9.4 |
| localCIDER | 0.1.21 |
| numpy | 2.1.0 |
| pandas | 2.2.2 |
| scipy | 1.14.1 |
| scikit-learn | 1.5.1 |
| matplotlib | 3.9.2 |

### Sort-seq read processing — `labtools2407` (Savio)

Reads were merged with **PEAR 0.9.11** before processing.

| Package | Version |
|---|---|
| python | 3.12.4 |
| numpy | 1.26.4 |
| pandas | 2.2.2 |
| scipy | 1.13.1 |
| scikit-learn | 1.4.2 |
| matplotlib | 3.8.4 |
| seaborn | 0.13.2 |

`labtools` itself is not on PyPI or conda and is therefore absent from the
environment file. Install it from source:

```
conda activate labtools2407
pip install git+https://github.com/staller-lab/labtools.git
```

### Edit distance — `editdistance` (Savio)

| Package | Version |
|---|---|
| python | 3.11.15 |
| editdistance | 0.8.1 |
| numpy | 1.26.4 |
| pandas | 3.0.2 |
| scipy | 1.17.1 |
| tqdm | 4.67.3 |

### Sequence features and clustering (WSL2, Ubuntu)

localCIDER, metapredict, sparrow and MMseqs2 were run under WSL2 rather than
Windows, in three separate environments: `wsl_localcider_env_*.yml`,
`wsl_sparrow_*.yml` and `wsl_mmseqs_env_*.yml`.

**`localcider_env`** — localCIDER sequence features

| Package | Version |
|---|---|
| python | 3.15.2 |
| localCIDER | 0.1.21 |
| biopython | 1.88 |
| numpy | 2.5.4 |
| pandas | 3.1.0 |
| scipy | 1.18.1 |
| matplotlib | 3.11.2 |
| statsmodels | 0.15.0 |

**`sparrow`** — sparrow and metapredict

| Package | Version |
|---|---|
| python | 3.14.7 |
| sparrow | 0.2.2 (commit `03aa232`) |
| metapredict | 3.0.2 |
| protfasta | 0.1.23 |
| idptools-parrot | 1.7.6 (commit `8d5c671`) |
| torch | 2.13.0 |
| numpy | 2.5.1 |
| pandas | 3.0.5 |
| scipy | 1.18.0 |
| scikit-learn | 1.9.0 |
| matplotlib | 3.11.0 |

**`mmseqs_env`** — sequence clustering

| Package | Version |
|---|---|
| python | 3.14.7 |
| MMseqs2 | 18.8cc5c |
| numpy | 2.5.2 |
| pandas | 3.0.5 |
| scipy | 1.18.0 |
| scikit-learn | 1.9.0 |
| matplotlib | 3.11.1 |
| statsmodels | 0.14.6 |

### Predictor comparison — Savio system Python

The notebooks in `notebooks/` that compare predictor outputs ran against the
cluster's Python module rather than a conda environment, so they have no
environment file.

| Package | Version |
|---|---|
| python | 3.10.10 (Savio module `langs/python/3.10`, Scientific Linux 7 stack) |
| numpy | 1.23.5 |
| pandas | 1.5.3 |
| scipy | 1.10.0 |
| matplotlib | 3.7.0 |
| seaborn | 0.12.2 |

That software stack has since been retired on Savio in favour of Rocky
Linux 8. These versions are recorded for provenance; the analyses do not depend
on the stack and the versions can be installed anywhere.

### Activation-domain predictors

PADDLE, TADA, ADhunterLite, ADhunterEnsemble and the Kotha composition model
were run from the source released with their respective publications rather
than as installed packages, so they carry no version numbers. ADpred was
installed as a conda package, version 1.3.1. Model development and the
proteome-scale predictor runs are in
[Madrigal-development](https://github.com/staller-lab/Madrigal-development).

### A note on versions

These environments were exported after the analyses were complete, so some
package versions are newer than those used at the time; where an executed
notebook's own metadata differs, the notebook metadata reflects the version
that produced the published result.

## Key data files

| File | Contents |
|---|---|
| `supplementary_tables/18750designed_activities_preds.csv` | Measured activities and predictions for the 18,750 tested designs |
| `data/Controls_supptable_allactivities.csv` | Controls library activities, used to harmonize across sorts |
| `data/allactivities_seqs_in_sort2.csv` | Per-sequence activities from the Sort 2 ratio sort |
| `data/SynAD_Glu_2_allinfo.csv` | Per-barcode reads and bin distributions |
| `data/MinSynth_to_Proteome.csv`, `data/MinSynth_to_Train.csv`, `data/Synthetics_min_edit_dist.csv` | Minimum edit distance from the designed sequences to the proteome, to the training set, and within the designed set |
| `data/*_Yeast14byBCrep_AD_ID.csv` | Per-barcode yeast measurements for the 14 retested designs |
| `data/HEK293T14_{EpCAM,ICAM1}_AD_ID.csv` | HEK293T CRISPRa flow results |
| `data/Plant_Yeast14byBCrep_AD_ID.csv` | *N. benthamiana* leaf measurements |
| `data_predictions/500000_shuffled_*_preds_dask.csv` | Per-predictor scores for the 500,000 generated sequences |
| `data_predictions/*_roc_prc_data.csv` | Precomputed ROC/PRC curves per dataset and predictor |

### Raw predictor scores for the 500,000 generated sequences

The per-predictor output files are included in `data_predictions/`:

| File | Predictor |
|---|---|
| `500000_shuffled_ADpred_preds_dask_first_half.zip`, `..._second_half.zip` | ADPred (zipped and split in two to stay under GitHub's file size limit; unzip both and concatenate before use) |
| `500000_shuffled_53_ADhunter_ensemble_preds.csv` | ADHunter (ensemble) |
| `500000_shuffled_ADHunter_preds_dask.csv` | ADHunter |
| `500000_shuffled_53_Paddle_preds_dask.csv` | PADDLE |
| `500000_shuffled_TADA_preds_dask.csv` | TADA |

### Data not stored in this repository

Four sets of files are not tracked here, either because they are
distributed with the paper or because they can be regenerated from the code
in this repository.

**Supplementary Table 1 (`500k_preds.csv`).** All six predictor scores for
the 500,000 generated sequences. This table is submitted with the
manuscript and will be available as a supplementary file with the published
paper. It can also be rebuilt from the per-predictor files above with
`notebooks/CombinePredictions.ipynb`.

**`500k_AA_SS_seqfeats.csv`** — all 500,000 generated sequences with state
size and localCIDER features. Generate with
`notebooks/Calculate_Sequence_Features_500k.ipynb`, using the generated
sequences from Supplementary Table 1 as input.

**`MinSynth500K_to_*.csv`** — minimum edit distance from the generated
sequences to each reference set (`proteome`, `choptf`, `train`,
`synth18750`, `self`). Generate with
`LevenshteinDistance/calculate_edit_distance.py`; see
`LevenshteinDistance/editdist.sbatch` for the cluster job used here. The
query and reference paths are set at the top of the script, under `BASE`,
`QUERY_PATH` and `TARGETS`, and point at the cluster directories used for
this study; edit them to your own copies before running. The query set is the
500,000 generated sequences from Supplementary Table 1, and the reference sets
are the tiled proteome, the Markov chain training sequences, the ChopTF set
and the 18,750 designed sequences.

**`consensus_model_all_six_predictions.tsv`** — six-predictor scores for
230,302 natural transcription factors. Generate with
[madrigal-predictor](https://pypi.org/project/madrigal-predictor/), run over
the natural transcription factor sequences, which are distributed in
[Madrigal-development](https://github.com/staller-lab/Madrigal-development)
under `compressed_data/` as `cl_seqs.fasta.gz` (Y1000+ fungal),
`PlantTFDB-all_TF_pep.fas.gz` (plant) and
`interpro_uniprot_evid_1_2.fasta.gz` (metazoan).
`synthetic_experimental_and_preds_w_parrot.csv` can be produced the same way.

Place these files at the relative paths listed in the tables above before
running `Figures.ipynb`, whose data-loading cell reads them directly. Every
other notebook runs without them.

### Externally sourced data

These are redistributed from published work and should be cited to their
original sources, not to this repository:

- `data/Morffy_41586_2024_7707_MOESM2_ESM.csv` — Morffy *et al.* 2024, *Nature* 632:166–173 (supplementary data).
- `data/staller2018data.csv` — Staller *et al.* 2018.
- `data/sanborn_mutants.csv` — Sanborn *et al.* 2021, *eLife* 10:e68068.

## Running the figure notebook

`figures_notebooks/Figures.ipynb` reads from `../data/` and
`../data_predictions/` using relative paths, so it runs in place once the
files listed under "Data not stored in this repository" have been generated
or downloaded into those folders. It saves PDFs to `../../Figures_v5/`, which is
outside this repository; change that path or create the folder before
running. Panels not included in the submission are saved with a
`NOTINCLUDED_` prefix.

Create the environment with
`conda env create -f envs/environment-figures.yml`; the pinned versions are
listed under "Software environments" above. Other notebooks use the other
environments in `envs/`, and additionally need MMseqs2 and the Staller lab
[labtools](https://github.com/staller-lab/labtools) package.

## Notes

- Some notebooks load data using absolute paths from the machine each analysis
  was run on, which record where the work was done. Point them at your own
  copies of the files before running those notebooks. `Figures.ipynb` uses
  relative paths and needs no changes.
