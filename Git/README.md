# Git

These are a few helpful tips for dealing with Git.

## Lola

This is a helpful command for visualizing the commit history of your repository. To exit lola, press q. Taken from http://blog.kfish.org/2010/04/git-lola.html, which itself has all the information you need, but for reference, copy this into gitconfig:

[alias]
        lol = log --graph --decorate --pretty=oneline --abbrev-commit
        lola = log --graph --decorate --pretty=oneline --abbrev-commit --all
[color]
        branch = auto
        diff = auto
        interactive = auto
        status = auto

## Removing file from history

I've tried half a dozen things, and for some reason I can't get this right, UNLESS I use the trick described in https://stackoverflow.com/a/2158271. Specifically, I use the filter-branch option. His tutorial is far better than anything I could ever make, so just look there.

## Excluding paths from diff

Sometimes you don't really care about changes made to particular paths. In ARCADE's case, for the most part, I don't care what goes on inside `src/test/resources`. To exclude a path from diff, tell it to `':(exclude)that/path'`. The full command will look something like this: `git diff 01b159a..1046a04 -- ':(exclude)src\test\resources\*'`. If using Windows, switch the single quotes for double quotes.