---
comments: true
date: 2011-12-02 08:35:14
layout: post
slug: adding-tab-completion-to-git-commands-in-bash
title: Adding tab-completion to Git commands in Bash
wordpress_id: 239
categories:
- Programming
tags:
- git
- github
---

I occasionally use the Git Bash shell in Windows and always miss the ability to tab complete the git commands when in other shells. A couple of days ago I found a bash script that enables this functionality when reading a [post on effectively using topic branches in Git by the mozilla web dev team](http://blog.mozilla.com/webdev/2011/11/21/git-using-topic-branches-and-interactive-rebasing-effectively/). It enables tab completion as shown below:

    
    $ git sta<tab><tab>
    stage    stash    status


I've added it to my [dotfiles repository on GitHub](https://github.com/mfoo/dotfiles/blob/master/.git-completion.bash), or you can access the latest file in the git release tarball at contrib/completion/git-completion.bash.
