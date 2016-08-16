---
layout: post
title: Broadcast Input To Current Session
date: 2015-10-26 13:46:19.000000000 +09:00
type: post
categories:
- Tmux
---
## Configuration

```
:setw synchronize-panes
```

## Shortcut
```
bind-key 키 set-window-option synchronize-panes
```

## References
* http://blog.sanctum.geek.nz/sync-tmux-panes/
* https://github.com/elasticdog/dotfiles/blob/master/tmux.conf
