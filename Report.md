# Project-work: symbolic regression  
The objective of the algorithm developed is, given a set of inputs x and the corresponding output y, to evolve a formula that best fits them. In order to do so, we developed an evolutionary algorithm that is described in the following.  
The entire project has been done in collaboration with __name__   

**Genotype**  

We decided to represent formulas through a tree with a maximum depth of 10 in order to avoid bloating, a phenomena that makes formulas grow generation after generation with small changes in fitness.  
The tree is represented with a list of lists, where, depending on the number of operands taken by the operation, the structure is:  
- `[operator, [first_operand], [second_operand]]` if it is a binary operation;
- `[operator, [operand]]` if it takes only one operand.  

Constants and variables are simply represented through a string, e.g.: 'x[0]' and '9.48'.    

**Phenotype**  

The Phenotype is represented by a mathematical formula that can be evaluated with the corresponding input dimensions (1 or more in the problems that we had to face).  

**Fitness**    

The fitness evaluates the goodness of the solution. There were different possibilities in literature to represent the fitness of an individual in the context of symbolic regression. We started by using MSE as it is done to evaluate the performance of the delivered algorithm but, as soon as we started to execute different problems, we noticed that the scale changes a lot among them. Thus, it was difficult to compare the performance of the algorithms on different problems. We tried to substitute it with the MAPE, computed as:  

```python
float(np.mean(np.abs((y - predictions) / y)) * 100)
```  

When using MAPE (Mean Absolute Percentage Error) and MSE (Mean Squared Error) as fitness measures in symbolic regression, discrepancies between these metrics were observed. MAPE measures the relative percentage error between predicted and actual values, making it highly sensitive to errors on samples with small actual values. In contrast, MSE measures the average squared error and is heavily influenced by errors on large values because the errors are squared.

In practice, this means that if your predictive formula is inaccurate on samples with large actual values, it could result in a low MSE (if errors are uniformly distributed across small and large values) while the MAPE could be high if most of the relative errors occur on samples with small actual values.  

This is the reason why, at the end, we decided to use the MSE as fitness and we didn't introduce other measures: our objective was exactly to minimize that value for the final evaluation. So, it doesn't matter if it has different values depending on the problem, it is simply related to the scale and the MAPE can be printed as used to have an idea of the performance of the algorithm. Moreover, given the aforementioned difference in scale, we decided to remove the penalization term that we had at the beginning in the fitness regarding the length of the formula. Our algorithm is already checking that the maximum depth is below 10 for every mutation and after the crossover, so we noticed no advantage in penalizing formulas based on the length.   

On the other hand, we had to penalize programs with errors (a negative argument to the logarithm, for instance). We simply did it by setting the fitness to infinite in that case, in a way that the formula is discarded by the evolutionary process itself.  


**Operations**  

We have conducted several trials concerning the choice of operations. In this context, there are two possible approaches: including all potential operations or limiting the set to a smaller subset. The latter is supported by Taylor's theorem, which demonstrates that any function can be well approximated by a polynomial.  

Our operation set, following the second strategy, includes: addition, subtraction, division, multiplication and power as binary operations, while we had sin,cos,exp log and sqrt as unary operations.  

At the beginning we also tried to associate different probabilities to each operation, in a way that sum and subtraction were picked more with respect to sin and cosine, for example. However, we now consider all of them with the same probability because, after many trials, we have noticed that this is more effective.   

**Individual**  

Our individual is represented by a genome and the corresponding fitness. In order to do so, we used dataclass to ensure that the fitness is computed only once when the individual is generated or mutated and then used in different steps of the algorithm.  

**Initial population**  

Before that the EA algorithm effectively starts to run, the initial population has to be generated. In order to do so, we designed a function called `generate_program()` that takes as argument the input dimension. Then, for each dimension, it generates a random tree (calling `random_program()` ) for each dimension with a max_depth=2 and combines through randomly selected operations all the subtrees to generate the final tree.  
This allows an initial population where there is a lot of genetic material related to all the dimensions of the problem and we can create more different combinations. 

**Evolutionary algorithm**

For what concerns the evolutionary algorithm, we decided to use a steady-state model where the offspring is added to the population, then they compete against parents for survival.  
We also tried, at the beginning, to use a generational model with the elitist strategy, but this strategy was less effective than the chosen one. 
We set all the parameters of the EA algorithm through empirical tests and, in particular, the population size is set to 1000, which ensures enough genetic material for the crossover without slowing too much each execution. The number of generations is set to 50 to obtain a result in a reasonable amount of time. We noticed, through experiments and collecting statistics about the population at each generation, that the diversity is preserved and the algorithm keeps evolving the formula and improving the fitness up to the last generation, so it would for sure benefit from a longer execution.    
Starting from the initial population, parents are selected and the offspring is generated with two possible operations: mutation and crossover, that will be better explained in the following sections. The crossover is the main operation and it is executed with a 70% probability, while the remaining times mutation on a single individual is performed.  
This is the hyper-modern GA flow: 

- A genetic operator is selected with a given probability;
- the correct number of parents is selected;
- the offspring is generated.  

When the offspirng has been generated, the population is extended with the offspring and they compete against each other for survival. After they have been merged together, they are sorted and only the best `population_size` individuals are kept for the next generation.


**Parent selection**    

We implemented torunament selection with fitness hole for selecting parents in order to generate the offspring. Starting from the initial population, tau=10 members are randomly selected and a tournament among them is performed. With a 90% probability, the best out of these tau individual is returned. The remaining times, the less fit individual is returned.  
Implementing a fitness hole as described can actually be beneficial in overcoming the challenges posed by an adaptive change that requires multiple intermediate steps. By intentionally allowing less fit individuals a chance to win, fitness holes can help navigate the evolutionary pathway where direct progression is hindered by intermediate steps that reduce overall fitness. This approach ensures that even though the final adaptation is advantageous, the evolutionary path to achieve it can successfully bypass 'fitness holes' that would otherwise deselect the intermediates before the final adaptation is achieved.


**Mutation**
We introduced three different types of mutation: subtree mutation, hoist mutation and point mutation.

- Subtree mutation: mutates a subtree with another one randomly generated;
- Hoist mutation: to replace a subtree with a smaller one within the selected subtree;
- Point mutation: to mutate a node (operator, variable or constant) of the program.

All of these functions are designed to achive a valid mutated solution.
Hoist and point mutation do not increase the depth of the solution (moreover Hoist mutation could be used to simplify the solutions), while subtree mutation can. To mantain the constraint on the max depth (set to 10) we defined a function called `cut_program` which simply cuts the program if the depth is bigger than the max depth. To do so, we simply replace the subtree with a single number randomly chosen.


**Crossover**




**Avoiding bloating**  

In order to avoid bloating, we introduced a function called `cut_program` that recursively travels the tree that represents the program and substitutes one or more subtrees with a constant if the depth is too high after mutation or crossover.  





