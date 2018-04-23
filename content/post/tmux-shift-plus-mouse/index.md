+++
title = "tmux Shift + Mouse"
author = ["Jimmy.m.Gong"]
description = "Using mouse to copy/paste in `tmux` _panes_."
date = 2014-08-28T16:47:46-04:00
tags = ["tmux", "mouse", "copy", "paste"]
categories = ["unix"]
draft = false
creator = "Emacs 25.3.1 (Org mode 9.1.9 + ox-hugo)"
[versions]
  tmux = "2.6+"
  tcsh = "6.17.00"
  xterm = "X.Org 6.8.99.903(327)"
+++

I had been missing the _"select and middle-click"_ method for copying
and pasting stuff in `tmux` panes.

Thanks to [this](http://superuser.com/questions/598718/how-do-i-select-entire-words-with-tmuxs-mouse-mode) post, I learned that I can use the <kbd>Shift</kbd> key and
bypass `tmux`'s own copy and paste method.

| Key/Mouse Binding                                 | Action                                                       |
|---------------------------------------------------|--------------------------------------------------------------|
| <kbd>Shift</kbd> + Mouse left button double-click | Copies the double-clicked word                               |
| <kbd>Shift</kbd> + Select using mouse             | Copies the selection                                         |
| <kbd>Shift</kbd> + Mouse middle button click      | Pastes the copied text using above method in the `tmux` pane |

[//]: # "Exported with love from a post written in Org mode"
[//]: # "- https://github.com/yssource/home"
