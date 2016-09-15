---
layout: post
title:  "Git合并多次提交"
date:   2015-10-24
keywords: ["git","rebase"]
description: "git rebase"
category: "Git"
tags: ["Git"]
---
{% include JB/setup %}
在工作中，有时候会遇到多次针对一个问题的多次提交，因而会产生很多的提交记录，这让人看起来很不爽。
那么怎么样才能将多次的提交合并呢？答案是通过rebase命令。

当你执行了`git rebase --interactive HEAD~2 master`命令后，将会出现如下的交互式画面：

    {% highlight ruby %}
    Code:
    pick e30746d modify ruby case statement
    pick d870d17 fix display problems

    # Rebase 15e28f4..d870d17 onto 15e28f4
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    {% endhighlight %}

注释掉的部分是git给出的操作提示，这里选择`squash`选项来替换掉`pick`，需要注意的是，最上方的那一行`pick`
需要保留，否则会出现`Cannot 'squash' without a previous commit`的错误提示。如果你不小心替换掉了，那么
需要使用`git rebase --abort`再来一遍。

退出编辑器之后，将会进入到下一个界面，如下所示：

    {% highlight ruby %}
    Code:
    # This is a combination of 2 commits.
    # The first commit's message is:

    modify ruby case statement

    # This is the 2nd commit message:

    fix display problems
    {% endhighlight %}

没有被注释掉的部分是前两次的提交记录，可以选择保留，也可以选择删掉重写，然后退出就可以了。
之后就可以通过`git log --oneline --decorate`查看提交记录，发现前几次的提交已经合并掉了。

另外一种合并的方式：

    {% highlight ruby%}
    code:
    $ git reset HEAD~5
    $ git add .
    $ git commit -am "Here's the bug fix that closes #28"
    $ git push --force
    {% endhighlight %}

如果你想修复之前的提交，又不想有多次记录，那么可以在修复的这次提交中使用：

    {% highlight ruby %}
    Code:
    git commit --fixup bbb2222
    git rebase --interactive --autosquash bbb1111
    #bbb1111为你需要修复的提交的前一次提交
    {% endhighlight %}



参考文献:

- [Auto-squashing Git Commits](https://robots.thoughtbot.com/autosquashing-git-commits "Auto-squashing Git Commits")
- [How can I merge two commits into one?](http://stackoverflow.com/questions/2563632/how-can-i-merge-two-commits-into-one "How can I merge two commits into one?")
