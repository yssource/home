+++
title = "Send a command to every pane/window/session in tmux"
author = ["Jimmy.m.Gong"]
description = """
  Faster way to send the same command to each and every _pane_ in your
  tmux _session_.
  """
date = 2014-03-06T09:50:21-05:00
tags = ["tmux", "pane", "window", "session"]
categories = ["unix"]
draft = false
creator = "Emacs 25.3.1 (Org mode 9.1.9 + ox-hugo)"
[versions]
  tmux = "2.6+"
+++

<span class="timestamp-wrapper"><span class="timestamp">&lt;2018-03-20 Tue&gt;</span></span>
: Optimize the `list-panes` + `send-keys`

`tmux` commands, rewrite the post.

---

Ever wondered how you would send the `clear` command to _each pane_,
in _each window_, in _each session_ in `tmux`, or how you would do
source your shell config file in each after each tweak?

Here are few excerpts from my [`.tmux.conf`](https://github.com/kaushalmodi/dotfiles/blob/master/tmux/dot-tmux.conf) that allow doing just
that.


## Send command to all panes in **all** sessions {#send-command-to-all-panes-in-all-sessions}

Thanks to the tip in comments from _Bob Fleming_, I learned that `tmux` has a `-a`
switch for the `list-panes` command.

From `man tmux`,

> ```text
> list-panes [-as] [-F format] [-t target]
>               (alias: lsp)
>         If -a is given, target is ignored and all panes on the server
>         are listed.  If -s is given, target is a session (or the
>         current session).  If neither is given, target is a window (or
>         the current window).  For the meaning of the -F flag, see the
>         FORMATS section.
> ```

With that knowledge, the [older version](#tmux-send-cmd-to-all-panes-old) of the <kbd>E</kbd> binding now reduces
to,

````docker
# Send the same command to all panes/windows/sessions
bind E command-prompt -p "Command:" \
       "run \"tmux list-panes -a -F '##{session_name}:##{window_index}.##{pane_index}' \
              | xargs -I PANE tmux send-keys -t PANE '%1' Enter\""
````


### Usage {#usage}

-   Type the following binding in any `tmux` pane: <kbd>C-z E</kbd>[^fn:1]
-   Enter a command that you would want to send to all the panes, like
    `source ~/.alias; clear` _(this is entered in the tmux command
    prompt)_.
-   That will source the `~/.alias` in **all** panes, and then clear the
    terminals as well.


### About the `##` {#about-the-double-hashes}

<div class="note">
  <div></div>

The `#` character needs to be escaped by another `#` and typed as
`##`, only when used inside the `run-shell` command.

</div>

.. because otherwise, `tmux run-shell` command will replace the
unescaped `#{session_name}`, `#{window_index}` and `#{pane_index}` with
their current values **before** executing the command.

With the hashes escaped, those variables will be evaluated _at run
time_.

But if you were to type the above command directly in the terminal,
without the `run-shell` command wrapper, you would use only single
`#`:

````text
tmux list-panes -s -F "#{session_name}:#{window_index}.#{pane_index}"
````


## Send command to all panes in **current** session {#send-command-to-all-panes-in-current-session}

The `list-panes` command has another useful switch: `-s`, which takes
an optional argument, a _session name_. If that argument is not
supplied, it takes the current session name by default.

Below <kbd>C-e</kbd> binding is used to send a command to all panes, in all
windows, but **only in the current session**.

````docker
bind C-e command-prompt -p "Command:" \
         "run \"tmux list-panes -s -F '##{session_name}:##{window_index}.##{pane_index}' \
                | xargs -I PANE tmux send-keys -t PANE '%1' Enter\""
````


## Older version (circa 2014) {#tmux-send-cmd-to-all-panes-old}

````docker
# Send the same command to all panes/windows/sessions
bind E command-prompt -p "Command:" \
       "run \"tmux list-sessions                                           -F '##{session_name}' \
              | xargs -I SESS          tmux list-windows  -t SESS          -F 'SESS:##{window_index}' \
              | xargs -I SESS_WIN      tmux list-panes    -t SESS_WIN      -F 'SESS_WIN.##{pane_index}' \
              | xargs -I SESS_WIN_PANE tmux send-keys     -t SESS_WIN_PANE '%1' Enter\""
````

[^fn:1]: I have set my tmux prefix to <kbd>C-z</kbd>.

[//]: # "Exported with love from a post written in Org mode"
[//]: # "- https://github.com/yssource/home"
