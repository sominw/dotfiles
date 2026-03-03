## Terminal UX Hardening Plan (Incremental, Hybrid iTerm+tmux, Dual Vim+Neovim)

### Summary
We will keep your current stack (`oh-my-zsh` + `powerlevel10k` + Vim + tmux + iTerm), remove a few risky defaults, and add high-leverage ergonomics for mouse-driven editing, pane-heavy coding, and cross-platform portability (macOS + Linux remotes).  
This plan is intentionally incremental: no major workflow reset, no aggressive remaps.

Current-state findings informing this plan:
- [`/Users/sominw/dotfiles/.zshrc`](/Users/sominw/dotfiles/.zshrc) has minor hygiene/perf issues (duplicate alias, ordering that can be improved).
- [`/Users/sominw/dotfiles/.vimrc`](/Users/sominw/dotfiles/.vimrc) has two dangerous global settings: `set binary` and `set noeol`.
- [`/Users/sominw/dotfiles/.tmux.conf`](/Users/sominw/dotfiles/.tmux.conf) is very minimal; missing clipboard/vi-copy/term features.
- iTerm profile is configured with low scrollback (`1000`) and old defaults that limit modern ergonomics.

---

### Phase 0: Safety + Baseline
1. Snapshot dotfiles:
- `git -C /Users/sominw/dotfiles checkout -b codex/terminal-hardening`
- `git -C /Users/sominw/dotfiles add -A && git -C /Users/sominw/dotfiles commit -m "baseline before terminal hardening"`

2. Baseline measurements:
- `time zsh -i -c exit`
- `tmux -V`
- `vim --version | rg 'clipboard|mouse|termguicolors'`

---

### Phase 1: Shell Improvements (`.zshrc`)
File: [`/Users/sominw/dotfiles/.zshrc`](/Users/sominw/dotfiles/.zshrc)

1. Reorder init for reliability:
- Keep `p10k-instant-prompt` block at top.
- Keep `plugins=(...)` before OMZ load.
- Move `[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh` to after `source $ZSH/oh-my-zsh.sh`.

2. Hygiene fixes:
- Remove duplicate `alias reload`.
- Keep your existing key habits and aliases unchanged otherwise.

3. Add portable history and completion defaults:
- `HISTSIZE=200000`, `SAVEHIST=200000`, `HISTFILE=~/.zsh_history`
- `setopt inc_append_history share_history hist_ignore_all_dups hist_reduce_blanks`
- case-insensitive completion + menu select styles.

4. Add cross-platform guard helpers used by functions:
- `is_macos()` and `is_linux()` shell helpers.

5. Harden `newmeeting()` for Linux remotes:
- Keep mac behavior (`open "$file"`).
- Linux fallback: `${EDITOR:-vim} "$file"`.

6. Add optional tool integrations (only when binaries exist):
- `zoxide init zsh`
- `fzf` keybindings/completion
- `direnv hook zsh`

---

### Phase 2: tmux Ergonomics for Hybrid Usage
File: [`/Users/sominw/dotfiles/.tmux.conf`](/Users/sominw/dotfiles/.tmux.conf)

1. Preserve your existing choices:
- Keep prefix `C-a`.
- Keep split bindings (`|` and `_`).
- Keep mouse enabled.

2. Add robust defaults:
- `set -g history-limit 100000`
- `setw -g mode-keys vi`
- `set -s escape-time 10`
- `set -g set-clipboard on`
- `set -g renumber-windows on`
- `set -g focus-events on`
- `set -g default-terminal "tmux-256color"`
- RGB support via terminal features for `xterm-256color`.

3. Add working-directory-aware splits/windows:
- Splits and new windows start in `#{pane_current_path}`.

4. Add pane navigation/resizing without remapping your core habits:
- Prefix-based pane moves on `h/j/k/l`.
- Prefix-based resize on `H/J/K/L`.

5. Add copy-mode clipboard portability:
- `copy-mode-vi` bindings (`v` select, `y` copy).
- macOS pipe to `pbcopy`; Linux fallback to `xclip`/`wl-copy`; final fallback to normal tmux buffer copy.

---

### Phase 3: Vim Hardening + Dual Neovim Track
Primary file: [`/Users/sominw/dotfiles/.vimrc`](/Users/sominw/dotfiles/.vimrc)  
New file: [`/Users/sominw/dotfiles/.config/nvim/init.vim`](/Users/sominw/dotfiles/.config/nvim/init.vim)

1. Fix risky global options:
- Remove `set binary`
- Remove `set noeol`

2. Add mouse and clipboard behavior for terminal workflow:
- `set mouse=a`
- `set clipboard=unnamed,unnamedplus`
- `set ttymouse=sgr`

3. Improve edit safety and productivity:
- Enable persistent undo (`undofile`, `undodir` with auto-create).
- Keep existing leader and mappings intact.
- Keep `wrap` behavior only for text-like filetypes via autocmd; default to `nowrap` for code buffers.

4. Search ergonomics update:
- Keep current mappings.
- Add `rg`-backed search command/mapping fallback so current `Ack`-style flows don’t silently break if `Ack` isn’t installed.

5. Dual Vim+Neovim setup (no migration yet):
- Install `neovim`.
- `init.vim` sources your existing `.vimrc` to keep one behavior surface.
- Add a lightweight `nv` command/alias for optional Neovim usage.

---

### Phase 4: iTerm Profile Tuning (UI-level, no repo file)
1. Update iTerm2 to current stable.
2. Keep terminal type `xterm-256color`.
3. Increase scrollback (target: 50k+ or unlimited).
4. Keyboard meta strategy:
- Left Option: `Esc+` (for meta mappings).
- Right Option: Normal (for symbols/accents).
5. Enable shell integration features useful for pane-heavy workflows (marks, prompt tracking, jump/navigation).
6. Keep your existing pointer actions; validate click-to-position behavior specifically inside `vim` and `tmux`.

---

### Phase 5: Tooling Install (small core set)
Install exactly this set:
- `brew install fzf fd bat eza zoxide direnv git-delta neovim`
- Run fzf installer in non-RC mode and wire hooks in `.zshrc` manually.

No broad tooling expansion beyond this set.

---

### Public Interface Changes (Aliases/Functions/Keybindings)
1. Shell interfaces:
- Existing aliases preserved.
- `newmeeting` gains Linux-safe opener behavior.
- Optional new helper commands (additive only): `nv` (Neovim), and fzf/fd/zoxide-driven helpers.

2. tmux key interface:
- Existing `C-a`, `|`, `_` preserved.
- New prefix pane nav/resize and vi-style copy-mode keys (`v`, `y`).

3. Vim interface:
- Existing leader `,` and mappings preserved.
- Mouse + clipboard enabled for click-heavy workflow.
- New optional `rg` mapping/command for project search.

---

### Test Cases and Validation Scenarios
1. Shell startup and reliability:
- `zsh -i -c 'echo ok'` works with no prompt init errors.
- Startup time is not worse than baseline (target: improve or hold steady).

2. Mouse-driven editing:
- Inside `tmux` -> `vim`, single-click repositions cursor reliably.
- Drag selection works predictably in copy-mode and normal mode.

3. Clipboard correctness:
- tmux copy-mode `v` then `y` reaches macOS clipboard (`pbpaste` check).
- On Linux remote, fallback copy path works (xclip/wl-copy or tmux buffer fallback).

4. Pane workflow:
- New split opens in current project directory.
- Prefix navigation/resizing behaves consistently across sessions.

5. Vim safety:
- Saving normal text/code files no longer forces binary/noeol behavior.
- Undo persists across Vim restarts.

6. Dual editor behavior:
- `vim` and `nvim` share core config behavior.
- No regressions in existing mappings.

---

### Succinct “Don’t Forget” Tips to Include at the End
Add a tiny cheat sheet file and shell command:
- File: `/Users/sominw/dotfiles/TERMINAL_CHEATSHEET.md`
- Command: `tkeys` prints top 12 bindings.

Cheat sheet content target:
- `tmux`: `C-a |`, `C-a _`, `C-a h/j/k/l`, `C-a H/J/K/L`, copy-mode `v` then `y`
- `vim`: `,w`, `,bd`, `,ss`, click-to-place cursor
- shell: `reload`, `newmeeting <project> <type> "<attendees>"`, `nv`

---

### Assumptions and Defaults Chosen
1. Keep `oh-my-zsh` + `powerlevel10k`; no shell framework migration.
2. Preserve existing keybindings by default; only additive/non-conflicting mappings.
3. Keep Vim as primary editor; Neovim added as parallel optional path.
4. Optimize for macOS local use with Linux-compatible fallbacks for remote work.
5. Install only the agreed small tool set; skip broader toolchain expansion.
