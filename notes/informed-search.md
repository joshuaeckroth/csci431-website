---
title: Informed search
layout: note
---

# Informed search

Most search problems can be solved more efficiently and find better solutions than breadth-first or depth-first search. All we need is a way of judging which actions should be examined first based on some function of goodness, called a "heuristic." Based on exactly how that function is defined, we have one of several possible informed search strategies, of which some are more appropriate than others for certain tasks.

## Heuristics

In the uninformed searches, the random search used no particular
technique for choosing the next state to check. Breadth-first search
(BFS) checked the earliest discovered state, and depth-first search
(DFS) checked the most recently discovered state.

BFS is probably an acceptable solution if the goal state is not deep in
the search graph. DFS is probably an acceptable solution if the opposite
is the case. Naturally, random search is probably never a good idea.

However, many problems do not fit simple descriptions like "the goal
state is not deep in the graph." The goal state may be anywhere, and
in different occasions may be deep or shallow in the graph.

Rather than choose how to search based solely on breadth-first or
depth-first, we can often come up with better *heuristics*. A
heuristic is a "rule of thumb," or some kind of rule that's "usually a
good idea." BFS and DFS have their own heuristics but their heuristics
(how to choose the next states to check) do not change depending on
the problem. Normally, we talk about heuristics in a more
problem-specific sense. A heuristic would be applied in step `4.a.`,
in which we choose the next state to check.

The heuristic always represents "estimate of how close this state is to the goal," so you always want to look at states with a lower heuristic value. As we'll see below, the heuristic value is not the whole story; we will also want to know the path cost to this state (cost so far) plus the heuristic (estimated cost from here to the goal).

### Heuristics for 8-puzzle

Here are some heuristics we might apply to the 8-puzzle problem. Each
heuristic is a function of a state, i.e., $h(s) = n$ where $s$ is a
state and $n$ is an integer:

- OP: The number of tokens that are out of place.

- MD: The sum of "[Manhattan-distances](https://en.wikipedia.org/wiki/Taxicab_geometry)" between each token and its
  goal position.

- RC: Number of tokens out of row plus number of tokens out of column.

These heuristics will be compared below after we discuss our first informed search strategy, "best-first search."

## Best-first search

Best-first visits first whichever state in the openset has the lowest heuristic value. This preference occurs at step `4.a.` in the generic search algorithm. The preference makes best-first search appear to jump around the search tree, like a
random search, but of course best-first search is not random. The
memory requirements for best-first search are not as bad as
breadth-first. This is because breadth-first search does not use a
heuristic to avoid obviously worse states.

Here is a comparison of the different 8-puzzle heuristics. We see that MD, Manhattan distance, is best in all ways so we'll continue only with that heuristic in the remainder of our analyses.

| Heuristic | Avg. path cost | Avg. checked states | Avg. max openset |
|---|
| OP | 22.04 | 168.09 | 116.13 |
| MD | 14.58 | 60.79 | 52.01 |
| RC | 18.30 | 70.56 | 65.02 |

### Beam search

Beam search is a variant of best-first search that limits the size of the openset to a maximum number of states. Suppose this maximum is 10 states. Then whenever a newly discovered state is discovered, its heuristic value is calculated and the state is added to the openset (ordered by heuristic value). Next, if the openset has grown too large, only the best 10 states are retained.

Beam search therefore caps the memory requirements for search. This is essential in very large search spaces such as speech recognition and machine translation, where "possible transitions" includes all the words in the dictionary. Beam search is also useful for simulating agents with limited memory, such as humans and other animals.

## Just a heuristic is not enough

Unfortunately, both best-first and beam search can
yield terrible solutions. Imagine a maze-navigating robot in a maze
that has some windows in the walls. The robot can sometimes see the
goal through windows. A best-first or beam search robot may
believe it is close to the goal because it can see it through some
windows, but in actuality it is quite far from the goal, and must
actually back up to make progress. Instead, however, the robot keeps
going deeper and deeper into the maze, enticed by the illusory
nearness of the goal.

If the robot simply kept track of how "deep" it had gone into the
maze, and if it had reason to believe that the goal cannot possibly be
"this deep," it would back up before going even deeper.

In other words, there is more to an estimate of "goodness of a state" than the heuristic that estimates 
closeness to the goal. One must also keep in mind how many steps have
already been taken. For example, if we are solving the 8-puzzle game,
we should consider how many moves have already been made. If we are
planning a driving route, we should consider how far we have already
traveled (or planned to travel); perhaps there is a shorter route if
we abandon the path we are on. (These considerations take place during the route-finding process, not while actually on the road, since you cannot just transport yourself to an alternative state from the openset!)

Thus, a good cost function actually has two parts: $f(s) = g(s) +
h(s)$, where $s$ is a state, $g$ computes the *actual* cost of arriving at $s$
from the initial state (the number of moves to $s$ or the distance
traveled so far; we know this value by examing the state's parents), and $h$ is our heuristic, i.e., our estimate of how
far away $s$ is from the goal.

In the 8-puzzle, $g$ is simply the number of moves made so far (to get
to state $s$); in the routing problem, $g$ is the miles traveled so
far. In the 8-puzzle, $h$ is one of the heuristics described above
(OP, MD, RC); in the routing problem, $h$ is perhaps an
"as-the-crow-flies" distance (i.e., Euclidean distance) from $s$ to
the goal.

## A\* search

Let's modify best-first search to use the new cost function $f(s)
= g(s) + h(s)$. This is the algorithm known as A (not A\*).

If $g$ is the number of moves made so far (number of "hops" in
the search tree) and $h$ is always equal to 0.0, then we have
breadth-first search. Recall that breadth-first search is optimal for
unweighted graphs (graphs where the edges always have weight 1.0).

Actually, the A algorithm is optimal as well, even on weighted graphs,
if we keep $h$ constantly equal to 0.0. This is because it acts like
breadth-first search, in that it always considers better alternatives
before going "deeper." The A algorithm here is optimal on weighted
graphs because it simply considers the true path cost (unlike
breadth-first search), and proceeds from the lowest cost path. 

Furthermore, if the heuristic $h(s)$ always *underestimates* the true cost of a path
from $s$ to the goal, then the A algorithm is optimal. If we call the
true cost function from $s$ to the goal $h^\*(s)$, then we are saying
that if $h(s) \leq h^\*(s)$ for all $s$, then A is optimal. When this
restriction is met, we call the algorithm A\*.

An $h$ that meets this *underestimation* criterion is an *admissible*
heuristic, and turns the A algorithm into A\*, which we call an
admissible search algorithm. Furthermore, A\* is the most efficient
algorithm (called "optimally efficient") that uses some particular
heuristic $h(s)$. This means that any other search algorithm using the same
heuristic will check no fewer states than A\*.

How poorly $h$ estimates the actual cost to the goal makes one $h$
better than another. With $h(s)=0.0$ for all $s$, we have a very poor
heuristic (unless the cost truly is always 0.0, but then why
search?). Ideally, $h$ is as close as possible (or equal to) $h^\*$ but
still underestimates (if not equal).

If $h$ overestimates the true cost to the goal, then our search
algorithm will obviously make the wrong choices. It will move the
robot farther from the end of the maze rather than towards the end.

### IDA\* (Iterative Deepening A\*)

The A\* algorithm has very bad space complexity (see below) because it
might, in the worst case, keep a memory of every state ever
visited. An alternative approach is to use an iterative deepening
strategy, just like IDDFS, except that we bring over the complex
heuristic $f(s) = g(s) + h(s)$. IDA\* is IDDFS except that the search
stops looking deeper if $f(s) > t$ for some threshold $t$. This
threshold is increased (by $1$, say) for every iteration. Thus, IDA\*
is the same as IDDFS except that IDDFS use a "depth" threshold while
IDA\* uses a heuristic cost threshold. Similar to how IDDFS improves on
BFS, IDA\* can reach "deep" solutions faster than A\* but also find just
as optimal solutions since it uses the same estimator function $f(s)$
as A\*.

## Theoretical comparison

Refer to the [search notes](/notes/search.html) (at the bottom) for an
explanation of the variables $n$ (total possible states), $b$ (branch
factor), $d$ (search depth of least-cost solution), and $m$ (maximum
search depth, which may be $\infty$).

| Metric                            | BFS      | DFS              | Beam | Best-first | A\*       |
|-----------------------------------+----------+------------------+---------------+------------+----------|
| Complete?                         | Yes      | Yes              | No            | Yes        | Yes      |
| Always optimal? (informed search) | No       | No               | No            | No         | Yes      |
| Time complexity                   | $O(b^d)$ | $O(b^d)$         | $O(m)$        | $O(b^d)$   | $O(b^d)$ |
| Space complexity                  | $O(b^d)$ | $O(m)$ or $O(n)$ | $O(m)$        | $O(b^d)$   | $O(b^d)$ |

Note that A\* is, in the worst case, just as bad as BFS in terms for
time complexity. But, we can give A\* a good heuristic function and its
time complexity will decrease, while BFS will stay the same. Also, we
know that A\* is optimally efficient for some heuristic function; that
is to say, no other algorithm can use the same heuristic function to
find the goal faster. A\* makes the best use of a heuristic function by adding the value of the heuristic to the actual path cost.

## Experiments

Now we'll look at the results of experiments with the 8-puzzle
problem. The breadth-first search (BFS) results are the $y=0$
line. Results from other searches are shown as how far they differ
from BFS. So, if a search is higher than the $y=0$ line in these results
it performs worse (in all graphs, higher is worse); if it is under the
$y=0$ line, it performs better than BFS.

The x-axis in the graphs shows the complexity of the 8-puzzle
problem. Starting with a solved 8-puzzle, we perform some number of
random moves. The optimal solution to the puzzle is at most that many
moves (the moves we made may "cancel each other out" in some cases, so
the optimal solution back to the starting state may involve fewer
moves). Since random search performs so badly, we do not show its
performance characteristics in some of the graphs after 4 moves.

### Number of checked states (time)

![States checked](/images/8puzzle-search-checked.png)

We use "number of checked states" as a proxy for computational
time. Random is obviously quite bad. Interesting, IDDFS is also bad;
this is because, at least how our algorithm works, many states are
checked more than once when the algorithm restarts with greater depth
limits.

Notice A\* gets to the goal fastest. This is because A\* is "optimally
efficient."

### Maximum number of states in memory

![States in memory](/images/8puzzle-search-memory.png)

"Memory" is measured as the maximum size ever encountered of the
`openset`. BFS uses a lot of memory because it adds all possible transitions from a level of the search tree before proceeding to the next level and visiting those new states.

### Length of path (goodness of solutions)

![Search path](/images/8puzzle-search-path.png)

We learned that A\* is optimal (with respect to the path cost of the
solution). Breadth-first search is also optimal since we have a graph
with edge weights all equal to 1.0. So, A\* and BFS perform just as
well, and nothing performs better.

Best-first and beam search got lost "in the weeds," producing
terrible solutions, because they do not consider the complexity if
their search path (the depth of their search tree). They just keep
searching further. DFS would do the same except that we are using
IDDFS instead (depth-limited DFS); IDDFS finds shorter solutions
before longer solutions, and thus nearly always finds the optimal
solution. Interestingly, random search finds near-optimal solutions,
possibly because it is more likely it will look at a shallow node
(there are up to four from each state) than a deep node.

## More experiments

### Maze path-finding

All numbers are compared to breadth-first search, the control
case. The numbers reflect averages across 20 runs.

| Search type   | Resulting path length | Largest closedset | Largest openset |
|---------------+-----------------------+-------------------+-----------------|
| A\*            |                   0.0 |            -524.8 |            -0.1 |
| Beam |                +213.0 |           -2166.9 |           -17.9 |
| Best-first    |                   0.0 |              +0.8 |            +0.2 |
| Depth-first   |                +505.4 |             -83.5 |           +43.6 |

### Discussion

A\* and best-first again find the best paths, which aren't any better
than those found by breadth-first, since all movements cost the same.

However, A\* considers far fewer states. Beam likewise, but
beam is terrible for finding optimal paths (in this case).

As expected, depth-first also does not find short paths.
