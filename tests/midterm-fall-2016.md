---
layout: note
title: Midterm for Fall 2016
categories: [tests]
---

# Midterm for Fall 2016

Submit your solutions on bitbucket in a repository named csci431-midterm or similar. Create a folder for each question. **Answers are due by Fri Oct 28, 11:59pm.** You may only look at these course notes and standard documentation for Drools, Prolog, etc.

## Question 1 (2pts)

Suppose somebody told you that the Turing Test had been passed. Further suppose that you are asked to join a community of scientists, philosophers, etc. to validate that the test had indeed been passed.

In 3-5 sentences, answer the following question: What proof would you need to be sure that a computer program has passed the Turing Test?

## Question 2 (5pts)

Given the graphical representation of possible transitions and action costs below, list the order that states are visited in **breadth-first** and **A\*** search. (Assume that depth-first follows children in alphabetical order.) In the diagram, the heuristic value for each state is also shown in the state as "h = value". The starting state is shown as a box (node "A"), and the goal state is shown as a double circle (node "G").

![Transitions](/images/midterm-search-fall-2016.png)

## Question 3 (5pts)

Extend the PDDL domain and problem files from the [midterm review solutions](/guide/midterm-review-solutions.html) to support two washers and two dryers and four baskets of clothes. Set the initial state so that all baskets are full of dirty clothes, and all washers and dryers are inactive and empty. Keep the same constraints as the original problem: clothes must be clean before adding them to the dryer; soap must be added to the washer before it can be turned on; etc. Your resulting plans should make sense (logically and physically).

## Question 4 (3pts)

Label the following minimax search tree for tic-tac-toe. Add an upward arrow from the best state at each choice point ("best" according to min or max, depending) and its parent state. Also write the value (utility), in terms of the computer player X, next to each upward arrow and inside each leaf node. Use utility values 1, 0, or -1. Note that each choice point should have just one upward arrow and utility; and the top state should have an upward arrow coming from the best move for X, among the three possible. Answer this question by adding labels to the image, or print the image and then take a picture of the solution.

![Tic-tac-toe](/images/midterm-ttt-fall-2016.png)

## Question 5 (5pts)

Define a genetic representation, fitness function, crossover function, and mutation function for the following problem. Be specific: indicate exactly how the representation and functions work. You will lose points if it is unclear exactly how they work. Be sure to specify crossover and mutation functions that only produce valid solutions.

> Suppose we have 1000 network routers, each of which can connect to at most five other routers, and we want to find an optimal or near-optimal network configuration that minimizes the number of connections (some routers are far apart and the connection could be costly) while maximizing performance (measured by simulating a range of network traffic patterns) and requiring that there is a path from every router to every other router (perhaps requiring multiple hops).

Note, valid solutions are only those for which "there is a path from every router to every other router." You do not need to tell me how that constraint is checked, just assume some function exists that determines if the constraint is met. Refer to this function when appropriate in your genetic representation, fitness function, etc.

## Question 6 (5pts)

Write a Drools rules file that represents the logic described below. The Drools file must have the correct syntax, but you do not need to provide the rest of the Java program to run the rules.

> Suppose we want to build a news alerting service that sends subscribers a unique email based on their preferences. Every news article in the database has a subject (e.g., Sports, Politics, etc.) and an age (in days). Subscribers have a preference for one or more subjects and an alert frequency (daily or weekly). Write rules that encode the logic for collecting the news stories that should be in each subscriber's alert. Suppose we have a class named "EmailAlert" with a constructor "EmailAlert(Subscriber s)" and a method "addArticle(Article a)"; in other words, the EmailAlert class represents the email that will be sent to a subscriber, and contains a collection of articles that will make up the alert.

Note, this problem requires that you make some assumptions about which Java classes are available and how they are structured. Use your best judgment and be explicit when necessary.

## Question 7 (5pts)

Write Prolog facts/rules that find the score for a hand of Blackjack. In Blackjack, a player has between two and five cards. The cards 2 through 10 are scored as 2 through 10 points each. The face cards (jack, queen, and king) are scored as 10 points. The goal is to come as close to a score of 21 as possible without going over 21. The ace can count as either 1 or 11, whichever is better for the player. For example, an ace and a 10 can be scored as either 11 or 21. Since 21 is a better score, this hand is scored as 21. An ace and two 8s can be scored as either 17 or 27. Since 27 is a "busted" score, this hand is scored as 17.

The following queries should be true:

```
score([2, 4, 7], 13).
score([ace, 2], 13).
\+(score([ace, 2], 3)).  % ace, 2 is not scored as 3
score([ace, queen], 21).
score([ace, ace, jack], 12).
\+(score([ace, ace, jack], 32)).  % ace, ace, jack is not scored as 32
score([ace, 8, 8], 17).
\+(score([ace, 8, 8], 27)).  % ace, 8, 8 is not scored as 27
score([3, 10, 5, king], 28).
```

Do not use any semicolons `;` (which means "or") in your rules. Do not write any extra code that is not directly related to solving this problem.