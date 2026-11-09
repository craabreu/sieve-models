# sieve-models

Pretrained [Sieve](https://github.com/craabreu/sieve) models for predicting
atomic partial charges, fitted to the DASH dataset.

## Models

| Model | Training data | Records | Atoms |
|---|---|---:|---:|
| `dash-qmugs` | QMugs subset of DASH (higher quality) | 514,735 | 8,091,819 |
| `dash-extra` | remaining DASH subsets (lower quality) | 510,109 | 6,476,329 |

Using both models amounts to a model fitted to the full DASH dataset.

All models share one configuration: element-only atom attributes, WL depth 5,
the `continuation` class estimator and empirical-Bayes shrinkage. Their element
vocabulary includes boron, which only `dash-extra` has seen.

## Layout

```
models/<model-name>/part-1-of-3.npz
                    part-2-of-3.npz
                    part-3-of-3.npz
                    manifest.json
```

Each model is split into parts that fit under GitHub's file-size limit. A part
is a complete Sieve model fitted to a subset of the training data, so parts
merge exactly: the sum of a model's parts is that model. Since every part has
the same configuration, parts from different models merge too.
`manifest.json` lists the cross-validation shard fits that make up each part.

## Usage

Clone this repository and install Sieve:

```bash
git clone https://github.com/craabreu/sieve-models.git
pip install "sieve[chem] @ git+https://github.com/craabreu/sieve.git"
```

Then, from the repository root, merge the parts of the models you want:

```python
from pathlib import Path

import sieve

model = sum(
    sieve.SieveModel.load(part)
    for name in ["dash-qmugs", "dash-extra"]
    for part in sorted(Path("models", name).glob("part-*.npz"))
)
```

[`predict-charges.ipynb`](predict-charges.ipynb) carries on from there: it
predicts charges and their variances for a few drug molecules, optionally
corrects each molecule's charges to sum to its formal charge, and draws them.
