This is an important type of problem as Quantum Annealers (QA) can ONLY solve QUBOs. Let's first start with Constrained case.

## Constrained Binary Optimization:
In QCBO problems, the variables can only take values from the set {0, 1}. This makes it harder to solve, as one can not use smooth gradients to arrive at the solution. 

### Example: Knapsack Type of Problems:
We have 3 generators: 
* $G_1$ : Yields 10 MW, costs 5 bucks
* $G_2$ : Yields 20 MW, costs 12 bucks
* $G_3$ : Yields 30 MW, costs 25 bucks

Goal is to minimize cost, how many dollars spent.
Constraint is that we need at least 40 MW.
Assuming we named our variables $x_1$, $x_2$, $x_3$; we get the problem:
$$\text{Minimize Cost:} \quad Z = 5x_1 + 12x_2 + 25x_3$$
$$\text{Subject to (Constraint):} \quad 10x_1 + 20x_2 + 30x_3 \ge 40$$

The hardship with these type of problems is that there is no algorithm to solve it, we can't just take the derivative and arrive a solution after finding the zeros. Machines must try all the combinations to find the optimal answer -which is $[1, 0, 1]$-. 

### Formal Definition:
For a variable set $x_i \in$ {0, 1}, the cost set $C_i$, and the constraint set $P_i$ and constraint boundary D and S as the tolerance over boundary (the slack variable) -remember that these are the running costs in dollars and the power they generate, other problems may have other concepts you need to model-, the equation are:
$$\min Z = \sum_{i=1}^{n} C_i x_i$$
$$\sum_{i=1}^{n} P_i x_i \ge D$$
The machine will try to solve this by substituting for every combination -not really, there are some optimizations for it but that's the idea-.

## Quadratic Unconstrained Binary Optimization
After this, you use Lagrange multipliers to mash these two together and get a unified equation to give QA:
$$E(x) = \sum_{i=1}^{n} C_i x_i + \lambda((\sum_{i=1}^{n} P_i x_i) - (40 + S))^2$$
Because we used Lagrange multipliers and then we squared the constraints, it became *Quadratic Unconstrained* Binary Optimization. 

In the literature, there are some ways to tackle such problems:
### Exact Solvers (like Gurobi):
If you need the 100% guaranteed absolute best answer, you use an Exact Solver. The industry standard software for this is **Gurobi**.
- **The Algorithm:** They use techniques like **Branch and Bound**. Instead of blindly checking every single $2^n$ combination, the algorithm builds a "decision tree" and uses clever math to chop off entire branches that it knows will lead to bad answers.
- **The Flaw:** Even with clever branch-chopping, as the grid grows to hundreds or thousands of generators, the computational time escalates exponentially and crashes the classical CPU.

## Heuristic Solvers (like Simulated Annealing)
If you don't have time to wait for the exact answer, you use heuristics like **Simulated Annealing** or **Genetic Algorithms**.
- **The Algorithm:** Imagine the $E(x)$ equation as a physical landscape of hills and valleys. Simulated Annealing drops a marble onto the landscape and lets it roll downhill into a valley (a low-cost solution). It is reminiscent of Gradient Descent algorithms.
- **The Flaw:** It easily gets trapped in "local minima." It might roll into a shallow valley and think it found the bottom, completely missing the "global minimum" (the true lowest cost) hiding just over the next mountain.

### 3. Quantum Annealing
This is where the D-Wave machine steps in to solve $E(x)$.
- **The Algorithm:** It doesn't run line-by-line code. It physically maps your $E(x)$ equation onto microscopic magnetic loops (qubits).
- **The Magic:** Instead of rolling a marble over the hills like classical Simulated Annealing, the quantum hardware uses a property called **Quantum Tunneling**. The qubits can literally tunnel _through_ the energy barriers (the mathematical mountains) to instantly reach the absolute lowest energy state

We assume that this is the best of the both worlds when we are setting out into this field to study.