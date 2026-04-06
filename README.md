# DMD for Neuroscience — Learning Notes & Insights

A structured collection of notes and findings from exploring Dynamic Mode Decomposition (DMD) and its variants for resting-state EEG analysis. All implementations use [PyDMD](https://github.com/PyDMD/PyDMD) directly.

> **PyDMD Documentation:** https://pydmd.github.io/PyDMD/dmdbase.html

---

## Why DMD? Advantages Over Other Decomposition Methods

Neural signals are high-dimensional, noisy, and fundamentally *dynamic* — they aren't just patterns in space, they are patterns that evolve in time. Most classical decomposition methods capture one of these dimensions well but not both. DMD is designed for exactly this problem.

### DMD vs Common Alternatives

| Method | What It Captures | Key Limitation |
|---|---|---|
| **PCA / SVD** | Spatial variance across time | No temporal dynamics; treats time as a nuisance |
| **ICA** | Statistically independent spatial components | No frequency or temporal evolution information |
| **Fourier (FFT)** | Global frequency content | Assumes stationarity; no spatial structure |
| **Wavelets** | Time-frequency content at fixed scales | Basis is fixed, not learned from data |
| **NMF** | Non-negative spatial + temporal factors | No dynamical model; no frequency interpretation |
| **DMD** | Spatiotemporal modes with frequency + growth rate | Assumes linear dynamics (addressed by variants) |

> DMD can be thought of as performing PCA in space and power spectral analysis in time — simultaneously, in a single decomposition.

### What Makes DMD Uniquely Suited for Neuroscience

**1. Spatiotemporal modes in a single step**
Each DMD mode is simultaneously a spatial pattern *and* a temporal oscillation — you get both where activity is happening and how it evolves in time, jointly. PCA gives you spatial components; FFT gives you frequencies. DMD gives you both at once.

**2. Modes are oscillatory by construction**
Every DMD mode has a well-defined frequency and growth/decay rate encoded in its eigenvalue. This maps naturally onto neural rhythms (delta, theta, alpha, beta, gamma) without imposing a fixed frequency basis on the data.

**3. Data-driven — no basis assumption**
Unlike wavelets or Fourier methods, DMD learns its decomposition basis from the data itself. It adapts to whatever spatiotemporal structure is actually present, rather than projecting onto a predetermined set of functions.

**4. Fits a forward dynamical model**
DMD doesn't just decompose snapshots — it fits a linear operator *A* such that x_{k+1} ≈ Ax_k. This means you get a model of how the system *evolves*, enabling prediction, control design, and characterization of brain state stability — not just a static picture.

**5. Koopman operator connection**
DMD approximates the Koopman operator — a rigorous mathematical framework for studying nonlinear systems through a linear lens. This gives DMD strong theoretical grounding for brain dynamics research, where the underlying system is nonlinear but a linear approximation over short windows is tractable.

**6. Modes can grow or decay**
Unlike FFT (which assumes constant-amplitude oscillations), DMD modes can grow or decay over time. This is more realistic for transient neural events like bursts, state transitions, or evoked responses.

---

## The Core Problem: DMD Breaks Down Over Long Windows

DMD gives you rich spatiotemporal information — but only reliably within a short time window, typically **less than 5 seconds** for biological data. This is because DMD assumes the underlying dynamics are linear and stationary within the window it analyzes. For resting-state EEG, this assumption quickly fails:

- The brain is never truly stationary — cognitive states drift continuously without discrete boundaries
- Neural dynamics are non-linear; a single linear operator can only approximate them locally
- The best-fit modes for one 3-second window may look completely different from the next — not because the brain changed dramatically, but because DMD has no way of knowing they represent the same underlying process

**The result:** If you apply DMD window-by-window, you get a pile of disconnected local decompositions. The "alpha mode" in window 1 and the "alpha mode" in window 3 might be the same neural phenomenon drifting slightly in frequency or spatial distribution — but standard DMD treats them as unrelated. There is no mechanism to stitch them into a coherent long-horizon picture.

This is the fundamental bottleneck, and it is not solved by simply choosing a better DMD variant.

---

## DMD Variants — Addressing Specific Limitations

Each variant below addresses a specific weakness of standard DMD, but none directly solves the mode-coherence problem across time.

### Overview

| Variant | Key Idea | Primary Strength |
|---|---|---|
| **Standard DMD** | Least-squares regression on state transitions | Baseline; degrades fastest with window length |
| **Optimized DMD** | Rank-constrained optimization; minimizes reconstruction error directly | Most noise-robust; recommended starting point |
| **Extended DMD (EDMD)** | Augments state with nonlinear observable functions | Captures non-linear brain dynamics |
| **Multi-Resolution DMD (mrDMD)** | Decomposes across multiple temporal resolutions | Isolates co-existing rhythms (delta through gamma) |
| **Wavelet DMD (WDMD)** | Merges wavelet transforms with DMD | Local + global frequency features simultaneously |
| **Total Least Squares DMD (TLS-DMD)** | Assumes noise corrupts all measurements, not just outputs | Reduces bias in universally noisy EEG |

### Detailed Notes

**Optimized DMD**
Recasts DMD as a rank-constrained optimization problem, directly minimizing reconstruction error rather than relying on standard least-squares. This distributes errors more uniformly across modes, making it significantly more robust to noise. It is the recommended starting point for biological data and, notably, serves as the backbone of NS-DMD (see below).

**Extended DMD (EDMD)**
Standard DMD can only capture linear dynamics. EDMD addresses this by augmenting the state space with nonlinear observable functions (e.g., polynomials, radial basis functions), enabling a better approximation of the Koopman operator. This is the primary route for handling the non-linearity of neural signals without abandoning the DMD framework.

**Multi-Resolution DMD (mrDMD)**
Combines multiresolution analysis with exact DMD to decompose time-series data into multiple levels of temporal resolution. Well-suited for EEG, which is inherently multi-scale — slow delta rhythms and fast gamma oscillations co-exist and require different temporal resolutions to isolate cleanly.

**Wavelet DMD (WDMD)**
Merges wavelet transforms with DMD to capture both local (high-frequency, short-duration) and global (low-frequency, sustained) features simultaneously. Complementary to mrDMD — where mrDMD separates scales hierarchically, WDMD integrates them within a single decomposition.

**Total Least Squares DMD (TLS-DMD)**
Standard DMD assumes noise only affects the output snapshot, not the input. TLS-DMD relaxes this by treating all measurements as noisy — far more realistic for EEG, where every electrode measurement is corrupted by sensor noise, muscle artifacts, and environmental interference.

---

## Key Findings

- **Optimized DMD** was the most consistently noise-robust method across resting-state conditions.
- **Non-stationarity** — specifically the inability to track modes coherently across time — was the primary bottleneck limiting long-horizon performance across all variants, including those designed for noise and non-linearity.
- **mrDMD and WDMD** captured co-existing frequency bands more faithfully than single-resolution methods, but remained sensitive to drifting dynamics over longer windows.
- **Standard DMD** degraded fastest with window length, confirming its unsuitability for long-horizon resting-state decomposition.
- Most DMD methods work accurately only for **short windows (< 5 seconds)** on biological data — a fundamental constraint regardless of variant choice.

---

## The Open Problem: NS-DMD

Non-Stationary DMD (NS-DMD) directly addresses the mode-coherence problem that no standard variant solves.

Its approach: run Optimized DMD on each short window independently, then apply a matching algorithm to identify which modes *recur and persist* across windows. Modes that appear consistently — even if drifting slightly in frequency or spatial distribution (e.g., 10Hz → 10.2Hz → 10.4Hz across consecutive windows) — are linked into a single continuous trajectory. Modes that are spurious or window-specific are discarded. The result is a small set of **global, coherent spatiotemporal modes** that describe how the system evolves over the full recording.

This is precisely what's needed for resting-state EEG: not just a decomposition at each moment, but a coherent picture of which modes persist, drift, and modulate across the entire session.

> NS-DMD paper: https://ieeexplore.ieee.org/document/10288443
> NS-DMD implementation: https://github.com/learning-2-learn/nsdmd

---

## Tools & Libraries

- **[PyDMD](https://github.com/PyDMD/PyDMD)** — all DMD implementations used directly from this library
- **[NS-DMD](https://github.com/learning-2-learn/nsdmd)** — proposed extension for non-stationary signals



