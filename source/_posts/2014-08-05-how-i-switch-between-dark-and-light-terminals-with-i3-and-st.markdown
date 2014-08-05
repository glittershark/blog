---
layout: post
title: "How I switch between dark and light terminals with i3 and st"
date: 2014-08-05 12:51:37 -0400
comments: true
categories: linux
---

I'm pretty happy with my current Linux desktop environment - I'm running Arch
Linux, which is a barebones, simplicity-first do-it-all-yourself distribution.
I'm using the absolutely amazing [i3 tiling window manager](http://i3wm.org/),
and I just recently switched to a fairly hacked-on version of the [st terminal
emulator](http://st.suckless.org/), which is basically as stripped-down and
simple as terminal emulators get.

I've got two versions of `st` compiled and sitting in `/usr/bin` - one with the
dark version of the [Solarized](http://ethanschoonover.com/solarized) color
scheme, and one with the light version. I've got a couple of scripts that I
wrote to easily switch between those two, which I do based on the whims of my
corneas. 

<!--more-->

First, I've got this script that writes a basic boolean to a dotfile in my home
folder:

```bash ~/bin/swap_dark.sh https://github.com/glittershark/dotfiles/blob/master/bin/swap_dark.sh
#!/bin/bash

FILE="/home/smith/.term_dark"

if [ "$(< $FILE)" = 't' ]; then
  echo 'f' > $FILE
else
  echo 't' > $FILE
fi
```

And then I've got this script which I use to actually launch the terminal
emulators themselves: 

```bash ~/bin/term.sh https://github.com/glittershark/dotfiles/blob/master/bin/term.sh
#!/bin/bash

if [ "$(< ~/.term_dark)" = 't' ]; then
  exec /usr/local/bin/st-dark $@
else
  exec /usr/local/bin/st-light $@
fi
```

I have the first launch on `$mod+Backspace` in i3, and the second launch using
the default `$mod+Enter`. Here's what that config looks like

```
...
bindsym $mod+BackSpace exec /home/smith/bin/swap_dark.sh

bindsym $mod+Return exec /home/smith/bin/term.sh -e tmux
...
```


----

As an aside, here's the command I used to get those snippets off my filesystem
while editing this post in vim:
```vim
:r !grep -i backspace ~/.i3/config
:r !grep -i term.sh ~/.i3/config
```

