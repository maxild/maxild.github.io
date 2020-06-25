+++
title = "My VIM notes"
date = 2020-05-27
[taxonomies]
tags = ["vim"]
categories = ["programming"]
+++

VIM is a language, and like all languages the smaller commands can be composed
into bigger more powerfull commands. The hope is that this expresses better
how one thinks, and in the long run more commands (obscure key sequences)
can be memorized.

```
<number><verb><modifier><noun>
<number><verb><modifier><subject>
```

```
<number><command><modifier><movement>
```

## The second most important fact: VIM has modes

Vim has multiple operating modes.

* Normal: for moving around a file and making edits
* Insert: for inserting text
* Replace: for replacing text
* Visual (plain, line, or block) mode: for selecting blocks of text
* Command-line: for running a command

Keystrokes have different meanings in different operating modes.

## VIM survival guide

This will make it possible to use VIM as notepad

### Basics of command-line mode

* `:q` quit (close window)
* `:w` save (“write”)
* `:wq` save and quit
* `:e {name of file}` open file for editing
* `:ls` show open buffers
* `:help {topic}` open help
    * `:help :w` opens help for the :w command
    * `:help w` opens help for the w movement

### Movement

You should spend most of your time in normal mode, using movement commands to
navigate the buffer. Movements in Vim are also called “nouns”, because they
refer to chunks of text.

* Basic movement: `hjkl` (left, down, up, right)
* Words: `w` (next word), `b` (beginning of word), `e` (end of word)
* Lines: `0` (beginning of line), `^` (first non-blank character), `$`
(end of line)
* Screen: `H` (top of screen), `M` (middle of screen), `L` (bottom of screen)
* Scroll: `C-u` (up), `C-d` (down)
* File: `gg` (beginning of file), `G` (end of file)
* Line numbers: `:{number}<CR>` or `{number}gg` (line {number})
* Misc: `%` (corresponding item)
* Find: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    * find/to forward/backward {character} on the current line
        `,` / `;` for navigating matches
    * Search: `/{regex}`, `n` / `N` for navigating matches

### Examples of deleting

You can extend the examples below with numbers

* dl
* dw
* dw, de, db, dW, dE, d
* d0, d^, d$, dd

The way better examples that is the reason I am looking forward to love VIM
even when I am learning (hating) VIM.

* diw, daw
* di", da"
* etc...


### Learn movements by using visual mode

That is place cursor where you want to anchor the selection, and press the
following keys

```
v<modifier><movement>....inspect the visual feedback.....<esc>
```

### Don't use visual mode for anything else than learning

Always try to do your editing without going in to visual mode. This will make
live easier in the long run. For example the powerfull dot command will be
useful if your editing is atomic (and therefore can be repeated).

## Basic Movements (Nouns, aka Subjects)

* h, j, k, l
* `gg`, `G`, `{n}gg` (navigate to a line)
* 0, ^, $ (navigate within a line)

# Basic keystrokes to go to insert mode

TODO: Move to top (VIM as Notepad, the basic survival mode, add save and quit)

* a
* A
* i
* I
* o
* O

* c/r...todo

## Language/Syntax Movements

* l (letter)
* w, e, b, W, E, B (word: whitespace vs punctuation, what is a word?)
* s (sentence)
* p (paragraph)
* { }
* ( )
* [ ]

## Basic Screen Mavigation

### Cursor Scrolling

* C-d, C-u

- Ctrl+U, Ctrl+D: scroll pages by half of screen
- Ctrl+B, Ctrl+F: scroll pages by a screen
### Screen Scolling (cursor stays, viewport moves)

* C-e, C-y

### ????

* C-b, C-f (by half a screen)
* C-u, C-d (by a screen)

## Screen navigation (cursor moves, viewport stays)

* H, M, L (home, middle, bottom)

## Commands (Verbs)

### Undo

* `u`: undo
* `C-r`: redo

### Single char

* r (???)
* `x` delete character (equal to `dl`)
* `s` substitute character (equal to `xi`)

### General

* d (delete, cut)
* y (yank, copy)
* c (change)
* > (indent)
* < (unindent)
* v (visual)

### Power macro

* dot (.)

## Text Objects (modfiers): Movements on steroids!

TODO: link to blog post

* i (inner content)
* a (around/outer content)

### Builtin text objects

These (except w) can only be used as text object movements

* w (word)
* s (sentence)
* p (paragrap)

These can be used bot with open/left and closing/right keys

* ], b (bracket)
* } (brace)
* ) (paranthesis)

These are literals

* " (double quote)
* ' (single quote)
* ` (backtick)

These are for markdown

* > (tag element)
* t (tag innertext)

* B (block)

Note: Many more general purpose text objects can be installed via plugins

* surround
* commentary
* replace-with-register
* indent
* entire
* line

## Hole Line command shortcuts

* dd
* yy
* cc

## To EOL command shortcuts

* D
* Y
* C

## Search Movements

* `f{char}`:
* `F{char}`:
* `t{char}`:
* `T{char}`:

* `/{char}`:
* `?{char}`:

* `*`
* `#`

### Examples

* ct{char}
* c/{search}<Enter>
* cf}

## Substitutions (sed like commands)

* `:s/{pattern}/{text}/`
* `:s/{pattern}/{text}/g`
* `:s/{pattern}/{text}/gc`
i

## tutorial

vimtutor

## Getting help

:h key-strokes
:h command

## Copy/Paste: registers

* +"y
* +"p

TODO: Explain registers

"<name>

register names:
* "
* 0,1,2,3,4,5,6,7,8,9
* a,b,...,z

:reg command to show all registers
: reg <name> to a register

## Resources

* `vimtutor` is a tutorial that comes installed with Vim
* [Vim Adventures](https://vim-adventures.com/) is a game to learn Vim
* [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)
* [Vim Advent Calendar](https://vimways.org/2019/) has various Vim tips
* [Vim Golf](http://www.vimgolf.com/) is [code golf](https://en.wikipedia.org/wiki/Code_golf), but where the programming language is Vim’s UI
* [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
* [Vim Screencasts](http://vimcasts.org/)
* [Practical Vim](https://pragprog.com/book/dnvim2/practical-vim-second-edition) (book)
