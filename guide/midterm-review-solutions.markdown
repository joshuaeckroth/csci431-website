---
layout: note
title: Midterm review solutions
categories: [studyguides]
---

# Midterm review solutions

## Turing Test

> Understand what the Turing Test is testing, and what is required to pass it. Be able to explain how to run a proper Turing Test. Be able to explain the significance of a computer program that passes the Turing Test.

The Turing Test is intended to definitively decide whether a machine is *as intelligent as* a human. The machine is not required to process or "think" just like a human. It is only required to be indistinguishable from a human, in terms of textual conversation, in an adversarial environment. The adversarial environment is important so that a machine does not pass the test just by chance. No machine or software has ever come close to passing a Turing Test; every attempt has produced laughable results. Thus, passing a Turing Test would be quite the accomplishment, though it is not clear exactly the implications. Perhaps we would begin to doubt the authenticity of every form of written communication, unable to be certain a human authored the text or, instead, some robot is pushing some malevolent robot agenda.

## Uninformed and informed search

> Given a particular initial state, fully-specified possible transitions for all states, goal criteria, action cost for every state transition, and heuristic function, be able to list the order that states are visited in a breadth-first or depth-first or best-first or A* search. (Assume the heuristic is "admissible," i.e., an underestimate, and assume that breadth-first and depth-first follow children in alphabetical order.) Also be able to list the sequence of transitions ("path") that each algorithm finds from the intial state to the goal.
>
> For example, here is a graphical representation of possible transitions and action costs. The initial and goal states are also indicated. The heuristic value for each state is also shown in the state as "h = value". The starting state is shown as a box (node "A"), and the goal state is shown as a double circle (node "G").
>
> ![Transitions](/images/midterm-review-search.png)

Here is depth-first search ordering:

```
1. A
2. B (rather than C due to alphabetical ordering)
3. E
4. D (rather than F)
5. C (rather than F or G, and E is not available since we've already been there)
6. F (D is not available, already been there)
7. G (goal found, we're done)
```

Here is best-first search ordering (using only heuristic value):

```
1. A
2. C (preferred over B since h=3 for C is lower than h=5 for B)
3. D (preferred over F)
4. G (preferred over E and F; goal found, we're done)
```

## Planning

> Be able to write a PDDL file to represent a planning problem given in English.
>
> For example, consider this planning problem:
>
> > The goal is to wash and dry a load of laundry. There is one washer and one dryer. The clothes must be washed before they can be dried. In order to wash clothes, the clothes have to be loaded into the washer, soap added, and the washer turned on. There can't be clothes already in the washer. Likewise, the clothes cannot be moved to the dryer if there are already clothes in the dryer. The initial state is that the washer is empty, the dryer is empty, and we have a basket of dirty clothes. Ultimately, we want both the washer and dryer to be unloaded, and the basket to have clean clothes.

Here is one possible solution. First, the `domain.pddl` file:

```lisp
(define (domain clothes)
  (:requirements :strips)

  (:predicates
    (empty ?x)
    (clothes-in ?x)
    (active ?x)
    (soap-added)
    (clothes-clean)
    (clothes-dry)
  )

  (:action
    add-clothes-to-washer
    :precondition (and (clothes-in basket)
                       (empty washer)
                       (not (clothes-clean)))
    :effect (and (not (clothes-in basket))
                 (clothes-in washer)
                 (empty basket)
                 (not (empty washer)))
  )

  (:action
    add-soap-to-washer
    :precondition (and (clothes-in washer)
                       (not (active washer))
                       (not (soap-added)))
    :effect (soap-added)
  )

  (:action
    turn-on-washer
    :precondition (and (clothes-in washer)
                       (soap-added)
                       (not (active washer)))
    :effect (active washer)
  )

  (:action
    wait-for-washer-to-finish
    :precondition (active washer)
    :effect (and (not (active washer))
                 (clothes-clean)
                 (not (clothes-dry)))
  )

  (:action
    move-clothes-to-dryer
    :precondition (and (clothes-in washer)
                       (clothes-clean)
                       (not (clothes-dry))
                       (not (active washer))
                       (not (active dryer))
                       (empty dryer))
    :effect (and (not (clothes-in washer))
                 (clothes-in dryer)
                 (not (empty dryer))))

  (:action
    turn-on-dryer
    :precondition (not (active dryer))
    :effect (active dryer)
  )

  (:action
    wait-for-dryer-to-finish
    :precondition (active dryer)
    :effect (and (not (active dryer))
                 (clothes-dry))
  )

  (:action
    unload-dryer
    :precondition (and (clothes-dry)
                       (clothes-in dryer)
                       (empty basket))
    :effect (and (not (clothes-in dryer))
                 (not (empty basket))
                 (clothes-in basket))
  )
)
```

Now an example problem file, e.g., `prob1.pddl`:

```lisp
(define (problem p)
  (:domain clothes)
  (:objects basket washer dryer)
  (:init
    (empty dryer)
    (empty washer)
    (clothes-in basket))
  (:goal
    (and (clothes-clean)
         (clothes-dry)
         (clothes-in basket)))
)
```

Here is the resulting plan found by fast-downward:

```
(add-clothes-to-washer )
(add-soap-to-washer )
(turn-on-washer )
(wait-for-washer-to-finish )
(move-clothes-to-dryer )
(turn-on-dryer )
(wait-for-dryer-to-finish )
(unload-dryer )
; cost = 8 (unit cost)
```

## Adversarial search

> Given some starting tic-tac-toe board, be able draw and label a tic-tac-toe search tree, like the one below, with min/max labels on downward arrows, and utility values (1 or 0 or -1) on upward arrows.
>
> For example, starting with this board:
>
> ![Tic-tac-toe start](/images/midterm-guide-ttt-start.png)

Answer:

![Tic-tac-toe solution](/images/midterm-guide-ttt-solution.png)

## Genetic algorithms

> Be able to define a genetic representation, fitness function, crossover function, and mutation function for a scenario specified in English. Be specific: indicate exactly how the representation and functions work. You will lose points if it is unclear to me exactly how they work.
>
> For example, consider the following problem:
>
> > Suppose we have six surveillance cameras and need to install them in the ceiling of a grocery store in order to view and record as much floor area as possible. Suppose the store is rectangular but some aisles are taller than others, blocking a camera's view from certain angles. Assume the cameras cannot pan, tilt, or zoom once they are positioned. We want to find an optimal or near-optimal placement of cameras so that the most area is covered.

One possible approach:

- genetic representation: represent the locations/angles of the six cameras as a list of six n-tuples, where each n-tuple is (x,y,angle) for that camera. E.g., a specific genetic string might be [(50,20,45), (10,20,180), (90,40,90), (15,45,0), (20,0,270), (50,50,45)].
- fitness function: the fitness function would calculate how much floor area can be seen by the six cameras for the specific genetic string.
- crossover function: given two strings, randomly decide for each of the six cameras if the position/angle for that camera will be copied from parent A or parent B.
- mutation function: Randomly generate a new position or angle for just one camera among the six.

## Rules engine

> Given a set of rules described in English, be able to write Drools rules that represent the same logic. The Drools file must have the correct syntax, but you do not need to provide the rest of the Java program to run the rules.
>
> For example, consider the following logic about an issue tracking (ticketing) system:
>
> > A ticket is either open or closed, every ticket has an integer priority, as well as a "newCustomer" boolean field. Whenever a customer creates a ticket, the ticket gets added to the database. If the customer has never issued a ticket before, the ticket should have its "newCustomer" field set to "true". Each ticket starts at priority 0. If the description of the ticket matches an existing open ticket (assume a matching function exists), then its priority is changed to the priority of the existing ticket plus 1; the priority of the existing tickets are left unchanged. If the customer has submitted more than 2 tickets with the same description, the priority of the new ticket is increased by 1. If the resulting priority of the ticket is greater than 3, an email is sent to the customer stating that "we are working hard on this problem." If the description of the ticket matches a closed ticket, this ticket is closed as well (and priority is set to -1).

One possible solution (partial):

```
rule "Ticket from new customer"
  when
    $c : Customer(ticketCount==1)
    $t : Ticket(customer==$c, newCustomer==false)
  then
    $t.setNewCustomer(true);
end

rule "Increase priority matching"
  when
    $t : Ticket()
    $t2 : Ticket( matches($t) )   // i.e., call $t2.matches($t), which should return a boolean
  then
    $t2.setPriority($t2.getPriority() + 1);
end

rule "Increase priority customer duplicates"
  when
    $c : Customer()
    $t : Ticket()
    // example of 'collect': 
    // https://docs.jboss.org/drools/release/5.2.0.Final/drools-expert-docs/html/ch05.html#d0e5306
    $cts : ArrayList ( size >= 2 )
           from collect ( Ticket(customer==$c, matches($t)) )
  then
    $t.setPriority($t.getPriority() + 1);
end

rule "Send email with ticket priority >= 3"
  when
    $c : Customer()
    $t : Ticket(customer==$c, priority >= 3)
  then
    sendEmail($c, $t, "we are working hard on this problem");
end

rule "Close ticket if matches already-closed ticket"
  when
    $t : Ticket()
    $tc : Ticket(matches($t), closed==true)
  then
    $t.setClosed(true);
end
```

## Prolog

> Know how to write Prolog code that represents a given set of facts and rules, stated in English.
>
> Here are some examples:
>
> - Familial relations: mother, father, aunt, uncle, niece, nephew, cousin, half-brother, half-sister, etc.
> - Or, from Monty Python: "A witch is a female who burns. Witches burn - because they're made of wood. Wood floats. What else floats on water? A duck; if something has the same weight as a duck it must float." (quoted from [this Prolog tutorial](http://www.allisons.org/ll/Logic/Prolog/Examples/witch/)) The resulting Prolog code should be able to prove that witches float.

Familial relations: See [this example source code](https://github.com/andschwa/uidaho-cs470-prolog/blob/master/family.pl).

Monty Python: See [this example source code](http://www.allisons.org/ll/Logic/Prolog/Examples/witch/).