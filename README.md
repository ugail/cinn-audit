# cinn-audit: Attribution, Sensitivity and Validity in the Measurement of Neural Network Activity

**by H Ugail, N Howard, N V Hoai and C T Tien

Measurements of temporal correlation, synchronisation and representational structure are used to describe how the internal activity of a neural network is organised, how that organisation changes with training, and what happens when part of the network is altered. The activity they are computed from belongs to the network and to its input at the same time, so a value on its own cannot say where a pattern came from or whether it means what it is taken to mean. `cinn-audit` is a measurement-audit framework that separates three questions about any such measurement and answers each with its own experiment:

- **Attribution.** How much of the variation in a measurement is associated with the system, how much with the input, and how much with their interaction, measured in a crossed design with repeated runs. For observational designs, where the predictors are collinear, the order-independent (Shapley) decomposition is reported alongside the conditional increments.
- **Conditional sensitivity.** Whether a controlled change to the system produces a consistent change in the measurement when the input is held fixed, measured by paired comparison on two scales, repeated over independent realisations of the intervention.
- **Validity and persistence.** Whether the evidence supports the property the measurement is supposed to capture, with scale-free organisation measured by the DFA exponent as the worked example and the same template instantiated for a synchronisation measure, and whether the activity continues when the input is removed.

<img width="2092" height="753" alt="Figure3" src="https://github.com/user-attachments/assets/17da0bd4-96ce-415f-a0ec-5e0d91b2180a" />


This repository contains the notebook that reproduces every experiment of the accompanying paper, together with the inlined `cinn-audit` library it depends on.

## What does this show?

1. **Persistent activity does not establish scaling.** On 33 reference systems from six families with known mechanisms, the validity checks accepted a scaling interpretation only for heterogeneous-timescale and critical-branching networks. Chaotic and oscillator networks sustain activity on their own and give exponents that depend strongly on the system, yet every one of them was rejected, and the rule classified all 33 systems correctly. The verdicts are unchanged under a joint sweep of the thresholds except at the strictest local-slope tolerance, and are reproduced when the thresholds are estimated from one scale-free family instead of fixed by hand. A bootstrap acceptance frequency, computed separately for each condition, shows how far each decision sits from the thresholds. Under the zero-input condition it was 1.00 for the branching networks and between 0.84 and 1.00 for the heterogeneous-timescale networks, while under the driven condition, where all sixteen runs must pass, it was 0.69 to 0.74 and 0.25 to 1.00, because several driven runs had only four or five of eight channels passing. It was 0.00 for every other system under both conditions.
2. **Attribution describes the design but does not settle the interpretation.** The system explained 92% of the exponent variance in the chaotic networks, which are not scale-free. A large system share says that a statistic discriminates between systems, and nothing more.
3. **Sensitivity depends on the statistic and on the intervention.** Removing a quarter of the units from a chaotic network left the DFA exponent unchanged while moving the spectral slope and metastability on every input. Weight noise did the opposite.
4. **Input dependence and sensitivity coexist in language models.** In Pythia-410M, 1.4B and 2.8B, the identity of the text explained 54 to 86% of the variance in the hidden-state exponent and the model variant only 2 to 3%, yet pruning attention heads lowered the exponent on 89 of 90 replayed texts with paired effect sizes between 1.9 and 4.0. The effect held for ten further independent random head selections at every size (effect sizes between 1.9 and 4.5, the exponent falling in every selection), whereas the geometric summaries changed sign between selections at the smaller sizes. Weight noise calibrated to a similar loss produced a different response, opposite in direction at the smallest size.
5. **The record length decides what can be claimed.** Along about 200 tokens the fitted DFA scales span less than a decade, so the language-model exponent is reported as a descriptive summary and no claim of scale-free organisation is made for it. On such records the crossover check is the one that becomes unreliable, and the diagnostics show which checks keep their meaning.
6. **The validity template transfers.** Instantiated for the metastability measure, the template shows that in the 8 to 13 Hz band of the motivating index no reference family has the oscillatory structure a metastability reading presupposes, and that in a band matched to the oscillators it separates a locked network from a metastable one, with verdicts stated per record (the eight driven runs with the first noise seed and the first zero-input run of each system).

## Contents

- `cinn_audit_experiments.ipynb`: the reproduction notebook. It opens with the problem, the three questions and a description of the experiments, then runs them. The `cinn-audit` library is inlined in one cell, so the notebook is self-contained.
- `requirements.txt`: the packages the notebook installs.
- `LICENSE`, `README.md`.

Results and figures are not stored in the repository. The notebook writes every result as a CSV or JSON file and bundles them as `cinn_audit_results.zip`.

## Quick start

Open `cinn_audit_experiments.ipynb` in Google Colab with a GPU runtime (a T4 is sufficient), set `PROJECT_DIR` in the **Paths** cell, and run all cells. For a local run:

```bash
git clone https://github.com/ugail/cinn-audit.git
cd cinn-audit
pip install -r requirements.txt
jupyter notebook cinn_audit_experiments.ipynb
```

Set `USE_DRIVE = False` in the **Paths** cell when running outside Colab. Set `DRY_RUN = True` in the **Parameters** cell to check the whole pipeline on tiny stand-ins in a few minutes before committing to the full run.

## What the notebook runs

| Section | Experiment | Runtime |
|---|---|---|
| E2v2 | Six families of reference systems (feedforward filters, leaky networks, chaotic rate networks swept across gain with the connection matrix held fixed, Stuart–Landau oscillators, heterogeneous-timescale networks, critical branching networks), 33 systems driven by the same eight inputs with replicate noise seeds and run on zero input. Reports the variance shares, the validity and persistence verdicts, and the paired response of the chaotic network to unit removal and weight noise. | CPU, about 25 minutes |
| E4v2 | Pythia-410M, 1.4B and 2.8B, each compared with a head-pruned and a loss-calibrated weight-noised version of itself. Every generated text is replayed through every variant. Reports the attribution of five hidden-state statistics to text and variant, the increments in both directions with a permutation null, the paired changes under each intervention, and the discrimination between interventions by temporal and geometric measures. | GPU, about 40 minutes |
| E4v3 | The E4v2 texts replayed through models pruned with ten independent random head selections (`E4_PRUNE_SEEDS`). Reports the mean paired change and effect size of each temporal statistic per selection and their spread across selections. | GPU, about 15 minutes per model |
| E5 | Diagnostics of the validity protocol on the E2v2 systems: a joint sweep of the thresholds, the behaviour of each check on truncated records, a bootstrap acceptance frequency for every verdict under the driven and the zero-input condition, thresholds estimated from data, and mixtures of mechanisms. The same cell instantiates the validity template for the metastability measure and computes the order-independent (Shapley) attribution of the E4v2 hidden-state exponent with the range of each sequential increment over all entry orders. | CPU, about 10 minutes |

The E2v2, E4v2 and E4v3 cells skip any model or experiment whose result files already exist, so an interrupted run can be resumed by running the notebook again, and E4v3 regenerates the E4v2 texts from the same seeds, so it can be run on its own once E4v2 has written its calibration files. Every directory is defined once in the **Paths** cell.

## What cinn-audit reports

For each measurement and experiment the notebook writes

- **Attribution shares** for system, input, interaction and repeats, as sequential increments in $R^2$ of the model $z_{ijk} = \mu + A_i + B_j + C_{ij} + \varepsilon_{ijk}$, which in the balanced crossed design are order-independent and coincide with the Sobol decomposition, together with the increments in each direction and, for the unbalanced language-model design, a permutation null that shuffles variant labels within each text and the Shapley value of each predictor group.
- **Conditional sensitivity**: the mean paired change, the paired effect size $d_z$, the sign consistency across inputs, a Wilcoxon signed-rank test, the change scaled by the between-input standard deviation, and a bootstrap interval for every paired analysis (prompt-level for grouped inputs). For head pruning, the distribution of these quantities across independent head selections.
- **Validity**: a per-channel verdict from the log-log linearity ($R^2 \geq 0.98$), local-slope crossover (within 0.35 of the global exponent), spectral concentration (at most half the power in the three largest bins), stationarity (half-record variance ratio at most 3), scale range (at least a decade) and phase-randomised surrogate checks, with the fraction of passing channels, a majority rule applied to every run of the assessed condition and a bootstrap acceptance frequency for the verdict under each condition. The thresholds were fixed in the library before the reference systems were simulated and were not calibrated on separate records, so the threshold sweep and the data-derived thresholds of E5 are the checks on that choice. For metastability, the matched checks of a spectral peak in the band (mean Welch power in the band against the geometric mean of two flanks of equal logarithmic width, 4.9 to 8 and 13 to 21.1 Hz for the 8 to 13 Hz band), saturation, stationarity, a channel-wise surrogate null and record length.
- **Persistence**: whether activity under zero input retains at least 5% of the driven variance without decay, within a stated system boundary that includes any internal noise.

## Who is this for?

- Researchers who use dynamical or representational statistics to compare networks, checkpoints or damaged models, and want to know whether a difference belongs to the system or to the input.
- Researchers choosing a statistic to detect a particular kind of structural change, who need the evidence of a paired comparison rather than a convention.
- Reviewers and readers who want to see, for a reported exponent, whether the fluctuation function supports the interpretation attached to it.

## Citation

H. Ugail, N. Howard, N.V. Hoai and C.T. Tien, "Attribution, Sensitivity and Validity in the Measurement of Neural Network Activity," under review, 2026.

If you use the framework, please cite the paper and this repository.

## License

MIT License. See `LICENSE`.
