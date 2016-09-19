---
title: Evolving solutions with genetic algorithms
layout: note
---

# Evolving solutions with genetic algorithms

Genetic algorithms (GAs) are used when a search goal is not entirely describable. Instead, a pool of random candidate solutions (having the right structure but not necessary good solutions) are "evolved" in order to produce better solutions. The goodness of a solution is measured by a "fitness" function. Evolution happens by combining two candidate solutions and applying "crossover" and "mutation" functions to copy features from both inputs but also randomly vary the result. Eventually, we decide to terminate the algorithm and pick out the highest fitness solution found so far.

## Terminology

GAs are modeled after biological evolution, so many of the terms we use in GAs are borrowed and redefined:

  - **genetic representation** -- a format for describing the parameters
       of an individual; often a string of 0's and 1's (binary coding)
       or string of letters is used

  - **fitness function** -- a function that receives as input a genetic
       representation and returns as output a number such that higher
       fitness values means the genetic representation describes an
       individual closer to the goal

  - **selection** -- a function that examines the fitness of the individuals in the population and chooses two to "breed" using crossover and mutation; the selection function may also choose to eliminate some of the lowest-fitness individuals

  - **crossover** -- a function that receives two genetic representations
                 and produces a new genetic representation by
                 combining a (random) portion (or portions) of each
                 input; the portions are copied verbatim (no mutation
                 here); the crossover function ensures the "child" candidate solution shares some of the partial solutions found by both parents

  - **mutation** -- a function that randomly modifies a genetic
                representation; how many locations are modified is
                typically determined by a parameter (e.g., 5% of the
                genetic representation will be randomly swapped with
                different codes); the mutation function helps avoid "local minima"

It is important to note that **the choice of genetic representation, fitness function, etc. is entirely left to the researcher; you can use whatever is best for your problem.** If you find that mutation is not helpful, for example, then don't use it. Do not limit the potential of your search procedure by attempting to fit a kind of "GA purity."

## Genetic representation

In order to more easily define the crossover and mutation functions, most GA search procedures encode candidate solutions in a constant-length string. This string may contain binary values or symbols from a larger set. The greatest challenge in applying GA to a new problem is to find an appropriate genetic representation. 

The crossover function copies the beginning part of one parent and the ending part of the other parent. Thus, the genetic representation should be designed so that a left or right portion encodes a partial solution. If the genetic representation is ordered completely randomly, this will not be the case and the crossover function will produce new candidate solutions that are completely bogus.

Likewise, the mutation function is typically defined as random perturbations of random sections of the string, so it is necessary for the genetic representation to still make sense after random perturbations. It is not necessary for every possible random string to be a candidate solution (maybe some strings describe impossible solutions), but if the genetic representation is too specialized, then most random perturbations will be invalid and the GA won't make much progress in its search.

A genetic representation that is just a dump of the bits representing a data structure is probably a poor genetic representation because it does not meet the constraints described above. Rather, each symbol in the genetic representation should describe an aspect of the solution (to allow mutation), and nearby symbols should describe related parts of the solution (to allow crossover).

## Fitness function

The fitness function is one of the most important features of a
genetic algorithm. It should accurately measure the quality of a candidate solution. With the selection function, only the highest fitness candidate solutions will "breed" and the lowest may be eliminated from the pool. Thus, the fitness function decides whether or not your GA search procedure will actually find a good solution in the end.

The fitness function should be quite fast, because it will
be executed *many* times. Typical GA algorithms run for maybe 50,000
or 100,000 generations. However, if the fitness function is something
like, "the simulated efficiency and safety of the commercial airplane
described by this genetic representation," then that fitness function
is probably too complex to use in a GA. Though GAs seem like a good
idea for finding good designs for complex systems, the fitness
functions may become so unweildy that GAs may turn out to be the wrong
approach.

![xkcd 534: "Just make sure you don't have it maximize instead of minimize.](/images/xkcd-genetic_algorithms.png "xkcd 534: 'Just make sure you don't have it maximize instead of minimize.'")

## When to stop?

A GA has no particular stopping condition. New candidate solutions can be generated forever. Typically, GAs are stopped when the fitness value of the most fit individual candidate solution does not increase much, or after a fixed number of steps.

## Python implementation

~~~ python
for generation in range(MAX_GENERATIONS):
    # calculate fitness values and put these values, along with
    # the individuals, in a new "weighted" list
    weighted_population = []
    for individual in population:
        fitness_val = fitness(individual)
        weighted_population.append((individual, fitness_val))

    # select two individuals to breed; individuals with higher
    # fitness values are more likely to be selected
    ind1 = selection(weighted_population)
    ind2 = selection(weighted_population)

    ind3 = crossover(ind1, ind2) # breed, creating a new individual
    ind3 = mutate(ind3) # do some random mutating
    population.append(ind3)

# generations are completed; find maximum fitness individual
max_fit = fitness(population[0])
max_fit_individual = population[0]
for individual in population:
    fitness_val = fitness(individual)
    if fitness_val > max_fit:
        max_fit = fitness_val
        max_fit_individual = individual
print max_fit_individual
~~~


