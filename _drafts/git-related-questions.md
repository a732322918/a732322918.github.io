---
layout: default
title: 使用git出现的相关问题
---

1. 使用`git svn dcommit`提交到`svn`服务器出现的问题
`ERROR from SVN:
A repository hook failed: Commit blocked by pre-commit hook (exit code 1) with output:
"请填写提交信息, 至少4个汉字或8个英文字符!" `

原因：使用`git`添加提交信息过短。

解决办法：使用`git rebase -i head~3`修改提交信息。