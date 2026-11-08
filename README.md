# seqlab

Sequence classification when **labels are scarce, noisy and of mixed trust**, with
the evaluation hygiene that makes the result believable.

> **Status: design only. There is no code in this repository yet.** What is here
> is the plan: the problem, the data contract, the modules, the experiments and
> the acceptance criteria. Nothing below is implemented or measured. Progress is
> tracked in the [roadmap](#roadmap).

---

## The problem

Recurrent models (a bidirectional GRU is the baseline) are easy to train and easy
to fool yourself with. On small multichannel time-series datasets the usual
failure is not the architecture, it is:

- **leakage:** windows from the same recording or subject land in both train and
  validation, so the score is inflated;
- **label quality:** some labels come from people, some from rules, some are
  doubtful, and treating them equally teaches the model the rules' mistakes;
- **too few labels** for the model size, with no evidence of how many would be enough;
- **missing or unreliable frames** handled as if they were data;
- **invariances** (mirroring, scale, shifts) applied ad hoc and never tested;
- **no baseline**, so nobody knows whether a recurrent model was needed.

seqlab is a small toolkit and benchmark that makes each of those explicit and testable.

## What it will contain

```
seqlab/
  data.py          SequenceDataset: X[N,T,D], mask[N,T], y[N], step_labels[N,T], group[N], provenance[N]
  splits.py        group-aware k-fold; leakage checker that fails loudly
  synth.py         configurable synthetic multichannel sequences (classes, noise, label noise, group shift, dropped frames)
  models.py        bidirectional GRU with a sequence head and an optional per-step head (PyTorch)
  augment.py       augmentations with a declared invariance and a test that the invariance holds
  train.py         weak-label training: per-provenance weights, class weights, masked frames
  evaluate.py      out-of-fold metrics, seed averaging, bootstrap CIs, calibration, abstention, per-trust subsets
  baselines.py     majority class and logistic regression on summary statistics
  experiments/     learning curves vs label count and label noise
```

## The experiments it is meant to answer

1. **How many labels are enough?** Accuracy as a function of the number of
   labelled sequences, for the recurrent model and for the baselines.
2. **How much does label noise cost?** The same curve under increasing label noise,
   and how much per-provenance weighting recovers.
3. **Does the invariance help?** Each augmentation on and off, with the invariance test.
4. **Does it survive a new group?** Train on some groups, test on unseen ones,
   with controlled group shift.
5. **Can it abstain usefully?** Accuracy against coverage when the model may decline to answer.
6. **Was the recurrent model needed?** Against majority class and a regularised
   linear model on summary statistics, with confidence intervals.

## Principles

- **Evaluation is out-of-fold, grouped and averaged over seeds.** Never a single
  split, never a single seed, never windows of one recording on both sides.
- **Report by trust.** Metrics are given on the subset of reliable labels and on
  all labels, separately.
- **Baselines first.** A recurrent model that cannot beat a regularised linear
  model on summary statistics is not worth its cost.
- **Declare invariances, then test them.** An augmentation without a test is a bug waiting to happen.
- **Synthetic data shows the mechanics, not the world.** Conclusions about a real
  dataset need that dataset; this repository will say so wherever it reports a number.

## Roadmap

- [ ] Data contract and group-aware splits, with a leakage test
- [ ] Synthetic generator with controllable noise, label noise, group shift and dropped frames
- [ ] Bidirectional GRU with a sequence head and an optional per-step head
- [ ] Weak-label training with per-provenance and per-class weights, and frame masks
- [ ] Augmentations with declared, tested invariances
- [ ] Evaluation: out-of-fold, seed-averaged, bootstrap CIs, calibration, abstention
- [ ] Baselines and the "was it needed?" comparison
- [ ] Learning-curve experiments and a short write-up with the limits stated plainly

See [docs/DESIGN.md](docs/DESIGN.md) for the contracts and acceptance criteria.

## License

[MIT](LICENSE)
