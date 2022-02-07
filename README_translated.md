I recently worked on a group project with colleagues who had experience of using Git and GitHub for solo projects, but not for collaboration. This is a compilation of an introduction and related tips I wrote to help them get started, especially with Git branches as well as GitHub issues and pull requests. My colleagues asked great questions, which are included for context. 

## Branches, general introduction
> Could you write a short Git/Github guide? For example, I don't know how to use branches at all, and it feels important to learn now that we are so many working on the same repo.

The most important commands to start with are:

* `git branch foo` create new branch locally
* `git checkout foo` switch to specified branch (ie switch from, for example, __main__ to __foo__)

Once you are on a branch, everything you do will be connected only to that branch. If I do `git add .` and` git commit -m "msg"`, then only the branch I am on will be affected. If I then switch over, `git checkout main`, I'll notice that all files are 'rolled back' to the state/contents they were in before I switched to foo.

It's good to know that all the different 'states'/versions of files are actually stored in the '.git' folder, there's nothing magical about it, which is why it's important to never delete the '.git' folder.

When I've worked for a while in foo and I'm satisfied with the results, so that I feel it's time to put (merge) them into the main branch, I can do this in two different ways.

** The first way: GitHub first **
Do `git push origin foo`. This creates a new branch at 'origin' (ie GitHub in our case), and uploads the state/commits of the _local_ __foo__ branch to the _remote_ (GitHub) __foo__ branch. At this time, the main branch hasn't been affected at all.

If I now open my web browser and go to the repository on github.com, I can see that there are two branches: __main__ and __foo__. If I look closer at __foo__, I see that it 'is ahead' with x number of commits compared to main. If I click on 'Contribute' (near the top of the page) I can then choose to do a _Pull Request_ (PR).

With my PR I 'ask for permission' to bring in commits from __foo__ to __main__. It provides a good opportunity to get an overview of what changes have been made before merging them, making them more 'official'.

Once the PR has been approved and changes have been merged into __main__, you can run `git checkout main` and` git pull origin main` locally for the changes to reach your local copy / repo.

I would recommend that we usually stick to the first way.

** The other way: Merge locally **
Run `git checkout main` and then` git merge --ff-only foo`. With this, changes from __foo__ are directly merged into the local __main__ branch. You can then push to GitHub with `git push origin main`, skipping the whole PR process. 


## Prevent issues related to divergent branches
> I did a git pull, opened the directory in Visual Studio Code and found that a file was marked in yellow (the one that colleague X had edited). Still, I proceeded to work on the repo, ran `git add` and then `git push`, but Git/GitHub wouldn't let me do the push. Any recommendations?

I don't know exactly what went wrong in this case, but here are sugggestions on what you can try to do to prevent problems, and debug them when they occur:

`git branch` (ie without any arguments): Check which branch you are on and which others you have locally. The active branch is marked with an asterisk (and often in green, depending on the command line interface you're using, I think).

`git status`: Check which branch you are on, if you have any files that haven't been staged (ie not added with` git add`) what has been staged (ie has been `git add`ed but not` git commit`ed ), and how one's own branch status is compared to the connected 'remote' (Github) branch. For example:
```bash
> git status
On branch main
Your branch is up to date with 'origin / main'.
```

This means that I'm on the main branch locally, and it's linked to the 'remote' (Github) branch __main__. I have no local commits that have not been sent to remote yet, and _as far as my local Git knows_ the remote branch does not have any commits that my local repo lacks. If Git had said something like `your branch is behind origin / main by 3 commits and can be fast-forwarded` then I could have simply run (after double checking with` git fetch`, see below) `git pull origin main --ff-only` to 'catch up' with the GitHub branch in my local repo branch. If it had said `is behind ...` and _did not_ end with `can be fast-forwarded`, then there probably would have been a conflict between local commits and GitHub commits. In that case, things become more complicated, and you'll want to search for more information on how to solve this.

`git fetch origin foo`: Retrieve updated information about which commits the remote branch (__origin/foo__, ie the __foo__ branch on GitHub) has. Now run `git status` again - it may turn out that there are commits on GitHub that your local Git didn't know about. **As a rule, it's a good habit to run `git fetch` before` git status` to be sure that you do not miss any commits on GitHub**.

Before `git pull` or` git push`, run `git fetch` and` git status`. And instead of 'just' `git pull`, run` git pull origin <branch> `, where` <branch> `is the name of the 'GitHub branch' from which you want to retrieve data. With `git status`, you see that you are on the right branch locally, and that there are no 'missing' commits locally (and/or conflicts). With `git pull / push origin <branch>` you ensure that you interact with the correct branch on GitHub (otherwise Git assumes that you want to do things with the local branch's connected 'remote' branch, ie the one it says in `git status` output, which may not be what you want). 

## Example of work flow: working on branch locally, push to GitHub, and start PR
> At the moment, we only have a 'main' branch for the GitHub repository I and (name) are working on.
> So, I run `git pull origin main`, to make sure that I have the latest commits from GitHub. Then, I create a git branch __tables__ (example) where I work on database table-related things. I can then commit several times (and create tags, right?), and finally when I'm happy with the code I can merge them.
> To make it simple then, I can pull the branch, continue to work on it, and push commits. And when I'm satisfied, I can merge the commits.

Yes, so in detail:

In the beginning you have
* The branch __main__ on GitHub
* The branch __main__ in your local repo (if you've eg run `git clone`)

When you run `git pull origin main` locally, you ensure that your local __main__ branch has the very latest commits from the GitHub branch.

You run `git branch tables`. This new branch now shares commit history with your local 'main' (and thus also 'main' on GitHub of course, since you synced them with `git pull`).

You run `git checkout tables` to switch to your local __tables__ branch. (can be checked with `git branch`)

Say you modify a file 'foo.txt'. If you run `git status` at this point you will see that you 1) are on the local branch __tables__ 2) that __tables__ is not connected to any remote (GitHub) branch yet, and 3) there is a change in 'foo.txt 'that is not in the staging area.

You run `git add foo.txt`. If you then run `git status` you'll notice that 'foo.txt' has been added to the staging area, but not been committed yet.

You run `git commit -m 'add foo.txt'`. Running `git status` will inform you 'nothing to commit, working tree clean', as there are no 'non-staged' or 'staged but not committed' changes.

You run `git push origin tables`. This creates a __tables__ branch on GitHub that shares commit history with the local __tables__ branch.

Via GitHub's web interface, you go to your repo, switch to the __tables__ branch, and click the button for starting a PR (near the top).

You approve the PR on GitHub. __main__ on GitHub has now been 'fast forwarded' in commit history so that it has caught up with __tables__.

Back in your CLI you run `git checkout main`. This local branch knows nothing about what happened in __tables__ or on GitHub (as can be seen with `git status`/`git log`). However, if you run `git fetch origin main` and then `git status`, the local repository will be made aware of the new commits on GitHub. If you want to make the local __main__ branch 'catch up', run `git pull origin main`.


## Name branches and being consistent about commits
> I did a branch the other day (!!!) and named it __tables__. But to be honest, it didn't turn out quite as clean as I had in mind - I made a lot of other changes, not related to database tables. Do you have any practical recommendations on how to name branches and what to do when you need to correct several (disparate) things at the same time in parallel?

I also find it difficult to name branches, and I think it is extremely common to fix various small things you come across while working on a branch. It's a good thing if you can solve problems when you notice them like this, instead of forgetting. But the branch name isn't very important, in my opinion. Once you've done a PR and merged with __main__, you'll often delete the 'development' branch ayway, and its branch name will only be visible in the commit message linked to the PR. In other words, you will almost never see it afterwards. I use the branch name mostly as a reminder to myself of what I am working on. If I create a branch __feature/login__, for example, if I lose track or go too far off on a tangent, I can write `git branch` and remind myself 'oh right, my main goal is implementing login functionality'. I should mention though, that if you work in a really large team you might want to be more strict about what parts of the code you're working on, to avoid any conflicts with colleagues' code.

Prefixes like those in 'feature/login' or 'debug/login' can help you keep track of whether you're working on something new, or on fixing something that already exists.

In my opinion, it's more important to try to commit often and with clear commit messages. I always write in imperative, which is the convention for git as far as I understand. This helps me keep messages consistent. As an example, I'd write `git commit -m 'add scooter PUT route and corresponding controller'`, instead of ` ... 'added scooter PUT ...`. But the most important thing is to try to commit fairly often and, if possible, with a concise and descriptive commit message (not, say, `... -m 'fix CSS for login, registration, customer overview views, fix stuff in routing, add Customer service with a bunch of functions ... '`).
