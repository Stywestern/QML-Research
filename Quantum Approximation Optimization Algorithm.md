We talked about QUBO problems and how Quantum Annealing solves it. This algorithm is to achieve similar objectives with gate-based methods. Because gate-based quantum computers have significantly less qubits (for now), this method is a hybrid of classical-quantum framework.

In QAOA, we construct two competing mathematical "forces" (Hamiltonians):
- **The Mixer ($H_I$):** This represents pure quantum superposition. It is the state of total indecision. If you apply the Mixer, the qubits have an equal probability of being in any of the 8 possible generator combinations (from `000` to `111`).
- **The Problem ($H_F$):** This is the QUBO penalty equation built in the other notebook. It applies "phases" (invisible quantum rotations) to the qubits based on how good or bad their current combination is. A bad combination (like `000`, causing a blackout) gets a heavy rotation, while our optimal `101` state gets a favorable rotation.

QAOA works by alternating between these two forces like a layered cake. This is measured in "depth" ($d$):
1. **Start:** Put all 3 qubits in perfect superposition (equal chance of all 8 answers).
2. **Layer 1:** 
	* Apply the Problem QUBO ($H_F$) for a specific duration of time called $\gamma$ (Gamma).
    - Apply the Mixer ($H_I$) for a specific duration of time called $\beta$ (Beta).
3. **Measurement:** We read the 3 qubits. Because of quantum interference, the bad answers cancel each other out, and the good answers amplify.

If we just ran that circuit once, we would probably get a random, bad answer. The magic of QAOA is that it works in a continuous loop with a classical laptop.
1. **Quantum Run:** The IBM machine runs the circuit using a random guess for $\gamma$ and $\beta$, measures the qubits, and gets an answer (e.g., `111`).
2. **Classical Scoring:** Your classical laptop takes that answer, plugs it into the QUBO equation, and calculates the "Expectation Value" (the average score of the quantum guesses).
3. **Classical Update:** A classical optimization algorithm (like Gradient Descent or COBYLA) looks at the score and says, _"Okay, those angles were bad. Let's adjust $\gamma$ and $\beta$ slightly and try again"_.
4. **Repeat:** The quantum computer runs the circuit again with the new, better angles.
Over hundreds of loops, the classical computer mathematically "tunes" the quantum computer like a radio dial until the probability of measuring the optimal answer (`101`) reaches nearly 100%.

## Example
I always understand better with examples so let's do one. Assume we have 3 qubits:

First step is to translate $x_i$ binary values into the qubit realm, we do this by operating on the standart qubits with Pauli-Z gates, equivalent to:
$$x_i = \frac{1 - Z_i}{2}$$
If we substitute x with Z in the in the main QUBO equation, that gives us the $H_f$, the Final Hamiltonian. After this, we need our $H_I$, the Mixer Hamiltonian, for example:
$$H_I = X_1 + X_2 + X_3$$
This mixer would just invert all three qubits once.

First, the three qubits are put in a superposition 
$$\vert{}\phi_0\rangle = \vert{}+\rangle \otimes \vert{}+\rangle \otimes \vert{}+\rangle$$
Next, we run the alternating layers. Let's just do $1$ layer for simplicity. We apply our Problem ($H_F$) for a duration of time $\gamma$, and then our Mixer ($H_I$) for a duration of time $\beta$. The resulting quantum state $\vert{}\phi(\beta, \gamma)\rangle$ is written as:
$$\vert{}\phi(\beta, \gamma)\rangle = e^{-i\beta H_I} e^{-i\gamma H_F} \vert{}\phi_0\rangle$$
Here exponential terms are just examples of rotation by those gates.

The quantum computer runs the equation above, measures the qubits, and spits out a binary string (like `101`). Classical laptop takes over and calculates the **Expectation Value**. This is essentially the "average QUBO score" of the quantum computer's guesses. We write the Expectation Value by sandwiching the Problem Hamiltonian between the quantum states:

$$\langle H_F \rangle = \langle\phi(\beta, \gamma)\vert{} H_F \vert{}\phi(\beta, \gamma)\rangle$$

- If the score is **high** (bad), the classical laptop runs a Gradient Descent algorithm to pick slightly different numbers for $\gamma$ and $\beta$.
- It feeds the new $\gamma$ and $\beta$ back to the quantum computer (Step 3).
- The loop repeats until $\langle H_F \rangle$ hits the mathematical floor (the optimal `101` state)!
This is the exact mathematical protocol happening under the hood when a researcher imports QAOA from IBM's Qiskit library.