---
layout: note
title: Midterm review
categories: [studyguides]
date: Wed Oct 28
---

# Midterm review

## Turing Test

Understand what the Turing Test is testing, and what is required to pass it. Be able to explain how to run a proper Turing Test. Be able to explain the significance of a computer program that passes the Turing Test.

## Uninformed and Informed search

Given a particular initial state, fully-specified possible transitions for all states, goal criteria, action cost for every state transition, and heuristic function, be able to list the order that states are visited in a breadth-first or depth-first or best-first or A* search. (Assume the heuristic is "admissible," i.e., an underestimate, and assume that breadth-first and depth-first follow children in alphabetical order.) Also be able to list the sequence of transitions ("path") that each algorithm finds from the intial state to the goal.

For example, here is a graphical representation of possible transitions and action costs. The initial and goal states are also indicated. The heuristic value for each state is also shown in the state as "h = value". The starting state is shown as a box (node "A"), and the goal state is shown as a double circle (node "G").

![Transitions](/images/midterm-review-search.png)

## Planning

Know how to read and write PDDL files. Be able to generate a valid plan from given PDDL problem and domain files, without using any software (be able to do it "by hand").

Be able to write a PDDL file to represent a planning problem given in English.

## Adversarial search

Given some starting tic-tac-toe board, be able draw and label a tic-tac-toe search tree, like the one below, with min/max labels on downward arrows, and utility values (1 or 0 or -1) on upward arrows.

For example, starting with this board:

![Tic-tac-toe start](/images/midterm-guide-ttt-start.png)

Answer:

![Tic-tac-toe solution](/images/midterm-guide-ttt-solution.png)

## Genetic algorithms

Be able to define a genetic representation, fitness function, crossover function, and mutation function for a scenario specified in English. Be specific: indicate exactly how the representation and functions work. You will lose points if it is unclear to me exactly how they work.

For example, consider the following problem:

> Suppose we have six surveillance cameras and need to install them in the ceiling of a grocery store in order to view and record as much floor area as possible. Suppose the store is rectangular but some aisles are taller than others, blocking a camera's view from certain angles. Assume the cameras cannot pan, tilt, or zoom once they are positioned. We want to find an optimal or near-optimal placement of cameras so that the most area is covered.

## Rules engine

Given a set of rules described in English, be able to write Drools rules that represent the same logic. The Drools file must have the correct syntax, but you do not need to provide the rest of the Java program to run the rules.

For example, consider the following logic about an issue tracking (ticketing) system:

> A ticket is either open or closed, every ticket has an integer priority, as well as a "newCustomer" boolean field. Whenever a customer creates a ticket, the ticket gets added to the database. If the customer has never issued a ticket before, the ticket should have its "newCustomer" field set to "true". Each ticket starts at priority 0. If the description of the ticket matches an existing open ticket (assume a matching function exists), then its priority is changed to the priority of the existing ticket plus 1; the priority of the existing tickets are left unchanged. If the customer has submitted more than 2 tickets with the same description, the priority of the new ticket is increased by 1. If the resulting priority of the ticket is greater than 3, an email is sent to the customer stating that "we are working hard on this problem." If the description of the ticket matches a closed ticket, this ticket is closed as well (and priority is set to -1).

## Prolog

Given a particular "trace" of a Prolog search process for given rules, be able to list the facts that must be present. These facts will not be given to you, they must be inferred from the trace.

Know how to write Prolog code that represents a given set of facts and rules, stated in English.

Here are some examples:

- Familial relations: mother, father, aunt, uncle, niece, nephew, cousin, half-brother, half-sister, etc.
- Or, from Monty Python: "A witch is a female who burns. Witches burn - because they're made of wood. Wood floats. What else floats on water? A duck; if something has the same weight as a duck it must float." (quoted from [this Prolog tutorial](http://www.allisons.org/ll/Logic/Prolog/Examples/witch/)) The resulting Prolog code should be able to prove that witches float.

