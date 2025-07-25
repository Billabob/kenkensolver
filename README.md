# KenKen Solver
A simple webpage for learning scripts, principally for Geoguessr

[https://billabob.github.io/KS/kenkensolver.html](https://billabob.github.io/KS/kenkensolver.html)

## How to use:

On the left side is the grid display. This is not interactive yet but it displays the current state of candidates on the board.

On the right side is all the solver buttons & logs. I'll list them off from top to bottom along with their purpose.

### Solver mode

Clicking "Batch mode" takes you to the batch solver, documentation below.

"Enter puzzle import string:"<br>
This is currently the only way you can import a puzzle into the solver. You paste in a string in Simon Tatham's format and click "Import". These strings may be obtained from [Tatham's implementation of KenKen](https://www.chiark.greenend.org.uk/~sgtatham/puzzles/js/keen.html) - generate a puzzle and click "Game > Enter Game ID..." to export it. Of course this is not ideal, you have to convert puzzles manually if they come from any other source. It'll have to do for now until I implement a UI that lets you input puzzles. Tatham's format is documented in his source code if you're interested in manually converting puzzles.

"Open in Sudokuwiki solver" is self-explanatory. Note this solver is limited to 6x6 puzzles and subtraction/division cages may not span more than 2 cells.

"Show candidates" may only be used immediately after a puzzle is imported. It performs the initial calculations for cage combinations and fills in Naked Singles, also removing candidates if Singles are placed and then continuing on by filling in any resulting Naked Singles and removing peer candidates after that etc... If a puzzle is easy enough it'll be solved by this step alone.

"Step" will find and execute the easiest technique currently available in the puzzle, according to the hierarchical solver.

"x100" will do this 100 times.

"Solve" will keep stepping until the puzzle is finally solved.

Solve log:

"Detailed logs" is an option that will log all the candidates & combinations removed by the constraint propagation steps. It's way too much information and the information isn't particularly valuable so there's no reason to ever turn this on.

"Log naked singles" is an option that does what it says on the tin. It's on by default, turn it off if the logs are too long for your liking.

### Batch mode

Clicking "Back to solver" brings you back to the solver.

The first box with default text "Paste JSON here" is an input field where you can paste a JSON object containing as many puzzles as you want. An example:<br>
```
[
    "6:a_a_aa_a_a_9a3ba__aa_a__aa__aa_a_,s2a9m6s3m18a5s2s3a6d3a7s2m12m20m24d2",
    "6:_4a_3aab__a_5aacaa_aa_aa__a_3a__a_,m10a7d3d3m12s2s1m12d2s1a6d2s4m90a13a5",
    "6:aa_aa_3a4babb_a_3b_3a_5a_3a_a_,a5a7s1s1m6m6d3m360a8s2a9s1d2m60m12",
    "6:a__a__a_12b_a9_aa_3a__ab,a11s3a13a6d2m12a9m6d3s2d2m12s1m60s1d2",
    "6:aa_aa__aa_5a_3abaa_aa_4a__ab_a4,a17d3s4s3a6a6m30m48s1s2d3d3m300m6d2"
]
```

Or you can upload a .txt file with the same format.

Mode:<br>
This is a single character for a variety of sets of solver options.<br>
n = Normal, respects whatever your current solver settings are<br>
c = Cage combinations, disables every technique except for constraint propagation and the cage combination bruteforce solver.<br>
b = Both, will output the results for n and c next to each other.

"Save output to file" is self-explanatory, as is "Run" and the Batch output text box.

## Solver

This page implements a hierarchical solver & grading system, similar to the SE rating scale used to rate Sudoku puzzles. Everything is currently a work in progress and impermanent. Expect rankings & weights to change in the near future as I revamp things and implement more techniques.

Each step of the solver starts with the most basic techniques and climbs the ladder of difficulty until the puzzle finally yields and a candidate/cage combination is eliminated. After this a new step begins and the solver goes back to the basic techniques and climbs again. This repeats until the puzzle is solved. Puzzles are graded based on the hardest technique required to solve the puzzle, no matter how many times the technique is used.

The technique ratings are as follows:<br>
0:   All the candidate busywork from constraint propagation etc.<br>
0:   Naked single<br>
1:   Hidden single<br>
1.5: Cage-region intersection (pointing/claiming)<br>
2:   Hidden pair<br>
2:   X-Wing<br>
2.1: Hidden Triple<br>
2.1: Swordfish<br>
2.2: Hidden Quadruple<br>
2.2: Jellyfish<br>
N-Sized Hidden subset/Fish will give rating 2+(N-2)/10, capped at 2.9.<br>
3:   Region Parity<br>
3:   Region Sum<br>
3.1: Multi-region Parity<br>
3.1: Multi-region Sum<br>
3.2: Region sum (2 unknown cages in 1 region)<br>
3.3: Region sum (2 unknown cages in >1 regions)<br>
3.4: Region sum (3 unknown cages in 1 region)<br>
3.5: Region sum (3 unknown cages in >1 regions)<br>
A sum with N unknown cages will give rating 3+(N-1)/5, capped at 3.9.<br>
4:   Combination of 2 cages<br>
4.2: Combination of 3 cages<br>
4.4: Combination of 4 cages<br>
Combination of N cages will give rating 4+(N-2)/5 with no maximum limit.

You should be familiar with most of these techniques if you're a competent KenKen solver. The only one that bears explaining is the cage combination solver.

Every cage has a certain set of combinations that could be valid solutions. For example in a 6x6 puzzle, a two-cell "6x" cage can only be 1x6 or 2x3, plus the permutations of those digits. A two-cell "9+" cage can only be 3+6 or 4+5, plus the permutations of those digits. Now what happens if these two cages are next to each other, like so:<br>
```
.___.___.___.___.
| 6x    | 9+    |
'---'---'---'---'
```
The cage combinations solver will go through every pair of two cages, including this pair. It will check which combinations of digits can satisfy both cages without contradicting each other. If the 6x cage contains 1x6 then the 9+ cage cannot contain a 6, so it must be 4+5. If the 6x cage contains 2x3 then the 9+ cage cannot contain a 3, so again it must be 4+5. This allows us to eliminate 3+6 and 6+3 as potential solutions for this cage.

The solver will check every pair of cages in this manner, then every triple, then every quadruple, then every N-tuple until it either finds an elimination or runs out of cages.

This stage is the highest-rated because it's effectively bruteforcing larger and larger subdivisions of the puzzle until everything is bruteforced at once - it can only fail if the puzzle has more than one solution as the application "ad absurdum" would be bruteforcing the entire puzzle. These bruteforce eliminations should be covered by other techniques when they're implemented in the future, especially AIC.

## TODO list:

Solver:<br>
- AIC!<br>
- Set Equivalence Theory<br>
- Region Products<br>
- Check for valid assignments in region total solver<br>
- Look into saving calculated sums/parities & applying them as strong links in the AIC solver

Site:<br>
- Interactive grid input<br>
- Playable puzzles<br>
- Combination view per cage<br>
- r1c1 = a1 toggle<br>
- Visually-friendly hint button that displays move without executing it

Other stuff:<br>
- Solve every 4x4<br>
- Puzzle generator & fast bruteforce solver

## Changelog

### v0 - 2025-07-24

First release with Naked/Hidden Single, Cage-Line Reduction, Hidden Subsets, Fish, Region Totals, Region Parity, Cage Combinations.