# Stacked PRs on git and github

Notes to myself about doing stacked PRs on github.

Initially, I was going to make my own CLI tool to handle a lot of the repetitive
tasks for keeping branches and PRs up-to-date.  But I did some googling and
found `git machete` which is most of the way there.

## `git machete`

`git machete` uses a single config file describing a DAG of branches.  Each
branch has an upstream.  In the typical workflow, all topic branches have the
same upstream: `main` or `master`.  But with stacked PRs, the first branch in a
stack is the upstream of the second, and so on.  The DAG is the easiest way to
describe this relationship.

Example git machete config file:
```
main
    pr-stack/1
        pr-stack/2
```

Git machete understands that any changes to `main` should be merged/rebased into `pr-stack/1`,
and then those changes to `pr-stack/1` should be merged/rebased into `pr-stack/2`.

It will (interactively) do all the necessary merging/rebasing and pushing for
all branches in the DAG.

It can also create and manage github PRs.  If you change the config file, it can update
the upstream branch for each PR.  From the example above, if `pr-stack/1` is merged to `main`, then you
want to change the upstream of `pr-stack/2`'s PR to `main`.

Quick-reference of commands:

```shell
# Discover branches, make a best-guess config file
git machete discover

# Add new branch to config
git machete add <branch name>

# Open the config file
code $(git machete file)

# Merge upstream changes into all branches in the tree
git machete traverse --merge --no-edit-merge -w

# Show the configured upstream for a given branch
git machete show up <branch name>

# If PRs already exist, discover and add their IDs to config file
git machete github anno-prs

# Create PR
git machete github create-pr

# set the upstream branch on an existing PR, if config changes
git machete github retarget-pr
```

## Older notes about building my own tooling

```
# Verify that swapping to other branches will succeed
git fetch
git checkout stacked-pr/branch1
git merge --no-ff origin/main
git checkout stacked-pr/branch2
git merge --no-ff stacked-pr/branch1

git stacked-prs merge-upstream
git stacked-prs push
git stacked-prs create # create branches with empty commits, push them, create GH PRs

git stacked-prs create <comma-separated PR list>

git stacked-prs update # fixup PRs to set their merge targets, after a branch has been removed from the stack


git config --set pr-stack.my-feature.


gh pr list --search 'head:name base:name'
```