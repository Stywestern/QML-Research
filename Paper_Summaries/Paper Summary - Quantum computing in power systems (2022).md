There is a need for faster algorithms because Distrubuted Energy Systems (DERs) -like renewable energy or batteries- are complicated, quantum advantage offers a promising solution to this.

Variational Quantum Circuit (VQC) is one such algorithm which works like a Neural Network, more on that here: (other notebook link I will do later). Basically, think of a QPU-CPU hybrid whenever you see the word "variational". It has something to do with circuit depth being important and currently it can't go far deep.

There are two schools for these algorithms, gate-based (which is most of them) and quantum annealing. Gate-based is like normal computers but bits are qubits, quantum annealing is different that it is about physically embedding a math problem to a quantum system and then extracting the solution from it after some operations. 

Power flow (static analysis):
A power grid consists of nodes (**buses**) and lines connecting them.
At every bus $i$, there are 4 main variables:
1. Voltage Magnitude ($V_i$)    
2. Voltage Phase Angle ($\delta_i$)
3. Active Real Power ($P_i$ — measured in Watts)
4. Reactive Power ($Q_i$ — measured in VARs, which maintains magnetic fields in motors/transformers)
**The Power Flow Problem:** Given specified power generation and load consumption at each bus, **find the unknown voltages ($V_i$) and phase angles ($\delta_i$) across the whole network.**
Non-linear active power: $P_i = \sum V_i V_j (G_{ij}\cos\delta_{ij} + B_{ij}\sin\delta_{ij})$ * Solved iteratively via Newton-Raphson Algorithm: $$\begin{bmatrix} \Delta P \\ \Delta Q \end{bmatrix} = \mathbf{J} \begin{bmatrix} \Delta \delta \\ \Delta V \end{bmatrix}$$
* **Bottleneck:** Classical linear solver for Jacobian $\mathbf{J}$ scales as $\mathcal{O}(N^3)$.
Quantum Solution: Hybrid VQLS Pipeline 
* Use Variational Quantum Linear Solver (VQLS) inside Newton-Raphson loop.
* Encodes state vector $|\beta\rangle \propto [\Delta P, \Delta Q]^T$. * Optimizes VQC parameters $\theta$ to minimize residual: $$\mathcal{C}(\theta) \to 0 \implies |\psi(\theta)\rangle \approx \mathbf{J}^{-1}|\beta\rangle$$
State Estimation (SE): 
* Solve for **Bus States** $\mathbf{x} = [\boldsymbol{\delta}, \mathbf{V}]^T$.
* **Power Flow:** Deterministic solution to $f(\mathbf{x}) = 0$. 
* **State Estimation:** Filters noisy SCADA/PMU sensor data $\mathbf{z} = h(\mathbf{x}) + \mathbf{e}$ via Weighted Least Squares (WLS): $$\min_{\mathbf{x}} [\mathbf{z} - h(\mathbf{x})]^T \mathbf{R}^{-1} [\mathbf{z} - h(\mathbf{x})]$$
* **QML Role:** Accelerated via VQLS (matrix inversion of Gain Matrix $\mathbf{G}$) or replaced by direct data-driven Quantum Neural Networks.

Grid Sensors: SCADA vs. PMU 
* **SCADA:** Legacy sensors. Slow (1 sample / 2–4s), asynchronous, measures magnitudes ($V, I$) only. Requires heavy State Estimation math to estimate $\delta$. 
* **PMU (Phasor Measurement Unit):** Modern sensors. Ultra-fast (30–120 Hz), GPS-synchronized, directly measures **Phase Angles ($\delta$)**. 
* **QML Context:** QNNs ingest streaming PMU/SCADA time-series data vectors $|\mathbf{z}\rangle$ for real-time fault detection and transient analysis.

SCADA systems are easily managed by classical computers, so the research is on mostly PMU datasets -sometimes SCADA+PMU hybrids- as they generate more data to work with.

Dynamic EMTP & QEMTP Algorithms 
1. Classical EMTP Bottleneck 
	* **Need:** Low inertia grids -high IBRs require microsecond-level transient wave tracing. 
	* **Math:** Discretized DAEs via Trapezoidal Integration: $$\mathbf{G}_{eq} \mathbf{v}(t + \Delta t) = \mathbf{i}(t + \Delta t) + \mathbf{i}_{hist}(t)$$
	* **Bottleneck:** Solving $\mathbf{G}_{eq} \mathbf{v} = \mathbf{b}$ up to $100,000\times$ per second of simulation; scales polynomially $\mathcal{O}(N^3)$. 
	
2. QEMTP & QSFA Solutions 
	* **VQLS-QEMTP:** Replaces classical linear solver with a VQC variational loop per time step. 
	* **QSFA (Quantum Shifted Frequency Analysis):** Shifts high-frequency carrier wave to envelope frequency, allowing larger time steps $\Delta t$. 
	* **Trade-Off / Bottleneck:** Quantum solvers yield *approximate* solutions $|v\rangle$; historical state updates cause error accumulation over time without error correction.

Shifted Frequency Analysis (SFA) & QSFA
* **Problem:** High-frequency $60\text{ Hz}$ carrier waves force classical EMTP to take tiny microsecond time-steps ($\Delta t \le 10 \ \mu\text{s}$).
* **SFA Principle:** Shifts signal down to $0\text{ Hz}$ (DC baseband) by extracting the dynamic complex envelope $\hat{V}(t)$:
  $$v(t) = \sqrt{2} \operatorname{Re}\left\{ \hat{V}(t) e^{j \omega_0 t} \right\}$$
* **QSFA (Quantum SFA):** 1. Uses **SFA** to increase time-step size ($\Delta t \gg 10 \ \mu\text{s}$), reducing total steps.
* Uses **VQLS** to solve envelope matrix equations at each step with quantum acceleration.

VQLS vs. QPE for Transient Solvers (QEMTP)
1. Why QPE/HHL Fails on NISQ Hardware 
	 * **HHL / QPE:** Uses Quantum Phase Estimation to calculate matrix eigenvalues $\lambda_j$ directly. 
	 * **Bottleneck:** Requires high circuit depth, QFTs, and fault-tolerant error correction; fails due to rapid gate decoherence on current hardware
2. The VQLS Advantage 
	 * **Hybrid Approach:** Replaces deep QPE circuits with shallow Variational Quantum Circuits (VQCs). 
	 * Decomposes system matrix $\mathbf{A} = \sum c_l A_l$ into Pauli strings. 
	 * Uses shallow **Hadamard Tests** to compute cost function $\mathcal{C}_G(\theta)$ on QPU, while classical CPU updates $\theta$. 
	 * ***Trade-off:** NISQ-friendly shallow depth vs. *approximate* solution requiring error mitigation.

Stochastic Risk Assessment (QAE vs. MCS)
1. Classical Monte Carlo Simulation (MCS) Bottleneck 
	* Used for Probabilistic Power Flow & risk assessment under renewable uncertainty. 
	* ***Error Scaling:** $\epsilon = \mathcal{O}(1/\sqrt{N}) \implies N = \mathcal{O}(1/\epsilon^2)$. 
	* Requires millions of sequential AC Power Flow runs for high precision
2. Quantum Amplitude Estimation (QAE) 
	* Leverages Grover's Search Algorithm operator $\mathcal{Q}$ to amplify probability amplitude $\sqrt{p}$. 
	* **Quantum Speedup:** Quadratic Speedup with error scaling $\epsilon = \mathcal{O}(1/N) \implies N = \mathcal{O}(1/\epsilon)$. 
	* **Applications:** Value-at-Risk in energy trading, line overload probabilities, reserve scheduling.

Grid Optimization & Control Hierarchy 
* **Unit Commitment (UC):** Day-ahead plant ON/OFF scheduling. Mapped to QUBO/Ising Model for Quantum Annealing & QAOA. 
* **Energy Management Systems (EMS):** Real-time power balancing solved via Q-ADMM. 
* **Energy Trading:** Market bidding & Value-at-Risk under renewable uncertainty, accelerated via QAE. 
* **Emergency Control:** Sub-second blackout defense (load shedding) driven by fast QNN classifiers on streaming PMU data.

NP-Hard Combinatorial Problems in Power Grids 
1. Unit Commitment (UC): Deciding ON/OFF status of $N$ generators over $T$ hours ($2^{N \times T}$ states). 
2. Distribution Network Reconfiguration: Choosing binary switch states to minimize line losses while keeping a radial tree topology. 
3. Discrete Optimal Power Flow (D-OPF): Mixed-Integer Non-Linear Program (MINLP) for transformer tap positions and capacitor switching. 
4. Emergency Load Shedding: Sub-second binary selection of feeder trips to halt blackout propagation. 
* **Unified Formulation:** All mapped to QUBO / Ising Model ($H = \mathbf{x}^T \mathbf{Q} \mathbf{x}$) for acceleration on Quantum Annealers or via QAOA.

Universal Framework: QUBO & Ising Hamiltonians 
1. Mathematical Equivalency 
	* Any binary decision problem ($x_i \in \{0,1\}$) formulated as a **QUBO**: $$\min_{\mathbf{x}} \mathbf{x}^T \mathbf{Q} \mathbf{x}$$ Is mapped to a physical **Ising Spin Glass Hamiltonian** via $x_i = \frac{1 - \sigma_z^i}{2}$: $$H_{Ising} = \sum_{i} h_i \sigma_z^i + \sum_{i < j} J_{ij} \sigma_z^i \sigma_z^j$$
	* Ground State of $H_{Ising} \equiv$ Optimal Binary Solution $\mathbf{x}^*$. 

2. Solver Execution Paradigms 
	* **Quantum Annealing (D-Wave):** Physical ground-state search via continuous magnetic field tuning. 
	* **QAOA (Gate-Based QPU):** Variational circuit simulating adiabatic evolution via alternating cost and mixer unitary layers ($e^{-i \gamma H_C} e^{-i \beta H_B}$). 
	* **Scope:** Universal across power grids (Unit Commitment), finance, logistics, and machine learning.

Quantum ADMM (Q-ADMM) for Unit Commitment 
1. Why Q-ADMM is Necessary 
	* Unit Commitment is a **Mixed-Integer Non-Linear Program (MINLP)**: 
		1. Discrete Binary Variables ($x_{i,t} \in \{0,1\}$): Generator ON/OFF status. 
		2. Continuous Variables ($P_{i,t} \in \mathbb{R}$): Exact Megawatt outputs. 
	* Quantum devices handle QUBO binary decisions well, but struggle with continuous constraints. 
2. Q-ADMM Decomposition Pipeline 
	1. **Binary Sub-Problem (QPU):** Formulates ON/OFF status as a QUBO solved via Quantum Annealing or QAOA. 
	2. **Continuous Sub-Problem (CPU):** Solves exact MW generation outputs using fast classical non-linear solvers. 
	3. **Dual Update Loop (ADMM Coordinator):** Iteratively updates Lagrange multipliers ($\lambda$) until discrete and continuous states converge to an global optimal dispatch plan.

Transient Stability Assessment (TSA) & QML 
1. Classical TSA Bottleneck 
	* **Goal:** Evaluate if grid generators remain synchronized after a severe fault ($0.1\text{--}10\text{s}$ horizon). 
	* **Math:** Non-linear **Swing Equations**: $$\frac{2 M_i}{\omega_0} \frac{d^2 \delta_i}{dt^2} = P_{m,i} - P_{e,i}(\boldsymbol{\delta}, \mathbf{V})$$
	* **Bottleneck:** Classical Time-Domain Simulation (TDS) is too slow ($>\text{seconds}$) for low-inertia grids collapsing in $<500\text{ms}$. 
2. QML Solution Pipeline 
	* **Approach:** Convert TDS into a real-time **Binary Classification Problem** ($0 = \text{Stable}, 1 = \text{Unstable}$). 
	* Ingests high-speed PMU synchrophasors $[\mathbf{V}, \boldsymbol{\delta}]$ into a Quantum Feature Map. 
	* Uses Parameterized VQCs / QNNs for sub-millisecond stability inference, enabling automated Emergency Control.


Quantum Generative Adversarial Networks (QGANs)
1. Purpose in Smart Grids 
	* Generates realistic synthetic time-series data for volatile DERs (solar/wind) and rare fault scenarios for TSA training.
2. QGAN Architecture 
	* Replaces classical Generator/Discriminator with Variational Quantum Circuits (VQCs): $$\min_G \max_D V(D, G)$$
	* Leverages $2^n$-dimensional Hilbert space to model complex multi-bus correlations with high sample efficiency. 
3. Current NISQ Bottlenecks 
	* Limited to small input vectors due to qubit count constraints and Barren Plateaus. 
	* High noise and shot-noise variance in quantum state measurement tomography.



---
## Raw Annotations & Quotes
“The unprecedentedly ultra-scale computational requirements make existing analysis algorithms, from probabilistic power flow to electromagnetic transients program (EMTP), unscalable and unable to offer real-time, highfidelity results needed for managing massive distributed energy resources (DERs) and ensuring resilient operations[” (Zhou et al., 2022, p. 170)

“Quantum-Engineered Smart Grids (Quantum Grids, or QGrids” (Zhou et al., 2022, p. 171)

“For a review of quantum circuits and implementations of recent quantum devices, please see the review paper ref. [30]” (Zhou et al., 2022, p. 172)

“variational quantum circuit (VQC). In the VQC, one has some pre-determined circuit structure, e.g., composed of fixed CNOTs and some single-qubit rotation gates, whose rotation angles are variational parameters. The VQCs are used in the variational quantum eigensolver (VQE) algorithm, in which the goal is to optimize some cost function or the expectation of a certain energy operator by using VQCs and measurement to yield some classical values, which in turn are used to infer how to change the variational parameters.” (Zhou et al., 2022, p. 172)

“For a recent review of the VQE algorithms and their applications, we refer the readers to a recent article published in Nature Reviews Physics[32]. Variational quantum circuits are also used in many quantum machine learning designs; for the latter, see a recent review[33” (Zhou et al., 2022, p. 172)

“While most quantum devices are gate-based (e.g., IBM, Google, Xanadu), D-Wave pursues another path using specialized quantum annealing techniques[57]. A quantum annealer does not rely on quantum circuits for computing. Instead, it reformulates the problem into ground state searching problems, an excellent match to various optimizing issues. Annealing-based quantum computers appear to be more scalable than gate-based ones in terms of the number of qubits manufactured in a single processor. While most gatebased quantum computers possess no more than 200 qubits, D-Wave already achieves the level of thousands of qubits. With more than 5, 000 qubits and over 15 couplers per qubit, D-wave systems are capable of calculating problems with more than 10, 000 variables[” (Zhou et al., 2022, p. 172)

“Power system static analysis, represented by power flow and state estimation, is the keystone of various power system analytics. Under the unprecedented integration of renewables, a tremendous amount of repetitive static analysis is required to analyze the impact of uncertainties. However, if solved by the conventional iterative algorithms, the computation complexities of power flow and state estimation scale polynomially with the problem scale.” (Zhou et al., 2022, p. 173)

“Prominent AC power flow algorithms include the Newton−Raphson algorithm[65], the Gauss−Seidel algorithm[66] and fast-decoupled methods[67]. An indispensable step of the aforementioned algorithms (i.e. the iterative nonlinear algorithms) is to solve a set of linear algebraic equations. Therefore, the critical bottleneck of power flow analysis lies in the inefficiency of the linear solvers” (Zhou et al., 2022, p. 173)

“data from supervisory control and data acquisition (SCADA) systems and phasor measurement units (PMUs), which are extremely valuable for online system operations” (Zhou et al., 2022, p. 174)

“Transient analysis is another cornerstone for power system analytics. Today’s power systems are facing a risk of diminishing inertia due to the deep integration of inverter-based resources and the retirement of synchronous generators powered by fissile fuel or nuclear reactors[” (Zhou et al., 2022, p. 174)

“EMTP becomes indispensable[76,77]. Although EMTP is capable of precisely tracing the electromagnetic waveforms, its daunting computational complexity, which scales polynomially with the system size, formidably hinders its application in very large power systems. This subsection reviews quantum-enabled EMTP (QEMTP) algorithms, which tackle the EMT computation problem through quantum computing.” (Zhou et al., 2022, p. 174)

“The VQLS-enabled QEMTP employs a hybrid quantum-classical framework. A VQC is constructed for solving Eq. (2), which does not involve the complicated eigendecomposition quantum circuits required by QPE” (Zhou et al., 2022, p. 175)

“In addition, we have also employed the philosophy of shifted frequency analysis (SFA)[81−83] in QEMTP to develop a quantum shifted frequency analysis (QSFA) to enable QEMTP computation with larger timestep.” (Zhou et al., 2022, p. 175)

“However, it should be noted that quantum linear solvers (either noise-free or noisy approaches) can only approximately estimate the solution of LSP, which is different from the classical solvers which can return the full solution. Therefore, error correction is still indispensable for the QEMTP algorithms” (Zhou et al., 2022, p. 176)

“The Quantum amplitude estimation (QAE) algorithm[90], which takes advantages of the Grover’s search algorithm[92], has already demonstrated the quadratic speedup and convergence over the classical MCS technique in estimating an uncertain variable” (Zhou et al., 2022, p. 176)

“such as unit commitment[93], energy management[94], energy trading[95], and emergence control[96]” (Zhou et al., 2022, p. 176)

“While the traditional combinatorial optimization is an NP-hard problem, quantum optimization leveraging quantum mechanisms is expected to achieve a super-polynomial advantage for complicated combinatorial optimization problems.” (Zhou et al., 2022, p. 176)

“Quantum approximation optimization algorithm (QAOA) is one of the most prominent quantum optimization algorithms[97]. As established in ref. [98], the solution to a quadratic unconstrained binary optimization (QUBO) problem is equivalent to the ground state of a corresponding Ising Hamiltonian. Several methods have been developed to establish the Hamiltonian of the Ising model[99]. QAOA aims to find feasible solutions to the QUBO problems by minimizing the expected value of the Hamiltonian” (Zhou et al., 2022, p. 177)

“Then, a multi-block decomposition of the alternating direction method of multipliers (ADMM) is used for coordinating different sub-problems. The overall procedure of the quantum ADMM (Q-ADMM)-enabled UC is as follows. First, initialize the iteration index , decision variables, penalty factor, and stopping criteria. Second, solve the QUBO sub-problem to update the binary decision variables. Then, solve the continuous sub-problem to update the continuous decision variables. Followingly, update the dual variables of Q-ADMM based on the obtained decision variables. The above sub-routines interact until convergence, which returns the optimized solution. Figure 5 depicts the schematic diagram of the quantum distributed UC for power systems. More details are presented in refs” (Zhou et al., 2022, p. 177)

“Transient stability assessment (TSA) is a long-standing obstacle for power system operations. It evaluates whether a power system can reach a steady-state operating point after large disturbance” (Zhou et al., 2022, p. 177)

“a novel quantum distributed controller (QDC) has been proposed” (Zhou et al., 2022, p. 178)

“A quantum version of GAN is the quantum generative adversarial network (QGAN). It uses two quantum components to represent the generator and the discriminator, respectively. Through the quantum-mechanical phenomenon, it promises to reduce computational complexity. However, many existing works on QGAN only focus on simple cases where limited input data points are involved” (Zhou et al., 2022, p. 180)