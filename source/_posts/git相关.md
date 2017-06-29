---
title: "git 相关技巧"
tags:
 - git
---


- 当执行 `pull` 操作时想要保留本地更改：
    - 先用 `git stash` 将本地的所有修改就都被暂时存储起来 stash 藏匿，
    - 可通过 `git stash list `查看保存的信息
    - 暂存起来之后就可以通过 `git pull`来进行更新
    - 还原暂存的内容：`git stash pop stash@{0}`
    - 如出现合并冲突，进行手动解决