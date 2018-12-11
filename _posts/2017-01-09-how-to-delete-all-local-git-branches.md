---
layout: post
title:  "How to delete all local git branches except the currently selected branch?"
date:   2017-01-09 23:31:31 +0100
comments: true
tags: git
---
If you are working as part of a team on a project hosted inside of a git repository, you may find yourself branching around in an excessive manner. This will lead to a huge pile of branches you don’t necessarily need to store locally if they are already merged or saved on a remote.

To clean this up and delete all local branches except the currently selected branch, issue the following command.

```bash
for b in `git branch | grep -v \*`; do git branch -D $b; done
```

What does this line in detail?
1. Show all branches and filter the current branch out. As the this branch is marked with a leading asterisk symbol, `grep -v \*` will match and remove it from the list.
2. Loop over the just selected branches and delete each of them.

Simple as that.
