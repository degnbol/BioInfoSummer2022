# Presenter notes

## Intro slides

## Walkthrough
https://github.com/degnbol/BioInfoSummer2022/blob/main/walkthrough.md

## Extra time
Open e.g. README.md with :set number
- gitsigns (GUIs have similar)
- git blame. <leader>gb
- move between hunks. ]h [h
- show diff (default to HEAD). <leader>gd
- reset hunk. <leader>gr. Undo this with u
- stage a hunk. <leader>gs
- undo stage. <leader>gu
- stage selection. visual + <leader>gs
- Stage file with :G stage %
- Commit with :G commit -m "fugitive" (or no message)
- :G push

Merge tool:
```bash
git reset --hard ':/conflict'
git reset --hard HEAD~
git checkout ':/os'
git switch -c dev
git switch main
git merge dev
vi src/myscript.py +'DiffviewOpen -uno' +DiffviewToggleFiles
git merge --abort
git pull
```

