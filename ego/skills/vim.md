**motions**
`operator [number] motion`

`d` - delete operator
`motion` - what the operator will operate on

`d [number] motion`

`dw` - until start of the next word
`de` - to end of the current word
`d$` - to end of the line.
`db` - to start of the previous word if on the start of cur word else         till start of cur word.
`d0` - to start of line

here `w e $ b 0` are the motions.

*typing a number before a motion repeats it that many times.*

`2w` - move 2 words forward
`3e` - end of the third word
`20` - start of second line
`$` - end of line

`dd` - delete a whole line
`2dd` - delete two lines

`u` - undo last cmd
`U` - return line to its original state


`u` - *undo command*
`p` - *put command*, put previously deleted text after the cursor
`r` - *replace command*

`c` - *change operator*
`ce` - change until end of a word.

`c [number] motion`

`<C-g>` - to show your location in a file and the file status.

`G` - bottom of file
`g` - top of file
`493G` - go to line 493

`/text` - search for text, `n` next `N` prev
`?text` - search in backward direction

`<C-o>` - go back to where you came from
`C-i>` - goes forward.

`%` - to find a matching `) ] }`

`:s/old/new/g` - to substitute "new" for "old", `g` to substitute globally in the line.

`:#,#s/old/new/g` - change occurences between two lines

`:%s/old/new/g` - change occurence in entire file

`:%s/old/new/gc` - ask for substitution

`:s/\cold/new/gc` - ignore case

`:s/\Cold/new/gc` - match exact case

`:!` - execute external command

`:w filename` - save a file

use `v` to save part of a file
`v` - *visual mode*

`:r filename` - retrieve contents of a file and place it below cursor
`:r !ls` - retrieve output of cmd and put it below cursor

`o` - *open command*, open a line below the cursor
`O` - open line above the cursor

`a` - *append command*, append text after cursor

`R` - replace more than one character

`y` - *yank operator*, copy text

`H M L` - first, middle, last line of screen

`ZZ` - save and quit  `ZQ` - force quit

`f<char>` - jump and land on char
`t<char>` - jump and land before char

`<` `>` - indent selection by one block

`ddp` - swap position of consecutive lines, dd then p


**cmdline shortcuts**

`Ctrl + A` - move to start of line
`Ctrl + E` - move to end of line
`Alt + B` - move back one word
`Alt + F` - move forward one word

`Ctrl + U` - del from cursor to start of line
`Ctrl + K` - del from cursor to end of line
`Ctrl + W` - delete word before cursor
`Alt + D` - delete word after cursor
`Ctrl + D` - delete character under cursor

`Ctrl + Y` - paste last deleted text

`Ctrl + P` - previous cmd
`Ctrl + N` - next cmd
`Ctrl + R` - reverse search
`Ctrl + G` - cancel history search

----
`B` `W` - ignore punctuations
`20G`
`fa` `;`

`PS1` - prompt string 1

