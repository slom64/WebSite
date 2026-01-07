

| Command                   | Description                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| :1,$s/^/Your String Here/ | append specific string at the start of each line. `1,$`:Specifies the range of lines to operate on. <br>`s`:The command to search for a pattern and replace it with a specified string.<br>`/^/`:The caret (`^`) is a **regular expression anchor** that matches the **beginning** of a line.<br>`/Your String Here/`:This is the actual string you want to insert.<br> |
|                           |                                                                                                                                                                                                                                                                                                                                                                         |






---

| Category | Command               | Description                              |
| -------- | --------------------- | ---------------------------------------- |
| Modes    | `i`                   | Enter Insert mode (insert before cursor) |
|          | `I`                   | Enter Insert mode at beginning of line   |
|          | `a`                   | Enter Insert mode (append after cursor)  |
|          | `A`                   | Enter Insert mode at end of line         |
|          | `o`                   | Enter Insert mode on new line below      |
|          | `O`                   | Enter Insert mode on new line above      |
|          | `v`                   | Enter Visual mode (character-wise)       |
|          | `V`                   | Enter Visual mode (line-wise)            |
|          | `<Ctrl-v>`            | Enter Visual mode (block-wise)           |
|          | `R`                   | Enter Replace mode                       |
|          | `<Esc>` or `<Ctrl-[>` | Exit to Normal mode                      |
|          | `:`                   | Enter Command-line (Ex) mode             |
|          | `/`                   | Enter search forward                     |
|          | `?`                   | Enter search backward                    |

| Navigation | Command | Description |
|------------|---------|-------------|
| Basic Movement | `h` / `j` / `k` / `l` | Left / Down / Up / Right |
| | `0` or `^` | Move to beginning of line (0: absolute, ^: first non-blank) |
| | `$` | Move to end of line |
| | `w` / `W` | Forward to next word / WORD |
| | `b` / `B` | Backward to previous word / WORD |
| | `e` / `E` | Forward to end of word / WORD |
| | `ge` / `gE` | Backward to end of word / WORD |
| Larger Jumps | `gg` | Go to first line |
| | `G` | Go to last line |
| | `{n}G` | Go to line {n} |
| | `{n}%` | Go to {n}% in file |
| | `(` / `)` | Previous / Next sentence |
| | `{` / `}` | Previous / Next paragraph |
| Screen | `<Ctrl-f>` / `<Ctrl-b>` | Page forward / backward |
| | `<Ctrl-d>` / `<Ctrl-u>` | Half-page down / up |
| | `H` / `M` / `L` | Top / Middle / Bottom of screen |
| | `zt` / `zz` / `zb` | Scroll current line to top / center / bottom |

| Inserting & Editing | Command | Description |
|---------------------|---------|-------------|
| Insert | `i` / `a` / `o` etc. (see Modes) | As above |
| Delete | `x` / `X` | Delete character under / before cursor |
| | `dw` / `d$` / `dd` | Delete word / to end of line / entire line |
| | `d{n}w` / `d{n}d` | Delete {n} words / {n} lines |
| | `D` | Delete to end of line (like d$) |
| Change | `cw` / `c$` / `cc` | Change word / to end of line / entire line |
| | `r{char}` | Replace single character |
| | `~` | Toggle case of character |
| | `J` | Join next line to current |
| Repeat | `.` | Repeat last command |
| | `{n}.` | Repeat last command {n} times |

| Copy, Cut, Paste | Command | Description |
|-------------------|---------|-------------|
| Yank (Copy) | `yw` / `y$` / `yy` | Yank word / to end of line / entire line |
| | `y{n}w` / `y{n}y` | Yank {n} words / {n} lines |
| | `"a yy` | Yank line to register a (named register) |
| Delete (Cut) | `dw` / `dd` etc. | Deletes also yank to unnamed register |
| | `"a dw` | Delete word to register a |
| Put (Paste) | `p` / `P` | Paste after / before cursor |
| | `"a p` | Paste from register a |
| | `{n}p` | Paste {n} times |

| Search & Replace | Command | Description |
|-------------------|---------|-------------|
| Search | `/pattern<Enter>` | Search forward for pattern |
| | `?pattern<Enter>` | Search backward for pattern |
| | `n` / `N` | Next / previous match |
| | `*` / `#` | Search forward / backward for word under cursor |
| Replace | `:%s/old/new/g` | Replace all old with new in file (global) |
| | `:s/old/new/` | Replace first occurrence on current line |
| | `:s/old/new/g` | Replace all on current line |
| | `:{range}s/old/new/g` | Replace in range (e.g., 1,10) |
| Highlight | `:set hlsearch` / `:nohl` | Enable / disable search highlighting |

| Undo & Redo | Command | Description |
|-------------|---------|-------------|
| Undo | `u` | Undo last change |
| | `{n}u` | Undo {n} changes |
| | `U` | Undo all changes on current line |
| Redo | `<Ctrl-r>` | Redo last undone change |
| | `{n}<Ctrl-r>` | Redo {n} changes |

| File Operations | Command | Description |
|------------------|---------|-------------|
| Save | `:w` | Write (save) file |
| | `:w filename` | Write to new filename |
| | `:wq` or `ZZ` | Write and quit |
| Quit | `:q` | Quit (fails if unsaved) |
| | `:q!` or `ZQ` | Quit without saving |
| | `:qa` | Quit all |
| Edit | `:e filename` | Edit new file |
| | `:enew` | New empty buffer |
| Buffers | `:ls` | List buffers |
| | `:b{n}` | Switch to buffer {n} |
| | `:bn` / `:bp` | Next / previous buffer |
| Splits | `:sp` / `:vsp` | Horizontal / vertical split |
| | `<Ctrl-w> w` | Switch windows |
| | `<Ctrl-w> q` | Close window |

| Visual Mode | Command | Description |
|-------------|---------|-------------|
| Enter | `v` / `V` / `<Ctrl-v>` (see Modes) | As above |
| Select | Movement keys | Extend selection |
| Operations | `d` / `x` | Delete selection |
| | `y` | Yank selection |
| | `c` | Change selection |
| | `~` | Toggle case |
| | `>` / `<` | Indent / unindent (line or block) |
| | `gv` | Reselect last visual selection |

| Ex Commands & Settings | Command | Description |
|------------------------|---------|-------------|
| Set Options | `:set nu` / `:set nonu` | Show / hide line numbers |
| | `:set ic` / `:set noic` | Ignore / respect case in search |
| | `:set hlsearch` / `:set nohl` | Highlight / no highlight search |
| | `:set wrap` / `:set nowrap` | Wrap / no wrap long lines |
| Macros | `q{reg}` | Start recording macro to register {reg} |
| | `q` | Stop recording |
| | `@{reg}` | Play macro from {reg} |
| | `{n}@{reg}` | Play {n} times |
| Help | `:help {topic}` | Open help on topic |
| | `:help` | Open general help |
| Shell | `:!command` | Run shell command |
| | `<Ctrl-z>` | Suspend Vim, fg to resume |

| Advanced Editing | Command | Description |
|-------------------|---------|-------------|
| Marks | `m{letter}` | Set mark at cursor (a-z local, A-Z global) |
| | `'{letter}` / ``{letter}` | Jump to line / position of mark |
| Registers | `"{reg}` before command | Use register {reg} for next delete/yank/change |
| | `:reg` | List registers |
| Folding | `zf` (visual) | Create fold |
| | `zo` / `zc` | Open / close fold |
| | `zR` / `zM` | Open / close all folds |
| Diff | `:diffthis` | Enable diff mode in window |
| | `:diffoff` | Disable diff |
| Completion | `<Ctrl-n>` / `<Ctrl-p>` (insert) | Next / previous completion |
| | `<Ctrl-x><Ctrl-f>` | File name completion |