# Presentation video here
[https://youtu.be/Q8kTB6uLX-w](https://youtu.be/Q8kTB6uLX-w)

Your team must produce a cohesive story of your project.

* **If In-Person:** A live 5-10 minute presentation to the judges.
* **If Remote:** A 5-10 minute voice-over slide deck (MP4 format) included in your github repository.

**Example Presentation Agenda:**

1. The Plan & The Pivot:
Quantum-Enhanced Memetic Tabu Search for LABS Optimization

Executive Summary

This document synthesizes the technical framework of a hybrid optimization approach designed for the Low Autocorrelation Binary Sequence (LABS) problem. The system integrates a Quantum-walk (SamBa-GQW, X-mixer) seeding mechanism with a classical Memetic Tabu Search (MTS). The core objective is to identify \pm 1 binary sequences of length N that minimize energy, defined by the sum of squared autocorrelations.

The process is divided into two distinct phases:

1. Quantum Seeding (P_Q): Utilizing a sample-based guided quantum walk to generate a high-quality initial population.
2. Classical Optimization: Refinement of the population through uniform crossover, mutation, and a specialized Tabu Search local optimizer.

While the quantum component provides a sophisticated "seed" for the population, the implementation is memory-intensive, restricted to sequences of length N \le 20 due to the use of 2^N statevectors.


--------------------------------------------------------------------------------


Technical Framework: The LABS Problem

The primary goal of the algorithm is to minimize the energy E(s) of a \pm 1 sequence s of length N. The energy is calculated as:

E(s) = \sum_{k=1}^{N-1} C_k(s)^2

Where C_k(s) is the autocorrelation at shift k: C_k(s) = \sum_{i=0}^{N-k-1} s[i] \cdot s[i+k]

The algorithm facilitates this by converting 0/1 bitvectors to \pm 1 sequences (where 0 \rightarrow -1 and 1 \rightarrow +1) before performing energy calculations.


--------------------------------------------------------------------------------


Phase I: Quantum-Walk Seed Generation (SamBa-GQW)

The initial population is not generated randomly but is "seeded" through a simulated quantum evolution process. This phase uses the Sample-Based Guided Quantum Walk (SamBa-GQW) with an X-mixer.

1. Offline Sampling (Algorithm 2)

Before the quantum evolution, a sampler identifies the energy landscape to create an Energy Schedule (ES).

* Process: The sampler takes q candidates (default N^2).
* Gap Calculation: It computes the normalized energy difference (\Delta) between a state x and its Hamming neighbors y (1-bit flips).
* Energy Schedule (ES): A list of mean maximum energy gaps, sorted in decreasing order and ending at 0.0. This schedule dictates the "gaps" used in the quantum layer schedule.

2. Layer Schedule Construction

The algorithm builds a QAOA-like schedule based on the ES. Each interval between energy levels ES[l] and ES[l+1] is divided into p layers.

* Time Parameter (\tau_l): Calculated as \frac{\pi}{2\sqrt{2}} \cdot \frac{1}{\Gamma_l} (where \Gamma_l is the current energy gap).
* Cost and Mixer Evolution: Each layer consists of a cost phase (\Delta t_{cost}) and a mixer phase (\Delta t_{mixer}), where the mixer is an X-mixer (represented as \otimes RX(2 \Delta t_{mixer})).

3. Quantum State Evolution

The quantum state evolves from an initial uniform superposition across a 2^N dimension statevector.

* Cost Phase: Application of exp(-i \cdot \Delta t_{cost} \cdot H_C) where H_C is the diagonal cost matrix.
* Mixer Phase: Application of the X-mixer across all qubits.
* Symmetry Pruning: To reduce redundancy, the system accounts for global spin flips and sequence reversals.

4. Measurement and Population Initialization (P_Q)

After evolution, the statevector is converted into a probability distribution. A high number of "shots" are taken to sample the distribution.

* Elite Fraction: Only the top percentage of samples (based on energy) are considered "elites."
* Population P_Q: The final population of size K is populated with these unique elite sequences.


--------------------------------------------------------------------------------


Phase II: Memetic Tabu Search (MTS)

Once the quantum-walk provides the initial population, the algorithm enters a classical memetic loop (Algorithm 1 skeleton) to iteratively improve the sequences.

1. Evolutionary Operators

The population undergoes transformation through two primary mechanisms:

* Uniform Combination (Crossover): Two parents are chosen to create a child. A mask is applied where each bit of the child has a 50% chance of coming from either parent.
* Mutation: A bit-flip mutation is applied to individual sequences based on a probability p_{mut}.

2. Memetic Step: Tabu Search

The most critical refinement occurs in the Tabu Search, which acts as a local optimizer for every child produced.

* Neighborhood Search: The optimizer examines all "single spin flip" neighbors (flipping one \pm 1 value).
* Tabu List: To avoid cycles and local minima, the indices of recently flipped bits are added to a "tabu" list. These indices cannot be flipped again for a duration defined by the Tabu Tenure.
* Iteration: The search continues for a set number of iterations (tabu_iters), constantly tracking the best sequence found.

3. Population Replacement

The algorithm uses a steady-state approach. After a child is generated and refined via Tabu Search, it replaces a random individual in the population if it offers a competitive energy level. The global best sequence is tracked throughout the maximum generations (G_{max}).


--------------------------------------------------------------------------------


Operational Parameters and Constants

The effectiveness of the hybrid search is governed by the following parameters:

Parameter	Description	Typical Value/Constraint
N	Sequence Length	Practical limit \le 18-20
K	Population Size	e.g., 200
G_{max}	Maximum Iterations	e.g., 1000
p_{comb}	Probability of Combination	0.7 (70%)
p_{mut}	Probability of Mutation	0.01 (1%)
Elite Fraction	Portion of samples used for seeding	0.05 (5%)
Tabu Tenure	Duration an index remains tabu	50 iterations
Local Mixer Gap	Constant for sampler	2.0

Conclusion

The SamBa-GQW seeded MTS represents a high-complexity approach to the LABS problem. By leveraging quantum state evolution to find promising regions of the search space (the "seed"), it provides the Tabu Search with a significant head start compared to random initialization. However, the reliance on statevector simulation limits the scalability of the current implementation to relatively small sequence lengths.


2. Results: 

* Present your results based on the Success Metrics defined in your PRD. For example, use charts to visualize your Time-to-Solution and Approximation Ratio compared to the baseline.

3. The Retrospective:

* Conclude with a personal touch. Each team member should share one specific technical or strategic takeaway from the challenge (e.g., "I learned that moving data to GPU is slower than computing on CPU if the batch size is too small").

