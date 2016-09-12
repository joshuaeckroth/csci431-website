---
title: Adversarial search
layout: note
---

# Adversarial search

The search techniques we have learned so far (A\*, best-first, planning, etc.) are not suitable for *games* in which there are at least two agents who act as adversaries. In adversarial situations, we must find a move that maximizes our chances of winning the game, assuming our adversary is playing optimally.

{% comment %}
> "It was a watershed event, but it doesn't have to do with computers
becoming intelligent," said Douglas Hofstadter, a professor of
computer science at Indiana University and author of several books
about human intelligence, including *Gödel, Escher, Bach*, which won a
Pulitzer Prize in 1980, with its witty argument about the connecting
threads of intellect in various fields of expression. "They're just
overtaking humans in certain intellectual activities that we thought
required intelligence. My God, I used to think chess required thought.
Now, I realize it doesn't. It doesn't mean Kasparov isn't a deep
thinker, just that you can bypass deep thinking in playing chess, the
way you can fly without flapping your wings." --- Bruce Weber,
"[Mean Chess-Playing Computer Tears at Meaning of Thought](http://www.rci.rutgers.edu/~cfs/472_html/Intro/NYT_Intro/ChessMatch/MeanChessPlaying.html),"
*The New York Times*, Feb 19, 1996
{% endcomment %}

## Zero-sum games

Chess, tic-tac-toe, connect-four, checkers, go, etc. etc. are
*zero-sum games*, meaning in the end, the gains and losses among the
players add up to zero. In the examples listed, one player wins,
another loses, so the sum of "utility" for each player at the end is
zero.

Poker is another zero-sum game, although there is one winner and many
losers. However, the amount the winner wins is equal to the sum of the
amounts the losers lose.

Study of zero-sum games began the study of game theory, which is a
mathematical subject that covers any situation involving several
agents attempting to make decisions that maximize their own utility.

We will not be studying game theory, as a whole, but we will learn how
to "play" zero-sum games. In fact, we will only focus on deterministic games with perfect information (both players see the full game state), as shown in the following table:

| | Deterministic | Stochastic |
|---|
| <b>Perfect information</b> | Chess, Checkers, Connect Four,<br/>Go, Othello, Tic-tac-toe | Backgammon, Monopoly |
| <b>Imperfect information</b> | Battleship | Bridge, Poker, Scrabble |

Table from [Leif Kusoffsky](https://www.nada.kth.se/kurser/kth/2D1350/progp02/lecture2.pdf).

## Expansion of the search problem

The components of a search problem included the following:

- **Initial state**
- **Possible transitions**

Note that we have left out "Goal criteria" and "Action cost." In order
to support adversarial search, we add the following:

- **Players** -- a list of players (we'll look at 2-player games) and
             a `switch_player(player)` function that switches to the
             player following `player`

- **Terminal tests** -- a function or functions that test for final states
                    (winning/losing/tied states)

- **Utility function** -- a function `utility(state)` that returns
     a real number (or integer) representing the value of
     `state` to the computer player; presumably, winning states have the highest utility,
     losing states the lowest utility, tied states have zero utility,
     and states that are not terminal have a utility calculated by
     looking at the utilities of deeper states

## I move, you move, etc.

The simplest zero-sum games are those that have two players who take
turns. Chess, tic-tac-toe, etc. fit this definition. Suppose a human
is playing a computer. The computer needs to figure out which move is
best (the computer wants to maximize the utility of its move). We
learned how to search (BFS, DFS, A\*, and so on), but even though the
computer will be engaging in a search, those search algorithms will
not help us here.

BFS, DFS, A\* and so on attempt to maximize (or minimize) every choice
equally (for example, we want to find the shortest route). But in a
game search, instead we want to maximize the payoff of our choices but
minimize the payoff of the other player's choices.

The way to do this is to treat the opponent's positive utility as our
negative utility. So if the opponent makes a winning move, that move
is worth 1.0 to the opponent but worth -1.0 to us (a tie game has
utility 0.0 for both players). We want to find the move that maximizes
our utility (best case, 1.0) and minimizes the opponent's utility
(best case, -1.0).

## Minimax algorithm

The minimax algorithm does exactly that: it finds the move that
minimizes the maximum utility the opponent can obtain. You can call it
"maximin" if you'd like: maximize the minimum utility we (not the
opponent) can obtain. They are the same algorithm, just the order of
the steps (maximize or minimize) is different.

First, we consider all of our possible moves. If a move is a final
move (winning/losing/tie), then we calculate its utility directly (1.0
for a win, -1.0 for a loss, 0.0 for a tie). If it's not a final move,
we have to look further to calculate its utility. We choose one move
from which to "look further," and consider all the possible moves the
opponent might make. If the opponent wins, the utility of the
opponent's move is -1.0. Of all the possible moves the opponent might
make, we find the worst (minimum) utility. That utility is the value
of our move.

Of all our possible moves, we choose the one that has the maximum
utility; it is the maximum of the minimum utilities generated by the
opponent's moves. We are assuming the opponent is also choosing the
best move, so when we choose our move, we want to find the move that
performs best (max) assuming the opponent also performs best (min;
i.e., max for the opponent, min for us).

~~~ python
def minimax(state, player, minimizing = False, depth = 0):

  # keep track of best move
  best_move = None
  best_u = None

  moves = possible_transitions(state, player):
  for move, nextstate in moves.iteritems():
    # if the state is a win/loss/tie, stop searching (base case of recursion)
    if is_terminal(nextstate):
      u = utility(nextstate)
    else:
      # after making move, find the best move the other player can make
      player = switch_player(player)
      minimizing = not minimizing # flip state
      u = minimax(nextstate, player, minimizing, depth + 1)

    # if we're minimizing, find the minimum utility; if maximizing, find the maximum utility
    if best_u is None or (minimizing and u < best_u) or (not minimizing and u > best_u):
      best_move = move
      best_u = u

  # recursive calls expect a utility to be returned; original call expects a move to be returned
  if depth == 0:
    return best_move
  else:
    return best_u
~~~

Here is an example search tree for tic-tac-toe. We basically have a
depth-first search but also propogate information about utilities (max
or min) upwards.


<div style="text-align: center">
<OBJECT CLASSID="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"
WIDTH="474" HEIGHT="550"
CODEBASE="http://active.macromedia.com/flash5/cabs/swflash.cab#version=5,0,0,0">
<PARAM NAME="MOVIE" VALUE="./flash/ttt-minimax-example.swf">
<PARAM NAME="PLAY" VALUE="true">
<PARAM NAME="QUALITY" VALUE="best">
<PARAM NAME="LOOP" VALUE="true">
<EMBED SRC="/flash/ttt-minimax-example.swf" WIDTH="474" HEIGHT="550" PLAY="true" LOOP="true" QUALITY="best"
PLUGINSPAGE="http://www.macromedia.com/shockwave/download/index.cgi?P1_Prod_Version=ShockwaveFlash">
</EMBED>
</OBJECT>
<br/>
<a href="/images/ttt-minimax-example.png">View final image</a>
</div>

## Alpha-beta pruning

The minimax search is very costly. Far too many states are
searched. We can use a very simple, and somewhat obvious, optimization
to search only a fraction as many states while coming up with the
exact same answer as minimax.

The key insight is that when we are performing the "max" step (or
equivalently, the "min" step), if we find that a certain move gives a
maximum utility X (by searching its entire subtree), we can avoid any
searches of subtrees whose minimum value is less than X.

~~~ python
##### NOTICE updated function parameters:
def minimax(state, player, minimizing = False, depth = 0, alpha = None, beta = None):

  # keep track of best move
  best_move = None
  best_u = None

  moves = possible_transitions(state, player):
  for move, nextstate in moves.iteritems():
    # if the state is a win/loss/tie, stop searching (base case of recursion)
    if is_terminal(nextstate):
      u = utility(nextstate)
    else:
      # after making move, find the best move the other player can make
      player = switch_player(player)
      minimizing = not minimizing # flip state

      ##### PASS current values of alpha, beta to recursive call
      u = minimax(nextstate, player, minimizing, depth + 1, alpha, beta)

    # if we're minimizing, find the minimum utility; if maximizing, find the maximum utility
    if best_u is None or (minimizing and u < best_u) or (not minimizing and u > best_u):
      best_move = move
      best_u = u

  ##### NEW code for alpha-beta pruning

  if minimizing:
    # if the utility we just found is smaller than alpha, and
    # these utilities can only get smaller (because we're in a
    # min stage), then there is no reason to check further
    if alpha is not None and u <= alpha:
      break
    # if the utility we just found (from the max stage) is
    # smaller than beta, we found a new smallest max; this
    # will restrict future searches not to look further if
    # they are maximizing and they find a utility greater than
    # beta
    if beta is None or u < beta:
      beta = u

  else:
    # if the utility we just found is greater than beta, and these
    # utilities can only get bigger (because we're in a max
    # stage), then there is no reason to check further
    if beta is not None and u >= beta:
      break
    # if the utility we just found (from the min stage) is
    # greater than alpha, we found a new greatest min; this
    # will restrict future searches not to look further if
    # they are minimizing and they find a utility less than
    # alpha
    if alpha is None or u > alpha:
      alpha = u

  ##### END of new code

  # recursive calls expect a utility to be returned; original call expects a move to be returned
  if depth == 0:
    return best_move
  else:
    return best_u
~~~

Suppose `minimizing` is True. Then the variable `u` here will only get smaller as each move is checked. We can avoid searching all possible moves if `u` gets smaller than (or equal to) some threshold. We know that the utility returned from this function may be used by the minimax function that called us, and that parent function must be *maximizing*. Thus, our parent function will only use the largest utility from the various child functions it calls. The $\alpha$ parameter is given by the parent function to the child function to say, "one of my children has already returned a utility of $\alpha$, and I'm maximizing; so if you (child) find that you're going to return a utility smaller than $\alpha$, just stop searching right then and there." (If $\alpha$ is `None`, then we have no parent function, so none of this matters and alpha-beta pruning doesn't do anything.)

In summary, when minimizing, the $\alpha$ parameter is used to stop the search. The $\alpha$ parameter is passed by the parent function. It represents the largest value seen so far in the parent function's search. Note that we're sure the parent function is maximizing, since the current function is minimizing; so the parent only wants utilities larger than $\alpha$. When `u` gets minimized so much in the current search that it goes lower than $\alpha$, the search is stopped. Values lower than $\alpha$ will not be used by the parent function anyway.

The $\beta$ parameter is used in exactly the same way, except that it represents the lowest utility seen so far, and the maximizing search will stop early if it finds a utility greater than or equal to $\beta$. Thus, the minimizing search sets $\beta$ and passes it along to a child maximizing search in order to help the child stop its search early if the utilities it finds start creeping up on $\beta$ and will thus be ignored by the parent.

It's very important to realize that the alpha-beta pruning procedure
does not produce different answers than minimax. Alpha-beta pruning
simply does not bother checking subtrees when it is certain those
subtrees will not change the answer produced by minimax.

Check out the [final search graph](/images/ttt-alphabeta-example.png) of another tic-tac-toe example, this
time with alpha-beta pruning. For comparison, [this is the search graph
without alpha-beta pruning](/images/ttt-minimax-example-2.png).

Alpha-beta pruning performs *much* better than vanilla minimax:

| Number of x/o tokens on the board | Average minimax states checked | Average $\alpha,\beta$ states checked |
|-----------------------------------+--------------------------------+-------------------------------------|
|                                 0 |                       549946.0 |                             29019.0 |
|                                 2 |                         7293.0 |                              1308.0 |
|                                 4 |                          176.2 |                                81.8 |
|                                 6 |                           11.0 |                                 9.2 |
|                                 8 |                            1.3 |                                 1.3 |

## A poor example?

Tic-tac-toe is such a simple game that minimax and alpha-beta pruning
are not really necessary. Apparently, here is optimal play (from
[xkcd](http://xkcd.com/832/)):

![Optimal tic-tac-toe](/images/xkcd-tic_tac_toe.png)


This is also instructive (from [xkcd](http://xkcd.com/1002/)):

![xkcd](/images/xkcd-game_ais.png)

