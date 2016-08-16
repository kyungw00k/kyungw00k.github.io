---
layout: post
title: Cygwin Configuration for C/C++ developer
date: 2009-01-07 13:38:08.000000000 +09:00
type: post
published: true
status: publish
categories:
- Log
---

### 1. Create .inputrc in your home root directory.

#### `.inputrc` for cygwin

```
set meta-flag On
set convert-meta Off
set output-meta On

"\e[3~": delete-char
# this is actually equivalent to

 "\C-?": delete-char
# VT
"\e[1~": beginning-of-line
"\e[4~": end-of-line

# kvt
"\e[H":beginning-of-line
"\e[F":end-of-line

# rxvt and konsole (i.e. the KDE-app...)
"\e[7~":beginning-of-line
"\e[8~":end-of-line
```

### 2. .bashrc

```

# .bashrc

# Get the aliases and functions
if [ -f /etc/.bashrc ]; then
    . /etc/.bashrc
fi

# User specific environment and startup programs
PATH=$PATH:~/bin:.

export PATH

unset USERNAME

alias cp='cp -i'
alias df='df -h'
alias du='du -h'
alias grep='grep --color'
alias l.='ls -dl .[a-zA-Z]*'
alias less='less -r'
alias ll='ls -al'
alias ls='ls -F --color=auto --show-control-char'
alias mc='. /usr/share/mc/bin/mc-wrapper.sh'
alias mv='mv -i'
alias rm='rm -i'
alias vi='vim'
alias whence='type -a'

export LS_COLORS='no=00:fi=00:di=01;34:ln=01;36:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:ex=01;32:*.ta>

export PS1='\[\e[0;31m\]\u\[\e[0;37m\]@\[\e[0;33m\]\h\[\e[0;36m\](\W)\[\e[0;0m\]\$ '

export MANPATH=$MANPATH:~/.man
```

### 3.  .vimrc configuration
- Refer to http://kldp.org/node/35701

### 4. Useful Plugins for Vim
- http://www.vim.org/scripts/script.php?script_id=527
- http://www.vim.org/scripts/script.php?script_id=1764
- http://www.vim.org/scripts/script.php?script_id=1318
- http://www.vim.org/scripts/script.php?script_id=1218
- http://www.vim.org/scripts/script.php?script_id=1520
- http://www.vim.org/scripts/script.php?script_id=2358

### 5. etc
- Use PuTTY as a Cygwin Terminal([http://vany.tistory.com/entry/PuTTYcyg-PuTTY-as-a-CygWIN-Terminal](http://vany.tistory.com/entry/PuTTYcyg-PuTTY-as-a-CygWIN-Terminal))
- sshd configuration
- Multiple User
