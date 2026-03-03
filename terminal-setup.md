# Terminal Prompt Setup — Replication Guide

This document replicates the following prompt setup:

- **Left prompt (line 1):** OS icon → directory path (Nerd Font folder icons, ancestors shortened, muted steel-blue pill) → git branch/status
- **Left prompt (line 2):** `❯` cursor in cyan
- **Right prompt:** active Python virtualenv name (yellow, hidden when none) → current time (HH:MM, grey)
- **Transient prompt:** previous prompts collapse to a single line after each command

---

## macOS (iTerm2 + zsh + oh-my-zsh)

### Prerequisites

- macOS with Homebrew installed (`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`)
- iTerm2 installed (https://iterm2.com)
- Internet access for cloning repos

---

### Step 1 — Install zsh and set it as the default shell

macOS ships with zsh since Catalina. Verify:

```bash
zsh --version   # should print zsh 5.x or higher
echo $SHELL     # should print /bin/zsh
```

If not already default:

```bash
chsh -s $(which zsh)
```

---

### Step 2 — Install oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

This creates `~/.oh-my-zsh` and writes a starter `~/.zshrc`. If you already have a `~/.zshrc` it will be backed up as `~/.zshrc.pre-oh-my-zsh`.

---

### Step 3 — Install Hack Nerd Font

The prompt uses Nerd Font glyphs (folder icons, branch icon, etc). Hack Nerd Font is used here.

```bash
brew tap homebrew/cask-fonts
brew install --cask font-hack-nerd-font
```

Then set iTerm2 to use it:

1. Open iTerm2 → Preferences → Profiles → Text
2. Change the font to **Hack Nerd Font** (Regular, size 13 or your preference)
3. Tick "Use a different font for non-ASCII text" and set that to **Hack Nerd Font** as well

---

### Step 4 — Install community zsh plugins

```bash
# zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions

# zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
  ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

---

### Step 5 — Install Powerlevel10k

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ~/.oh-my-zsh/custom/themes/powerlevel10k
```

---

### Step 6 — Write `~/.zshrc`

Replace the contents of `~/.zshrc` with the following (or merge carefully if you have existing customisations you want to keep):

```zsh
# Enable Powerlevel10k instant prompt. Should stay close to the top of ~/.zshrc.
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="powerlevel10k/powerlevel10k"

plugins=(
  git
  docker
  npm
  vscode
  macos
  web-search
  history-substring-search
  zsh-autosuggestions
  zsh-syntax-highlighting
)

source $ZSH/oh-my-zsh.sh

# Aliases
alias zshconfig="vi ~/.zshrc"
alias ohmyzsh="vi ~/.oh-my-zsh"
alias cleanup="find . -type f -name '*.DS_Store' -ls -delete"
alias path='echo -e ${PATH//:/\\n}'
alias reload="exec ${SHELL} -l"
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'
alias sudo='sudo '

# History
HISTFILE="$HOME/.zsh_history"
HISTSIZE=200000
SAVEHIST=200000
setopt inc_append_history
setopt share_history
setopt hist_ignore_all_dups
setopt hist_reduce_blanks

# Completion
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Za-z}'
zstyle ':completion:*' menu select

is_macos() { [[ "$OSTYPE" == darwin* ]]; }
is_linux() { [[ "$OSTYPE" == linux* ]]; }

# Optional integrations — only loaded when the tool is installed
if command -v zoxide >/dev/null 2>&1; then
  eval "$(zoxide init zsh)"
fi

if command -v direnv >/dev/null 2>&1; then
  eval "$(direnv hook zsh)"
fi

# iTerm2 shell integration
if [[ -r "$HOME/.iterm2_shell_integration.zsh" ]]; then
  source "$HOME/.iterm2_shell_integration.zsh"
elif is_macos && [[ -r "/Applications/iTerm.app/Contents/Resources/iterm2_shell_integration.zsh" ]]; then
  source "/Applications/iTerm.app/Contents/Resources/iterm2_shell_integration.zsh"
fi

# fzf
if [[ -o interactive && -t 0 && -t 1 ]]; then
  if [[ -f /opt/homebrew/opt/fzf/shell/completion.zsh ]]; then
    source /opt/homebrew/opt/fzf/shell/completion.zsh
  elif [[ -f "$HOME/.fzf/shell/completion.zsh" ]]; then
    source "$HOME/.fzf/shell/completion.zsh"
  fi
  if [[ -f /opt/homebrew/opt/fzf/shell/key-bindings.zsh ]]; then
    source /opt/homebrew/opt/fzf/shell/key-bindings.zsh
  elif [[ -f "$HOME/.fzf/shell/key-bindings.zsh" ]]; then
    source "$HOME/.fzf/shell/key-bindings.zsh"
  fi
fi

if command -v nvim >/dev/null 2>&1; then
  alias nv='nvim'
fi

# To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```

---

### Step 7 — Write `~/.p10k.zsh`

Create `~/.p10k.zsh` with the exact contents below. Do not run `p10k configure` — this file replaces the wizard output entirely.

```zsh
# Powerlevel10k prompt configuration.
# Generated manually — no wizard output.

typeset -g POWERLEVEL9K_INSTANT_PROMPT=quiet

# ─── Prompt layout ──────────────────────────────────────────────────────────

typeset -g POWERLEVEL9K_PROMPT_ON_NEWLINE=true
typeset -g POWERLEVEL9K_RPROMPT_ON_NEWLINE=false
typeset -g POWERLEVEL9K_MULTILINE_FIRST_PROMPT_PREFIX=''
typeset -g POWERLEVEL9K_MULTILINE_LAST_PROMPT_PREFIX='%F{cyan}❯%f '

# Left segments
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
  os_icon
  dir
  vcs
  newline
)

# Right segments
typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
  virtualenv
  pyenv
  time
)

# ─── OS icon ────────────────────────────────────────────────────────────────

typeset -g POWERLEVEL9K_OS_ICON_FOREGROUND=250
typeset -g POWERLEVEL9K_OS_ICON_BACKGROUND=235

# ─── Directory ──────────────────────────────────────────────────────────────

typeset -g POWERLEVEL9K_DIR_MAX_LENGTH=3
typeset -g POWERLEVEL9K_SHORTEN_STRATEGY=truncate_to_unique
typeset -g POWERLEVEL9K_SHORTEN_DELIMITER=''
typeset -g POWERLEVEL9K_DIR_ANCHOR_BOLD=false
typeset -g POWERLEVEL9K_DIR_ANCHOR_FOREGROUND=117

typeset -g POWERLEVEL9K_DIR_SHOW_WRITABLE=v3
typeset -g POWERLEVEL9K_LOCK_ICON='󰌾'

typeset -g POWERLEVEL9K_DIR_BACKGROUND=24
typeset -g POWERLEVEL9K_DIR_FOREGROUND=252
typeset -g POWERLEVEL9K_DIR_SHORTENED_FOREGROUND=245

typeset -g POWERLEVEL9K_DIR_PREFIX='  '

typeset -g POWERLEVEL9K_HOME_ICON=''
typeset -g POWERLEVEL9K_HOME_SUB_ICON=''
typeset -g POWERLEVEL9K_FOLDER_ICON=''
typeset -g POWERLEVEL9K_ETC_ICON=''

# ─── VCS (git) ──────────────────────────────────────────────────────────────

typeset -g POWERLEVEL9K_VCS_BRANCH_ICON='\uE0A0 '
typeset -g POWERLEVEL9K_VCS_CLEAN_BACKGROUND=2
typeset -g POWERLEVEL9K_VCS_CLEAN_FOREGROUND=0
typeset -g POWERLEVEL9K_VCS_MODIFIED_BACKGROUND=3
typeset -g POWERLEVEL9K_VCS_MODIFIED_FOREGROUND=0
typeset -g POWERLEVEL9K_VCS_UNTRACKED_BACKGROUND=5
typeset -g POWERLEVEL9K_VCS_UNTRACKED_FOREGROUND=0

# ─── Virtual environment ────────────────────────────────────────────────────

typeset -g POWERLEVEL9K_VIRTUALENV_SHOW_PYTHON_VERSION=false
typeset -g POWERLEVEL9K_VIRTUALENV_SHOW_WITH_PYENV=false
typeset -g POWERLEVEL9K_VIRTUALENV_FOREGROUND=0
typeset -g POWERLEVEL9K_VIRTUALENV_BACKGROUND=220
typeset -g POWERLEVEL9K_VIRTUALENV_PREFIX=' '

typeset -g POWERLEVEL9K_PYENV_FOREGROUND=0
typeset -g POWERLEVEL9K_PYENV_BACKGROUND=220
typeset -g POWERLEVEL9K_PYENV_PROMPT_ALWAYS_SHOW=false
typeset -g POWERLEVEL9K_PYENV_SHOW_SYSTEM=false

# ─── Time ───────────────────────────────────────────────────────────────────

typeset -g POWERLEVEL9K_TIME_FORMAT='%D{%H:%M}'
typeset -g POWERLEVEL9K_TIME_FOREGROUND=0
typeset -g POWERLEVEL9K_TIME_BACKGROUND=7
typeset -g POWERLEVEL9K_TIME_PREFIX=' '
typeset -g POWERLEVEL9K_TIME_UPDATE_ON_COMMAND=true

# ─── General segment styling ────────────────────────────────────────────────

typeset -g POWERLEVEL9K_LEFT_SEGMENT_SEPARATOR='\uE0B0'
typeset -g POWERLEVEL9K_RIGHT_SEGMENT_SEPARATOR='\uE0B2'
typeset -g POWERLEVEL9K_LEFT_SUBSEGMENT_SEPARATOR='\uE0B1'
typeset -g POWERLEVEL9K_RIGHT_SUBSEGMENT_SEPARATOR='\uE0B3'

typeset -g POWERLEVEL9K_LEFT_PROMPT_FIRST_SEGMENT_START_SYMBOL=''
typeset -g POWERLEVEL9K_RIGHT_PROMPT_LAST_SEGMENT_END_SYMBOL=''

# ─── Transient prompt ───────────────────────────────────────────────────────

typeset -g POWERLEVEL9K_TRANSIENT_PROMPT=always

(( ${#} )) || true
```

---

### Step 8 — Activate

```bash
exec zsh
```

The new prompt should appear immediately. If icons look like boxes or question marks, confirm the iTerm2 font is set to Hack Nerd Font (Step 3).

---

## Linux (bash/zsh + GNOME Terminal or similar)

> These instructions assume a Debian/Ubuntu-based distro. Adjust package manager commands (`apt` → `dnf`, `pacman`, etc.) for other distros. The terminal emulator is assumed to be GNOME Terminal, Konsole, or any other that supports custom fonts.

---

### Step 1 — Install zsh

```bash
sudo apt update && sudo apt install -y zsh git curl
chsh -s $(which zsh)
```

Log out and back in for the shell change to take effect.

---

### Step 2 — Install oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

---

### Step 3 — Install Hack Nerd Font

```bash
mkdir -p ~/.local/share/fonts
cd /tmp
curl -fLo "HackNerdFont.zip" \
  https://github.com/ryanoasis/nerd-fonts/releases/latest/download/Hack.zip
unzip -o HackNerdFont.zip -d ~/.local/share/fonts/HackNerdFont
fc-cache -fv
```

Then set your terminal emulator to use **Hack Nerd Font**:

- **GNOME Terminal:** Edit → Preferences → your profile → Text → Custom font → select "Hack Nerd Font"
- **Konsole:** Settings → Edit Current Profile → Appearance → Font → select "Hack Nerd Font"
- **Other terminals:** look for the font setting in profile/preferences

---

### Step 4 — Install community zsh plugins

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
  ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

---

### Step 5 — Install Powerlevel10k

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ~/.oh-my-zsh/custom/themes/powerlevel10k
```

---

### Step 6 — Write `~/.zshrc`

Replace `~/.zshrc` with the following. Note the `macos` plugin is removed since it is macOS-only, and the iTerm2 integration block is omitted.

```zsh
# Enable Powerlevel10k instant prompt.
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="powerlevel10k/powerlevel10k"

plugins=(
  git
  docker
  npm
  web-search
  history-substring-search
  zsh-autosuggestions
  zsh-syntax-highlighting
)

source $ZSH/oh-my-zsh.sh

# Aliases
alias zshconfig="vi ~/.zshrc"
alias ohmyzsh="vi ~/.oh-my-zsh"
alias path='echo -e ${PATH//:/\\n}'
alias reload="exec ${SHELL} -l"
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'
alias sudo='sudo '

# History
HISTFILE="$HOME/.zsh_history"
HISTSIZE=200000
SAVEHIST=200000
setopt inc_append_history
setopt share_history
setopt hist_ignore_all_dups
setopt hist_reduce_blanks

# Completion
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Za-z}'
zstyle ':completion:*' menu select

# Optional integrations
if command -v zoxide >/dev/null 2>&1; then
  eval "$(zoxide init zsh)"
fi

if command -v direnv >/dev/null 2>&1; then
  eval "$(direnv hook zsh)"
fi

# fzf
if [[ -o interactive && -t 0 && -t 1 ]]; then
  if [[ -f /usr/share/fzf/completion.zsh ]]; then
    source /usr/share/fzf/completion.zsh
  elif [[ -f "$HOME/.fzf/shell/completion.zsh" ]]; then
    source "$HOME/.fzf/shell/completion.zsh"
  fi
  if [[ -f /usr/share/fzf/key-bindings.zsh ]]; then
    source /usr/share/fzf/key-bindings.zsh
  elif [[ -f "$HOME/.fzf/shell/key-bindings.zsh" ]]; then
    source "$HOME/.fzf/shell/key-bindings.zsh"
  fi
fi

if command -v nvim >/dev/null 2>&1; then
  alias nv='nvim'
fi

[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```

---

### Step 7 — Write `~/.p10k.zsh`

The p10k config is identical to macOS, **except** `os_icon` will show a Linux/distro logo instead of the Apple logo automatically — no change needed. Create `~/.p10k.zsh` with the exact same contents as the macOS Step 7 above.

---

### Step 8 — Activate

```bash
exec zsh
```

If icons still render as boxes after setting the font, try:

```bash
fc-cache -fv
```

Then close and reopen the terminal.

---

## Verification checklist

After activating on either platform, confirm:

- [ ] Left prompt shows OS icon → folder-icon path (shortened ancestors) → git branch when inside a repo
- [ ] Right prompt shows time in HH:MM format
- [ ] Right prompt shows virtualenv name when a Python venv is active (`source venv/bin/activate`)
- [ ] Ancestors in the path are shortened to their unique prefix (e.g. `~/p/my-project`)
- [ ] After running a command, the previous prompt collapses to a single line (transient prompt)
- [ ] No boxes or question marks where icons should be (indicates font not set correctly)
