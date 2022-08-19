# Some GraphQL studies and other git related stuff

[Git PRO related site](https://about.gitlab.com/blog/2018/06/07/keeping-git-commit-history-clean/)

## Git commits
Git commits are the best friends of developers, either they are working with others or not.
Git commits must follow a pattern and must be "atomic".

Some rules of thumb:
- Separate subject from body with a blank line
- Limit the subject line to 50 characters
- Capitalize the subject line
- Do not end the subject line with a period
- Use the imperative mood in the subject line (clean your room, close the door, take out the trash)
    ```txt
    if applied, this commit will ________
    Examples:
    if applied, this commit will clean your room
    if applied, this commit will close the door
    if applied, this commit will take out the trash
    ```
    Obs 👉🏻: Remember: Use of the imperative is important only in the subject line. You can relax this restriction when you’re writing the body.
- Wrap the body at 72 characters
- Use the body to explain what and why vs. how

## How to git rebase
First, verify your `git logs`:
```bash
git log --all --decorate --oneline --graph 
```

Then, you need to select the commit ID right before the commit you want to change.
<img src="./docs/img/GitRebase.png" align="center">
<caption>In the image above, we want to change the commit "Page Navigation View", then we need to rebase to the commit "Add routes for navigation" which is the commit right before.</caption>

After that, do:
```bash
git rebase -i <commit-id>
```
This will launch your default editor (vi/vim/nano) and you'll be presented with some options that look like the ones below:
```bash
pick 4155df1cdc7 Page Navigation View
pick c22a3fa0c5c Render navigation partial
pick aa0a35a867e Add styles for navigation

# Rebase 8d74af10294..aa0a35a867e onto 8d74af10294 (3 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove Git commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```
Change from "pick" to "edit" for the commit you want to change. In our case, it'd be:
```bash
pick 4155df1cdc7 Page Navigation View  -> edit 4155df1cdc7 Page Navigation View
```

<quote>Tip 💡: In `vi` you can edit a file by pressing `i`. When you've edited what you wanted, press `ESC`. To quit and save the file, `wq`.</quote>

After that,
```bash
git add .
git rebase --continue
```

It'll ask for some merge changes. Select the ones you want to maintain (fix merge issues) and then keep doing `git rebase --continue` until you find yourself at the HEAD.
When you're there:
```bash
git push --force-with-lease
```

## How to git squash/fixup
It's mainly used to combine commits.

This is usual when we do a lot of "fix" git commit messages and then it becomes a mess.
Start by calling our well-known `git rebase -i` to some commit-id and then change from 
"pick" to either "squash" or "fixup".

The difference between them are:
- `squash`: keeps the gix fix commit messages in the description.
- `fixup`: forget the commit messages of the fixes and keep the original.

First get the commit-id you want to rebase to:
```bash
 git log --all --decorate --oneline --graph
 git rebase -i <commit-id>
```
Example:
```txt
pick 4155df1cdc7 Page Navigation View
pick c22a3fa0c5c Render navigation partial
pick aa0a35a867e Add styles for navigation
pick 62e858a322 Fix a typo
pick 5c25eb48c8 Ops another fix
pick 7f0718efe9 Fix 2
pick f0ffc19ef7 Argh Another fix!
```

Now if you wanna combine the fixes into `c22a3fa0c5c Render navigation partial`, you should:
- move the fixes up so that they are right below the commit you want to keep in the end;
- change `pick` to `squash` or `fixup` for each of the fixes.

And then you'll end up with something like:
```txt
pick 4155df1cdc7 Page Navigation View
pick c22a3fa0c5c Render navigation partial
fixup 62e858a322 Fix a typo
fixup 5c25eb48c8 Ops another fix
fixup 7f0718efe9 Fix 2
fixup f0ffc19ef7 Argh Another fix!
pick aa0a35a867e Add styles for navigation
```

Save the changes and then `git push --force-with-lease`.

You'll end up with:
```txt
pick 4155df1cdc7 Page Navigation View
pick 96373c0bcf Render navigation partial
pick aa0a35a867e Add styles for navigation
```

## Restarting a git commit history
Whenever we're working on, usually, a big project, there are some times that our git commit messages don't
make sense.
If you want to start from scratch while maintaning all your code, you can create a `patch`.

Basically a `patch` is a box containing all your code, but that can be transported to a new location.

Imagine you have a branch called `add-page-navigation`. Let's see the step-by-step to do that:

1) Making sure your branch has all the changes present in the `master` and has no conflict with the same.
    - `git rebase master` while you're in `add-page-navigation` to get all the changes from `master` to your branch;

2) Creating the `patch` file:
    ```bash
    git diff master add-page-navigation > ~/add_page_navigation.patch
    ```
    Once the command is run and you don't see any errors, the patch file is generated.

3) Deleting the `add-page-navigation` branch:
    - check out to the `master` branch: `git checkout master`;
    - delete the `add-page-navigation` branch: `git branch -D add-page-navigation`;

4) Creating a new branch with the same name:
    - while in the `master` branch, run `git checkout -b add-page-navigation`.
    At this point this is a fresh new branch with no differences from `master` branch

5) Finally, apply your changes from the patch file:
    - while in the new created branch, run `git apply ~/add_page_navigation.patch`.

6) Now commit each part of the code as you want.
    
    