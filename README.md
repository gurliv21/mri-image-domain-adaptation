# Unsupervised Domain Adaptation for Medical Image Segmentation

## Overview

This project investigates **unsupervised domain adaptation (UDA)** for medical image segmentation across different MRI scanners.

The model is trained using **labeled GE 3T MRI scans** and adapted to **unlabeled Philips 3T MRI scans**.

The main challenge is the **scanner-domain shift**: although both datasets contain the same type of medical images, differences in image appearance and scanner characteristics can cause a segmentation model trained on GE scans to perform poorly on Philips scans.

This project evaluates several adaptation strategies, including:

- Source-only training
- DANN (Domain-Adversarial Neural Network)
- Self-training
- Histogram matching
- Feature-space adversarial adaptation
- Output-space adversarial adaptation
- Combined histogram matching and output-space adversarial adaptation

---

## Problem Setup

```text
Source Domain: GE 3T
├── MRI images
└── Ground-truth segmentation masks

Target Domain: Philips 3T
└── MRI images only
    (no segmentation labels during training)
```

The goal is to improve segmentation performance on the **unlabeled Philips domain** without using Philips ground-truth masks during training.

---

## Method

A U-Net segmentation model is used as the base architecture.

Different domain adaptation strategies are evaluated to determine which component best addresses the GE → Philips scanner shift.

### Histogram Matching

Histogram matching modifies the **pixel-intensity distribution** of GE images to make their visual appearance more similar to Philips images while preserving the original GE segmentation masks.

### DANN

DANN performs **feature-space domain adaptation** using a domain classifier and Gradient Reversal Layer (GRL). The encoder is encouraged to learn features that are less dependent on the scanner domain.

### Output-Space Adversarial Adaptation

Instead of aligning internal features, the discriminator operates on the **segmentation predictions**.

The discriminator attempts to distinguish source-domain predictions from target-domain predictions, while the segmentation network learns to make target predictions appear more similar to source predictions.

### Combined Approach

The strongest configuration combines:

```text
Histogram Matching
        +
Output-Space Adversarial Adaptation
```

This achieved the best target-domain performance in the current experiments.

---

## Results

| Step | Method | Philips Dice | Gap Closed |
|---:|---|---:|---:|
| 0 | Source-only (GE) | 0.8433 | 0% |
| 1 | Control: more training, no DANN (λ=0) | 0.806 | -27% |
| 2 | DANN, λ=0.1 | 0.896 | 38% |
| 3 | DANN, λ=0.3 | 0.898 | 39% |
| 4 | DANN λ=0.3 + self-training | 0.898 | 40% |
| 5 | Histogram matching only (λ=0) | 0.926 | 59% |
| 6 | DANN λ=0.3 + histogram matching | 0.940 | 70% |
| 7 | Output-space adversarial (control, adv weight 0) | 0.939 | 69% |
| **8** | **Output-space adversarial + histogram matching** | **0.9618** | **85%** |
| 9 | Oracle | 0.983 | 100% |

### Best Result

**Output-space adversarial adaptation + histogram matching**

- **Philips Dice:** **0.9618**
- **Gap closed:** **85%**
- **Oracle Dice:** 0.983

The combined approach substantially reduces the performance gap between source-only segmentation and the fully supervised target-domain oracle.

---

## Key Findings

### 1. DANN improves cross-scanner generalization

Source-only training achieved a Philips Dice of **0.8433**.

DANN increased this to approximately **0.898**, showing that feature-level domain alignment helps reduce the GE → Philips domain shift.

### 2. Self-training provided little additional benefit

Adding self-training to DANN resulted in approximately **0.898 Dice**, indicating that pseudo-label-based adaptation did not provide a substantial improvement in the current setup.

### 3. Histogram matching was highly effective

Histogram matching alone increased Philips Dice to **0.926**.

This suggests that **scanner-specific intensity and appearance differences are an important component of the domain shift**.

### 4. Combining approaches produced the strongest result

The combination of histogram matching and output-space adversarial adaptation achieved:

> **0.9618 Philips Dice**

This corresponds to approximately **85% of the source-to-oracle performance gap being closed**.

---

## Comparison

```text
Philips Dice

Source-only                         0.8433
DANN                                0.898
Histogram Matching                  0.926
DANN + Histogram Matching           0.940
Output-space Adversarial            0.939
Output-space Adv. + Histogram      0.9618
Oracle                              0.983
```

The results suggest that **input-level intensity harmonization and output-space adaptation can complement each other**.

---

## Research Question

The project investigates:

> **Can unsupervised domain adaptation improve medical image segmentation when the target scanner has no segmentation labels available?**

The experiments further investigate whether the remaining domain gap is better addressed through:

- feature-level alignment,
- image-intensity harmonization, or
- prediction-space alignment.

---

## Technologies

- Python
- PyTorch
- U-Net
- Domain-Adversarial Neural Networks (DANN)
- Gradient Reversal Layer (GRL)
- Histogram Matching
- Output-Space Adversarial Adaptation
- MRI Image Segmentation
- Unsupervised Domain Adaptation

---

## Dataset Setup

| Domain | Scanner | Labels | Role |
|---|---|---|---|
| Source | GE 3T | Available | Model training |
| Target | Philips 3T | Unavailable during training | Domain adaptation/testing |

The Philips labels are **not used during adaptation training**. They are used only for evaluation.

---

## Conclusion

The experiments demonstrate that scanner-related domain shift can significantly reduce segmentation performance when transferring between MRI scanners.

While DANN improves generalization, **histogram matching provides a larger improvement in this particular GE → Philips setting**. The strongest result is obtained by combining histogram matching with **output-space adversarial adaptation**, achieving a Philips Dice of **0.9618** and closing approximately **85% of the source-to-oracle performance gap**.

The results indicate that addressing both **image appearance differences** and **segmentation-output differences** can be more effective than relying on feature-level adversarial alignment alone.
