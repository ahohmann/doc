# Vim

This document tracks my vim learning curve starting with the first step of
getting used to vim's "modal" editing all the way to switching most tools to
their respective vi modes and even writing a few little extension functions.

## Resources

The are *a lot* of resources out there teaching vim on all levels. Some of the
most useful I encountered were:

* vimtutor: The obvious place to start.
* Drew Neil's [Practical Vim][practical-vim]: Next step after vimtutor, shows
  how to use many of the built-in features.
* Drew Neil's http://vimcasts.org/
* Steve Losh's [Learn Vimscript the Hard Way][vimscript-the-hard-way]: Besides
  teaching vimscript, this book explains vim configuration.
* cheat sheets
  - www.viemu.com: keyboard layout, good start
  - http://michael.peopleofhonoronly.com/vim/: keyboard layout, more complete
  - http://tnerual.eriogerg.free.fr/vimqrc.html: good reference card

[practical-vim]: https://pragprog.com/book/dnvim/practical-vim "Practical Vim"
[vimscript-the-hard-way]: http://learnvimscriptthehardway.stevelosh.com/ "Learn Vimscript the Hard Wary"

## First Steps

I used vi many years ago on my first "real" job in a production environment,
but then quickly switched back to emacs when moving to development (C++, Python
at the time).  Surprisingly, my fingers still remembered the hjkl movements and
even simple key combinations such as `cw` or `dd`, but I realized quickly that
I had barely scratched the surface back then.  I don't remember the same steep
learning curve when starting to use emacs, but then again this was even before
my first vim exposure, and I'm not using the same level of powerful commands in
my normal emacs editing.

Learning vim feels like learning a music instrument.  One starts fairly slow,
but with every day of practice learns something new and finally reaches the
state where one does not have to think about the individual commands anymore
and can stay focused on the piece as a whole.

After a day or two, editing speed is back to normal in comparison to "normal"
editors, only interrupted by a few hickups caused by fast typing in command
mode resulting in a furry of unintended actions followed by as many hits of the
undo key.  The first thing I noticed was that I typed less and moved my hands
much less while accomplishing the same amount of editing.  After that, the
speedup set in once certain key patterns became automatic and I started using
more powerful commands.

What I found useful is to read the books off work (the daily commute on the bus
or train works well) and then try to apply a selected group of commands to a
concrete editing task during the day.

I had to change a few habits to make working with vim practical.  For plain
text, for example, I finally gave in and added a second space after each
sentence so that vim recognized them.

A few quirks one simply has to get used to.  For example, entering insert mode
with `i` and returning to command mode with `<esc>` moves the cursor one step
back unless you were already at the beginning of the line.  One can press
`<A-l>` (which is equivalent to `<esc>l`) instead, but that's hardly
convenient.  In the end, one just has to accept that the insert point is before
the cursor (between the characters, typically shown with a beam in the
graphical versions of vim) and that `<esc>` places the cursor on the character
before insert.

The other aspect to keep in mind is that vim is based on a line editor. Several
commands (such as `f`, `F`, `t`, `T`) operate only on the current line, and the
current line is the default range (`.`) of all ex commands apart from `global`.

### Initial Setup

To use vim efficiently it is essential to have Escape in a convenient location,
because reentering command mode has to become second nature.  I'm using a
[Kinesis](https://www.kinesis-ergo.com) keyboard most of the time, which is
incredibly valuable for keyboard shortcuts relying on the Ctrl and Alt key that
are so common in emacs and IDEs.  However, the Escape key is the most difficult
key to reach.  I therefore remapped CapsLock on my Kinesis keyboards to the
escape key.  On normal keyboards, CapsLock is a perfect location for a Ctrl
key, so that I left this mapping and use Ctrl-[ (pressing CapsLock) for Escape.

As often recommended, I added a few initial lines to my `.vimrc` to get started:

```
set nocompatible
let mapleader=" "
set incsearch ignorecase smartcase hlsearch
nnoremap <cr> :noh<cr>
```

I'm picking the space bar as the leader key, because its normal function is
redundant and it's very easy to use.  I did not choose the often used `,`,
because its original function is useful (especially in view of the symmetric
`;` and `,` commands) and I want to stay close to the original mapping just in
case I have to use vim without my settings.

The last line allows for a quick way to get rid of the highlighting after a
search.  Overriding the return key was the best option I could find.  The
`noremap` command means (as I learned from "Learn Vimscript the Hard Way")
non-recursive mapping in normal mode, where the "non-recursive" refers to the
fact that the keys used in the key binding will have their original meaning and
not the one defined in some other mapping.

Especially during the first few weeks it's good to be able to make quick
changes to the configuration file with little interruption of the normal
workflow. As recommended in "Learn Vimscript the Hard Way", I therefore added
two bindings for editing and reloading my `.vimrc` file:

```
:nnoremap <leader>ev :vsplit $MYVIMRC<cr>
:nnoremap <leader>sv :source $MYVIMRC<cr>
```

I also mapped `<c-s>` to the write command, because I'm used to it (at least
outside of emacs):

```
nnoremap <C-S> :w<CR>
inoremap <C-S> <Esc>:w<CR>
```

As I'm writing a lot of plain text (such as these notes), I also added a
mapping for formatting the current paragraph (emacs' `M-q`) once I had figured
it out:

```
nnoremap <c-q> vipgq
inoremap <c-q> <esc>vipgq
```

Initially, I found the following comprehensive status line format:

```
set statusline=%F[%{strlen(&fenc)?&fenc:'none'},%{&ff}]%h%m%r%y%=%c,%l/%L\ %P
```

It shows the full file name (`%F`), file encoding if available (`fenc`), file
format (`ff`), modification marker, and the position in terms of line, column,
and percentage.  Later I switched to
[airline](https://github.com/bling/vim-airline) which has a good format out of
the box.

### Windows

Before even starting to use vim for real I had to make sure that it supported
my normal workflows working with many files at the same time, either in
multiple frames on a large screen or switching quickly between them on a
smaller screen.  Fortunately, the multi-window split-screen support turned out
to be so good that it became one of the main reasons to install Vrapper in
eclipse (see below).  I added the following key bindings to make the shortcuts
for horizontal and vertical split more mnemonic.

```
nnoremap <C-w>- <c-w>s
nnoremap <C-w>\| <c-w>v
```

I'm using the same split keys in my tmux configuration following Brian Hogan's
"tmux: Productive Mouse-Free Deveopment".  Finally, the
[vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator) allows
for using `<c-h>`, `<c-j>`, `<c-k>`, and `<c-l>` to navivate vim windows and
tmux panes.

## Operator-Pending Mode, Moves, and Text Objects

I started realizing vim's power (this time around) when first hitting one of
the operator-plus-text-object combinations such as `2das` (deleting this
sentence and the next) or `ci"` to change the text within double quotes.  What
makes these commands so powerful and (eventually) easy to use is the consistent
count + operator + movement-or-text-object pattern (count and operator can be
swapped) and the "orthogonality" of its parts.  Any movement or text object can
be combined with any operator.

The movement can be simple movement (such as w, b, e, ge, (, ), and so forth),
a search (/ or ?), or a jump (e.g., to a mark).  The text object themselves
follow the pattern scope + object, where scope is either `i` for "in" or
"inside" or `a` for "around" or "a".  `dap`, for example, deletes the current
paragraph (dap = delete a paragraph), `2d/and<cr>`, deletes up to the second
following occurrence "and" (where "and" could be any regular expression), and
`cit` changes the content of an XML tag.

Furthermore, these commands can be preceded by a register (" + some letter)
which causes the involved text to be stored in named register instead of the
default ones (the `"` register and additionally the yank register `0` for
yanks).  `"a2cw`, for example, changes the following two words and stores the
original content in register 'a', ready to be pasted somewhere else.  Capital
letters append to the register (in contrast to marks where capital letters are
global marks).  If you want to throw away the copied text and not copy it into
the default register, you can use the "black hole" (or /dev/null) register `_`.

## Cheat Sheet

The following sections cover only those commands that I either tend to (or used
to) forget or that I noticed somewhere and still want to integrate into my
workflow.

### Miscellaneous Hints

* In visual mode, `p` and `P` replace the selected text (like normal editors,
  but unlike emacs) and put the replaced text into the default register.
* `nH`, `nL`: place cursor n lines from top, bottom
* `xp`, `Xp`: swap current character with next, previous
* `*` and `#` search the current word forwards and backwards, respectively. The
  `g*` and `g#` variants also match inside words.
* `qxq` clears register `x`
* useful ex commands: s, S (with vim-abolish), yank, sort
* ex range limits may a expressions involving patterns (representing the next
  line containing the pattern), e.g., `.+1,/foo/-1`
* `:cw` opens quickfix window, `<cr>` selects location, `:ccl` closes the window

### Marks

One of the features one may overlook initially, but which is so useful one will
immediately miss it when using other editors.   `mx` defines mark `x` where `x`
can be any letter, ``` `x``` jumps back to the mark, and `'x` jumps to the
beginning of the mark's line.  The automatic marks `.` (last change), `^` (last
insertion), ``` ` ``` (position before last jump) are useful as well.  `:marks`
shows the list of marks.

Vim maintains a list of change positions that can be navigated with `g;` and
`g,`.  Similarly, `gi` jumps to the last insert position and enters insert
mode.  This is useful when coding in some part of a file and in between looking
at (or copying from) other places in the file.

### help

Vim's help system is very good, but takes a while to get comfortable with (in
part because it obviously relies on vi commands for navigation).

* `C-]`, `C-t`, `C-o`: follow link, jump back
* `:h <c-w>`: looks up help for key code `<c-w>`

### Windows

* `<c-w>q`, `<c-w>o`: close current, other window(s)
* `:windo *cmd*`: run command in all windows
* `<c-w>-`, `<c-w>+`, `z{nr}<cr>`, `<c-w><`, `<c-w>>`, `<c-w>|`: descrease,
  increase, set height, width
* `<c-w>r`, `<c-w>R`, `<c-w>x`: rotate, exchange windows
* `<c-w>H`, `<c-w>J`, `<c-w>K`, `<c-w>L`: move current window to left, bottom,
  top, right

### XML Editing

* `</<c-x><c-o>` (omni complete): closes tag

### Repeats

* `.`repeats last change (moving around in insert mode resets change)
* `@:`repeats last ex command
* `&`repeats last substitution (`:s/foo/bar`)
* `@x`repeats macro x (defined with `qx{changes}q`)

### Key Bindings

* ctrl-space is bound using `<nul>` (not `<c-space>` as one might guess).

### ctags

Steps:

* Generate tags file with `ctags -R .`
* Jump to definition with `<C-]>`, jump back with `<C-t>` (as in help navigation).
* Search tag with `:tag /regex
* Use `:ts`, `:tn`, `:tp`, `:tf`, `:tl` for show, next, previous, first, last.

### grep

Vim has a built-in `:vimgrep` and can also call the external grep command.

* ```:vimgrep /pattern/ *.js```
* `:cn`, `:cN`: next, previous occurrence (quickfix list)

### Colors: solarized

I'm using the [solarized](http://ethanschoonover.com/solarized) color scheme
for vim (and the terminal in general). Installation on Ubuntu:

* Add new gnome-terminal profile (menu Edit/Profiles...), e.g., "solarized",
  and make it the default.
* Install the solarized colors into this profile using
  [gnome-terminal-colors-solarized](https://github.com/Anthony25/gnome-terminal-colors-solarized).
* Add the solarized plugin `Plugin 'altercation/vim-colors-solarized'`.
* Select the solarized color scheme:

```
set t_Co=16
set background=dark
colorscheme solarized
```

* For vim to show the right colors in tmux, also set TERM in `.bashrc` (see [Stackoverflow][solarized-tmux-vim]):

```
export TERM=screen-256color-bce
```

[solarized-tmux-vim]: http://stackoverflow.com/questions/23118916/configuring-solarized-colorscheme-in-gnome-terminal-tmux-and-vim

## Plugins

Using Vundle, a plugin can be installed with:

* `<leader>ev` to edit `.vimrc`
* add Plugin line, e.g., `Plugin 'tommcdo/vim-exchange'`
* `<leader>sv` to load the new 'settings'
* `:PluginInstall` to install the new plugin

Essential plugins are

* scrooloose/nerdtree: The better explorer.
* tommcdo/vim-exchange
  - `cx{motion}` applied twice (the second time often with `.`) will exchange
    the two selection regions
  - `cxx` selects current line
  - `cxc` clears a first selection
* scrooloose/nerdcommenter: comment/uncomment blocks in many languages
  - `<leader>cc`: toggles comment
* kien/ctrlp.vim: fuzzy selection of files and buffers (see `:h ctrlp-mappings)
  - `<esc>`, `<c-c>`: quit
  - `<c-f>`, `<c-b>`: next, previous mode
  - `<c-n>`, `<c-p>`: next, previous in prompt history (old searches)
  - `<c-s>`, `<c-v>`: open in new horizontal, vertical split window
  - `<c-y>`: create new file including parent directories
  - `<c-z>`, `<c-o>`: toggle mark, open marked files
* tpope/vim-unimpaired: Nice symmetric key bindings using `[` and `]`.
  - `b`: buffers, `q`: errors (:cn, :cp)
  - `<space>`: add newline ([ = before, ] = after current line)
  - `u`: URL encode/decode, `x`: XML encode/decode
* tpope/vim-abolish: Gives you emacs-like smart case substitution and more.
  - `:%S/part{y,ies}/celebration{,s}/g` works as expected for Party and PARTIES
    as well
* mileszs/ack.vim: Vim integration of ack, `:Ack` opens result quickfix list
  and first location
  - `h`, `v`: open with horizontal, vertical split window
  - `<cr>`, `o`, `go`: open, preview (see `?` for more)
  - `q`: close quickfix window
  - without args, `:Ack` locks for current word
* terryma/vim-expand-region: `+`, `__` select more, less
* christoomey/vim-tmux-navigator: easy window pane/window navigation across vim
  and tmux
* bling/vim-airline: lightweight statusline

### Ultisnips

The `SirVer/utilsnips` plugin provides the equivalent of textmate's snippets or
emacs' yasnippets.

### Instant Markdown

Nice plugin (using redcarpet for the markdown formatting and a node.js server)
that shows the formatted markdown in Chrome.  The installation requires a bit
more work:

```
$ sudo apt-get install ruby-dev
$ sudo gem install pygments.rb
$ sudo gem install redcarpet
$ npm -g install instant-markdown-d
```

## [Vimium](http://vimium.github.io/)

Vim bindings and more in Chrome.

* `?`: help
* `h`, `j`, `k`, `l`, `gg`, `G`, `d`, `u`: scroll left, down, up, right, top, bottom, page down, page up
* `gs`: page source
* `yy`: copy URL to clipboard 
* `f`, `F`: open link on page with keyboard
* `o`, `O`, `b`, `B`: open anything, bookmark in current, new tab
* `K`, `gt`, `J`, `gT`, `g0`, `g$`, `yt`, `x`, `X`: navigate, duplicate, close, restore tabs
* `H`, `L`: back, forward in history

## Shell

To switch to vim bindings in the shell add `set -o vi` to your `.bashrc` or
`.zshrc`.  The shell will start in insert mode after a prompt.  By default, the
mode is not visible, but it can be turned on with the following `.inputrc`
configuration (showing a + for insert and a : for normal mode at the beginning
of the line):

```
set show-mode-in-prompt on
set editing-mode vi
set keymap vi-command
```

## Vrapper

Turns Eclipse editors into vim-like editors (including split windows, search,
etc.) while maintaining Eclipse's language specific features (syntactical
selection, refactoring, source generation).

However, this is a tough choice, because eclim in combination with ycm provides
the most useful Eclipse functions as well (e.g., code completion, rename
refactoring, and constructor and accessor generation) while staying in the vim
environment.

## Eclim

[Eclim](http://eclim.org) uses eclipse as an "IDE server".  The eclim Eclipse
plugin exposes the eclipse compiler functionality as a server, and the eclim
vim plugin talks to this server to make this functionality available in vim.
The eclim server is run as a separate daemon (`eclimd`).  Although it does not
use the UI portion of Eclipse, it still needs an X server on UNIX systems.  To
go truly [headless](http://eclim.org/install.html#install-headless) (e.g., when
using eclim remotely), one can start an xvfb (X virtual frame buffer) server
and let eclimd talk to this X server:

```
Xvfb :1 -screen - 1024x768x24 &
DISPLAY=:1 eclimd -b
```

Most important commands:

* `:Validate` (performed automatically when saving a file), marks errors in
  location list (which can be navigated with `[l`, `]l` when using unimpaired).
