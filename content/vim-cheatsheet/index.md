+++
title = "My VIM notes"
date = 2020-05-27
[taxonomies]
tags = ["vim"]
+++

## Fact number 1: VIM editing is based on a language

VIM is a language, and like all languages the smaller commands can be composed
into bigger more powerfull commands. Try to think of the commands you learn to
type (in normal mode) as functions. Also try to think of motions and text
objects as functions. The goal is realise that commands, motions and text
objects are easier to memorize when they take arguments and can be composed.
This means that fewer shortcuts have to be memorized in the long run (albeit at
first VIM seems weird and irritating).

We want to execute something a `<number>` of times, where something is a
`<command>` that operates on some text scoped by `<modifier><movement>`. This
last thing that denotes the target of text we operate on is called movements and
text objects in VIM.

Formally we have

```
<number><verb><modifier><noun>
<number><verb><modifier><subject>
```

In VIM this is also expressed as

```
<number><command><modifier><movement>
```

where

* `number` is repetition
* `command` is an action (delete, change etc)
* `modifier` will be explained below in text objects (it changes the scope of
  the edit in a syntactic/structural way, because VIM understands different
  objects in your text)
* `movement` is another concept that determines the scope/range (what text) of
  the edit (end of line, begining of line etc)

Text objects and motions are the 2 central concepts to understand when using the
power of VIM.

## Fact number 2: VIM has modes

Vim has multiple operating modes.

* Normal: for moving around a file and making edits
* Insert: for inserting text
* Replace: for replacing text
* Visual (plain, line, or block) mode: for selecting blocks of text
* Command-line: for running a command

Keystrokes have different meanings in different operating modes.

## VIM survival guide

This will make it possible to use VIM as notepad

* `:q` quit (close window)
* `:w` save (“write”)
* `:wq` save and quit
* `:e {name of file}` open file for editing
* `:ls` show open buffers
* `:help {topic}` open help
    * `:help :w` opens help for the :w command
    * `:help w` opens help for the w movement

## Overview

You should spend most of your time in normal mode, using movement commands to
navigate the buffer. Movements in Vim are also called “nouns”, because they
refer to chunks of text.

* Basic movement: `hjkl` (left, down, up, right)
* Words: `w` (next word), `b` (beginning of word), `e` (end of word)
* Lines: `0` (beginning of line), `^` (first non-blank character), `$`
(end of line)
* Basic movement to go into insert mode:
    * `a`: Append text following current cursor position
    * `A`: Append text to the end of current line
    * `i`: Insert text before the current cursor position
    * `I`: Insert text at the beginning of the cursor line
    * `o`: Open up a new line following the current line and add text there
    * `O`: Open up a new line in front of the current line and add text there
* Screen:
    * `H` (top of screen), `M` (middle of screen), `L` (bottom of screen)
    * `C-e` (line down), `C-?` (line up)
    * ???
* Cursor scrolling:
    * `C-u` (up), `C-d` (down)
    * `C-U` (Up half screen), `C-D` (down half screen)
    * `C-B` (Backward screenful), `C-F` (Forward screenful)
    * `C-b` (One page backward), `C-f` (One page forward)
    *
* File: `gg` (beginning of file), `G` (end of file)
* Line numbers: `:{number}<CR>` or `{number}gg` (line {number})
* Misc: `%` (corresponding item)
* Find: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    * find/to forward/backward {character} on the current line
        `,` / `;` for navigating matches
    * Search: `/{regex}`, `n` / `N` for navigating matches

### Examples of deleting text

The delete command is one of the simplest commands in vim. We can use it to
illustrate the many ways text can be scoped by motions and socalled text objects.


The way better examples that is the reason I am looking forward to love VIM
even when I am learning (hating) VIM.

* `x`: Delete character under cursor
* `X`: Delete character before cursor
* `dw`: Delete word from cursor on
* `diw`: Delete (inner) word.
* `daw`: Delete (outer, a) word
* `db`: Delete word backward
* `dd`: Delete line
* `d$`: Delete to end of line
* `d^`: Delete to beginning of line
* `di"`: Delete the (inner) content of a double quoted string
* `da"`: Delete a double quoted string (also the pair of double quotes)

## Surround: Wrap/Unwrap text object with pair of things

After installing the vim-surround plugin a new text object is available, it's
called with `s`.

A bit more formally the structure is:

```
<command><surround object>[count]<surround target>[replacement]
```

The `<command>` is one of the standard Vim ones such as delete (d), change (c)
or visual (v). The plugin also add a new command to add surrounding pair of things
around text objects and motions. The add command is called with `y`.

The `<surround object>` is called with `s` which tells Vim we're trying to operate
on something surrounding some text.

The next parameter is the `<surround target>` which is a text object, such as a
bracket or a quote mark.

The last parameter is the `<replacement>` which is required when you're changing
(with c) or adding (with y) something. It is the new thing (quote, bracket, tag)
that you want to wrap around the inner surround target.

### Delete surrounding pair of things

* `ds"`: delete pair of double-quotes (string literals)
* `dst`: delete surrounding tags (html, xml)
* `ds[`: delete surrounding brackets (list, array literals)

### Change surrounding pair of things

To change a surrounding we use `cs{pair}{another_pair}`, this deletes the supplied text-object
and replaces it with the second argument. The surrounding text object (called
pair) is what is replaced, not the inner contents of the text object.

There are two ways to use it:

* Change one surround to another: `cs<surround target><replacement>`
* Change one surround to another, putting the new ones on separate new lines: `cS<surround target><replacement>`

Some examples are

* `cs"'`: change from double quoted to single quoted string literal
* `cS'<p>`: wrap single quoted text in p tag.

### Add surrounding

Add a surrounding uses `ys`, the following item is the additional mark-up,
brackets or text that will placed around the inner content on either side.

There four ways to use the new add command (called with `y`):

* Wrap the vim motion or text object in the second argument: `ys<motion|text-object><addition>`
* Change the current line and wrap it with the second argument: `yss<addition>`
* Change the specified motion, putting the new addition on new lines and
indenting the text: `yS<motion><addition>`
* Whole line and do the above: `ySS<addition>`

Note that `ys` is like a 2-arg function, and `yss` is like a partial application
of this function, where the target motion/textobject have been defined to be the
hole current line (that is is `yss` is a 1-arg function).

Some examples are

* `ysw'`: add single quotes around a word
* `ys$"`: add double quotes around some text at the end of a line
* `ys3w[`: put some square brackets around 3 words
* `yssB`: put some (clang) brackets around the current line


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

## Basic Screen Navigation

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

* `d` (delete, cut)
* `y` (yank, copy)
* `c` (change)
* `>` (indent)
* `<` (unindent)
* `v` (visual)

### Aliases (short commands)

Current line motion commands

* dd = d_: d command with d motion (_ motion, because there is no d motion).
  Underscore (_) is an undocumented motion that refers to the current line.
* yy = y_
* cc = c_

To end-of-line motion commands

* D = d$
* Y = y$
* C = c$

* x = dl
* X = ?
*   = dh
*   = cl
*   =

### Power macro

* dot (.)

## Text Objects (modfiers): Movements on steroids!

TODO: link to blog post

* `i` (inner content)
* `a` (around/outer content)

### Builtin text objects

These (except w) can only be used as text object movements

* `w` (word)
* `s` (sentence)
* `p` (paragraph)

These can be used both with open/left and closing/right keys

* `]` (bracket)
* `},` `B` (brace)
* `),` `b` (paranthesis)

These are literals

* `"` (double quote)
* `'` (single quote)
* `` ` `` (backtick)

These are for markdown

* `>` (tag element)
* `t` (tag innertext)

* `B` (block)

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
