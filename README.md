# 🔬 License Plate Super-Resolution via SVD & Truncated SVD

<p align="center">
  <img src="https://img.shields.io/badge/MATLAB-R2020b-orange?style=for-the-badge&logo=mathworks&logoColor=white"/>
  <img src="https://img.shields.io/badge/Domain-Image%20Processing-blue?style=for-the-badge&logo=opencv&logoColor=white"/>
  <img src="https://img.shields.io/badge/Method-SVD%20%7C%20TSVD-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge"/>
</p>

---

## 📌 Project Overview

This project addresses a real-world **image super-resolution problem** applied to **vehicle license plates** captured from CCTV footage or handheld cameras.

### 🎯 The Problem
Cameras capturing **moving vehicles** often produce:
- 🌫️ Blurry / motion-degraded images
- 📉 Low-resolution number plates
- ❌ Unreadable characters due to undersampling

### 💡 The Solution
Rather than collecting privacy-sensitive data from live CCTV systems, data was collected **manually** and the algorithm focuses **exclusively on the license plate region** as the region of interest (ROI). The goal is to **reconstruct a high-resolution image from a low-resolution (degraded) observation** using linear algebra — specifically the **Moore-Penrose Pseudoinverse** via **SVD** and **Truncated SVD (TSVD)**.

---

## 🗂️ Repository Structure

```
📁 license-plate-super-resolution/
│
├── 📄 feed_29.bmp          # Input image — vehicle photo with license plate
├── 📄 RUN_ME.m             # Core pipeline: downscaling operator construction (cameraman demo)
├── 📄 RUN_ME_edit.m        # Extended pipeline: Normal Equation + SVD upscaling on feed_29.bmp
├── 📄 Upscale_test.m       # Full upscaling test with SVD pseudoinverse on cropped ROI
├── 📄 tsvd.m               # Custom Truncated SVD (TSVD) function
└── 📄 README.md            # Project documentation
```

---

## ⚙️ Mathematical Pipeline

The super-resolution problem is framed as a **linear inverse problem**:

> **b = A · x**

Where:
| Symbol | Meaning |
|--------|---------|
| **x** | Unknown high-resolution image (vectorized) |
| **A** | Downscaling operator (sparse matrix) |
| **b** | Observed low-resolution image (vectorized) |

### 🔢 Step-by-Step Algorithm

```
1. 📷  Image Acquisition
       └── Load feed_29.bmp → Convert to grayscale → Crop license plate ROI

2. 🏗️  Construct Downscaling Operator A  (Sparse Matrix)
       ├── Nearest-Neighbor: picks every 2nd pixel (subsampling)
       └── Bilinear (Linear): averages 2×2 pixel blocks → weights = 1/4

3. 📉  Simulate Degradation
       └── b = A · x  →  produces low-resolution observation

4. 📐  Formulate Normal Equation
       └── A'A · x = A'b  (converts underdetermined system to symmetric form)

5. 🔬  Solve via SVD (Full Pseudoinverse)
       └── B = A'A
           [U, S, V] = svds(B)
           A† = V · S⁻¹ · U'     (Moore-Penrose Pseudoinverse)
           x̂  = A† · (A'b)

6. ✂️  Solve via TSVD (Regularized)
       └── Keep only singular values > threshold (0.1)
           Truncate remaining → reduces noise amplification
           x̂_TSVD = V_r · S_r⁻¹ · U_r' · b

7. 🖼️  Reconstruct & Visualize
       └── Reshape solution vector → display super-resolved image
```

---

## 📁 File Descriptions

### `RUN_ME.m` — Downscaling Demo (Baseline)
> A clean, minimal demonstration of the downscaling operator on the standard `cameraman.tif` image.

- Builds sparse operator **A** using **nearest-neighbor** or **bilinear** interpolation
- Visualizes the **sparsity pattern** of A (`spy` plot)
- Produces and displays a **downscaled image** alongside the original
- 📌 *Use this to understand the forward problem before running the full pipeline*

---

### `RUN_ME_edit.m` — Full Inverse Problem on License Plate
> The main experimental file applied to `feed_29.bmp`.

- Crops the license plate region: `img = A1(451:550, 1251:1400)` — a 100×150 patch
- Constructs the **linear operator A** using bilinear interpolation
- Solves the **Normal Equation**: `B = A'A`, `f = A'b`
- Computes the **SVD pseudoinverse**: `A† = V · S⁻¹ · U'`
- Reconstructs the upscaled image from the degraded observation
- ⚠️ *Contains commented-out TSVD block for experimentation*

---

### `Upscale_test.m` — Upscaling Validation Test
> A self-contained test that runs the full downscale → upscale pipeline on a larger ROI.

- Uses a **200×200 crop** of `feed_29.bmp`: `A1(401:600, 1201:1400)`
- Constructs A, downscales, then **inverts via SVD pseudoinverse**
- Visualizes: original → `A'b` approximation → final SVD reconstruction
- 📌 *Good reference for validating reconstruction quality*

---

### `tsvd.m` — Truncated SVD Function
> A custom regularization utility to suppress noise amplification.

```matlab
function [U, S, V] = tsvd(A)
```
- Uses `svds(A, 20)` — computes the **top 20 singular triplets**
- **Thresholds** singular values: keeps `σᵢ` only if `σᵢ > 0.1`, zeroes out the rest
- Returns the **filtered** U, S, V for use in a regularized pseudoinverse
- 📌 *Truncation prevents division by near-zero singular values, which would amplify noise*

---

## 🖼️ Visual Pipeline

```
┌─────────────────────┐
│  feed_29.bmp (RGB)  │
│  Full vehicle image │
└────────┬────────────┘
         │  rgb2gray + crop ROI
         ▼
┌─────────────────────┐
│  Grayscale Patch    │  ← License plate region only
│  (100×150 pixels)   │
└────────┬────────────┘
         │  Apply operator A (bilinear)
         ▼
┌─────────────────────┐
│  Low-Res Image b    │  ← 50×75 pixels (downscaled 2×)
└────────┬────────────┘
         │  Solve Normal Equation A'Ax = A'b
         ▼
┌────────────────────────────────────┐
│  SVD  →  A† = V · S⁻¹ · U'        │
│  TSVD →  threshold σᵢ < 0.1       │
└────────┬───────────────────────────┘
         │  x̂ = A† · (A'b)
         ▼
┌─────────────────────┐
│  Super-Resolved     │  ← Reconstructed 100×150 image
│  License Plate      │
└─────────────────────┘
```

---

## 🚀 How to Run

### Prerequisites
- MATLAB (R2019b or later recommended)
- Image Processing Toolbox
- `feed_29.bmp` in the same directory as the scripts

### Execution Order

```matlab
% Step 1 — Understand the forward problem (downscaling only)
run('RUN_ME.m')

% Step 2 — Full inverse problem on license plate
run('RUN_ME_edit.m')

% Step 3 — Upscaling validation test
run('Upscale_test.m')

% Note: tsvd.m is a helper function, called automatically
```

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| **Inverse Problem** | Recover x from b = Ax where A is known |
| **Normal Equation** | A'Ax = A'b — transforms to symmetric solvable form |
| **Moore-Penrose Pseudoinverse** | A† = V S⁻¹ U' — least-squares solution |
| **SVD** | Singular Value Decomposition — factorizes A into U, S, V |
| **TSVD** | Truncated SVD — regularization by discarding small singular values |
| **Sparse Operator A** | Efficiently encodes the downsampling process as a matrix |

---

## 📚 Academic Context

This project was completed as part of the **MSc Computational Mathematics** program at **Friedrich-Alexander-Universität Erlangen-Nürnberg (FAU)**, Germany.

The work is grounded in the theory of **ill-posed linear inverse problems** and **regularization methods**, with practical application to the image reconstruction domain.

> 💡 *The TSVD approach is particularly important here because A'A is often nearly singular — direct inversion amplifies small errors catastrophically. Truncation stabilizes the solution.*

---

## 👨‍💻 Author

**Spondon** — MSc Computational Mathematics, FAU Erlangen-Nürnberg
Original framework by: **Daniel Tenbrinck** (daniel.tenbrinck@fau.de), 14.08.2020

---

## 📄 License

This project is for **academic and educational purposes only**.


---

## 🔒 Code Availability
AS this project was completed under academic supervision at FAU Erlangen-Nürnberg.  
Full source code is available **upon request** — feel free to reach out via
[LinkedIn]((https://www.linkedin.com/in/spondon-sarker-553544196/)) or write me over the email.
