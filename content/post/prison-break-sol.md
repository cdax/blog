+++
date = "2015-03-08T20:53:15+05:30"
draft = false
title = "(Solution to) Prison Break: A Simple Counting Problem"
tags = ["puzzles"]

+++

>Imagine a prison having 36 cells arranged like the squares in a 6-by-6 grid. All adjacent cells have doors between them; doors that you can open. You, the prisoner in the top-left corner cell, are told that you can have your freedom if you can find your way to the diagonally opposite corner cell, after passing through each other cell exactly once.

![](/img/prison-break.png)

Can you figure such a way out of the prison?

#### The Solution

With counting problems, as with most real-life problems, it makes a lot of sense to take a step back and think hard about the feasibility of a solution before actually setting out to discover possible solutions or to count them. Is it even possible for you, the prisoner, to escape by getting to the diagonally opposite corner cell after crossing each other cell exactly once?

Having solved similar problems before, you decide to colour the map like a chessboard – for ease of making logical deductions. Since adjacent cells have doors between them, you can imagine yourself navigating the chessboard-prison exactly the way [a pawn](http://en.wikipedia.org/wiki/Pawn_%28chess%29) would.

![](/img/prison-break-sol.png)

It’s easy to see that a pawn-move from one cell to an adjacent cell will always be a move from a white square to a black square, or vice-versa. This means that you’ll require an even number of pawn-moves (at least two) to get from a square of one colour, to another square of the same colour.

How many moves will you be making if you must pass through each cell exactly once? The answer is **35**, since 35 other prison cells need to be entered exactly once, with one pawn-move per cell. *No cell may be entered more than once.*

I hope you can see by now that you’ve been had, that you’ve been at the receiving end of the prison warden’s cruel joke. *Because diagonally opposite corner cells will always be the same colour,* and you can never get from one to the other in an odd number (35) of pawn-moves.
