---
title: "Variational Quantum Linear Solver applied to Gaussian Process Regression"
date: 2024-09-07
draft: false
---
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

### Variational Quantum Linear Solver (VQLS)

#### Motivation
Solving linear systems of equations is a fundamental task in many applications, including machine learning. Given a linear system problem \( A x = b \), where \( A \in \mathbb{R}^{N \times N} \) and \( b \in \mathbb{R}^N \), the goal is to find the solution vector \( x \in \mathbb{R}^N \). Traditional methods like matrix inversion yield the solution as \( x = A^{-1} b \), but this approach has a computational complexity of \( \mathcal{O}(N^3) \), making it impractical for large \( N \).
 
Quantum algorithms offer a promising alternative for large-scale computations, with some providing exponential (e.g., Shor's algorithm) or quadratic speedup (e.g., Grover's algorithm). Harrow, Hassidim, and Lloyd developed a fully quantum algorithm for solving linear systems that achieves exponential speedup, characterized by a complexity of:

$$
\mathcal{O}\left(\log(N) s^2 \kappa^2 / \epsilon\right)
$$

where \( s \) is sparsity, \( \kappa \) is the condition number, and \( \epsilon \) is the desired precision. However, this algorithm requires a quantum computer with a large number of error-corrected qubits, which is currently unattainable.

Today, we are in the Noisy Intermediate-Scale Quantum (NISQ) era, where a few hundred qubits with error mitigation can be utilized for small-scale quantum computations. Variational Quantum Algorithms (VQA) present promising near-term applications, combining classical and quantum algorithms to solve computational tasks.

#### Overview of VQLS
The Variational Quantum Linear Solver (VQLS) was introduced by Bravo-Prieto et al. as a hybrid quantum-classical algorithm suitable for NISQ applications. The system matrix \( A \) is decomposed into a linear combination of unitaries:

$$
A = \sum_{l=0}^{L} c_l A_l
$$

The goal is to find a quantum state \( A|x\rangle \) that is proportional to \( b \). A Parametrized Quantum Circuit (PQC) is utilized, consisting of a gate sequence \( V(\theta) \) with optimizable parameters \( \theta \). The state \( |x(\theta)\rangle = V(\theta) |0\rangle \) is prepared starting from the ground state \( |0\rangle \).

To quantify how close the state \( A|x\rangle \) is to \( b \), a local cost function is defined, where a value of \( 0 \) indicates that the linear system is solved. The cost function \( C(\theta) \) is minimized until it reaches a threshold value \( C(\theta) \leq \epsilon \). After executing the algorithm, the optimized parameters \( \theta_{\rm opt} \) can be used to prepare the solution state \( |x(\theta_{\rm opt})\rangle \).

#### Cost Function
##### Global Cost Function
One potential global cost function is the overlap between the projector \( |\psi\rangle\langle \psi | \) with \( |\psi\rangle = A |x\rangle \) and the subspace orthogonal to the state \( |b\rangle \):

$$
\bar{C}_G = \text{Tr}(|\psi\rangle\langle\psi|(\mathbb{I} - |b\rangle\langle b|)) = \langle x | H_G | x \rangle
$$

where the Hamiltonian \( H_G \) is defined as:

$$
H_G = A^\dagger (\mathbb{I} - |b\rangle \langle b |)A
$$

This cost function is small if \( |\psi\rangle \propto |b\rangle \) (the solution) or if the norm of \( |\psi\rangle \) is small, which can make it size-dependent. To overcome this, an alternative cost function is proposed:

$$
C_G = \frac{\bar{C}_G}{\langle \psi | \psi \rangle} = 1 - |\langle b | \Psi \rangle |^2
$$

where \( |\Psi\rangle=|\psi\rangle/\langle \psi | \psi \rangle \).

##### Local Cost Function
Global cost functions can lead to barren plateaus, where the gradient of the cost function vanishes exponentially with the number of qubits. A local cost function is defined as:

$$
\bar{C}_L =  \langle x | H_L | x \rangle , \qquad C_L = \frac{\bar{C}_L}{\langle \psi | \psi \rangle}
$$

with the alternative Hamiltonian given by:

$$
H_L = A^\dagger U \left( \mathbb{I} - \frac{1}{n} \sum_{j=0}^{n-1} |0_j\rangle \langle 0_j| \otimes \mathbb{I}_{\bar{j}}\right) U^\dagger A
$$

By reformulating this local Hamiltonian using the relation \( |0\rangle \langle 0 | = (\mathbb{I} + Z)/2 \) and the parametrized state \( |x\rangle = V(\theta) | 0 \rangle \), the cost function can be expressed as:

$$
C_L = \frac{1}{2} - \frac{1}{2n} \frac{\sum_{j} \sum_{ll'} c_l c_{l'}^* \langle 0 | V^\dagger A_{l'}^\dagger U Z_j U^\dagger A_l V | 0 \rangle}{\sum_{ll'} c_l c_{l'}^*\langle 0 | V^\dagger A_{l'}^\dagger A_l V|0 \rangle}
$$

This can be simplified using the compact notation:

$$
\beta_{ll'} = \langle 0 | V^\dagger A_{l'}^\dagger A_l V | 0 \rangle \, , \quad \quad \delta_{ll'}^{(j)} = \langle 0 | V^\dagger A_{l'}^\dagger U Z_j U^\dagger A_l V | 0 \rangle
$$

The final form of the cost function is:

$$
C_L = \frac{1}{2} - \frac{1}{2n} \frac{\sum_{j} \sum_{ll'}  c_l \, c_{l'}^* \, \delta_{ll'}^{(j)}}{\sum_{ll'}  c_l \, c_{l'}^* \, \beta_{ll'}} \,.
$$

---
