# Protein-Localization
Subcellular Protein Localization via Fluorescence Microscopy

This project implements a complete high-performance Machine Learning pipeline designed to classify sub-cellular protein locations (specifically targeting **Mitochondria** and **Nucleoli**) using fluorescent microscopy images. 

It features parallelized feature extraction, statistical spatial profiling, co-localization analysis, and a Random Forest Classifier evaluated using Stratified K-Fold cross-validation.

---

## Dataset Structure

The project utilizes 4-channel immunofluorescence microscopy images. Each protein sample has 4 separate grayscale PNG channel images:
*   **`_blue.png`**: Nucleus (Reference marker)
*   **`_red.png`**: Microtubules (Reference marker)
*   **`_yellow.png`**: Endoplasmic Reticulum (ER, Reference marker)
*   **`_green.png`**: Target Protein of interest

### Directory Hierarchy
Ensure your Google Drive has the following structure:
My Drive/ └── BMED3201_Projects/ └── Protein_Localization/ ├── training_labels.csv # Contains 'sample_id', 'Mitochondria', 'Nucleoli' └── protein_loc_project_data/ # Folder containing raw *.png images


---

## Methodology & Pipeline

### 1. High-Speed Local Copying
Because loading thousands of files directly from Google Drive during execution can be extremely slow due to I/O bottlenecks, the pipeline copies the entire image directory to the local Colab temporary runtime SSD (`/content/protein_loc_project_data`) before processing.

### 2. Parallelized Feature Extraction
For each sample, the pipeline extracts **27 spatial and intensity-based features**:
*   **Basic Intensity Statistics (20 features)**: Mean, Standard Deviation, 75th percentile, 90th percentile, and 99th percentile across each of the 4 channels.
*   **Reference Correlation (4 features)**: Pearson correlation coefficients between the target Protein (Green) and each structural reference channel (Blue, Red, Yellow).
*   **Mask Overlap / Colocalization (4 features)**: Quantifies the spatial overlap by thresholding (top 15-20% brightest pixels) and measuring how much of the target protein is inside the Nucleus, on Microtubules, or on the ER.
*   **Compartment Intensities (3 features)**: Mean target protein (Green) intensity strictly inside vs. outside the reference organelle masks.

This is processed concurrently across all available system cores using Python's `ProcessPoolExecutor`.

### 3. Machine Learning Modeling
*   **Multi-Class Strategy**: The binary target labels (**Mitochondria**, **Nucleoli**) are mapped to three mutually exclusive classes:
    *   `0`: Target protein is in Mitochondria only
    *   `1`: Target protein is in Nucleoli only
    *   `2`: Target protein is present in both organelles
*   **Model**: Random Forest Classifier (`n_estimators=300`)
*   **Validation**: 5-Fold Stratified Cross-Validation to guarantee stable, un-biased metrics.
*   **Inference**: Fits on the complete labeled set, generates predictions on the unlabeled test samples, and saves the output back to Google Drive as `predicted_protein_labels.csv`.

---

## Getting Started (Colab Workflow)

1.  **Mount Google Drive** to authenticate your storage access.
2.  **Run the copy block** to transfer the dataset to the local SSD runtime memory:
    ```python
    !cp -r "/content/drive/MyDrive/BMED3201_Projects/Protein_Localization/protein_loc_project_data" "/content/protein_loc_project_data"
    ```
3.  **Execute the classification script** to perform feature extraction, cross-validation, and wri
