Git is a very powerful tool that has become so ubiquitous in software development that it is one of the first things that newcomers to programming learn. Part of what makes it so powerful is how approachable it is. Even without Git clients, the concepts and basic commands are simple and repeatable enough to be learned in the span of an hour. As a result, it is easy to get by knowing only a handful of Git commands. Until recently I also followed a formulaic Git pattern. This post walks through how and why I have tweaked my Git workflow.

As a source control tool, Git fits well into a collaborative software engineering process. Websites like Github and Gitlab have popularized the use of Git in this manner. Additionally, the branching paradigm makes it easy for multiple people to work concurrently on the same codebase. Again, platforms built around Git have established workflows that take advantage of branching. Using Git to its full potential, however, requires intentionally documenting a thorough Git history.

Initially, I formed a habit of committing at the end of each day with a commit message of “initial commit” or “WIP”. As the mantra goes, I committed early and often. When a particular feature was done I would have a hand full of commits, sometimes with a semi-meaningful commit message. This was a result of two things:

1) My decision to make a commit was based almost entirely on time, not the progress or changes I had made. Aside from my last commit, each commit would usually happen before closing my laptop.
2) I exclusively used the `git commit -m` command to generate my commits

Both of these things were pitfalls for me. Understanding and practicing the principle of atomic commits fundamentally helped me fix both of these issues. Making a commit “atomic” implies that the commit makes one change, one fix or accomplishes one task. This task should be as small as reasonably possible while leaving the codebase in a functioning and healthy state. Changing the way I used Git tooling helped me achieve this goal.

The first part of achieving this goal is actually creating atomic commits. An atomic commit may often take several days to write. This does not mean that it is impossible or unwise to commit often. Having your work committed and stored somewhere outside of your computer is still a valuable precaution against data loss. This may mean you will have multiple “WIP” commit messages in your feature branch before you reach a point where you have made an atomic change. Using Git’s rebase feature, you can squash all of these commits down into a single commit and update the commit message. Assuming you have made five commits on the feature branch that you would like to squash, you can interactively rebase with `git rebase -i HEAD~5`. This will open up a text editor.

```
pick f33e3cf initial commit
pick 36ee5ed WIP
pick abfcb94 more work
pick 2c1c036 another day, another commit
pick dec283d finally done

# Rebase cbc60a2..dec283d onto cbc60a2 (5 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

All of the lines that begin with `#` will not affect the Git history after the rebase is complete. Most of the text editor contains instructions for using the interactive commit rebase command. Right now, each of the five commits is set to be picked. This implies that each commit and commit message will be used for the Git history, as it currently is. The commands below the listed commits can be used to manipulate the Git history. For the sake of brevity, I will not cover all of them. To achieve a single commit in this instance replace `pick` with `reword` for the first commit and replace `pick` with `fixup` for the subsequent commits. Once you do this and save and exit the file you will be prompted with another screen to update the commit message. Again, Git will only use the lines that do not begin with a `#` for the commit message.

The second part of creating an effective commit history is writing informative commit messages. This is an opportunity to let anyone else looking at your work know what you did and why you did it. It may even be helpful for yourself down the road as you reread your own code. A good commit message is comprised of a subject and body. The subject is a short (under 50 characters) description, written in title case that uses the present declarative tense to describe the changes. An example of this is “Update User Model to Require Age”. The body should be separated from the subject by a blank line and describe in one or two short paragraphs what was changed and more importantly why the change was made. Each line in the subject should be no longer than 75 characters. This commit will be far more informative to anyone looking back on code changes than a collection of half-written commit messages for half finished tasks.

If a task is short enough that it only requires committing once, then remove the `-m` flag from `git commit`. This will open up a code editor, allowing you to write an entire commit message. Using the inline commit message enables writing poor messages and makes it almost impossible to write effective ones.

As training wheels for writing effective Git commit messages, I have created a custom Git commit template. Git will open the template each time a commit is made. The template is pasted below. To use it, save it as `~/.git-commit-template` and then update your `~/.gitconfig` to include the file path.

> ~/.gitconfig

```
[commit]
          template = ~/.git-commit-template
```


> ~/.git-commit-template

```
# Git Commit Template
# Subject Line (e.g. Add Model for Cart) -------|


# Body - What, Why, (as opposed to how) ----------------------------------|


# Issue Tag (e.g. TST-1006, #41)
```