# Product Requirements Document (PRD)

**Project Name:** LABS-Solv-V1  
**Team Name:** 3kingdoms  
**GitHub Repository:** [Link](https://github.com/iseungsang01/2026-NVIDIA)

---

> **Note to Students:** > The questions and examples provided in the specific sections below are **prompts to guide your thinking**, not a rigid checklist. 
> * **Adaptability:** If a specific question doesn't fit your strategy, you may skip or adapt it.
> * **Depth:** You are encouraged to go beyond these examples. If there are other critical technical details relevant to your specific approach, please include them.
> * **Goal:** The objective is to convince the reader that you have a solid plan, not just to fill in boxes.

---

## 1. Team Roles & Responsibilities [You can DM the judges this information instead of including it in the repository]

| Role | Name | GitHub Handle | Discord Handle
| :--- | :--- | :--- | :--- |
| **Project Lead, GPU Acceleration PIC** (Manager, Builder) | [Seung Sang, Lee] | [@iseungsang01] | [@iseungsang] |
| **Quality Assurance PIC** (Architect, Verifier) | [Gwanwoo, Kim] | [@dualmath] | [@gimgwanu3753] |
| **Technical Marketing PIC** (Architect, Storyteller) | [jamesjoshuahamburgerhyeon ?] | [@handle] | [@jamesjoshuahamburgerhyeon] |

---

# 2. The Architecture

## Choice of Quantum Algorithm

### Algorithm
**Quantum Approximate Optimization Algorithm (QAOA)** with a standard X-mixer and a LABS-specific cost Hamiltonian composed of 2-body and 4-body Pauli-Z interactions.

### Motivation
The LABS problem is a highly nonlocal combinatorial optimization problem where the objective function depends on long-range autocorrelations of a binary sequence. QAOA is a natural fit for this structure for the following reasons:

1. **Direct Hamiltonian Encoding**  
   The LABS objective can be exactly rewritten as a diagonal Ising Hamiltonian consisting of 2-body and 4-body \( Z \)-type interactions. QAOA allows direct exponentiation of this Hamiltonian without relaxation or penalty terms.

2. **Progressive Correlation Capture**  
   Each QAOA layer increases the effective correlation length of the quantum state. This aligns well with LABS, where minimizing long-range correlations is the dominant difficulty rather than enforcing local constraints.

3. **Continuous Optimization of a Discrete Problem**  
   QAOA embeds the discrete LABS landscape into a continuous parameter space \((\gamma, \beta)\), enabling structured exploration via classical optimizers instead of brute-force bit flips.

4. **Benchmark Transparency**  
   QAOA has well-characterized behavior and known limitations. This makes it suitable not only as a solver but also as a diagnostic tool for understanding how quantum variational algorithms behave on sequence optimization problems.

---

## Literature Review

### Bernasconi (1987): *Low Autocorrelation Binary Sequences*
- **Relevance:**  
  This paper defines the LABS problem and its canonical energy function. All correctness checks, benchmarks, and target energies in this project are derived directly from this formulation.

### Hadfield et al. (2017): *From QAOA to the Quantum Alternating Operator Ansatz*
- **Relevance:**  
  This work generalizes QAOA beyond graph problems and provides the theoretical foundation for applying alternating-operator methods to problems like LABS that lack an underlying graph structure.

### Zhou et al. (2020): *Quantum Approximate Optimization Algorithm: Performance, Mechanism, and Implementation*
- **Relevance:**  
  This paper analyzes parameter concentration, optimization landscapes, and depth scaling in QAOA. These results inform our choice of shallow circuit depth, gradient-free optimizers, and realistic expectations for convergence under shot noise.

---

# 3. The Acceleration Strategy

## Quantum Acceleration (CUDA-Q)

### Strategy
We will use **CUDA-Q on NVIDIA L4 GPUs** to accelerate classical simulation of quantum circuits.

The motivation is throughput rather than hardware realism:
- QAOA requires repeated circuit evaluations during parameter optimization.
- LABS circuits contain many multi-body interactions, increasing gate depth.
- Shot-based expectation estimation introduces additional sampling overhead.

CUDA-Q enables parallel state evolution and sampling on the L4 GPU, significantly reducing wall-clock time per objective evaluation. This allows rapid iteration over QAOA parameters and circuit variants.

---

## Classical Acceleration (MTS)

### Strategy
The classical LABS energy evaluation involves nested summations over sequence shifts and indices, which becomes a bottleneck during optimization and verification.

We accelerate the classical side by:
- Vectorizing autocorrelation computations.
- Evaluating batches of candidate sequences in parallel.
- Offloading these computations to the GPU where appropriate.

This hybrid approach ensures that classical post-processing does not dominate the overall runtime.

---

## Hardware Targets

- **Development Environment:**  
  CPU-only execution for correctness validation, unit testing, and debugging.

- **Production Environment:**  
  NVIDIA L4 GPU for QAOA circuit simulation and batched classical energy evaluation.

This staged approach balances cost efficiency with performance.

---

# 4. The Verification Plan

## Unit Testing Strategy

### Framework
Standard Python testing frameworks (e.g., `pytest`) are used to validate correctness deterministically.

### AI Hallucination Guardrails
AI-generated code is treated as untrusted until it satisfies strict invariants:
- Property-based tests enforce theoretical bounds and symmetries.
- Results are cross-checked against brute-force solutions for small \( N \).
- Quantum sampling is always validated using an independent classical energy function.

---

## Core Correctness Checks

### Check 1: Symmetry
The LABS energy is invariant under global sign inversion:
\[
E(S) = E(-S)
\]
Any violation indicates an incorrect Ising mapping or sign convention.

### Check 2: Ground Truth
For small \( N \), exact optimal energies are known. The implementation must reproduce these values exactly before being used for larger instances.

---

# 5. Execution Strategy & Success Metrics

## Agentic Workflow

### Plan
The workflow separates generation, verification, and refactoring:
1. Code is generated or modified.
2. Unit and property tests are executed.
3. Failures are analyzed and fed back into the refinement loop.

Local documentation and API references are provided to the agent to prevent hallucinated interfaces.

---

## Success Metrics

- **Approximation Quality:**  
  Achieve energies close to known optima as circuit depth increases.
- **Speedup:**  
  Demonstrate meaningful runtime reduction compared to CPU-only baselines.
- **Scale:**  
  Successfully simulate and optimize LABS instances beyond trivial sizes.

---

## Visualization Plan

- **Time-to-Solution vs. N:**  
  Comparing CPU-only and L4-accelerated execution.
- **Energy vs. Iteration Count:**  
  Showing convergence behavior during QAOA optimization.

---

# 6. Resource Management Plan

## Plan
GPU usage is tightly controlled:
- All logic and verification are completed on CPU.
- The L4 GPU is used only after correctness is established.
- Long-running jobs are scheduled intentionally to avoid idle GPU time.

This ensures that computational resources are spent on validated experiments rather than debugging cycles.

