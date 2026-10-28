# AuroraLabs LOCI on AWS Marketplace

Example material for the LOCI execution-time models sold as SageMaker model
packages on AWS Marketplace: one notebook per model, a small sample input, and
the exact output the current model version returns for it.

| Model | Target | Notebook | Sample input | Sample output |
|---|---|---|---|---|
| LOCI v8 | ARMv8 (aarch64) | `notebooks/loci-v8-1m-Notebook.ipynb` | `data/arm_dataset.csv` (16 blocks) | `example_outputs/loci-v8-1m-output.npy` (16 x 4) |
| LOCI AT | Infineon TriCore (AURIX) | `notebooks/loci-at-1m-Notebook.ipynb` | `data/tricore_dataset.csv` (49 blocks) | `example_outputs/loci-at-1m-output.npy` (49 x 4) |

## Input

A CSV with one column, `r.asm`. Each row is one assembly **basic block**: a
straight-line sequence of instructions ending in a branch, call or return, one
instruction per line inside the quoted cell. Send it as `text/csv`.

## Output

A NumPy array (request it with `Accept: application/x-npy`) with one row per
input block and four columns, in this order:

| Column | Name | Meaning |
|---|---|---|
| 0 | `std` | predicted standard deviation of the execution time, in cycles |
| 1 | `mean` | predicted mean execution time, in cycles |
| 2 | `fd_score` | out-of-distribution score. Larger means the block is further from the model's training data and the prediction is less reliable |
| 3 | `threshold` | calibrated cut-off for `fd_score`, constant per model. Treat rows with `fd_score` above it with caution |

A subset of columns can be requested through the SageMaker custom-attributes
header, for example `{"predicted_outputs": ["std", "mean"]}` for the
two-column form. Columns always come back in the order above.

## Running the notebooks

```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/
```

Each notebook deploys the model package to a real-time endpoint on
`ml.g5.xlarge`, runs the sample input through it, prints one row per block and
deletes the endpoint again. Set `modelpackage_arn` to the ARN from your
Marketplace subscription and `role` to a SageMaker execution role before
running.

The sample outputs in `example_outputs/` were produced by the current model
version from the sample inputs in `data/`, so running a notebook unchanged
should reproduce them.
