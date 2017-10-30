Spot the Dot - Implementation of the 'Spot the Dot' Puzzle Game in R
===============================================================

[![Build
Status](https://travis-ci.org/daattali/lightsout.svg?branch=master)](https://travis-ci.org/daattali/lightsout)
[![CRAN
version](http://www.r-pkg.org/badges/version/lightsout)](https://cran.r-project.org/package=lightsout)

[![saythanks](http://i.imgur.com/L88apDa.png)]

Lights Out is a puzzle game consisting of a grid of lights that are
either on or off. Pressing any light will toggle it and its adjacent
lights. The goal of the game is to switch all the lights off. This
package provides an interface to play the game on different board sizes,
both through the command line or with a visual application (Shiny app).
Puzzles can also be solved using the automatic solver included. Play the
game at https://iamnik.shinyapps.io/spotthedot/

Play the game
-------------

`lightsout` provides a Shiny app that lets you play with a graphical
user interface. To run the game locally, install the package with
`install.packages("lightsout")` and run the `lightsout::launch()`
command. This will launch the provided app in a web browser.

Alternatively, you can see the app online at
[<https://iamnik.shinyapps.io/spotthedot/>](https://iamnik.shinyapps.io/spotthedot/).

The game looks like this:

![Shiny app image](inst/img/shinyapp.png)

In this image, there is a 5x5 Lights Out board. The darker green lights
represent lights that are off, and the brighter green lights are lights
that are on. Clicking on the *Show solution* button will highlight all
the lights that need to be pressed in order to solve the current board.

Quick start
-----------

Other than the Shiny app that lets you play the game visually, you can
also interact with `lightsout` using the command line.

### Create a game board

You can use the `random_board()` function to initialize a new Lights Out
game with a random configuration of lights. The generated board is
guaranteed to be solvable. The first argument to the function is the
board dimensions (number of rows and columns), which is restricted to be
either 3, 5, 7, or 9.

    library(lightsout)
    random1 <- random_board(3)
    random1

    #> Lights Out 3x3 board
    #> Game mode: classic 
    #> 
    #>  1 0 0
    #>  1 1 0
    #>  1 1 0
    #>  

    random2 <- random_board(5)
    random2

    #> Lights Out 5x5 board
    #> Game mode: classic 
    #> 
    #>  0 0 0 1 1
    #>  1 1 1 0 0
    #>  1 0 0 1 0
    #>  1 1 0 1 0
    #>  1 1 1 0 0
    #>  

You can also use the `new_board()` function to create a new board if you
want to provide your own defined set of lights. The first argument is
either a matrix or a vector of the lights, with all values being either
0 (light off) or 1 (light on). If a vector is provided, then it is read
row-by-row rather than column-by-column.

    lights_vector <- c(0, 0, 0,
                       0, 1, 0,
                       1, 1, 1)
    board1 <- new_board(lights_vector)
    board1

    #> Lights Out 3x3 board
    #> Game mode: classic 
    #> 
    #>  0 0 0
    #>  0 1 0
    #>  1 1 1
    #>  

Any board generated by `random_board()` is guaranteed to be solvable,
but since we defined this board ourselves, it's a good idea to make sure
it's solvable before attempting to play it.

    is_solvable(board1)

    #> [1] TRUE

### Press a light

Pressing a light is done with the `play()` function. Pressing a light
will cause that light and all its adjacent lights to toggle (in classic
mode, which is the default). All the coordinates in `lightsout` are
treated as (row,column). Let's press the light at (2,3) - row 2, column
3

    # Press light at (2,3)
    played <- play(board1, 2, 3)
    played

    #> Lights Out 3x3 board
    #> Game mode: classic 
    #> 
    #>  0 0 1
    #>  0 0 1
    #>  1 1 0
    #>  

Notice how the light at (2,3) and its 4 neighbours are flipped.

If you want to play the non-classic game mode, where pressing a light
toggles the entire row and column, then use the `classic = FALSE`
argument of either `new_board()` or `random_board()`. To demonstrate,
here is the same board initialized as non-classic mode, with the same
light being pressed.

    # Press light at (2,3) in non-classic mode
    new_board(lights_vector, classic = FALSE) %>% play(2, 3)

    #> Lights Out 3x3 board
    #> Game mode: entire row/column 
    #> 
    #>  0 0 1
    #>  1 0 1
    #>  1 1 0
    #>  

Note that this specific board does not actually have a solution in
non-classic mode, which means that no combination of lights being
pressed will result in all lights being off:

    new_board(lights_vector, classic = FALSE) %>% is_solvable()

    #> [1] FALSE

Notice also that `lightsout` works well with the `%>%` operator that
allows you to easily chain calls.

### Press multiple lights

By chaining calls, you can press multiple lights easily one after the
other:

    # Press light at (2,3) and then (1,2)
    board1 %>% play(2, 3) %>% play(1, 2)

    #> Lights Out 3x3 board
    #> Game mode: classic 
    #> 
    #>  1 1 0
    #>  0 1 1
    #>  1 1 0
    #>  

By the way, when pressing multiple lights, the order does not matter.
Pressing `y` after `x` will always have the same result as pressing `x`
after `y`.

If you want to press multiple lights in one call, you can pass a vector
of row numbers and a vector of column numbers instead of a single row
and column. The previous chain of presses can also be achieved with:

    # Press light at (2,3) and (1,2)
    board1 %>% play(c(2, 1), c(3, 2))

    #> Lights Out 3x3 board
    #> Game mode: classic 
    #> 
    #>  1 1 0
    #>  0 1 1
    #>  1 1 0
    #>  

Another way to press many lights with one call is to use a matrix
instead of rows and columns. The matrix must have the same dimensions as
the board, and every position in the matrix with a value of `1` means
that the light in that position will be pressed.

    # Press light at (2,3) and (1,2)
    board1 %>% play(matrix = matrix(nrow = 3, byrow = TRUE,
                                    c(0, 1, 0,
                                      0, 0, 1,
                                      0, 0, 0)))

    #> Lights Out 3x3 board
    #> Game mode: classic 
    #> 
    #>  1 1 0
    #>  0 1 1
    #>  1 1 0
    #>  

Now let's try to actually solve the board. The board we have is actually
super easy - it should be obvious that a single press at (3,2) will turn
off all the lights.

    play(board1, 3, 2)

    #> Good job, you won!

If you have a more complex board, it will generally require many more
presses and will not be as trivial to solve. Let's create a random board
of size 5.

    bigboard <- random_board(size = 5)
    bigboard

    #> Lights Out 5x5 board
    #> Game mode: classic 
    #> 
    #>  1 1 0 0 0
    #>  1 0 1 1 0
    #>  1 1 0 1 1
    #>  1 1 1 0 1
    #>  1 1 0 1 1
    #>  

This one isn't so simple, is it? You can try playing with it for a bit.

### Get the solution

If you want to see the solution to a board, use the `solve_board()`
function:

    solution <- solve_board(bigboard)
    solution

    #> 
    #>  1 0 0 1 0
    #>  0 0 1 1 1
    #>  0 1 1 1 0
    #>  0 1 0 0 1
    #>  0 1 1 0 0
    #>  

The solution is a matrix with the same dimensions as the board, with the
values of `1` representing the lights that need to be pressed. We can
verify that the solution does indeed solve the current board:

    bigboard %>% play(matrix = solution)

    #> Good job, you won!

Solving boards algorithmically
------------------------------

There are a few algorithms for solving Lights Out puzzles. This package
(specifically the `solve_board()` function) implements the Gaussian
Elimination technique, which uses linear algebra to solve a matrix
equation `Ax=b` in order to derive the solution. The matrix operations
are done in modulus 2, with `b` being the current board configuration,
`x` being the solution vector, and `A` is a special matrix that depends
on the board size and game mode. The reason that only boards with an odd
number of rows/columns are supported is simply because I couldn't figure
out how to derive the special matrix `A` for even-sized boards.

The Gaussian Elimination strategy does not guarantee the minimum number
of steps, and therefore some steps in the suggested solution may be
redundant. If you are interested in more details about how this puzzle
is solved mathematically, you can look at the source code or look at
resources online for the exact details of the algorithm and the
mathematical theory behind it.

I hope you enjoy this little game fully implemented in R, or get
inspired to create Shiny games!
