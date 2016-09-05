---
title: Planning Domain Definition Language (PDDL)
layout: note
---

# PDDL

In the late 1990s, the yearly International Planning Competitions (IPL) began in order to encourage development of efficient planning search algorithms. A new language, PDDL (Planning Domain Definition Language) was designed in order to have a single unified syntax for representing planning problems for the competition. PDDL has improved over time to support many sophisticated features, such as variable types, action costs, action preferences, deadlines, etc. even though no planning software available today supports all these features of PDDL.

The main inspiration for the PDDL language was the STRIPS language from the early 1970s, developed at SRI International. STRIPS (Stanford Research Institute (SRI) Problem Solver) was used to help [Shakey the Robot](https://en.wikipedia.org/wiki/Shakey_the_robot) solve various tasks.

## Syntax

Planning problems represented in PDDL are separated into two parts: the domain and one or more problems. The domain PDDL file lists the actions available in the domain. A problem PDDL file describes the initial state and goal criteria for a particular scenario. The idea is that the same domain file will be useful for many different problem scenarios.

Many examples of PDDL files are available on londo at `/home/jeckroth/csci431/pddl-examples`.

### Domain file

A domain file, typically named `domain.pddl`, has the following sections:

- the name of the domain
- a list of predicates and each predicate's variables (recall that a predicate represents a single property of a state)
- a list of actions; for each action, its parameters, preconditions, and effects may be stated

Here is an example domain file, for the "[blocks world](https://en.wikipedia.org/wiki/Blocks_world)" planning problem. Note that variables are marked with a question mark, as in `?x`, and the syntax generally uses Lisp's prefix notation and keywords (preceded by colons, e.g., `:parameters`).

~~~
;; domain.pddl for blocksworld

(define (domain blocksworld)
  (:requirements :strips)

  (:predicates (clear ?x)
               (on-table ?x)
               (holding ?x)
               (on ?x ?y))

  (:action pickup
           :parameters (?ob)
           :precondition (and (clear ?ob) (on-table ?ob))
           :effect (and (holding ?ob) (not (clear ?ob)) (not (on-table ?ob))))

  (:action putdown
           :parameters (?ob)
           :precondition (and (holding ?ob))
           :effect (and (clear ?ob) (on-table ?ob) 
                        (not (holding ?ob))))

  (:action stack
           :parameters (?ob ?underob)
           :precondition (and  (clear ?underob) (holding ?ob))
           :effect (and (clear ?ob) (on ?ob ?underob)
                        (not (clear ?underob)) (not (holding ?ob))))

  (:action unstack
           :parameters (?ob ?underob)
           :precondition (and (on ?ob ?underob) (clear ?ob))
           :effect (and (holding ?ob) (clear ?underob)
                        (not (on ?ob ?underob)) (not (clear ?ob)))))
~~~

### Problem file

A problem file lists the domain, objects that can be used in place of variables, a description of the initial state (using the predicates listed in the domain file), and goal criteria (again, using predicates).

~~~
;; problem file: blocksworld-prob1.pddl

(define (problem blocksworld-prob1)
  (:domain blocksworld)
  (:objects a b)
  (:init (on-table a) (on-table b) (clear a) (clear b))
  (:goal (and (on a b))))
~~~

### Conditional preconditions and effects

An extension known as [ADL](https://en.wikipedia.org/wiki/Action_description_language) (Action Description Language) allows PDDL actions to specify `forall` and `exists` conditions on preconditions and `forall` and `when` on effects (using `exists` on an effect doesn't make sense). To use ADL, `:adl` must be specified in requirements section of the domain file (alongside `:strips`), e.g.:

~~~
(define (domain mydomain)
  (:requirements :strips :adl)
...
~~~

Here is the sytnax for ADL:

~~~
(forall (?v1 ?v2 ...) condition_formula)
(exists (?v1 ?v2 ...) condition_formula)
(when condition_formula effect_formula)
~~~

Here is an example (in a domain file of course) for an elevator ("lift"):

~~~
(:action stop
  :parameters (?floor)
  :precondition (lift-at ?floor)
  :effect (and
               (forall (?passenger)
                  (when (and (boarded ?passenger)
                             (destin ?passenger ?floor))
                        (and (not (boarded ?passenger))
                             (served ?passenger))))
               (forall (?passenger)
                   (when (and (origin ?passenger ?foor)
                              (not (served ?passenger)))
                              (boarded ?passenger)))))
~~~

Here is an example for refrigerator repair:

~~~
(:action unfasten
         :parameters (?screw)
         :precondition (and (exists (?fridge)
                                  (exists (?compressor)
                                     (and (attached ?compressor ?fridge)
                                          (fits ?screw ?compressor)
                                          (not (fridge-on ?fridge)))))
                            (screwed ?screw))
         :effect (not (screwed ?screw)))
~~~

### Types

PDDL also supports typed variables. The syntax can be seen in an elevator planning problem. Domain file:

~~~
(define (domain elevator)
  (:requirements :strips)
  (:types passenger - object
          floor - object
  )
  (:predicates 
    (passenger-at ?person - passenger ?floor - floor)
    (above ?floor1 - floor  ?floor2 - floor)
    (boarded ?person - passenger)
    (lift-at ?floor - floor)
  )
  (:action board
    :parameters (?f - floor ?p - passenger)
    :precondition (and (lift-at ?f) (passenger-at ?p ?f))
    :effect (and (not (passenger-at ?p ?f))
                 (boarded ?p)))
etc.
~~~

Problem file:

~~~
(define (problem instance3)
   (:domain elevator)
   (:objects p0 p1 p2 - passenger
             f0 f1 f2 f3 f4 f5 - floor
   )
   (:init
     (above f0 f1)
     (above f0 f2)
     (above f0 f3)
etc.
~~~

When using ADL conditionals, types are often employed, e.g., the refrigerator example again:

~~~
(:action unfasten
         :parameters (?x - screw)
         :precondition (and (exists (?f - fridge)
                                  (exists (?c - compressor)
                                     (and (attached ?c ?f)
                                          (fits ?x ?c)
                                          (not (fridge-on ?f)))))
                            (screwed ?x))
         :effect (not (screwed ?x)))
~~~

## Fast Downward planner

Many planners are available, most of which were created for one or more IPC competitions over the years. As research projects, these planners are often incomplete and difficult to use. Helpful error messages for badly written PDDL files are often missing.

The [Fast Downward](http://www.fast-downward.org/HomePage) planner is one such competition-winning planner. It supports a subset of PDDL that is sufficient for our interests. It also allows the user to specify a search strategy (e.g., A\*) and heuristic. Fast Downward translates the PDDL specification into several graphical structures that it then searches with A\* or some other strategy. We will not examine how Fast Downward does so, but if you are interested in the details, several papers and presentations [are available](http://www.fast-downward.org/Literature).

### Running the planner

On londo, use this command in the same directory as your PDDL files:

~~~
fast-downward.py prob1.pddl --search "astar(blind())"
~~~

where `prob1.pddl` is the problem file (the domain file does not need to be specified if it's named in a conventional manner). Options for the `--search` parameter and the heuristic function (in this case, `blind()`) are discussed below. The resulting plan, should one be found, will be in a file named `sas_plan`.

### Search strategies

Refer to Fast Downward's [documentation](http://www.fast-downward.org/Doc/SearchEngine) for details.

- A\*: "A\* is a special case of eager best first search that uses g+h as
  f-function. We break ties using the evaluator. Closed nodes are re-opened.";
specify as `--search astar(heuristic)`
- Best-first (eager): specify as `--search eager_greedy(heuristic)`
- Best-first (lazy): specify as `--search lazy_greedy(heuristic)`

The difference between eager and lazy is described by [Richter and Helmert (2009)](http://www.avacs.org/fileadmin/Publikationen/Open/richter.icaps09.pdf) as the following: "To expand a state, Lazy only puts two pointers into the open list, while Eager generates and evaluates all successor states." They compare various heuristics and search techniques with Fast Downward, and state, "[t]he reason for examining both variants [eager and lazy] is that they can behave very differently, with Lazy being less informed than Eager, but paying a smaller price for wrong expansions. [...] Along the way, we answer the question whether deferred evaluation is useful in general [...]" They find that, "[c]omparing Eager with Lazy, we note that Eager performs better in most of the configurations; [...] The best performance obtainable from each search variant [...] is very similar. Lazy consistently evaluates fewer states than Eager and is usually faster. Eager, on the other hand, finds plans of better quality. This is not surprising as Eager is better informed with respect to heuristic values than Lazy."

### Heuristics

Each search strategy listed above makes use of (one or more) heuristic functions. (The Fast Downward documentation also refers to such functions as "evaluators.") The choice of heuristic can affect the time and memory required to find a plan as well as the quality of the plan. Just like we saw with [informed search](/notes/informed-search.html), not all heuristics are admissible. Recall that admissibility means the heuristic $h$ is always either perfect or an underesimate of the cost to the goal. A search algorithm like A can find optimal goals only if the heuristic is admissible (thus giving us the algorithm known as A\*).

Fast Downward supports [several heuristics](http://www.fast-downward.org/Doc/Heuristic), not all admissible. Also of critical importance to us is whether or not the heuristic supports all of the PDDL features we need (e.g., for [assignment A02](/assignments/A02.html)). Specifically, we need the feature "conditional effects." In the following table, only those heuristics that support conditional effects are included.

| Heuristic | Admissible? | Syntax |
|---|
| [Additive](http://www.fast-downward.org/Doc/Heuristic#Additive_heuristic) | No | `add()` |
| [Blind](http://www.fast-downward.org/Doc/Heuristic#Blind_heuristic) | Yes | `blind()` |
| [Context-enhanced additive](http://www.fast-downward.org/Doc/Heuristic#Context-enhanced_additive_heuristic) | No | `cea()` |
| [Causal graph](http://www.fast-downward.org/Doc/Heuristic#Causal_graph_heuristic) | No | `cg()` |
| [Constant](http://www.fast-downward.org/Doc/Heuristic#Constant_evaluator) | No | `const()` |
| [FF](http://www.fast-downward.org/Doc/Heuristic#FF_heuristic) | No | `ff()` |
| [Goal count](http://www.fast-downward.org/Doc/Heuristic#Goal_count_heuristic) | No | `goalcount()` |
| [Max](http://www.fast-downward.org/Doc/Heuristic#Max_heuristic) | Yes | `hmax()` |

With this table and the list of search strategies above, you can experiment with different search strategies and heuristics, such as:

~~~
--search "astar(blind())"
--search "astar(ff())"
--search "eager_greedy(const())"
--search "lazy_greedy(add())"
etc.
~~~


## Another example: Cargo

Here is another simple example of a STRIPS planning problem. The last steps show how to run Fast Downward and display the resulting plan.

### Domain file

~~~
(define (domain cargo)
  (:requirements :strips)

  (:predicates (in ?c ?p) (out ?c) (at-airport ?x ?a)
               (plane ?p) (airport ?a) (cargo ?c))

  (:action load :parameters (?c ?p ?a)
           :precondition (and (at-airport ?c ?a) (at-airport ?p ?a) 
                              (out ?c) (plane ?p) (cargo ?c) (airport ?a))
           :effect (and (not (at-airport ?c ?a)) (in ?c ?p) (not (out ?c))))
  
  (:action unload :parameters (?c ?p ?a)
           :precondition (and (in ?c ?p) (at-airport ?p ?a)
                              (plane ?p) (cargo ?c) (airport ?a))
           :effect (and (not (in ?c ?p)) (at-airport ?c ?a) (out ?c)))
  
  (:action fly :parameters (?p ?from ?to)
           :precondition (and (at-airport ?p ?from) 
                              (plane ?p) (airport ?from) (airport ?to))
           :effect (and (not (at-airport ?p ?from)) (at-airport ?p ?to))))
~~~

### Problem file

~~~
(define (problem cargo1)
  (:domain cargo)
  (:objects c1 c2 jfk sfo p1 p2)

  (:init (airport jfk) (airport sfo)
         (plane p1) (plane p2)
         (at-airport p1 jfk) (at-airport p2 sfo)
         (cargo c1) (cargo c2)
         (at-airport c1 sfo) (at-airport c2 jfk)
         (in c2 p1) (out c1))

  (:goal (and (at-airport c1 jfk) (out c1) (at-airport c2 sfo) (out c2))))
~~~

### Running the planner

~~~
fast-downward.py prob1.pddl --search "lazy_greedy(add())"
~~~

If a plan is found, show the plan with `cat`:

~~~
cat sas_plan
~~~

Here is the result:

~~~
(fly p1 jfk sfo)
(unload c2 p1 sfo)
(load c1 p2 sfo)
(fly p2 sfo jfk)
(unload c1 p2 jfk)
; cost = 5 (unit cost)
~~~


