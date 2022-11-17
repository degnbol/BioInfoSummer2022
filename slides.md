# git-good (git for collaborative science seminar)

## intro slide(s)
Who
- Researchers working on a project involving code (in general text files).

What
- Git: tool for your project/repo(sitory) to track
 - code **changes** over time.
 - who did what and combining work from multiple **collaborators**.
- GitHub: a remote (bare) copy of your repo.
 - easy to share with others (compared to copying from your local).
 - a stable backup.

Why (motivation slide)
- Better than "file.py" -> "file2.py" -> "file2_final.py" -> "file2_FINAL.py" 
  -> "file2_FINAL2.py" -> "file_FINAL_final.py" -> 
  "file_FINAL_forRealThisTime.py" -> "file_FINAL_forRealThisTime_2.py" -> ...
- Nothing is (almost) ever lost (don't be afraid to delete and cleanup)
 - restore old version of anything
- Multiple people working on the same file concurrently even offline
- Temporarily leave half finished work, try alternative approaches, ... 
  (branches)
- ...

## Endless options
- GUI screenshots, plugins
- % justifying that we will use the core commandline tool

## Setup
- Windows: https://gitforwindows.org/
- Mac & Linux: Open your favourite terminal (I'll be using kitty)
 - `git` to see if installed. Install with your package manager, e.g. homebrew 
 for Mac `https://brew.sh/`
 See `https://git-scm.com/downloads`

## Go through hypothetical collaborative project hosted on github.
- show git commit nodes as we go
- setup, ssh keys etc. private initially, make public later.
- invite collab
- add README.md, install.sh, Project.toml, requirements.txt, src/ folder, data 
  folder with wget command. Best practise suggestions: Talk about storing code 
  not data on git. Show github limits. Git gets slow with too many files. If 
  you wish to store large amounts of data then compress it, which means git 
  will not track within file changes, and you could consider having a bare git 
  repo for your lab set up on a server which can easily be set up for anyone 
  with ssh access.
- Add relevant branches, see below.
- deal with a conflict
- maybe mention that overleaf is git tracked, or that you can edit your latex 
  locally and still collab using git.

Commands
- `git config --global credential.helper store`
- `git status`
- `git add`: stage
- `git rm`: not the opposite of `add` at all, it's `rm` recorded in git.
- `git rm --cached`: stop tracking, but don't `rm`. Opposite of `add` for newly 
  added file.
- `git restore --staged`: the opposite of `add` for newly added changes to an 
  already tracked file. Maybe alias as `git unstage`

## Should I branch?
classics
- main vs dev branch
- issue branches
creative
- two or more ways of implementing or trying something new, quickly switch 
  between alt methods and try out on the same files. Show stash example.
- you have a nice clean structure in main branch that you can provide with your 
  publication, and then some more messy work stored away in other branches that 
  only you and collaborators are looking at.

## Config
Git has global and project settings.
- edit with commands or the config file directly
- Global in `~/.gitconfig` or `$XDG_CONFIG_HOME/git/config` where 
  `$XDG_CONFIG_HOME` == `~/.config` by default.
- Local in `<PROJECT_ROOT>/.git/config`
- List all: `git config --list`
- Edit global settings: `git config --global --edit`
- Editor: `git config --global core.editor <EDITOR>`. `<EDITOR>` examples:
 - `nano`: simple preinstalled terminal editor
 - `micro`: easy-to-use, not preinstalled (https://github.com/zyedidia/micro)
 - `vim`: standard modular terminal editor
 - `neovim`: modern reimplementation of vim
 - `emacs`: extensive non-modular terminal editor
 - `'open -nWa TextEdit'`: Mac GUI text editor (finish commit edit with save 
 then cmd+q)
 - Default: `$EDITOR` or `$VISUAL` (usually `vim`)

Settings

Aliases
- `git config --global alias.<SHORT> <LONG>`
- e.g. `git config --global alias.unstage 'restore --staged'`

## Tab-completion (might already work)
bash
- install bash-completion:
`brew/apt/yum/... install bash-completion`
Source the downloaded file on startup, e.g. `. /path/to/bash_completion`
  - homebrew: /opt/homebrew/etc/profile.d/bash_completion.sh
  - apt: /etc/bash_completion or /usr/share/bash-completion/bash_completion
  - otherwise find it: `sudo find /etc /usr /opt -name bash_completion`
zsh
- Just make sure general completion is enabled. Run on startup:
  `autoload -Uz compinit && compinit`

## help
- `git commit --help` == `git help commit` == `man git-commit`
  (`man` can be colored by zsh plugin 
  https://github.com/ael-code/zsh-colored-man-pages)
- Shorter help pages: `tldr git-commit` (https://github.com/tldr-pages/tldr)
- Short examples: `eg git` (https://github.com/srsudar/eg)
- Completion
- Google, StackOverflow, free ebook "git pro", ...

## Paths
- Can't work with abs paths, someone else isn't installing the same place or 
  has the same username.
- Relative doesn't work in all cases, e.g. referencing src/ scripts to run from 
  a workdir. It will be a lot of ../../../. It can also be problematic if you 
  want to move files later, e.g. to split work that has become large into 
  multiple subfolders.
- `git config --global alias.root 'rev-parse --show-toplevel'`
- `here` in R, etc. can find project root, however this requires that CWD is 
  inside git project, which means one has to cd into a Submodule before using 
  code within said submodule if it uses `git root` and subtree code will simply 
  have to be rewritten.
- Some approach this by simply writing all code relative to root and assumes 
  you are calling things from there.
- Env variable being set per session or in .zshrc to store location of root.

## Using work done by others
Sometimes you find a paper with an associated GitHub repo and want to download 
and use their work in your own. Options:
- Git clone (or just download)
 - outside your repo and mention dependence in README.
  - Requires additional install step
  - Assumes correct path
 - inside repo
  - either ignore dependency or tracking every file in it (messy double 
  tracking)
- Cool options
 - Subtree: basically git clone inside repo that your own repo understands
  - adds all files to your repo, allows for editing as usual
 - Submodule: git only records a reference to the dependency (specific commit 
 hash)
  - best if you are not going to edit the dependency

`git subtree add URL`
`git submodule add URL`
`git submodule update --init`
Can't necessarily assume:
`git clone --recurse-submodules URL`

## Extra tools
- https://git-scm.com/downloads/guis
- `git diff` vs `git difftool -y --tool=nvimdiff` vs `ydiff` vs `sdiff`
- VS code plugin
- GitHub GUI
- GitHub CLI (gh)
- (neo)vim
 - vim-fugitive
 - diff view https://github.com/sindrets/diffview.nvim
  - maybe learn to use its merge tool? looks pretty
 - gitsigns
  - showing who edited each line (git blame)
  - commit a hunk or selection (you have <leader>gs for git stage etc)

