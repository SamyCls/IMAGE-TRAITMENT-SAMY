# Group 23 - Corrective RAG for Image Caption Generation

## Project Title
A Comparative Study of RAG Techniques for Generating Image Captions - Corrective RAG (CRAG).

## Team Members
- Saadallah Rym
- Maria Bennabi
- Benmeziane Sarah
- Zina Oumahammed

## Project Overview
This project investigates Retrieval-Augmented Generation (RAG) techniques for image caption generation, with a focus on Corrective RAG (CRAG).

The objective is to improve image caption generation by retrieving relevant visual and textual information from a knowledge base.

## Dataset
The project uses the MS COCO 2017 dataset.

Current dataset preparation:
- 10,000 training images
- 100 validation images
- 1,000 test images
- 11,100 images in total

The original dataset images are not included in this submission.

## Technologies and Tools
- Python
- YOLO11n for object detection
- CLIP (ViT-B/32) for image and text embeddings
- FAISS for similarity search and retrieval
- NumPy and Pandas for data processing
- Matplotlib for visualization

## Project Structure

Groupe23/
|-- READ ME.txt
|-- README.md
|-- .gitignore
|-- check_environment.py
|-- preprocessing/
|-- results/
    |-- figures/
    |-- statistics/

## Current Progress
The current submission includes:

1. Dataset preparation and selection.
2. Dataset quality validation.
3. Image and caption metadata analysis.
4. Object detection using YOLO11n.
5. Image and text embedding generation using CLIP.
6. Knowledge base preparation using FAISS.
7. Initial retrieval evaluation.

## Preliminary Retrieval Evaluation

The initial retrieval evaluation was performed on 98 validation images.

- Recall@5: 0.004516
- Mean Reciprocal Rank (MRR): 0.962925
- Average similarity: 0.792525

These are preliminary retrieval results, not final CRAG caption-generation evaluation metrics.

## Important Note
The current submission documents the initial data preparation and retrieval stages. The complete Corrective RAG correction and caption-generation pipeline, together with its final evaluation, remain to be implemented.

## Results
Figures and statistical reports generated during the preliminary experiments are available in the `results/` directory.