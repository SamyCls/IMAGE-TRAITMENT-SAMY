# Dataset Description

## 1. Dataset Source

This project uses the **Microsoft Common Objects in Context (COCO) 2017** dataset for image-captioning and image-retrieval experiments.

COCO contains real-world images accompanied by natural-language captions and object annotations. This project uses images from the `train2017` and `val2017` subsets.

- **Training source:** `train2017`
- **Validation source:** `train2017`
- **Test source:** `val2017`
- **Image format:** JPEG
- **Image content:** Everyday scenes containing people, animals, objects, and indoor or outdoor environments.

The original COCO images and annotation files are not included in this repository. They must be downloaded separately from the official COCO dataset distribution.

## 2. Image Subsets Used in the Project

The project defines the following principal subsets:

| Subset | COCO source | Number of images | Purpose |
|---|---|---:|---|
| Knowledge Base | `train2017` | 10,000 | Main image collection used to build the retrieval knowledge base |
| Validation | `train2017` | 100 | Validation and technical checks |
| Principal Test Set | `val2017` | 100 | Final evaluation |
| Supplementary Test Set | `val2017` | 1,000 | Supplementary evaluation (includes the 100 principal test images) |

The Knowledge Base and validation image IDs were selected from `train2017` after shuffling the image IDs using the fixed random seed `42`. The first 10,000 IDs were assigned to the Knowledge Base, and the following 100 IDs were assigned to the validation set.

The 100 test image IDs were selected from `val2017`. Split verification confirmed that the three principal subsets have no overlapping image IDs.

### Principal test set (100 images) versus supplementary test set (1,000 images)

| | Principal test set | Supplementary test set |
|---|---|---|
| Images | 100 | 1,000 (the 100 principal images + 900 others) |
| Captions | not reported | 5,004 |
| Source | `val2017` | `val2017` |
| ID file | `data/splits/test_image_ids.json` (identical copy: `test_100_image_ids_legacy.json`) | `data/splits/test_1000_image_ids.json` |
| Construction | First 100 `val2017` IDs after shuffling (seed `42`) | The 100 principal IDs plus 900 IDs drawn at random (seed `42`) from the remaining `val2017` images, by `scripts/create_updated_test_split.py` |
| Role | Final evaluation (principal split configuration) | Supplementary evaluation, giving more stable statistics |
| Formal quality control (`scripts/quality_control.py`) | Yes, 10/10 checks passed | `quality_control.py` not re-run; assertions of `create_updated_test_split.py` and leakage checks of `test_retrieval_1000.py` (unique IDs, no overlap with the knowledge base or validation set) |
| Retrieval check | Validation images (100) in `reports/validation/` | 1,000 test images in `reports/test_1000/` (`scripts/test_retrieval_1000.py`) |

The two sets must not be confused. The 1,000-image selection contains the 100 principal images, and `test_image_ids.json` was deliberately left unchanged. Test images and captions are used only as queries: they are never added to the knowledge base or to the indexes.

## 3. Captions

The COCO dataset generally provides five captions per image, although the exact number may vary.

For the 10,000 Knowledge Base images, the project contains:

- **10,000 images**
- **50,028 captions**
- **9,972 images with five captions each**
- **28 images with six captions each**
- **No missing image references** in the caption preprocessing output

Two caption versions are retained:

- **Raw captions:** The original caption text, preserved for reference and evaluation.
- **Cleaned captions:** Normalized text used by the text-processing and retrieval pipeline.

The cleaning stage includes lowercasing, trimming leading and trailing whitespace, collapsing repeated whitespace, and removing spaces before punctuation. The original caption wording is otherwise preserved.

## 4. Image Preprocessing

The image preprocessing pipeline prepares the Knowledge Base images for visual feature extraction.

Images are handled as RGB inputs. The CLIP processor performs the model-specific resizing, cropping, normalization, and tensor conversion required by the selected model. The pipeline does not independently stretch images to a fixed aspect ratio before passing them to CLIP; the CLIP processor resizes the shorter side to 224 pixels and center-crops to 224x224, so the borders of landscape images are cropped.

The visual embedding model used in this project is **CLIP ViT-B/32** (`openai/clip-vit-base-patch32`).

## 5. COCO Object Annotations and Metadata

In addition to captions, COCO instance annotations are used to extract structured information about the objects appearing in the Knowledge Base images.

The metadata extraction stage records information such as:

- Object categories
- Bounding boxes
- Object areas
- Crowd indicators (`iscrowd`)
- Object counts per image
- Spatial relationships estimated from object positions

For the 10,000 Knowledge Base images, the extracted COCO metadata contains **71,773 object instances** and approximately **961,525 spatial relationships**.

These values describe the metadata extraction results, not the number of images or captions.

## 6. Additional YOLO Detections

The project also includes an additional object-detection stage using **YOLO26n**. This stage supplements the original COCO annotations; it does not replace them.

The detector was run on the 10,000 Knowledge Base images using a confidence threshold of `0.25`. The resulting outputs contain:

- **49,375 detected objects**
- **79 categories** represented among the detections
- **136 images** with no YOLO detections

The number 79 refers to the categories actually represented in the detector output, not to the total number of COCO categories.

The merged metadata file `data/metadata/kb_metadata_with_yolo.json` is approximately 143 MB and is not included in the Git repository because of its size. The generated YOLO detection outputs are stored separately under `data/visual_detections/full_10000/`, depending on the repository's tracking configuration.

## 7. Image and Text Embeddings

CLIP is used to encode both images and captions into a shared feature space for retrieval.

| Embedding type | Number of vectors | Dimension | Data type |
|---|---:|---:|---|
| Image embeddings | 10,000 | 512 | `float32` |
| Text embeddings | 50,028 | 512 | `float32` |

These embeddings are used by the project's retrieval pipeline. The 512-dimensional representation corresponds to the output dimension of the selected CLIP model.

## 8. Data Organization

The main dataset-related files are organized under the `data/` directory:

```text
data/
├── splits/
│   ├── knowledge_base_image_ids.json
│   ├── validation_image_ids.json
│   ├── test_image_ids.json
│   ├── test_100_image_ids_legacy.json
│   └── test_1000_image_ids.json
├── captions/
│   ├── kb_captions_raw.json
│   └── kb_captions_cleaned.json
├── metadata/
│   └── kb_metadata.json
├── visual_detections/
│   └── full_10000/
└── embeddings/

reports/
├── validation/    retrieval check on the 100 validation images
└── test_1000/     retrieval check on the 1,000 supplementary test images
```

The exact files available may vary as the pipeline evolves. Large source images, generated intermediate files, and some large merged artifacts may be excluded from Git tracking.

The split ID files and preprocessing outputs document which COCO images are used throughout the project.

## 9. Reproducibility Notes

- The principal Knowledge Base and validation split uses random seed `42`.
- The Knowledge Base and validation images are selected from `train2017`.
- The principal test images are selected from `val2017`.
- The principal split configuration contains 10,000 Knowledge Base images, 100 validation images, and 100 test images.
- The extended 1,000-image test selection is used for supplementary evaluation; it contains the 100 principal test images.
- Retrieval on the 1,000 supplementary test images (5,004 captions, top-10, CLIP ViT-B/32, CPU) is reported in `reports/test_1000/test_1000_summary.json`.
- COCO annotations and YOLO predictions are distinct sources of visual metadata and should be interpreted separately.
- The repository stores project code, split definitions, and selected processing outputs. Users may need to download the original COCO images and annotation files separately.

## 10. Dataset Reference

Lin, T.-Y., Maire, M., Belongie, S., et al. (2014). **Microsoft COCO: Common Objects in Context.** *European Conference on Computer Vision (ECCV).*

Project website: https://cocodataset.org/