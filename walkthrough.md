# Introduction to git collaboration
BioInfoSummer 2022
workflow instructions

https://bis.amsi.org.au/speakers/christian-madsen/


## Setup
- Windows: https://gitforwindows.org/
- Mac & Linux: Open your favourite terminal (I'll be using kitty)
  - `git` to see if installed. Install with your package manager, e.g. homebrew 
    for Mac https://brew.sh/
    See https://git-scm.com/downloads

## Make a playground project
```bash
mkdir myproject
cd myproject
git init
```
Congrats, it's a project.
You might get a message about default branch name which you can ignore for now 
or to use the same naming as implied here run e.g.
```bash
git config --global init.defaultBranch main
git branch -m main
```

```bash
ls -a
```
Concretely that means you made a hidden folder `.git/` with technical files 
that you should generally not edit directly.
```bash
git status
```
When you run any command it looks for a parent folder with `.git/`.

let's imagine we did some work:
```bash
touch experiment.log TODO.md notes.txt
url='https://www.ebi.ac.uk/thornton-srv/m-csa/media/flat_files/literature_pdb_residues.csv'
echo "wget $url" > download.sh
echo 'print("hello world")' > myscript.py
chmod +x download.sh myscript.py
python3 myscript.py > myrun.out
git status
```
git sees the files, git doesn't care.

## Staging
```bash
git add experiment.log
```
Look at `git status` after each command to see what we did. The file is staged, 
git has taken note of its **current** state.
```bash
echo 'Day 1: all experiments completed flawlessly on the first attempt.' > experiment.log
```
Git remembers the previous file state at the time where we called `add`. See 
the file change **overview** and **line-by-line** changes with (q to quit pagers):
```bash
git diff --stat
git diff experiment.log
```
By default `git diff` shows unstaged, uncommitted changes.  
You can always get short and long help for a command with `git diff -h` and 
`git diff --help` ([Getting help](#getting-help)).

```bash
git add -u
```
[u]pdate staging of files that are already tracked. Now that 
`experiment.log` is staged, if we wanted to see the changes again, we would have 
to use `git diff --staged experiment.log` instead.

```bash
mkdir -p src/ logs/ data/CSA/
git add .
```
Add files in the folder and subfolders. Notice that git only tracks files, not 
folders.  
You can track that `download.sh` is executable (`chmod +x`) with
```bash
git config --global core.filemode true
```

## Committing
We now want to save the changes to the project history. Note that we must stage 
those changes before committing, which we have. We always supply a [m]essage:
```bash
git commit -m 'Initial analysis.'
```
This command should **fail** if you haven't configured git before. Git takes note 
of the author for each commit, so as the error suggests, you can specify your 
email and name with
```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
Then rerun the commit command. If you rerun it once more, there is nothing in 
the staging area so nothing happens.

## Removing
Now that the files are in the history we can safely remove things and clean up 
without fear of losing work forever.
```bash
rm myscript.py
```
Oops, that wasn't the file we wanted to change. `git status` gives a few 
suggested commands, e.g. undo with `git restore`:
```bash
git restore myscript.py
rm notes.txt
git rm notes.txt
```
The initial `rm` call can also be omitted:
```bash
git rm TODO.md
```
In some cases we may want to delete a file from git but still keep the actual 
file locally (in the working tree), e.g. if it feels like unnecessary clutter for 
others to view.
```bash
git rm --cached myrun.out
git status
```
myrun.out is still listed under untracked. We can hide the untracked prints 
with [no] [u]ntracked
```bash
git status -uno
```
Or always ignore the filetype:
```bash
echo '*.out' >> .gitignore
# and on Mac .DS_Store while we're at it
echo '.DS_Store' >> .gitignore
git status
git add .gitignore
```

## Moving
We may want to rename or move files to reorganise
```bash
mv *.log logs/
```
Git cares about the folder `logs/` now that it has files.
```bash
git add logs/
```
Git thinks the moved/renamed file `experiment.log` is deleted.
```bash
git rm experiment.log
```
It now understands it was renamed.
Move/rename like above can be done with one command instead:
```bash
git mv myscript.py src/
```

## Code vs data storage

```
./download.sh
mv *.csv data/CSA/
git mv download.sh data/CSA/literature_pdb_residues.csv.sh
```
We're tracking the download script and not the actual datafile, since Git is not 
intended for data storage.
This approach also serves as simple history to take note of the raw data source.
> In this case the datafile is only a 124kb CSV so it would be fine to track.
> GitHub shows a warning for files larger than 50Mb, and blocks files larger than 100Mb 
> https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github
> GitHub offers the paid service Large File Storage.
> You could consider having a bare git repo for your lab set up on a server which 
> can easily be set up for anyone with ssh access.
> Git will also become slow if it contains **too many files**, e.g. if you run a 
> script that writes 100s or 1000s of files to a folder, consider tracking a zip 
> or tar of the folder, or other alternatives.
> Compressed files are **binary** rather than plaintext, which means git no longer 
> tracks **line-by-line** changes.

## Branching

Let's say this is a stable version of the project without bugs.
```bash
git commit -m "stable experiment 1"
```
Let's start some new untested work
```bash
echo "import numpy as " >> src/myscript.py
git add src/*.py
```
In the middle of this complicated new work we realise we want to run the stable 
version of the script for a different quick analysis but the script doesn't 
work in its current state if we run it. We quickly **stash** the changes.
```bash
git stash push src/myscript.py
python3 src/myscript.py
git stash pop
```
It might be a while before we are done working on the script so we move the 
work to a new branch.
A typical reason to branch is to keep different levels of **stability** separate, 
to have branches specific to writing new **parts** of the overall project, or even 
to have multiple parallel branches for **alternative** implementations to compare.
```bash
git branch dev
git switch dev
```
The staging area moves with us.  
We can add and commit in one go, when the file is already tracked:
```bash
git commit -m "untested new idea" src/myscript.py
cat src/myscript.py
```
We can now switch back to the stable version on the default branch.
We can call `git branch` to get an overview.
```bash
git branch # quit pager with q
git switch main
cat src/myscript.py
```
Now the file runs like before. Let's finish the new development.
```bash
git switch dev
sed 's/as /as np/' src/myscript.py > temp && mv temp src/myscript.py
```
We provide installation instructions in the top level folder:
```bash
echo 'numpy' > requirements.txt
echo '# My project

## Requirements
- pip, conda or other python package manager.

## Install
- python dependencies in requirements.txt, e.g. with `pip install -r 
  requirements.txt` or `conda install --file requirements.txt`.' > README.md
```
> Pip can often be installed simply with
> `python3 -m ensurepip --upgrade` or from https://bootstrap.pypa.io/get-pip.py
> and a minimal conda installation is on 
> https://github.com/conda-forge/miniforge.
> With numpy installed we can run the script and commit to it.

```bash
python3 src/myscript.py # optional
git add src/ README.md requirements.txt
git commit -m "numpy imported"
```

## Merging

Once we have finished developing we can merge the changes back into the stable 
main branch. By default git merges into your current branch, so we switch to main.

```bash
git switch main
```

We can get overview of the commits with `git log`.
We show a simpler overview with 1 line per commit, leaving out author and time. 
```bash
git log --oneline
git log --oneline --branches
```
We see an indication of HEAD which is where we are currently standing, i.e. a 
pointer to the current commit.
By adding `--branches` we list other branches, so the dev branch becomes 
visible as one step ahead of main. Since that's the case we can easily merge 
the work from dev into main.
```bash
git merge dev
```
It becomes a simple "**fast-forward**", which means we simply made the main branch 
point to one commit newer, and updated the working tree.  
Git can also easily handle multiple people editing a file, which we emulate by 
editing it in different branches.
```bash
# prepend a shebang
echo '#!/usr/bin/env python3' | cat - src/myscript.py > temp && mv temp src/myscript.py
git commit -m "shebang" src/
git switch dev
# append a comment
echo "# a comment" >> src/myscript.py
git commit -m "a comment" src/
git switch main
git merge dev -m "parallel editing is not scary"
cat src/myscript.py
```
We see git **automatically** added changes from both branches.
A **commit** and thus a [m]essage was necessary this time since an actual **merge** took place.
Let's imagine two different people each make changes to the end of the file:
```bash
echo 'import sys' >> src/myscript.py
git commit -m 'sys' src/
git switch dev
echo 'import os' >> src/myscript.py
git commit -m 'os' src/
git switch main
git merge dev
```
This time it didn't go so well. Auto-merge failed since both edits are at the **same** location in the file.
We could panic and call `git merge --abort` to undo the attempted merge or resolve the **conflict**.
```bash
cat src/myscript.py
```
Git has inserted 3 weird lines. We accept changes from both sides simply by deleting these lines.
```bash
grep -Ev '^<<<<<<<|^=======|^>>>>>>>' src/*.py > temp && mv temp src/*.py
cat src/*.py
git add src/
git commit -m "conflict resolution 101"
```
Often this merge conflict would be between you and another collaborator, but in 
this simple branching example we play both sides.
We can see the divergence and merging history with `--graph` on `git log`:
```bash
git log --oneline --branches --graph
```

## Rebasing
There is an alternative to merge, called rebase.
```bash
echo 'infile = sys.argv[1]' >> src/myscript.py
git commit -m "argv" src/
git switch dev
echo $'import pandas as pd\ndf = pd.read_table(infile)' >> src/myscript.py
git commit -m "pandas" src/
git rebase main
```
We have made a change to both branches. Earlier we merged into main. Now we are 
rebasing dev onto main, which means we attempt to "**replay**" applying the changes 
from dev (since divergence) on top of the changes in main (since divergence). 
Just like with merge, this can cause a **conflict** if the changes are at the *same* 
place.
We accept both again:
```bash
grep -Ev '^<<<<<<<|^=======|^>>>>>>>' src/*.py > temp && mv temp src/*.py
git add src/
git rebase --continue # close editor with :q if vim
git log --oneline --graph --branches
```
We now see the "pandas" commit has been applied on top of the "argv" commit.
Since we decided to combine the branches inside *dev* we have to do another step 
to update *main*. We can call `git merge` since it will just be a **fast-forward**.
```bash
git switch main
git merge dev
```
Merging instead of rebasing would have resulted in the same `src/myscript.py` but 
the log is cleaner. Basically, merging shows what really **happened** and rebasing 
**rewrites** history.
> :warning: That can be dangerous if you collaborate with others and they 
work based on history that no longer exists.

Another powerful use of rebase is **interactive** rebasing, where e.g.
`git rebase -i HEAD~4` lets you interatively select what to do with the last 4 
commits, e.g. drop one, combine (squash) two of them, edit another.
(`:cq` to cancel an interactive rebase in vim).

## Jumping around

We can **delete** branches no longer in use.
```bash
git branch -d dev
```
Note that a **branch** is simply a commit reference, that **moves** when a new 
commit is added on top. When the branch is deleted, all its history is not simply gone,
it can still be accessed, e.g. by the commit hash listed in the `git log`.
> Although if a branch is deleted that was ahead of all other branches, its 
commits will be orphans and will be deleted periodically by garbage collection.

We can switch back to where the dev branch has been with `git checkout 
<COMMIT HASH>` by finding the commit hash of e.g. the commit with name "os".
Since we know the commit message we can also use it directly 
with a reverse search:
```bash
git checkout ':/os' # or git switch ':/os' --detach
git log --oneline --graph --branches
```
We see that HEAD has moved and get a message about detached HEAD, since it is 
not currently on a branch.
So, we can easily move around in history, but let's go back to main.
```bash
git checkout main
```
We've used both `git checkout` and `git switch` to jump between commits. 
There's really no difference. Both functions exist because `git checkout` has 
been confusing for new users, as it also covers restoring old versions of 
specific files, so two new (currently experimental) functions were introduced: 
`git switch` and `git restore` (used above).

## Undoing
Let's try out similarly named functions `git revert` and `git reset`, which 
reverts commits (making a new commit), and resets your branch, respectively. 
Let's say we consider undoing the shebang. We could first remind ourselves what 
was done in that commit with `git show 0df52c8` where "0df52c8" should 
be the abbreviated commit hash. Press q to quit the pager and h for help. We 
revert with
```bash
git revert -n ':/shebang'
```
`-n` for [n]o commit, which means we can review the revert before committing.
We decide to go through with the revert and call
`git revert --continue --no-edit` where `--no-edit` means the message for the 
new commit is autogenerated. We realize the revert was a mistake, and should 
have called `git revert --abort` but it's too late. The revert commit was the 
latest commit, so we just need to reset to the commit before HEAD, i.e. its 
parent, which can be written as `HEAD~1` (or just `HEAD~`). We are 
not sure how aggresively to reset so we call
```bash
git reset --soft HEAD~
git status
```
With `--soft` we only reset the commit itself, which means the staging area 
contains the changes that were done by the revert, which is a deletion since 
the reverted commit was adding a shebang line.
We wanted to not only reset the commit (`--soft`),
but also the staging area (the default, explicit with `--mixed`),
and also the actual files (`--hard`).
So we undo the call `git reset --soft HEAD~` by calling
```bash
git reset --hard HEAD@{1}
```
Then undoing the revert by calling
```bash
git reset --hard HEAD~
cat src/myscript.py
```
The `HEAD@{1}` syntax means "where HEAD used to be 1 move ago" which will often 
be equivalent to `HEAD~1`. The raw history that `HEAD@{...}` refers to can be 
seen with `git log` by adding the flag `-g` or using `git reflog`.


## Editing
Let's add some more hypothetical work.
```bash
infile=literature_pdb_residues.csv 
rename_cmd='PDB ID,PDB,CHAIN ID,chain,RESIDUE NUMBER,resi'
echo "mlr --c2t --from $infile rename '$rename_cmd' + uniq -f resi,PDB,chain > CSA_sites.tsv" > data/CSA/CSA_sites.tsv.sh
chmod +x data/CSA/CSA_sites.tsv.sh
echo "- miller (https://miller.readthedocs.io/en/latest/installing-miller/)" >> README.md
git add data/CSA/*.sh README.md
```
So far we have been editing with shell commands like `echo` and `sed`, but of 
course we should write in a normal editor.

Let's commit without specifying a message with the `-m` option.
```bash
git commit
```
What happened now is git opened the default text editor so you can write the 
commit message here instead. What we have written with `-m` so far is the 
commit title that is written here on the first line. The next line should be 
left empty and then extra lines can describe the commit with more detail.
The default editor is usually `vim` (exit with `:q`) or `nano`, depending on 
git version and if env variables `$EDITOR` or `$VISUAL` are set.
Set your favourite editor with `git config --global core.editor <EDITOR>`.
`<EDITOR>` examples:
- `nano`: simple preinstalled terminal editor
- `micro`: easy-to-use, not preinstalled (https://github.com/zyedidia/micro)
- `vim`: standard modular terminal editor
- `neovim`: modern reimplementation of vim
- `emacs`: extensive non-modular terminal editor
- `'open -nWa TextEdit'`: Mac GUI text editor (finish commit edit with save then CMD+q to fully quit)

## Configuring git
Git's own config can also be edited with the chosen editor with `git config --global --edit`.
So far we have used `--global` but the alternative is **project local** settings 
which **overrule** global settings.
- **Local** in `<PROJECT ROOT>/.git/config`
- **Global** in `~/.gitconfig` or `$XDG_CONFIG_HOME/git/config` where 
  `$XDG_CONFIG_HOME` == `~/.config` by default.
- List all: `git config --list`
> - Config contains your **aliases**, which are useful shortcuts, e.g.
>   - `git config --global alias.st status` to write `git st` instead of `git status`
>   - `git config --global alias.unstage 'restore --staged'`
>   - Run **arbitrary** shell commands, e.g. resolve conflict with `git both`  
>     `!"f(){ grep -Ev '^<<<<<<<|^=======|^>>>>>>>' $1 > git.tmp && mv git.tmp $1; git add $1; }; f"`
>   - Your **shell** can also do shortcuts, e.g. `alias g=git`

## GitHub
Let's make a project copy that we can share. GitHub is standard for open source 
code sharing, but there are alternatives like BitBucket, GitLab, SourceForge, Codeberg...

Sign in or sign up:  
https://github.com/

Generate ssh keys if you have not already with `ssh-keygen`. A one-liner without prompts:
```bash
ssh-keygen -t ed25519 -f $HOME/.ssh/id_ed25519 -N "" # [t]ype, [f]ile, [N]ew passphrase
cat ~/.ssh/id_ed25519.pub
```
> :warning: This creates a private key in **plaintext**, which means if someone gains access to 
> your computer they can access your GitHub account.
> If you set a non-empty passphrase for added security you will have to type it 
> on each connection unless you manage it with "ssh-agent", in which case you type it once per session.
> You can additionally manage it with "Keychain Access" on Mac or similar tools on other OSs to never type it.
> https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

Copy the public ssh key to your github account at  
https://github.com/settings/keys  
Now your GitHub account will trust your computer over ssh.

Make a repository on  
https://github.com/new

When you decided on a name and private/public GitHub is very helpful and tells you what to do next.
Since we already have a project we need to push it to the remote, rather than `git clone` the remote.
We add a remote that we name "origin" that points to the GitHub repo with an SSH link 
("git@github.com:" instead of "https://github.com/").
```bash
git remote add origin git@github.com:USERNAME/REPO_NAME.git
git remote -v # [v]erbose
```
We copy our local project to GitHub, and also add the remote (we can have 
multiple) as [u]p-stream, 
which means we only need to write `git push` afterwards.
```bash
git push -u origin main
git log --oneline --graph --branches
```
Now the origin/main branch should be visible as well in the log.

Invite others to collaborate on your repo under settings → collaborators
https://github.com/USERNAME/REPO/settings/access
You can add me: "degnbol"
If you get someone to add you, you can then clone their project and start editing it. 
Otherwise use your own project in a different location.
```bash
git clone git@github.com:USERNAME/REPO_NAME.git
cd REPO_NAME
touch imInYourRepo.hi
git add imInYourRepo.hi
git commit -m "hi"
git push
```
After someone you invited edits your repo or you edit it in another location 
you can go back to the original and update it with the new changes.
```bash
git pull
```
`git pull` is `git fetch` then `git merge`. `git fetch` downloads the remote 
branch into origin/main, then `git merge` merges that into your local main. 
This is where merge conflicts may arise which we looked at earlier ([Merging](#merging)).

## Using work done by others
Sometimes you find a paper with an associated GitHub repo and want to download 
and use their work in your own. You could `git clone` (or just download)
- *outside* your repo and mention dependence in `README.md`.
  - Requires additional install step.
  - Assumes correct path.
- *inside* repo.
  - Git only tracks folder name.
  - Hide with `.gitignore`.

Cool options:
- ***Subtree***: basically git clone inside repo that your own repo understands.
  - Adds all files to your repo, git tracks changes.
- ***Submodule***: git only records a reference to the dependency (commit hash)
  - Best if you are **not** going to edit the dependency.

```bash
mkdir external
git subtree add -P external/git-good/ https://github.com/degnbol/git-good main
cd external
```
We can now easily edit files in "git-good" but if it was a large repo it would 
have been a lot to add.

Let's say we want to use Tudesco et al. work for calculating hypergraph centrality.
```bash
git submodule add https://github.com/ftudisco/node-edge-hypergraph-centrality
echo 'include("../external/node-edge-hypergraph-centrality/centrality_tools.jl")' > ../src/HG_cent.jl
cd -
# add Julia to README.md requirements:
vim README.md +/Requirements +'normal }O- Julia' +wq
cat README.md
git add src/
git commit -m "initial HG work"
git push
```

Now, anyone cloning your repo will be able to run `julia src/HG_cent.jl` if 
they used `--recurse-submodules` option for `git clone`.
You can be on the safe side with
```bash
echo 'git submodule update --init' >> install.sh
echo '- `./install.sh`' >> README.md
chmod +x install.sh
```
Which means calling `./install.sh` will make sure to download missing submodules.

---------------------------------------------------------------------------------
## Extras

### Paths
Now that we are collaborating with others it will be problematic to use absolute paths, 
i.e. `/Users/ME/myproject/install.sh`
Let's say we want to use some of the `src/` code in the work in `data/`

Relative path:
```bash
# cd PROTECT_ROOT
echo 'src/myscript.py data/CSA/CSA_sites.tsv.sh' > data/CSA/myanalysis.sh
chmod +x data/CSA/myanalysis.sh src/myscript.py
```
Assumes it's run from **project root**: `./data/CSA/myanalysis.sh`.

```bash
cd data/CSA/
echo '../../src/myscript.py CSA_sites.tsv.sh' > myanalysis.sh
```
Assumes it's run from the relevant **working directory**: `./myanalysis.sh`.
We can make it runnable from anywhere, e.g.
```bash
echo $'#!/usr/bin/env zsh\ncd $0:h' | cat - myanalysis.sh > temp && mv temp myanalysis.sh
cat myanalysis.sh
```

We can also write paths relative to project root, which assumes we don't `cd` out 
of the project before running things.
```bash
git config --global alias.root 'rev-parse --show-toplevel'
git root # alias prints project root
sed 's/.*src/`git root`\/src/' myanalysis.sh > temp && mv temp myanalysis.sh
cat myanalysis.sh
```
`git root` equivalent in different languages:
- Python: https://github.com/maxnoe/python-gitpath
  - install: `pip install git+https://github.com/maxnoe/python-gitpath`
- R: `here()` https://here.r-lib.org/
  - install (in R):
    ```r
    install.packages("here")
    ```
- Julia:
    ```julia
    readchomp(`git root`)
    ```

And there are of course many more options, such as setting an environment 
variable `export MYPROJECT=/path/to/myproject` and using that in scripts etc.


### Extra tools

GitHub CLI https://cli.github.com/

There are many alternative tools for each part of git, e.g. diff and merging.
`git diff` alternatives:
- with `--word-diff` option.
- `git difftool --tool-help` to list tools. I alias `git difftool -y --tool=nvimdiff`
- `ydiff -s -w0 README.md` (https://github.com/ymattw/ydiff)
- Custom diff script, e.g. sdiff
  ```bash
  echo 'sdiff "$2" "$5"' > mydiff.sh
  chmod +x mydiff.sh
  git config --global diff.external $PWD/mydiff.sh
  git diff README.md
  ```
- GUIs and plugins

Fuzzy finder (https://github.com/junegunn/fzf) use with git commands, e.g. alias
`git ls-files -m | fzf --multi --height 10 | xargs git add`

For more fine-grained control, git allows you to operate on parts of files, 
with so-called hunks, which are consecutive changed lines. E.g. edit file two 
places and call `git add --patch FILENAME`. Editors make this fine control 
nicer, e.g. in neovim with plugin gitsigns (https://github.com/lewis6991/gitsigns.nvim)
- add/commit a hunk or selection
- showing who edited each line (git blame)
Other popular (neo)vim plugins:
- vim-fugitive https://github.com/tpope/vim-fugitive
- diff view https://github.com/sindrets/diffview.nvim

### Getting help
We can get a short list of options with `git reset -h` and open a detailed 
description with any of `git reset --help`, `git help reset`, `man git-reset`
(`man` can be colored by zsh plugin https://github.com/ael-code/zsh-colored-man-pages).
The same naturally goes for any other git function.
Shorter help pages: `tldr git-reset` (https://github.com/tldr-pages/tldr)
Short examples: `eg git` (https://github.com/srsudar/eg)
Other ways of getting help are of course Google, StackOverflow, git-scm.com, etc.
There is also a free ebook "Pro Git".

### Tab-completion
Another way to get help is with tab-completion.
It might already work, try 
pressing `git <TAB>+<TAB>`.
- Bash
  - install bash-completion:
    `brew OR apt OR yum OR ... install bash-completion`
    Source the downloaded file on startup (`~/.bashrc`), e.g. `. /path/to/bash_completion`
      - homebrew: /opt/homebrew/etc/profile.d/bash_completion.sh
      - apt: /etc/bash_completion or /usr/share/bash-completion/bash_completion
      - otherwise find it: `sudo find /etc /usr /opt -name bash_completion`
      - If this doesn't work then download and source 
        https://github.com/git/git/blob/master/contrib/completion/git-completion.bash
- Zsh
  - To install, just make sure general completion is enabled: run on startup (`~/.zshrc`):
    `autoload -Uz compinit && compinit`
  - has option completion (try `git status -<TAB>`)

### Notes
- There is a lot of git not covered here, e.g. amending (`git checkout --amend`), and
  checking out multiple branches at the same time with `git worktree`.
- Overleaf is git tracked, so you can clone and edit a shared overleaf 
  document locally in your favourite editor.
- Track your git config itself with git in "dotfiles" https://dotfiles.github.io/  
  Inspiration from mine: https://github.com/degnbol/dotfiles/tree/main/config/git
- Line-by-line changes are not tracked in binary files but custom diff can be 
  configured, e.g. to see main text and comment changes in a pdf (see my dotfiles).

