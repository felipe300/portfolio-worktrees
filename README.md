# Git Worktrees

[Git Worktrees Crash Course](https://www.youtube.com/playlist?list=PL4cUxeGkcC9iUtQh7Aja3TGfbdd7Z-K0W) is a course from [Net Ninja](https://www.youtube.com/@NetNinja), here is the [starter repo](https://github.com/iamshaunjp/portfolio-worktrees).

Think of **Git Worktree** as a way to "checkout" multiple branches of the same repository at the same time into different folders.

Normally, a Git repository has **one working tree** (the files you see and edit). When you switch branches, Git swaps the files in that single folder. **Git Worktree** breaks this limitation by allowing you to have multiple linked working trees attached to a single repository.

**Why use it?**

- **Context Switching**: If you are in the middle of a massive refactor and a "hotfix" request comes in, you don't need to stash or commit unfinished work. You just create a new worktree in a different folder, fix the bug, and delete the folder when done.

- **Comparing Branches**: You can open two instances of your IDE side-by-side to compare how a specific logic behaves in `production` vs `development`.

- **Long-running tasks**: You can leave a heavy test suite or a build process running in one worktree while you continue working in another.

```sh
git init --bare ./.git # initialize a bare repo

git worktree add ../[FOLDER_NAME] [NEW_BRANCH_NAME] # create a new worktree, this will use the current `HEAD` as a starting point
git worktree add -b [NEW_BRANCH_NAME] ../[FOLDER_NAME] # create a new worktree, this will use the current `HEAD` as a starting point
git worktree add -b [NEW_BRANCH_NAME] ../[FOLDER_NAME] [STARTING_BRANCH] # create a new worktree, `main` branch as a starting point

git worktree list # list all worktree directories

# To remove a worktree, giving the path to the worktree itself.
# The changes must be added and committed. OR you can use `--force` flag at the end
# Be careful, because the branch will not be deleted, so you have to delete it manually
git worktree remove ../[FOLDER_NAME]
git branch -d [NEW_BRANCH_NAME]
```

You will get a "Main directory" (`git_worktrees`), then inside you will get your branches (`portfolio-worktrees/main`, `feature-a`).

```sh
mkdir git_worktrees && cd $_
git clone https://github.com/iamshaunjp/portfolio-worktrees.git
git worktree add -b feature-a ../feature-a
cd ../

git_worktrees
❯ tree
.
├── feature-a
│   ├── index.html
│   ├── README.md
│   ├── styles.css
│   └── welcome.html
└── portfolio-worktrees
    ├── index.html
    ├── README.md
    ├── styles.css
    └── welcome.html
```

Make a change in one of the `README` files, then do `git status` in both to check the changes.

> [!IMPORTANT]
> Delete these branches and the projects
> `rm -rf portfolio-worktrees`
> `rm -rf feature-a`
> later we will use `Bare repositories`

**Bare Repositories**

A **Bare Repository** is a Git repository created without a Working Tree.

When you run a standard `git init`, Git creates a storage area (the `.git` folder) and extracts the project files so you can edit them. In a Bare repository, the files you would normally find inside the `.git` folder are instead placed directly in the main directory, and no checkout of the working files is created.

Why use it?

Bare repositories are designed to be Central Repositories. You should never work directly inside them. Instead, they act as a "hub" where developers `push` their code and `pull` updates.

Main advantages:

- **Push-safe**: Since there is no working tree, there is no risk of someone’s uncommitted changes being overwritten by a `git push`.
- **Server-side standard**: Services like GitHub, GitLab, or Bitbucket use bare repositories on their servers to manage your code.
- **Efficiency**: It only stores the administrative data and the compressed object database, making it ideal for backup and synchronization.

| Feature                   | Standard Repo      | Bare Repo             |
| ------------------------- | ------------------ | --------------------- |
| Contains editable files?  | Yes                | No                    |
| Has a `.git` folder?      | Yes (hidden)       | No (it is the folder) |
| Purpose                   | Development/Coding | Sharing/Hosting       |
| Can you `git push` to it? | Not recommended    | Yes (expected)        |

```sh
git clone https://[GIT_URL].git --bare [DESTINATION_FILE]
git clone https://XXX.git --bare .git
ls -la # to see the hidden `.git` folder
git status # these command this thrown an error
cd .git
ls -la
>  branches   config   description   FETCH_HEAD   HEAD   hooks   info   objects   packed-refs   refs
```

It is recommended that the first "tree" you create is a "main" tree as a reference. But you shouldn't work in that directory directly, instead you will use new Work trees to do that. Most of the time you will use "main tree" to push or pull changes.

```bash
# In this case you will have already a bare repository
ls -la
> .git
git worktree add ./main
git worktree add -b feature-abc  ./feature-abc
# cd into feature and make a change to any file
git status # list the current changes
cd ../
git worktree add -b hotfix ./hotfix
git worktree list
# cd into hotfix, make a change and commit it, all from inside the directory "hotfix".
git push origin hotfix
# merge changes and delete remote branch
cd ../
git worktree remove hotfix
# cd into main to updates changes (this case pull from remote main)
git pull
```
