# Brain Tumor Classification: CNNs, Transfer Learning, and Conformal Prediction

## Overview
This project aims to develop an automated classification system for brain Magnetic Resonance Imaging (MRI) scans to detect three types of brain tumors: **Meningioma, Glioma, and Pituitary tumors**. 

The study compares the performance of four different Convolutional Neural Network (CNN) architectures, exploring custom-built models, data augmentation, class-weight balancing, and advanced **Transfer Learning** techniques (VGG16). 

Crucially, to ensure reliability in a medical context, the best-performing model is integrated with **Split Conformal Prediction** to quantify diagnostic uncertainty and provide mathematically guaranteed prediction sets.

## Dataset
The dataset was originally published by Jun Cheng (2017) and sourced from [Kaggle](https://www.kaggle.com/datasets/denizkavi1/brain-tumor). It contains **3,064 MRI images** divided into three highly imbalanced classes:
* **Meningioma (Class 1):** 708 images (Minority class)
* **Glioma (Class 2):** 1,426 images (Majority class)
* **Pituitary Tumor (Class 3):** 930 images

<img width="776" height="278" alt="mri_samples.png" />

## Models & Methodology
To address the challenges of medical image classification and class imbalance, four different approaches were sequentially implemented and evaluated:

### 1. Simple CNN (Baseline)
A custom 4-block convolutional architecture trained from scratch. 
* **Results:** Reached **94.1% accuracy** on the test set. 
* **Limitation:** Exhibited overfitting and severe bias towards the majority class. Sensitivity for Meningioma was only 85.9%, compared to >96% for the other classes.

### 2. Regularized CNN (Handling Imbalance)
To counteract overfitting and class imbalance, three techniques were introduced: **Data Augmentation**, **Dropout**, and **Class Weights** in the loss function.
* **Results:** Overall accuracy slightly dropped to **92.6%**, but the model achieved a **balanced accuracy (93-97%)** across all categories. Sensitivity for Meningioma increased significantly from 85.9% to 93.4%. 
* **Insight:** In a clinical setting, a fair and balanced model is vastly preferable to one that maximizes overall accuracy by sacrificing minority class detection.

### 3. VGG16: Feature Extraction
Leveraged Transfer Learning using the VGG16 base pre-trained on ImageNet as a static feature extractor, adding a new dense classifier on top.
* **Results:** Achieved **91.1% accuracy**.
* **Limitation:** Struggled to differentiate Meningiomas (Precision: 78.5%), producing many false positives. The VGG16 filters, optimized for natural images, were not perfectly suited for the specific textures of brain MRIs without further adaptation.

### 4. VGG16: Fine-Tuning (Best Model)
To adapt the pre-trained features to the medical domain, the last convolutional block (`block5`) of VGG16 was "unfrozen" and trained alongside the classifier. 
<img width="666" height="480" alt="loss_accuracy_curves.png" />

**Performance on Test Set:**
* **Accuracy:** 95.0% (95% CI: 0.925 - 0.968)
* **Kappa:** 0.921

**Confusion Matrix:**

| Prediction \ True Class | 0 (Glioma) | 1 (Meningioma) | 2 (Pituitary) |
| :--- | :--- | :--- | :--- |
| **0 (Glioma)** | **206** | 8 | 3 |
| **1 (Meningioma)** | 6 | **96** | 2 |
| **2 (Pituitary)** | 2 | 2 | **134** |

**Statistics by Class:**
* **Glioma (0):** Sensitivity 96.3% | Specificity 95.5% | Balanced Accuracy 95.9%
* **Meningioma (1):** Sensitivity 90.6% | Specificity 97.7% | Balanced Accuracy 94.2%
* **Pituitary (2):** Sensitivity 96.4% | Specificity 98.8% | Balanced Accuracy 97.6%

*Insight: The Fine-Tuned model successfully overcame the initial bias against the minority class (Meningioma), achieving >90% sensitivity across all categories, making it highly robust for clinical decision support.*
## Uncertainty Quantification: Conformal Prediction
Medical AI requires strict reliability. Instead of providing absolute point predictions (which can be misleading in ambiguous cases), **Split Conformal Prediction** was applied to the Fine-Tuned VGG16 model.

Given a confidence level of $1-\alpha$, this framework outputs a *Prediction Set* guaranteed to contain the true diagnosis with at least $1-\alpha$ probability.
* **Target Coverage:** 95%
* **Empirical Coverage:** **95.86%** (Theoretical guarantee successfully met).
* **Average Set Size:** **1.026**. This indicates high model confidence; in almost all cases, the prediction set contained only one class, effectively managing uncertainty without sacrificing diagnostic utility.

## 📂 Repository Structure
* `Brain_Tumor_CNN.Rmd`: The complete source code containing data preprocessing, model architectures, training loops, and evaluation metrics.
* `Brain_Tumor_CNN.md`: The rendered output of the analysis containing all visualizations, loss curves, and confusion matrices.

## 💻 Tech Stack
* **Language:** R
* **Deep Learning Framework:** Keras / TensorFlow
* **Methodologies:** CNNs, Transfer Learning, Data Augmentation, Conformal Prediction.
