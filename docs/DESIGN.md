# Design (planned, not implemented)

## Data contract

| Field | Shape | Meaning |
|---|---|---|
| `X` | `[N, T, D]` | `N` sequences, `T` steps, `D` channels (padded to a common `T`) |
| `mask` | `[N, T]` | 1 where the step is reliable, 0 where it is missing or untrustworthy |
| `y` | `[N]` | sequence label |
| `step_labels` | `[N, T]` (optional) | per-step label for the per-step head; `-1` = unlabelled |
| `group` | `[N]` | recording / subject / session id; the unit that must never straddle a split |
| `provenance` | `[N]` | where the label came from (for example `manual`, `rule`, `doubtful`) |

A `provenance -> weight` table is configuration, not code, so a user can state
how much to trust each source.

## Acceptance criteria per module

| Module | Done when |
|---|---|
| `splits` | A test builds a dataset with a deliberately leaky split and the checker raises. Group folds are reproducible from a seed. |
| `synth` | Class separability, label noise rate, group shift and frame-drop rate are parameters, and a test confirms each moves the measured quantity it claims to. |
| `models` | Output shapes are correct for variable `T`; masked steps do not change the output (test: perturb masked steps, output unchanged). |
| `augment` | Each augmentation declares an invariance and a test checks the label-relevant statistic is unchanged. |
| `train` | With all provenance weights equal the result matches an unweighted run (test). Seeds are reproducible. |
| `evaluate` | Metrics are computed only out-of-fold; bootstrap intervals contain a known value on a toy problem; calibration error is zero for a perfectly calibrated toy. |
| `baselines` | They run with the same splits and report the same metrics as the recurrent model. |

## Experiments (each is a script plus a table)

1. Learning curve: accuracy vs labelled sequences (log-spaced), all models.
2. Label-noise curve: same, at 0, 10, 20, 30 % noise, with and without weighting.
3. Augmentation ablation: each on/off, with confidence intervals.
4. Group shift: train on `k` groups, test on unseen groups, at increasing shift.
5. Abstention: accuracy vs coverage, by confidence threshold.
6. "Was the recurrent model needed?": gap to the best baseline, with intervals.

## Out of scope

- Any specific domain, dataset or feature extractor.
- Pose estimation, tracking or other perception.
- Claims about real-world performance.
- Architectures beyond a recurrent baseline, until the baselines comparison justifies them.

## Risks

- A small synthetic benchmark can be solved by anything; the generator must be
  hard enough that the baselines do not saturate it.
- Bootstrap intervals over few groups are optimistic; report the number of groups.
- Per-provenance weights can be tuned to the test set by accident; fix them
  before looking at the held-out groups.
