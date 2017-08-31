---
title: Biductive Prolog
layout: note
---

# Biductive Prolog

In order to isolate Prolog’s computational capabilities that extend beyond traditional programming techniques, we distinguish three types of computing: “deductive,” “abductive,” and “biductive.”

Deductive computing is computing of the usual sort, familiar to any computer science student. Deductive computing starts with fully-specified inputs or arguments and computes a fully-specified output or return value. The common function call in languages like C, Java, et al. takes the form of deductive computing.

Consider a familiar example of deductive programming: given a set of logic gates (and, or, not), wired connections between gates, and initial inputs, determine the output of each gate. Here is the full Prolog code with comments:

    knownGates(
        [(and, 2, [([1,1],1), ([0,1],0), ([1,0],0), ([0,0],0)]),
         (or,  2, [([1,1],1), ([0,1],1), ([1,0],1), ([0,0],0)]),
         (not, 1, [([1],0), ([0],1)])]).

    getWireState(Wires, WireName, State) :- member((WireName, State), Wires).

    evalCombLogic(_, []). % base case, no gates left to evaluate
    evalCombLogic(Wires, [(Gate, Inputs, Output)|RestGates]) :-
        knownGates(KnownGates),
        % extract gate info
        member((Gate, InputCount, Table), KnownGates),
        % ensure inputs list is right length
        length(Inputs, InputCount),
        % extract current state of the wires involved
        maplist(getWireState(Wires), Inputs, InputsWithState),
        % eval gate with those wire states
        member((InputsWithState, OutputState), Table),
        % ensure the wire's state is as computed
        getWireState(Wires, Output, OutputState),
        % look at rest of gates
        evalCombLogic(KnownGates, Wires, RestGates).

The code below is an example of a deductive query. The first argument, `[(a,1),(b,0),(c,CState),...]` gives the state of each wire, or unknown state in the case of `CState` and other capitalized variables. The second argument, `[(and, [a,b], c), ...]` gives the logic gates, in this case an “and” gate, and indicates their inputs (a and b) and single output (c). The query asks for the value of the outputs on wires c, d, and e simply by leaving those parts of the argument as variables.

    evalCombLogic([(a,1),(b,0),(c,CState),(d,DState),(e,EState)],
                  [(and, [a, b], c), (or, [b, c], d), (not, [d], e)]).

Result:

    CState = 0, DState = 0, EState = 1

We see that the wires c and d have state 0, while e has state 1.

Abductive computing works backwards from outputs to inputs. Abductive inference is a common pattern in artificial intelligence such as medical diagnosis, in which a system is presented with symptoms and must reason backwards to diseases. We may form an abductive query by specifying only the final output state on wire e and inferring the input states on wires a and b:

    evalCombLogic([(a,AState),(b,BState),(c,CState),(d,DState),(e,1)],
                  [(and, [a, b], c), (or, [b, c], d), (not, [d], e)]).

There are two solutions: `a=1 b=0`, and `a=b=0`. The same Prolog implementation for the deductive case supports the abductive case. Again, a Java program that forward-evaluates combinational logic gates would be incapable of determining initial inputs based on given outputs. Additional code must be written to explicitly handle the abductive case. Prolog code, on the other hand, naturally supports the abductive case.

Biductive computing combines deductive and abductive computing so that a single implementation may be used, with any or all arguments fully instantiated, partially instantiated, or entirely uninstantiated. For example, we may indicate some known wire states and some gates, but leave undecided the input to the "not" gate as well as the identity and inputs of the middle gate:

    evalCombLogic([(a,0),(b,BState),(c,CState),(d,DState),(e,1)], 
                  [(and, [a, b], c), (MiddleGate, MiddleInputs, d),(not, [NotInput], e)]).

One solution has wire `b=1`, an "and" gate in the middle with inputs a and b, and wire c as input to the "not" gate.

Biductive computing is not possible without an inference engine capable of searching for instantiations of variables according to specified criteria. In the case of Prolog, depth-first search in the form of SLD resolution and variable unification enable biductive computing. Imperative languages are unable to support biductive computing, so additional code must be written to support non-deductive use cases. We advise against writing this extra code as doing so would likely result in an unnecessary re-implementation of Prolog’s built-in inference engine.
