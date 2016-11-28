---
layout: post
title: Heuristic optimization algorithms for fun and (academic) profit
category: programming
tags: [Python, Algorithms]

---

Optimization algorithms are one of those things that you might learn about
in an undergraduate CS class, then quickly forget. But if you need a
good answer to an computationally intensive problem, there's really no
substitute for them. There are optimization algorithms with a strong
mathematical basis (such as gradient descent), but these are generally based on certain
assumptions of how the problem is defined and what your fitness
landscape is like. Heuristic algorithms (such as hill climbers,
simulated annealing, and evolutionary algorithms) make few assumptions
but no guarantees. They are fairly agnostic to the shape and structure of
your solution space and fitness function, but they make no promises
that you will ever find the *best* solution.

# Why use heuristic search algorithms?
Heuristic search algorithms are good for a very specific thing:
finding solutions to minimize or maximize some fitness function. In
general, the fitness function refers to some quantitative metric for
how good your solution is. You
could be trying to find a route to minimize the distance travelled for
a UPS driver (this is the well-known
[travelling sales rep problem](https://en.wikipedia.org/wiki/Travelling_salesman_problem)),
or you could be trying to choose the price point and production cost
in order to maximize units of clothing sold. Each of these problems has a well-defined
fitness function to minimize (total distance and
units sold, respectively). We don't know exactly
what the "fitness landscape" looks like, but we expect that similar
solutions will generally have similar fitnesses.
In both of these cases, there are far too many
possible solutions to search exhaustively, and the fitness function can't simply
be minimized by differentiating. All of these features make these
problems an excellent fit for a heuristic search algorithm. Ideally,
we would like to use some algorithm that would guarantee that we find
the globally best solution, but when the fitness space is
high-dimensional, non-differentiable, and impossible to search
exhaustively, heuristic algorithms are often the best we can do.

Fortunately, a pretty decent solution is often good enough for
practical purposes. UPS would rather their drivers take the most
efficient routes possible, but it's better to have a relatively
efficient route calculated in a short amount of time than the optimal
solution calculated after days or weeks of computation. So now that
we've decided to use a heuristic method, what are our options?

# Hill Climbers
Hill climbers are pretty boring algorithmically, but they are
incredibly useful for many problems, because they are so fast. Hill
climbers essentially greedily look for a local optimum. The general
flow of the algorithm goes as follows:

1. Take current solution and mutate it
2. Compare the fitness of the mutant to that of the current solution
3. If the mutant is better than the current, replace the current
solution with the mutant
4. Return to step 1

That's it! Again, the fitness here is a number that measures how good
your solution is-- this is the number you'll be minimizing or
maximizing. The mutation function is a function that makes some small
change to the current solution. You want this mutation to be
fairly small, so that you're searching locally in the solution
space. If you make big changes to your solution with every mutation
step, you're not really hill climbing, you're just trying out random
solutions, which isn't very efficient. For the travelling sales rep
problem, a solution will be a permutation of the destinations, so a
reasonable mutation function would be to randomly swap the order of
two destinations.

In general, you'll run a hill climber for a specified number of steps
and then stop. You can also code the algorithm so that you
automatically stop after rejecting N steps in a row. This will stop
you wasting computer time when your algorithm has already converged.

A simple extension to the hill climber that can improve performance is
to create several mutants in step 1, then take the best and compare it
to the current solution in step 2. This helps the algorithm climb the
steepest part of the hill, rather than meandering slowly upward.

Hill climbers are great because they have very little overhead. They
find local optima very quickly. However, if your fitness landscape has
multiple local optima (which is likely for many problems), it is easy
to get stuck in a suboptimal part of the solution space. One solution
is to run lots of hill climbers starting in different places. The hope
is that one of them will start near the global optimum, and since the
runs are relatively quick, it's not very costly to run a lot of
them.

I find that hill climbers are always a good place to start when
looking at a new optimization problem. However, for very complicated
fitness landscapes, their usefulness begins to break down. A hybrid
solution is to use something complicated like a genetic algorithm at
first, then take that solution and run a short hill climber, to make
sure it is locally optimized.

# Simulated Annealing

To understand why simulated annealing is useful, it's helpful to
understand the tradeoff between *exploration* and
*exploitation*. Algorithms *explore* the fitness landscape to find
promising regions. This means searching some crappy parts of the
landscape too, since you may have to cross a deep valley (assuming
you're maximizing) in the
landscape to find a higher peak. Algorithms *exploit* promising areas
by climbing local peaks. Once you've found a worthwhile part of the
fitness landscape, you want to find the absolute best solution in that
area, so it's worth spending time there searching for a local
optimum.

Hill climbers are focused entirely on exploitation, and have
no exploration phase. Simulated annealing allows for more exploration
by accepting suboptimal steps for a while. Early on, a simulated
annealing algorithm will accept almost anything, and will wander
around the fitness landscape. But over time, it will hone in on a
promising area, and becomes less and less likely to accept suboptimal
steps. By the end of the algorithm, it is essentially a hill climber.

Simulated annealing moves smoothly from exploration to exploitation
with a parameter called the "temperature" T, where T is strictly
positive. The temperature essentially
controls the probability of accepting a suboptimal step. When the
temperature is high, the algorithm works essentially like a random
walk, accepting most steps whether they're an improvement or not. When
the temperature is cold, few suboptimal steps are accepted, and the
algorithm operates essentially like a hill climber. By starting the
"chain" (a single run of the algorithm) at a high temperature which
slowly cools over time, the chain transitions slowly from exploration
to exploitation, and is better able to find a good solution without
getting stuck in local optima. Here is the general flow of the
algorithm, for a minimization problem:

1. Take current solution and mutate it
2. Compare the fitness of the mutant to that of the current solution
3.
	a. If the mutant is better than the current, replace the current
	solution with the mutant
	b. If the mutant is worse, accept mutant with probability:
	`exp((CurrentFitness - MutantFitness)/T)`
4. Update the temperature so that it is slightly cooler
5. Return to step 1

Note the equation in step 3. If the mutant is worse, its fitness will
be higher, making the value inside the exponential higher. The
exponential of something very negative will be close to zero, so as
your mutant gets worse, your probability of accepting it gets closer
and closer to zero. But as the temperature T increases, the
value inside the exponential becomes less negative, thereby increasing
the probability of acceptance. This is why high temperatures (large T)
are more permissive than cold ones (T close to 0). The "cooling
schedule" in step 4 is a complicated subject about which many papers
have been written, so I won't discuss it in depth here. A reasonable
starting point is to decrease T by some small fixed value at each step
(making sure never to allow T to become negative).

# Markov Chain Monte Carlo (MCMC)

If you're familiar with MCMC, you may be wondering why I'm mentioning
it here. MCMC isn't really designed as an optimization algorithm at
all. It's basically a way to approximate really complicated
integrals. In optimization terms, this would mean that it's designed
to output the whole fitness landscape, not just the optimal
solution. If you're at all familiar with Bayesian statistics, you may know that
MCMC is useful for numerically approximating the posterior
distribution, which is generally a really complicated integral. If you
aren't familiar with Bayesian stats, don't worry about it, it doesn't
matter for our purposes.

There are many variants, but at its core, MCMC operates kind of like
simulated annealing, but with a constant temperature. That means you
don't get the automatic transition between exploration and
exploitation at all.

So who cares? Why use it for optimization? Because...

# Metropolis-Coupled Markov Chain Monte Carlo (MCMCMC)

If you're really hardcore, two MCs aren't enough for you. You need
three! MCMCMC (MC^3) is also technically designed to solve complicated
integrals, but it also happens to work well as an optimization algorithm
for many problems. Each "MC" refers to a different mathematical property of the
algorithm. These are things you care about if you want to solve an
integral, but all we want is a pretty good solution to a hard
optimization problem, so these properties don't really matter to
us. In fact, ignore the real name. I'm going to call this
Multiple-Chain MCMC, because the multiple chains are what really make
this algorithm stand out.

MC^3 is an interesting blend between simulated annealing and MCMC,
with multiple chains running at once. The way it works is that we run
many chains simultaneously, each at a different temperature. These
temperatures will range from very hot (almost a random walk) to very
cold (almost a hill climber). We let these chains run for several steps,
but every so often, we give them a chance to swap.

This is really the critical part of MC^3. Starting from the hottest
temperature, we look and see how good the solution is. If the solution
is good enough, we swap it with the adjacent temperature. Then we move
to the next hottest, and repeat until all chains have had an
opportunity to swap. We call this a "bucket brigade" process because
good solutions will percolate up to cooler temperatures, being passed
up and up like water in a bucket brigade. This means that once we find
promising regions of the landscape, we quickly cool those solutions
and do a local search. The multiple chain setup allows us to both
explore and exploit at the same time. I've had fantastic results with
this algorithm, and I think it deserves more attention than it's
gotten in an optimization context.

If you're using MC^3 for Bayesian statistics, the probability of
accepting a swap is based on a likelihood ratio. Our fitness function
is not necessarily a mathematical likelihood, but we can still
calculate this value using fitnesses. Here it is for a minimization
problem:

```exp[(coldFit - hotFit)*(1/coldTemp - 1/hotTemp)]```

where `coldFit` is the fitness of the colder solution, `coldTemp` is
the temperature of the colder solution, and so on. Notice that this is
fairly similar to the probability used for simulated annealing, but
that the difference between the hot and cold temperatures matters
here. This is because if you're proposing to significantly cool a
chain, you want to make very sure that it's a promising area of the
solution space. The upshot is that it's more difficult to successfully
swap solutions if they are at very different temperatures. Choosing
how to space the temperatures of the different chains is a complicated
problem. My solution has been to try uniform and logarithmic spacing,
then using whichever performed better in trial runs.

The performance cost for the added power of MC^3 is the fact that you
have to run many chains
at once, which takes more time. There is also lots of parameters to
tune: number of chains, temperature spacing, and length of runs all
need to be tuned. This involves a lot of trial and error. To my
knowledge, there is no automated way to tune parameters for
optimization algorithms like this. However, for my money, the
additional runtime and complexity of the code and parameter tuning is
very worth it for many problems.

# Evolutionary Algorithms/Genetic Algorithms

Evolutionary algorithms (EAs), as the name suggests, are inspired by the
evolution of biological species. In general, EAs involve some
population of solutions, in which the fittest solutions reproduce more
often. Over time the population will evolve toward promising areas of
the solution space. There are many kinds of EAs, designed for all
sorts of optimization problems. For simplicity, I will focus on
genetic algorithms (GAs), which are designed to for problems with
discrete, rather than continuous, solutions.

There are many, many variants on GAs, but the general flow is as
follows: (1) Generate a population of N solutions. (2) Using some
selection algorithm, select N "parent" solutions. This selection
process generally involves a fitness function, along with some
randomness. Solutions with high fitness are likely to be a parent
multiple times, and less fit solutions may not reproduce at all. (3)
Mutate each parent solution to produce N "child" solutions. (4) Return
to step 2 with child solutions as the new population to select from.

While MC^3 maintains variability by allowing hot chains to continually
explore new territory, GAs maintain variability by maintaining a
population. This variability will decrease over time, as better
solutions continue to reproduce and take over the population. Thus,
the GA is able to transition from exploration to exploitation without
a temperature parameter.

The selection step is very important for determining how quickly your
population converges. You want less fit solutions to get lucky
and reproduce sometimes, because they allow you to explore different
areas of the solution space. I tend to use tournament selection. This
means that I randomly choose N solutions, and the fittest of the group
becomes a parent for the next generation. Larger values of N mean the
competition is fiercer, and suboptimal solutions are less likely to
reproduce. I usually use a tournament size of 2 or 3, making
competition relatively weak to keep the population variable for as
long as possible.

The mutation step for GAs works just like it does in the other
algorithms. However, because you are maintaining a population, you can
allow recombination, in which multiple parents are combined to form a
single child solution. Occasionally allowing solutions to recombine
can introduce new variability into the population, but it is not
appropriate for all problems. In the travelling sales rep problem, for
example, each destination should appear in the solution once. For
instance, if you want the optimal path between locations {A,B,C,D},
it's not obvious how you would combine the path A -> B -> C -> D with
D -> B -> A -> C.

GAs are very slow in comparison to the other algorithms, because it is
constantly maintaining and updating so many solutions. However, it
does a fairly good job of exploring the solution space, and the speed
cost can sometimes be worth the tradeoff.

# Conclusion

Heuristic search algorithms come in many varieties, and may perform
better or worse depending on the specific problem to be
solved. Unfortunately, it is very difficult to know ahead of time if
you have a problem where simulated annealing will succeed, or if an
evolutionary algorithm will be necessary, or if repeated hill climbers
will perform best. These algorithms can become computationally
expensive very quickly depending on your fitness function and your
input parameters (population size for GAs, number of chains for MC^3,
and so forth). My preferred approach is to try them all and see what
works best. Luckily, once you've written each algorithm once, it's
very easy to plug in different fitness and mutation functions. It's
easy to make the code modular, so you don't have to rewrite everything
for a new optimization problem.

In general, I have the most consistent luck with hill climbers and
with MC^3. I've found that simulated annealing and genetic algorithms
both have a tendency to converge too quickly, unless everything is
very carefully tuned. Hill climbers can just be run many times until
convergence, and MC^3 is constantly getting new variation via the hot
chains, so I've found that it takes less tuning to get good results. I
encourage you to try them all and see what works best for your
problem. Simple examples of each algorithm, as applied to the
travelling sales rep problem, can be found on my
[github](https://github.com/elsander/GoodEnoughAlgs). Happy coding!
