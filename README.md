# cinn-audit: Attribution, Sensitivity and Validity in the Measurement of Neural Network Activity

**Hassan Ugail** (Centre for Visual Computing and Intelligent Systems, University of Bradford) and **Newton Howard** (School of Individualized Study, Rochester Institute of Technology)

Measurements of temporal correlation, synchronisation and representational structure are used to describe how the internal activity of a neural network is organised, how that organisation changes with training, and what happens when part of the network is altered. The activity they are computed from belongs to the network and to its input at the same time, so a value on its own cannot say where a pattern came from or whether it means what it is taken to mean. `cinn-audit` is an experimental framework that separates three questions about any such measurement and answers each with its own experiment:

- **Attribution.** How much of the variation in a measurement is associated with the system, how much with the input, and how much with their interaction, measured in a crossed design with repeated runs.
- **Conditional sensitivity.** Whether a controlled change to the system produces a consistent change in the measurement when the input is held fixed, measured by paired comparison.
- **Validity and persistence.** Whether the evidence supports the property the measurement is supposed to capture, with scale-free organisation measured by the DFA exponent as the worked example, and whether the activity continues when the input is removed.

This repository contains the notebook that reproduces the two main experiments of the accompanying paper, together with the inlined `cinn-audit` library it depends on.

## What does this show?

1. **Persistent activity does not establish scaling.** On 33 reference systems from six families with known mechanisms, the validity checks accepted a scaling interpretation only for heterogeneous-timescale and critical-branching networks. Chaotic and oscillator networks sustain activity on their own and give exponents that depend strongly on the system, yet every one of them was rejected, and the rule classified all 33 systems correctly.
2. **Attribution describes the design but does not settle the interpretation.** The system explained 92% of the exponent variance in the chaotic networks, which are not scale-free. A large system share says that a statistic discriminates between systems, and nothing more.
3. **Sensitivity depends on the statistic and on the intervention.** Removing a quarter of the units from a chaotic network left the DFA exponent unchanged while moving the spectral slope and metastability on every input. Weight noise did the opposite.
4. **Input dependence and sensitivity coexist in language models.** In Pythia-410M, 1.4B and 2.8B, the identity of the text explained 54 to 86% of the variance in the hidden-state exponent and the model variant only 2 to 3%, yet pruning attention heads lowered the exponent on 89 of 90 replayed texts with paired effect sizes between 1.9 and 4.0. Weight noise calibrated to a similar loss produced a different response, opposite in direction at the smallest size.
5. **The record length decides what can be claimed.** Along about 200 tokens the fitted DFA scales span less than a decade, so the language-model exponent is reported as an operational summary and no claim of scale-free organisation is made for it.

## Contents

- `cinn_audit_experiments.ipynb`: the reproduction notebook. It opens with the problem, the three questions and a description of the two experiments, then runs them. The `cinn-audit` library is inlined in one cell, so the notebook is self-contained.
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

The E4v2 cell skips generation and replay for any model whose result files already exist, so an interrupted run can be resumed by running the notebook again. Every directory is defined once in the **Paths** cell.

## What cinn-audit reports

For each measurement and experiment the notebook writes

- **Attribution shares** for system, input, interaction and repeats, as sequential increments in $R^2$ of the model $z_{ijk} = \mu + A_i + B_j + C_{ij} + \varepsilon_{ijk}$, together with the increments in each direction and, for the unbalanced language-model design, a permutation null that shuffles variant labels within each text.
- **Conditional sensitivity**: the mean paired change, the paired effect size $d_z$, the sign consistency across inputs, a Wilcoxon signed-rank test, and the change scaled by the between-input standard deviation. For grouped inputs, prompt-level bootstrap intervals.
- **Validity**: a per-channel verdict from the log-log linearity ($R^2 \geq 0.98$), local-slope crossover (within 0.35 of the global exponent), spectral concentration (at most half the power in the three largest bins), stationarity (half-record variance ratio at most 3), scale range (at least a decade) and phase-randomised surrogate checks, with the fraction of passing channels and a majority rule per system.
- **Persistence**: whether activity under zero input retains at least 5% of the driven variance without decay, within a stated system boundary that includes any internal noise.

## Who is this for?

- Researchers who use dynamical or representational statistics to compare networks, checkpoints or damaged models, and want to know whether a difference belongs to the system or to the input.
- Researchers choosing a statistic to detect a particular kind of structural change, who need the evidence of a paired comparison rather than a convention.
- Reviewers and readers who want to see, for a reported exponent, whether the fluctuation function supports the interpretation attached to it.

## Citation

H. Ugail and N. Howard, "Attribution, Sensitivity and Validity in the Measurement of Neural Network Activity," under review, 2026.

If you use the framework, please cite the paper and this repository.

## License

MIT License. See `LICENSE`.
