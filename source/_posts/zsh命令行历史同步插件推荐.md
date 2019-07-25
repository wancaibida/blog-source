title: Zsh命令行历史同步插件推荐
author: 大丈夫没问题
tags:
  - linux
  - zsh
  - plugin
categories:
  - linux
date: 2019-07-25 21:30:00
---
公司和家里用的是两台电脑，经常遇到一些用过却记不住的命令，恰巧这些命令在另外台电脑执行过，就想找个能在不同机器上同步命令行历史的插件。

尝试过将用户目录文件夹下的`.zsh_history`文件软链接到dropbox，发现还是有点问题，比如多台机器同步是会发生文件冲突，一些情况下zsh会重新创建新的`.zsh_history`文件，导致数据丢失。期间又尝试了下[history-sync](https://github.com/wulfgarpro/history-sync)这个插件，发现有会将历史文件清空的BUG。

无意中发现了[zsh-histdb](https://github.com/larkery/zsh-histdb)，这个插件正好满足了我的要求：在多台电脑之间同步历史记录，自动合并冲突文件。配合[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)简直完美。


