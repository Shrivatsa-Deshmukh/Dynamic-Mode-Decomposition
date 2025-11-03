# Dynamic Mode Decomposition (DMD) and Its Variants

Dynamic Mode Decomposition (DMD) is a data-driven, equation-free technique used to extract coherent spatiotemporal structures from complex datasets. It decomposes a system's evolution into a set of **dynamic modes**, each associated with a specific temporal frequency and growth/decay rate.

This document provides an overview of DMD, its application to signals like EEG, and a summary of its most common and powerful variants.

## Python Implementation: PyDMD

A popular, full-featured Python library for DMD and its many variants.

* **GitHub Repository:** `https://github.com/PyDMD/PyDMD/tree/master`
* **Documentation:** `https://pydmd.github.io/PyDMD/dmdbase.html`

---

## Example Application: EEG Analysis

DMD is highly effective for analyzing complex, high-dimensional time-series data like EEG signals.

### Core Applications
* Isolating oscillatory patterns (e.g., alpha, beta, gamma waves) linked to specific cognitive states.
* Distinguishing between healthy and pathological brain activity by identifying abnormal modes.

### General Workflow
1.  **Construct Data Matrix:** Create a matrix `X` using "snapshots" of EEG signals over time.
2.  **Apply SVD:** Use Singular Value Decomposition (SVD) to reduce dimensionality and extract principal components.
3.  **Build Model:** Compute the best-fit linear operator `A` that advances the system in time (i.e., $x_{k+1} \approx A x_k$).
4.  **Analyze Modes:** Analyze the eigenvectors (DMD modes) and eigenvalues (temporal evolution) of the operator `A`.

---

## Key Challenges & Non-Stationary DMD (nsDMD)

While powerful, DMD and its variants face a major challenge in handling highly **non-stationary** and **non-linear** signals, which are characteristic of biological data like EEG.

* **Optimized DMD** (detailed below) often performs better than other variants for EEG analysis due to its robustness to noise.
* However, the underlying assumption of linear, stationary dynamics means most DMD methods only work accurately for very short time windows (often less than 5 seconds).
* **Non-Stationary DMD (nsDMD)** is a viable option proposed to resolve these issues by allowing the dynamic modes and eigenvalues to vary in time.

For more information on nsDMD, see this implementation:
`https://github.com/learning-2-learn/nsdmd`

---

## Advanced DMD Methodologies for EEG

> The following list details several DMD variants, **ordered by their potential importance for EEG analysis**. This list is not complete, and many other specialized variants exist, but these cover the primary challenges of non-stationarity, non-linearity, noise, and multi-scale dynamics.

### 1. Optimized Dynamic Mode Decomposition

* **Key Idea:** Recasts the problem as a **rank-constrained optimization**, directly minimizing the reconstruction error rather than relying on a standard least-squares regression.
* **Why It Matters for EEG:** This is a crucial variant for noisy data. It provides a more robust selection of modes by distributing errors more uniformly, making it a common foundation for other advanced methods (like nsDMD).

### 2. Extended Dynamic Mode Decomposition (EDMD)

* **Key Idea:** Augments standard DMD by using a larger set of **nonlinear observable functions** (e.g., polynomials, radial basis functions) instead of just the original state variables.
* **Why It Matters for EEG:** This is the primary DMD variant for tackling **non-linear dynamics**. It allows the method to approximate the Koopman operator more effectively, enabling the capture of complex brain dynamics that linear DMD would miss.

### 3. Wavelet-Based (WDMD) & Multi-Resolution (mrDMD)

* **Key Idea (WDMD):** Merges **wavelet transforms** with DMD to analyze multi-scale data, capturing both local (high-frequency) and global (low-frequency) features.
* **Key Idea (mrDMD):** Combines multiresolution analysis with Exact DMD to systematically decompose time-series data into **multiple levels of temporal resolution**.
* **Why It Matters for EEG:** These methods are ideal for EEG, which is inherently multi-scale (e.g., co-existing delta, theta, alpha, beta, and gamma rhythms). They can isolate dynamics evolving on vastly different timescales, which a single DMD model might blur or miss.

### 4. Adaptive Dynamic Mode Decomposition (ADMD)

* **Key Idea:** Incorporates time-delay coordinates, projection methods, and filtering (e.g., DFT) to **dynamically adjust** how data is processed.
* **Why It Matters for EEG:** This is another strong candidate for handling **non-stationary signals**. It allows the method to "track" changing dynamics and regime shifts (e.g., transitions between cognitive states) more effectively than a static method.

### 5. Total Least Squares (TLS) DMD

* **Key Idea:** Assumes that **all measurements are corrupted by noise**, not just the "output" data (as in standard least-squares).
* **Why It Matters for EEG:** EEG data is universally noisy. TLS-DMD reduces the bias introduced by standard OLS-based DMD when sensor error affects all snapshots, yielding a more accurate model in high-noise environments.

---

## Concluding Remarks

Each of these refinements to Dynamic Mode Decomposition addresses specific challenges encountered when analyzing complex dynamical systems. For EEG, the most critical variants are those that tackle **non-stationarity** (nsDMD, ADMD), **non-linearity** (EDMD), **noise** (Optimized DMD, TLS-DMD), and **multi-scale signals** (mrDMD, WDMD).

These methods provide more nuanced, accurate, and robust representations of the underlying system dynamics—beneficial for tasks such as prediction, control design, feature extraction, and theoretical insight into complex brain processes.
