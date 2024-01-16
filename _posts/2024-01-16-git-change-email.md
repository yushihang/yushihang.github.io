---
layout: post
title: Git修改提交历史中的邮箱
subtitle: 当你在不同组织的项目内混用了邮箱后缀...
categories: Git
tags: [Git, GitHub]
---

## Git 修改提交历史中的邮箱

先用如下命令修改当前 git 项目的邮箱

```bash
git config user.email "xxx@xxx.com"
```

如果要修改整个项目的历史提交记录

```bash
git rebase -r --root --exec "git commit --amend --no-edit --reset-author
git push -f
```

修改最近一次提交记录

```bash
git commit --amend --no-edit --reset-author
git push -f
```

修改从某个 commit 之后的历史记录

```bash
git rebase -r <commit hash>  --exec 'git commit --amend --no-edit --reset-author'
git push -f
```

还有一个待测试的脚本

```bash
#!/bin/sh

git filter-branch --env-filter '

an="$GIT_AUTHOR_NAME"
am="$GIT_AUTHOR_EMAIL"
cn="$GIT_COMMITTER_NAME"
cm="$GIT_COMMITTER_EMAIL"

if [ "$GIT_COMMITTER_EMAIL" = "your@email.to.match.example" ]
then
    cn="Your New Committer Name"
    cm="Your New Committer Email"
fi
if [ "$GIT_AUTHOR_EMAIL" = "your@email.to.match.example" ]
then
    an="Your New Author Name"
    am="Your New Author Email"
fi

export GIT_AUTHOR_NAME="$an"
export GIT_AUTHOR_EMAIL="$am"
export GIT_COMMITTER_NAME="$cn"
export GIT_COMMITTER_EMAIL="$cm"
'

```
