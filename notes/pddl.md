---
title: Planning Domain Definition Language (PDDL)
layout: note
---

# PDDL

- high
- level
- points

## Search strategies

## Fast Downward planner

note which heuristics support which features: http://www.fast-downward.org/Doc/Heuristic

## Example: Blocks world

### Domain file

~~~
;; domain file: blocksworld-domain.pddl

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

~~~
;; problem file: blocksworld-prob1.pddl

(define (problem blocksworld-prob1)
  (:domain blocksworld)
  (:objects a b)
  (:init (on-table a) (on-table b) (clear a) (clear b))
  (:goal (and (on a b))))
~~~

### Running the planner

Use this command:

~~~
java -cp . javaff.JavaFF blocksworld-domain.pddl blocksworld-prob1.pddl
~~~

You should see this output:

~~~
Parsed Domain file blocksworld-domain.pddl successfully
Parsed Problem file blocksworld-prob1.pddl successfully
Performing search as in FF - first considering EHC with only helpful actions
2
1
(pickup a)
(stack a b)
Instantiation Time =            0.07sec
Planning Time = 0.03sec
~~~

## Example: Cargo

### Domain file

~~~
;; domain file: cargo-domain.pddl

(define (domain Cargo)
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
;; problem file: cargo-prob1.pddl

(define (problem cargo1)
  (:domain Cargo)
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
C:\Users\Joshua\Documents\GitHub\teach-planning>java -cp . javaff.JavaFF cargo-domain.pddl cargo-prob1.pddl
Parsed Domain file cargo-domain.pddl successfully
Parsed Problem file cargo-prob1.pddl successfully
Performing search as in FF - first considering EHC with only helpful actions
6
4
3
1
(fly p1 jfk sfo)
(unload c2 p1 sfo)
(load c1 p2 sfo)
(fly p2 sfo jfk)
(unload c1 p2 jfk)
Instantiation Time =            0.075sec
Planning Time = 0.057sec

C:\Users\Joshua\Documents\GitHub\teach-planning>
~~~


