# pso_quantum

# IN PROGRESS !!

A 'quantum' particle swarm optimizer written in Python using several quantum inspired methods. Modified from the [adaptive timestep PSO optimizer](https://github.com/jonathan46000/pso_python) by [jonathan46000](https://github.com/jonathan46000). 

Now featuring AntennaCAT hooks for GUI integration and user input handling.

## Table of Contents
* [Particle Swarm Optimization](#particle-swarm-optimization)
* [Quantum Inspired Optimization](#quantum-inspired-optimization)
* [Quantum Particle Swarm Optimization](#quantum-particle-swarm-optimization)
* [Requirements](#requirements)
* [Implementation](#implementation)
    * [Time-step adaptation](#time-step-adaptation)
    * [Constraint Handling](#constraint-handling)z
    * [Boundary Types](#boundary-types)
    * [Multi-Object Optimization](#multi-object-optimization)
    * [Objective Function Handling](#objective-function-handling)
      * [Internal Objective Function Example](internal-objective-function-example)
* [Error Handling](#error-handling)
* [Example Implementations](#example-implementations)
    * [Basic PSO Example](#basic-pso-example)
    * [Detailed Messages](#detailed-messages)
    * [Realtime Graph](#realtime-graph)
* [References](#references)
* [Publications and Integration](#publications-and-integration)
* [Licensing](#licensing)  


## Particle Swarm Optimization

Particle Swarm Optimization (PSO) is a popular nature-inspired optimization algorithm introduced in "Particle Swarm Optimization" [1] (J. Kennedy & R. Eberhart, 1995). It is inspired by the social behavior animal groups, often compared to birds flocking or fish schooling. PSO is used to find approximate solutions to complex optimization problems.

PSO consists of a population (or swarm) of candidate solutions called particles. Each particle moves through the search space, influenced by its own best-known position and the best-known positions of the swarm. The algorithm combines exploration and exploitation to find the optimal solution.



## Quantum Inspired Optimization

Quantum-inspired algorithms take concepts from quantum mechanics, such as superposition and entanglement, and apply them in classical computation to solve optimization problems more effectively. 

1) Superposition

**Quantum Concept**: In quantum mechanics, a particle can exist in a superposition of multiple states simultaneously. For example, a quantum bit (qubit) can be in a state ∣0⟩∣0⟩, ∣1⟩∣1⟩, or any linear combination ∣ψ⟩=α∣0⟩+β∣1⟩∣ψ⟩=α∣0⟩+β∣1⟩, where αα and ββ are complex numbers.

**Classical Adaptation**: In quantum-inspired algorithms, superposition can be interpreted as a probability distribution over multiple states. Instead of particles having a single position, they are represented by a probability distribution, reflecting the potential to be in various positions simultaneously.

**Example in QPSO**: In the Quantum-inspired Particle Swarm Optimization (QPSO), a particle’s position is often updated using a probability distribution derived from both personal best and global best positions, rather than a deterministic position update. This allows particles to explore the search space more effectively.

2) Entanglement

**Quantum Concept**: Entanglement is a phenomenon where particles become interconnected such that the state of one particle directly affects the state of another, no matter the distance between them. This creates a strong correlation between the particles.

**Classical Adaptation**: In quantum-inspired algorithms, entanglement can be represented as a dependency or correlation between particles. When one particle updates its position, it influences the position updates of other particles, promoting cooperative behavior among the particles in the swarm.

**Example in QPSO**: In QPSO, the positions of particles might be updated using a combination of their personal best position and the global best position, creating a form of "entanglement" where particles are influenced by the best solutions found by the swarm, thus maintaining a level of coordination and cooperation.



## Quantum Particle Swarm Optimization





## Requirements

This project requires numpy and matplotlib. 

Use 'pip install -r requirements.txt' to install the following dependencies:

```python
contourpy==1.2.1
cycler==0.12.1
fonttools==4.51.0
importlib_resources==6.4.0
kiwisolver==1.4.5
matplotlib==3.8.4
numpy==1.26.4
packaging==24.0
pillow==10.3.0
pyparsing==3.1.2
python-dateutil==2.9.0.post0
six==1.16.0
zipp==3.18.1

```

## Implementation
### Time-Step Adaptation 
This particle swarm optimizers uses the mean absolute deviation of particle position as an adjustment to the time step, to prevent the particle overshoot problem.  This particle distribution is initialized to one when the swarm starts, so that the impact is boundary independent. 

### Constraint Handling
Users must create their own constraint function for their problems, if there are constraints beyond the problem bounds.  This is then passed into the constructor. If the default constraint function is used, it always returns true (which means there are no constraints).

### Boundary Types
This PSO optimizer has 4 different types of bounds, Random (Particles that leave the area respawn), Reflection (Particles that hit the bounds reflect), Absorb (Particles that hit the bounds lose velocity in that direction), Invisible (Out of bound particles are no longer evaluated).

Some updates have not incorporated appropriate handling for all boundary conditions.  This bug is known and is being worked on.  The most consistent boundary type at the moment is Random.  If constraints are violated, but bounds are not, currently random bound rules are used to deal with this problem. 

### Multi-Object Optimization
The no preference method of multi-objective optimization, but a Pareto Front is not calculated. Instead the best choice (smallest norm of output vectors) is listed as the output.

### Objective Function Handling
The optimizer minimizes the absolute value of the difference from the target outputs and the evaluated outputs.  Future versions may include options for function minimization absent target values. 

#### Internal Objective Function Example
The current internal optimization function takes 3 inputs, and has 2 outputs. It was created as a simple 3-variable optimization objective function that would be quick to converge.  
<p align="center">
        <img src="https://github.com/LC-Linkous/pso_python/blob/pso_basic/media/obj_func_pareto.png" alt="Function Feasible Decision Space and Objective Space with Pareto Front" height="200">
</p>
   <p align="center">Function Feasible Decision Space and Objective Space with Pareto Front</p>

```math
\text{minimize}: 
\begin{cases}
f_{1}(\mathbf{x}) = (x_1-0.5)^2 + (x_2-0.1)^2 \\
f_{2}(\mathbf{x}) = (x_3-0.2)^4
\end{cases}
```

| Num. Input Variables| Boundary | Constraints |
|----------|----------|----------|
| 3      | $0.21\leq x_1\leq 1$ <br> $0\leq x_2\leq 1$ <br> $0.1 \leq x_3\leq 0.5$  | $x_3\gt \frac{x_1}{2}$ or $x_3\lt 0.1$| 

This function has three files:
   1) configs_F.py - contains imports for the objective function and constraints, CONSTANT assignments for functions and labeling, boundary ranges, the number of input variables, the number of output values, and the target values for the output
   2) constr_F.py - contains a function with the problem constraints, both for the function and for error handling in the case of under/overflow. 
   3) func_F.py - contains a function with the objective function.

Other multi-objective functions can be applied to this project by following the same format (and several have been collected into a compatible library, and will be released in a separate repo)

## Error Handling

In the particle_swarm.py class, the objective function is called twice. Some optimizer/objective function/parameter combinations cause under/overflows when using numpy. It is a known bug in numpy that as of 5/2024 basic numpy cannot convert floats to longFloats or float128().

 * 1) When the constraints are called to verify if the particle is in bounds, and to apply the selected boundary method. At this point, the 'noErrors' boolean is used to confirm if the objection function resolves. If the objective function does not resolve, or the particle is out of bounds, the boundary conditions are applied.
 * 2) To evaluate the objective function as part of the traditional particle swarm algorithm

## Example Implementations

### Basic PSO Example
main_test.py provides a sample use case of the optimizer. 

### Detailed Messages
main_test_details.py provides an example using a parent class, and the self.suppress_output and detailedWarnings flags to control error messages that are passed back to the parent class to be printed with a timestamp. This implementation sets up the hooks for integration with AntennaCAT in order to provide the user feedback of warnings and errors.

### Realtime Graph

<p align="center">
        <img src="https://github.com/LC-Linkous/pso_python/blob/main/media/pso_graph.gif" alt="Example PSO Convergence" height="200">
</p>

main_test_graph.py provides an example using a parent class, and the self.suppress_output and detailedWarnings flags to control error messages that are passed back to the parent class to be printed with a timestamp. Additionally, a realtime graph shows particle locations at every step.

NOTE: if you close the graph as the code is running, the code will continue to run, but the graph will not re-open.

## References

[1] J. Kennedy and R. Eberhart, "Particle swarm optimization," Proceedings of ICNN'95 - International Conference on Neural Networks, Perth, WA, Australia, 1995, pp. 1942-1948 vol.4, doi: 10.1109/ICNN.1995.488968.

## Publications and Integration
This software works as a stand-alone implementation, and as one of the optimizers integrated into AntennaCAT.

Publications featuring the code in this repo will be added as they become public.

## Licensing

The code in this repository has been released under GPL-2.0


