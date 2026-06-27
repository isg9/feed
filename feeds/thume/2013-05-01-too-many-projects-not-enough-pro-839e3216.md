---
title: Too Many Projects, Not Enough pro
url: https://thume.ca/2013/05/01/too-many-projects-not-enough-pro/
published: "2013-05-01T00:00:00Z"
feed: thume
guid: https://thume.ca/2013/05/01/too-many-projects-not-enough-pro
---

# Too Many Projects, Not Enough pro

I have too many projects, so I started a new project to solve my problems. This project is a [little tool called pro](http://github.com/trishume/pro) which allows you to easily deal with all your git repositories.

It has a handful of very useful features, each of which solves a problem that I have experienced. I imagine they will be useful to others as well. You can get `pro` by running `gem install pro`.

Do note that a Unix system is required to use this, so it won’t work on Windows without Cygwin.

## CD’ing to a project’s repository

Cd’ing to your projects is harder than it should be. There are [many tools](https://github.com/rupa/z) that try and solve this problem using frequency and recency. Pro solves the problem by fuzzy searching only git repositories.

The `pd` command allows you to instantly CD to any git repo by fuzzy matching its name. You can install the `pd` tool (name configurable) by running `pro install`. Once you have it you can do some pretty intense cd’ing:

![pd demo](/assets/postassets/pro/pd_screen.png)

## State of the Repos Address

Oftentimes I find myself wondering which git repositories of mine still have uncommitted changes or unpushed commits. I could find them all and run git status but it would be nice to get a quick overview. `pro status` does this.

![pro status](/assets/postassets/pro/pro_status.png)

You can also run `pro status <repo>` to show the output of `git status` for a certain repo.

## Run all the commands!

Wouldn’t it be cool if you could run a command on all your repos and see a summary of the output? Now you can!

You can do this with `pro run <command>`. If you don’t pass a command it will prompt you for one.

For example, searching all your repos for ruby files:

![pro run](/assets/postassets/pro/pro_run.png)

Notice that it double checks before running so you don’t accidentally run `rm -rf *` on all your projects.

## The Pro Base

Pro can use a base directory to speed up its search for git repos. By default it uses your home folder.

To set the base directory either create a file at `~/.proBase` containing the base path or set the environment variable `PRO_BASE` to the path.

## Conclusion

`pro` is a handy tool that makes working with lots of git repos much easier. If you want to get it run `gem install pro`. You can also [check it out on Github](http://github.com/trishume/pro).
