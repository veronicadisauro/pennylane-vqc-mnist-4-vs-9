# Variational Quantum Classifier for MNIST 4 vs 9

A reproducible Quantum Machine Learning project developed in **PennyLane** to study a **Variational Quantum Classifier (VQC)** for binary classification of the MNIST digits **4 vs 9**.

The project focuses on controlled experimental analysis of the main design choices of a hybrid quantum-classical classifier: feature encoding, entanglement, ansatz depth, loss function, optimizer choice, initialization robustness, and circuit complexity.

## Overview

The dataset contains 6000 samples and is already reduced to three numerical features plus the class label. The classification task is therefore mapped to a **3-qubit VQC**.

The final experimental pipeline was implemented and validated in PennyLane using `default.qubit`. Earlier Qiskit-based experiments were used for preliminary study of variational quantum algorithms, but the quantitative comparison reported in this repository is based on the PennyLane implementation.

### Binary labels

- MNIST digit `4` → class `0`
- MNIST digit `9` → class `1`

### Dataset protocol

- Training set: 228 samples, balanced
- Internal test set: 12 samples, balanced
- Grading set: 2000 samples, balanced
- No overlap between train/test and grading data
- Explicit checks against data leakage

## Quantum model

The baseline circuit uses:

- 3 qubits
- RX/RZ feature encoding
- linear entanglement in the feature map
- trainable RY/RZ ansatz
- linear CNOT entanglement
- `FEATURE_REPS = 2`
- `ANSATZ_REPS = 4`
- Negative Log-Likelihood loss
- COBYLA optimizer
- small random initialization in `[-0.05, 0.05]`

The circuit returns computational-basis probabilities, which are aggregated into the two output classes.

## Experimental methodology

The project is organized around a fixed baseline and a series of controlled experiments.

The main one-factor studies investigate:

- optimizer choice
- local vs entangled feature maps
- entanglement topology
- NLL vs MSE loss
- ansatz depth
- feature-map depth

Additional combined experiments explore interactions between promising choices.

A separate robustness study, **EXP02-B**, evaluates the optimizer comparison over multiple shared initializations. For each initialization seed, all optimizers start from exactly the same initial point.

A supplementary experiment, **EXP09-C**, evaluates **Adam + MSE** on the baseline circuit and checks its behavior across the same initialization seeds.

### Optimizer robustness study: EXP02-B

The original optimizer comparison (EXP02) was performed using a single shared initialization, corresponding to seed `10599`. In that experiment, L-BFGS-B achieved the highest grading accuracy among the tested optimizers.

However, VQC optimization is non-convex and its outcome can depend on the initial values of the trainable parameters. Therefore, a second experiment, **EXP02-B**, was added to verify whether the ranking observed in EXP02 remained stable when the initialization changed.

EXP02-B repeats the optimizer comparison using three initialization seeds:

- `10599`
- `10600`
- `10601`

For each seed, COBYLA, Adam, L-BFGS-B, and SPSA start from exactly the same initial parameter vector. This paired design isolates the effect of the optimizer within each initialization.

The results show that the ranking changes across seeds: L-BFGS-B is best for seed 10599, Adam is best for seed 10601, while Adam and COBYLA tie for seed 10600. Therefore, the single-seed result of EXP02 cannot be interpreted as evidence that L-BFGS-B is universally superior.

Across the three tested initializations, Adam achieves the highest mean grading accuracy and the lowest variability, while L-BFGS-B reaches the highest single-run peak. The three main optimizers remain very close in absolute accuracy, whereas SPSA is consistently less competitive in the adopted configuration.

Because only three initialization seeds are considered, EXP02-B should be interpreted as a robustness check rather than a statistically exhaustive benchmark.
## Main results

The baseline grading accuracy is:

`0.7880`

The best single configuration observed in the current experimental campaign is:

`COBYLA + MSE → 0.7940`

The multi-seed optimizer robustness study shows that Adam, COBYLA, and L-BFGS-B achieve very similar grading accuracies, while SPSA is consistently worse in the tested configuration.

For EXP02-B:

- Adam mean grading accuracy: `0.787167`
- Adam standard deviation: `0.000577`
- L-BFGS-B mean grading accuracy: `0.786667`
- COBYLA mean grading accuracy: `0.785500`
- SPSA mean grading accuracy: `0.612667`

Adam is the most stable optimizer across the tested initializations, while L-BFGS-B reaches the highest single-seed peak.

For EXP09-C, Adam + MSE achieves:

- seed 10599: `0.7905`
- seed 10600: `0.7925`
- seed 10601: `0.7925`

showing good stability, but not exceeding the best COBYLA + MSE result.

## Circuit complexity

A simple analytical circuit-cost proxy is used:

`circuit_cost = single_qubit_rotations + 10 * CNOTs`

For the baseline:

- 62 single-qubit rotations
- 12 CNOTs
- circuit cost = `182`

This metric is not intended as a full hardware cost model. It is used only as a consistent proxy to compare circuits with different depths and entanglement structures.

## HPQC perspective

The project is also discussed from a High Performance and Quantum Computing perspective.

The VQC is treated as a hybrid CPU/QPU workflow:

1. the CPU prepares inputs and variational parameters;
2. the QPU executes the parameterized circuit;
3. measurement results are returned to the CPU;
4. the CPU computes the loss;
5. the classical optimizer updates the parameters;
6. the loop is repeated.

This highlights several relevant architectural bottlenecks for future CPU/QPU systems:

- CPU/QPU round-trip latency
- scheduling overhead
- repeated circuit execution
- shot cost
- batching opportunities
- parameterized-circuit reuse
- measurement and post-processing overhead
- circuit depth and entangling-gate cost

The experiments in this repository are executed on a classical simulator, so the project does **not** claim quantum computational advantage.

## Installation

Create a virtual environment and install the required Python packages.

```bash
python -m venv .venv
```

On Windows:

```bash
.venv\Scripts\activate
```

On Linux/macOS:

```bash
source .venv/bin/activate
```

Then install the dependencies used by the notebooks, for example:

```bash
pip install pennylane numpy pandas matplotlib scikit-learn scipy jupyter
```

If a `requirements.txt` file is included in the repository, prefer:

```bash
pip install -r requirements.txt
```

## Running the experiments

Open the project in VS Code or Jupyter and execute the notebooks in the order indicated by the experiment numbering.

For reproducibility:

- keep the dataset unchanged;
- preserve the reported train/test/grading split;
- use the specified initialization seeds;
- compare runs only when the experimental protocol is identical.

## Reproducibility notes

The main initialization used in the original single-seed campaign is:

`INIT_SEED = 10599`

The robustness analysis also uses:

- `10599`
- `10600`
- `10601`

All optimizer comparisons within the same seed use the same initial point.

## Scope and limitations

This repository studies the behavior of a small VQC under a controlled experimental protocol. It should not be interpreted as evidence that quantum classifiers outperform classical machine-learning models.

The main limitations are:

- execution on a statevector simulator rather than real quantum hardware;
- small number of qubits;
- reduced three-feature representation of MNIST;
- limited number of initialization seeds in the robustness analysis;
- optimizer comparisons are not fully normalized by wall-clock time or total quantum circuit evaluations.

## Technologies

- Python
- PennyLane
- NumPy
- Pandas
- SciPy
- scikit-learn
- Matplotlib
- Jupyter / VS Code

## License

Add a license before publishing the repository if you want to define how other users may reuse the code.

For an academic project, the MIT License is a common permissive option, but the appropriate choice depends on how you want the code to be reused.

## Author

Academic project developed for the **High Performance and Quantum Computing** course.
