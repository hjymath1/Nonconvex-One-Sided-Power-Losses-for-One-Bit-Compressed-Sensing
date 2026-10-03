# OSLp-BIHT Experiments

Reproducibility package for the numerical experiments accompanying

**Nonconvex One-Sided Power Losses for One-Bit Compressed Sensing**

by Chunyu Liu, Xiaoxuan Peng, Youpeng Ye, and Jingyao Hou.

This repository implements the stabilized one-sided residual-power BIHT
(OSLp-BIHT) iteration, the \(p=1\) BIHT-type endpoint, Gaussian one-bit
data generation, random / high-margin / low-margin sign corruption,
correlation initialization, support metrics, iteration-level weighting
diagnostics, and the controlled AOP-style sign-correction variants used in
the manuscript.

## What is reproduced here

The package contains scripts corresponding to the core numerical sections:

- residual-exponent sweep under clean, random, high-margin, and low-margin corruption;
- iteration-level weight and correction diagnostics;
- initialization and support-recovery diagnostics;
- flip-fraction sweeps for BIHT / finite-\(p\) OSLp-BIHT;
- AOPF-BIHT versus AOPF-OSLp;
- measurement-scaling experiments.

The manuscript also compares with **OSL0** and **HOW-1s**. Their full
executable specifications are not contained in the manuscript source used
to build this repository, so they are intentionally **not reimplemented
from inference**. See `third_party/README.md` for adapter guidance.

## Manuscript defaults

Unless otherwise stated, the manuscript uses

- `n = 500`
- `s = 10`
- `m = 1000`
- `p in {1, 0.8, 0.6, 0.4, 0.2}`
- `epsilon = 0.2`
- `mu = 2`
- flip fraction `q = 0.1`
- 50 Monte Carlo trials
- normalized angular error
  \[
  \pi^{-1}\arccos(\widehat x^\top x_\star).
  \]

The \(\ell_0\)-comparison and measurement-scaling experiments use
`n = 1000`, `s = 10`.

### One manuscript detail to confirm before archival release

The current manuscript source specifies an iteration count \(T\) in the
algorithm but does not state the numerical value used in the experiments.
This repository therefore exposes it in `configs/default.json` and uses
`iterations = 200` only as a transparent implementation default.

**Before tagging the exact paper-reproduction release, replace this value
with the iteration count used to generate the published figures.**

## Installation

Python 3.10+ is recommended.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -e .
```

On Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
pip install -e .
```

## Quick smoke test

```bash
pytest -q
python experiments/run_p_sweep.py --trials 3 --iterations 20 --quick
```

The generated CSV and PDF files are written under `results/`.

## Reproducing the core experiments

Use the manuscript-scale defaults:

```bash
python experiments/run_p_sweep.py
python experiments/run_mechanism.py
python experiments/run_init_support.py
python experiments/run_support_recovery.py
python experiments/run_flip_fraction.py
python experiments/run_aopf_comparison.py
python experiments/run_measurement_scaling.py
```

Or run the convenience script:

```bash
bash reproduce_core.sh
```

All random experiments are seed-controlled. The base seed can be changed
with `--seed`.

## Algorithm

For residuals

\[
r_i^t=[-y_i a_i^\top x^t]_+,
\]

the stabilized residual weight is

\[
w_i^t=\left(1+\frac{r_i^t}{\varepsilon}\right)^{p-1}.
\]

The pre-threshold update is

\[
z^{t+1}
=
x^t+\frac{\mu}{m}
\sum_{i:r_i^t>0}w_i^t y_i a_i,
\]

followed by hard thresholding to the largest `s` entries and normalization
to unit Euclidean norm. At `p = 1`, all violated constraints have unit
weight and the method reduces to the BIHT-type endpoint used in the
experiments.

## Corruption models

Let

\[
y_i^\star=\operatorname{sign}(a_i^\top x_\star),\qquad
\gamma_i=|a_i^\top x_\star|.
\]

For `k = round(q*m)` flips:

- `random`: choose `k` indices uniformly without replacement;
- `high`: flip the `k` largest clean margins \(\gamma_i\);
- `low`: flip the `k` smallest clean margins \(\gamma_i\);
- `clean`: no flips.

The margin-selective models are oracle constructions used to isolate the
effect of corruption location.

## Repository layout

```text
src/oslp_biht/
    algorithms.py      OSLp-BIHT and AOP-style corrected variant
    data.py            sparse signal, Gaussian sensing, sign corruption
    metrics.py         angular and support metrics
    utils.py           thresholding, normalization, CSV helpers

experiments/
    run_p_sweep.py
    run_mechanism.py
    run_init_support.py
    run_support_recovery.py
    run_flip_fraction.py
    run_aopf_comparison.py
    run_measurement_scaling.py

configs/default.json
docs/REPRODUCIBILITY.md
third_party/README.md
tests/
```

## Reproducibility notes

Each Monte Carlo trial constructs one signal, one sensing matrix, and one
corrupted bit vector, then reuses them across all methods compared within
that trial. This paired design matches the manuscript description.

The plotting scripts save both the aggregated numerical results (`.csv`)
and publication-friendly figures (`.pdf` and `.png`) so that plots can be
regenerated without rerunning the Monte Carlo simulations.

## Citation

See `CITATION.cff`.

## License

No software license is imposed by this generated package. Add the license
selected by the authors before public release (for example MIT, BSD-3-Clause,
or GPL-3.0).
