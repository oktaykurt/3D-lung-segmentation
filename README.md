# 3D Lung Segmentation

## Project Overview

This project implements a 3D U-Net model for segmenting lungs from CT scans. It aims to achieve accurate segmentation, focusing on reducing overfitting and enhancing generalization through a deep architecture and robust data augmentation techniques. The model was developed using TensorFlow/Keras and trained on the LCTSC dataset from The Cancer Imaging Archive (TCIA), originally part of the AAPM Thoracic Auto-segmentation Challenge.

![4d4af3df-8c4a-4dd2-866d-b0cb26b45ae5](https://github.com/user-attachments/assets/af353581-d1a3-49fb-8619-ab4fedafcb53)
![119eb1cd-c85c-4384-9b3b-77106c97f7ee](https://github.com/user-attachments/assets/0fec19e8-9bf9-4511-b301-5bd5821e60b2)
![fb484015-b1a5-4a58-9087-413411a1c563](https://github.com/user-attachments/assets/6c7fc4c0-6040-44bb-8b5a-3fd953a893ef)
![cdae8e9d-ca2c-431f-a8dd-7ad31e57498e](https://github.com/user-attachments/assets/bac931aa-f9df-48b0-aed6-7e8285a33ad7)

For a detailed explanation of the methodology and results, please see the full report:
[View Full Project Report (PDF)](https://github.com/oktaykurt/3D-lung-segmentation/blob/main/Lung_Segmentation_Report.pdf)

## Dataset

* **Source:** [LCTSC Dataset on The Cancer Imaging Archive (TCIA)](https://www.cancerimagingarchive.net/collection/lctsc/)
* **Content:** Thoracic CT scans (`ct.nii`) and corresponding lung masks (`lung.nii`).
* **Preprocessing:**
    * HU intensity values are clipped to the range [-1000, 300] and normalized to [0, 1].
    * Lung masks are binarized (0 or 1).
* **Splitting:** The dataset is typically split into training (80%), validation (10%), and testing (10%) sets based on patient IDs.

## Data Citation

If you use this dataset in your research, please cite the original publication:

> Yang, J., Sharp, G., Veeraraghavan, H., Van Elmpt, W., Dekker, A., Lustberg, T., & Gooding, M. (2017). Data from Lung CT Segmentation Challenge (LCTSC) (Version 3) \[Data set]. The Cancer Imaging Archive. https://doi.org/10.7937/K9/TCIA.2017.3R3FVZ08

## Features

* **Model:** A 5-level 3D U-Net architecture processes input volumes of shape (64, 64, 64, 1).
    * Encoder blocks use two Conv3D layers (3x3x3 kernel) + Batch Normalization + ReLU, followed by MaxPooling3D.
    * Decoder blocks use Conv3DTranspose for upsampling and concatenate skip connections from the encoder.
    * Output layer uses a 1x1x1 Conv3D with sigmoid activation.
* **Regularization:**
    * Dropout (rate=0.5) is applied within convolution blocks.
    * L2 Weight Decay (1e-5) is used on convolutional layers.
* **Data Augmentation:**
    * Random flips along depth, height, and width axes.
    * Random 3D rotations (±10 degrees).
    * Random 3D elastic deformations using SimpleITK.
* **Loss Function:** Combined Binary Cross-Entropy and Dice Loss.
* **Metrics:** Dice Coefficient, Intersection over Union (IoU).
* **Training:**
    * Adam optimizer (initial LR=5e-4).
    * `ReduceLROnPlateau` callback to adjust learning rate based on validation Dice (patience=6).
    * `EarlyStopping` callback to prevent overfitting based on validation Dice (patience=15).
    * Custom callback (`ValidationPlotCallback`) to visualize predictions on a validation sample during training.

## Setup & Requirements

1.  **Clone the repository:**
    ```bash
    git clone <https://github.com/oktaykurt/3D-lung-segmentation>
    cd <https://github.com/oktaykurt/3D-lung-segmentation>
    ```
2.  **Install dependencies:**
    ```bash
    pip install tensorflow numpy matplotlib nibabel scipy simpleitk glob2 # Add other specific versions if needed
    ```
3.  **Prepare Data:**
    * Download the LCTSC dataset from TCIA.
    * Organize the data into patient folders, each containing `ct.nii` and `lung.nii`.
    * Update the `data_dir` variable in the `lung_segmentation.ipynb` notebook to point to your dataset location.

## Usage

### Training

1.  Ensure your data is set up correctly and the `data_dir` path is correct in the notebook.
2.  Run the `lung_segmentation.ipynb` notebook.
    ```bash
    jupyter notebook lung_segmentation.ipynb
    ```
3.  Execute the cells sequentially. Training progress (loss, Dice, IoU) will be printed, and validation plots will be generated periodically.
4.  The best model weights will be restored based on validation Dice due to `EarlyStopping`.
5.  The final trained model will be saved as `lung_seg_model.h5` (Note: This file is generated by the notebook but not included in the repository).

### Inference / Evaluation

1.  The notebook includes a final evaluation section that loads the saved model (`lung_seg_model.h5`).
2.  Executing these cells will iterate through the test set, perform inference, calculate test metrics (Loss, Dice, IoU), and visualize predictions against ground truth for each test patient.

## Results

The model achieved the following performance on the held-out test set:

* **Test Loss:** 0.1586
* **Test Dice Coefficient:** 0.8864
* **Test IoU:** 0.7962

Training curves and visualizations of validation/test predictions demonstrate the model's effectiveness in learning lung structures and generalizing to unseen data. Overfitting was effectively managed through the use of dropout, weight decay, and extensive data augmentation.

## Files Included

* `lung_segmentation.ipynb`: Jupyter Notebook containing the full code for data loading, preprocessing, augmentation, model definition, training, and evaluation.
* `Lung_Segmentation_Report.pdf`: A report detailing the project, methodology, results, and discussion.
* `/data/lctsc/` (Example): A directory containing a subset of the patient data structure (users need to replace this with the full dataset).

## Future Work

Potential improvements include:

* Exploring attention mechanisms or residual blocks in the U-Net.
* Training with larger input resolutions (e.g., 128x128x128) if hardware permits.
* Utilizing cross-validation or a larger validation set for more stable evaluation during training.
