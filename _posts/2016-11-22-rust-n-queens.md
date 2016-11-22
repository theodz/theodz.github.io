---
layout: post
title:  "Solving the N queens problem in Rust"
categories: tutorial rust 
---

In this post we will proceed to solve the [classical 8 queens problem](https://en.wikipedia.org/wiki/Eight_queens_puzzle) using Rust, and even put in some overtime to go up to N queens on a board of any given size. This problem is a prime example in which [backtracking shines](https://en.wikipedia.org/wiki/Eight_queens_puzzle), and we will naturally use it.

Let's start by implementing a basic chess board and a function to display it. Having a way to view your data structures is a good first step in any development project.

```rust
struct Board {
    width: usize,
    height: usize,
    queens: Vec<(usize, usize)>
}

impl Board {
    fn display(&self) {
        let mut squares = vec![vec![false; self.height]; self.width];

        for queen in &self.queens {
            let &(qi, qj) = queen;
            squares[qi][qj] = true;
        }

        for row in squares {
            let mut line = String::from("");
            for square in row {
                match square {
                    true => line += "Q ",
                    false => line += ". "
                }
            }
            println!("{}", line);
        }
    }
}
```

The `display` method lets us visualize the board in a crude but efficient way. For instance a board with queens in a8 and b6 will be represented this way:

```
Q . . . . . . .
. . Q . . . . .
. . . . . . . .
. . . . . . . .
. . . . . . . .
. . . . . . . .
. . . . . . . .
. . . . . . . .
```

Not the pinnacle of visual art but very helpful to manually validate solutions. Now we are all set to go through with implementing the solution to our problem. Let's consider what we need. Conceptually, a backtracking algorithm relies on a recursive function that goes like this :

```
procedure backtrack(step):
	if incorrect(step) then return
	// No need to go deeper from the current state, we know we're off track anyway

	if correct(step) then output(state)
	// Yay, we found a solution!

	for next_step in followup(step):
		backtrack(next_step)
```

The core idea of backtracking is to take steps towards a possible solution until you find out the path you are following cannot be valid, and then move back to iterate over other paths until you either find a correct solution or conclude there is none. To go into more detail over the function used in the above pseudocode :

* The `backtrack` procedure evaluates each step of the way to determine what to do next : are we on a wrong path, did we find a solution, do we need to go deeper?
* The `incorrect` function straight up tells us whether there can be a good solution derived from our current state. If there is none we can just go back up and stop exploring this particular branch.
* The `correct` function is the one that can end the recursion successfully. It assesses whether the conditions of our problem are met at the current state, and if so lets us call the `output` function to communicate the solution we found.
* The `followup` function gives us all the possible steps following the one we are at. It lists all the directions we should explore next, so that we can call `backtrack` over them one at a time until we find a solution.

Enough explaining, let's do it! The `incorrect` and `correct` functions are rather simple as long as you know how to check diagonals, so let's start with them. (Tip: you can tell whether two chess pieces are on a same diagonal by checking if the differences in their horizontal and vertical positions are equal, which is what this code does.)

```rust
fn incorrect(board : &Board) -> bool {
    for queen in &board.queens {
        let &(qi, qj) = queen;
        for other_queen in &board.queens {
            let &(oi, oj) = other_queen;
            let h_diff = max(oi, qi) - min(oi, qi);
            let v_diff = max(oj, qj) - min(oj, qj);

            if oi == qi && oj == qj {
                continue;
            } else if oi == qi || oj == qj || h_diff == v_diff {
                return true;
            }
        }
    }

    return false;
}

fn correct(board : &Board, queen_number: usize) -> bool {
    if board.queens.len() == queen_number {
        return true;
    } else {
        return false;
    }
}
```

Now about the `followup` function, we have a slight problem; in its simpler form it would simply return a vector containing all boards with the previous queens plus a new one. But this approach would require us to store in memory a full board for each square at every backtracking step, which would be costly even at small board sizes. Maybe it would not make a noticeable difference in runtime for 8 queens, but it would still be better practice to go for a solution that scales better.

What we need is what is called a [generator](https://wiki.python.org/moin/Generators) in Python, that is to say an object that behaves like an iterator by yielding elements in sequence but without computing and storing them all first. There is no syntax sugar for this in Rust but nothing stops us from implementing it: all we need is a structure to hold the current state of our generator and a `next` method updating it and returning elements one at a time. 

```rust
struct BoardGenerator {
    initial_board: Board,
    i: usize,
    j: usize
}

impl BoardGenerator {
    fn next(&mut self) -> Option<Board> {
        self.j += 1;

        if self.j >= self.initial_board.height {
            self.i += 1;
            self.j = 0;
        }

        if self.i >= self.initial_board.width {
            return None;
        }

        match self.initial_board.queens.binary_search(&(self.i, self.j)) {
            Ok(_) => return self.next(),
            Err(idx) => {
                let mut next_board = Board::new(&self.initial_board);
                next_board.queens.insert(idx, (self.i, self.j));
                return Some(next_board);
            }
        }
    }
}
```

With that we are all set to complete our solver by putting the pieces together in the `backtrack` function, along a `main` function to call it:

```rust
fn backtrack(board : Board, queen_requirement: usize) -> Option<Board> {
    if incorrect(&board) { return None; }
    if correct(&board, queen_requirement) { return Some(board) }

    let mut generator = BoardGenerator::from(board);
    while let Some(next_board) = generator.next() {
        if let Some(solution) = backtrack(next_board, queen_requirement) {
            return Some(solution);
        }
    }

    return None;
}

fn main() {
    let board = Board { width: 8, height: 8, queens: vec![] };
    if let Some(solution) = backtrack(board, 8) {
        solution.display();
    } else {
        println!("No solution found.")
    }
}
```

And voila! A simple solver for an interesting problem. This configuration solves the 8 queen problem, but can also tackle larger or smaller ones simply by altering the initial board. The code of this program is available in a [GitHub repository](https://github.com/theodz/rust-n-queens) should you want to run it.

Bonus: an actual solution of the 8 queens problem!

```
. Q . . . . . .
. . . Q . . . .
. . . . . Q . .
. . . . . . . Q
. . Q . . . . .
Q . . . . . . .
. . . . . . Q .
. . . . Q . . .
```
