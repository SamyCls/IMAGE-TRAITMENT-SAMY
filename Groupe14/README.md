# A comparative study of RAG techniques for generating image captions: Phase 1 (database)

Group: **[number]** | Members: **[names]** | Course: Introduction to image processing, University of Algiers 1

This repository contains **Phase 1**: data exploration, cleaning and preparation of the shared database used by all groups
(MS COCO images + captions, object metadata, CLIP embeddings, FAISS indexes).

> Full description of every file, field and rule: see **[DATASET.md](DATASET.md)**.
> Short report with statistics and plots: see **[report/Phase1_Report.pdf](report/Phase1_Report.pdf)**.

## What Phase 1 produces

| Set | Images | Source | Role |
|---|---|---|---|
| Knowledge base (`train_kb`) | 10,000 | COCO train2017 | Retrieval corpus (50,000 captions) |
| Validation | 100 | COCO train2017, different from the knowledge base | Tuning and choices |
| Test A | 100 | COCO val2017, stratified | First test set, kept for comparison |
| Test B | 1,000 | COCO val2017, stratified, contains Test A | Main test set |

Main steps: audit and clean images, build the splits with an anti-leakage check, explore the data **before** preprocessing,
extract objects from the images with DETR, explore the data **after** preprocessing, compute CLIP embeddings,
build FAISS indexes, write the documentation, run extra database quality checks.

## Repository content

```
.
├── README.md
├── DATASET.md                  description of the database
├── requirements.txt
├── notebooks/
│   └── Phase1_v5.ipynb         the whole Phase 1 pipeline (Parts 1 to 4)
└── report/
    └── Phase1_Report.pdf
```

The images, embeddings and indexes are **not** in this repository (too large). They are on Google Drive: **[add the Drive link here]**.

## Installation

The notebook is made for **Google Colab** with a **GPU** (Runtime > Change runtime type > T4 GPU).
Most libraries are already installed in Colab; the notebook installs `faiss-cpu` itself.

To run somewhere else:

```bash
pip install -r requirements.txt
```

You also need the commands `wget` and `unzip` (Linux/macOS), and you must remove the Google Drive lines
(`from google.colab import drive` and `drive.mount(...)`) and change `BASE` in the Setup cell to a local folder.

## How to run (reproduction)

1. Open `notebooks/Phase1_v5.ipynb` in Colab and select a GPU runtime.
2. Run the **Setup** cell. It mounts Drive and sets `BASE = /content/drive/MyDrive/RAG-Phase1` (change it if you want another folder).
3. Run the cells **in order**: Part 1 (audit, cleaning, splits), Part 2 (exploration + objects), Part 3 (embeddings + index), Part 4 (quality checks).
4. Everything is saved in `BASE`. If Colab disconnects, run Setup and the first cell of the Part you were in; finished steps are loaded from Drive instead of recomputed.
5. To rebuild Part 1 from zero, delete `splits.json` from the `RAG-Phase1` folder.

Reproducibility: random seeds are fixed (42 for the main selection, 43 for the 900 extra test images), the split ids are saved in `splits.json`,
and the exact library versions of our run are saved in `metadata/phase1_documentation.json`.

Approximate durations in our run: DETR on 11,100 images took about 53 minutes on a Colab GPU; re-checking 10,000 images (fingerprints) took about 5 minutes.

## Use the database in the next phases

Load it with the code in section 9 of [DATASET.md](DATASET.md). The five rules to respect:

1. Retrieve only from the knowledge base.
2. Never use captions of validation or test images for retrieval or generation (evaluation only).
3. A test image must never be retrieved for itself.
4. Use `query_objects.json` (detected) for queries; `eval_objects.json` (human labels) is for evaluation only.
5. Do not modify the frozen splits.

## Main results

- 0 missing, 0 corrupted, 0 exact duplicate images; 0 shared ids or pictures between sets.
- Objects extracted from images with DETR (`facebook/detr-resnet-50`), threshold 0.9 chosen on the validation images (F1 = 0.937).
  Against human labels: knowledge base F1 0.920, Test B F1 0.850, Test A F1 0.853.
- CLIP `openai/clip-vit-base-patch32`, 512 dimensions, vectors of length 1.
- FAISS `IndexFlatIP` (exact search, cosine similarity): 10,000 image vectors (20.48 MB) and 50,000 caption vectors (102.4 MB).

## Limitations

- Duplicates are detected by exact file fingerprint (MD5); Part 4 also looks for near-identical pictures with CLIP.
- The detector knows 80 COCO classes; its recall on test images is about 0.80.
- Colours, shapes, context and relations come from caption words, not from pixels, and exist only for knowledge-base images.
- Test sets are balanced with human object counts; knowledge-base statistics use detector counts.
- The fingerprint file of the first run was lost, so audit tables describe the final sets.
- The threshold was chosen on only 100 validation images.

## Data source and license

Images and captions come from **MS COCO** (Lin et al., *Microsoft COCO: Common Objects in Context*, ECCV 2014).
Annotations are under CC BY 4.0; images follow their original Flickr licenses. Follow the COCO terms of use.
