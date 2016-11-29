---
layout: post
title:  "I3 configuration"
categories: tools i3
---

Now for something completely different, let's do some work environment configuration. Here we will adapt the default [I3](http://i3wm.org/) configuration to set it up not to interfere with [tmux](https://tmux.github.io/) and [Neovim](https://neovim.io/) shortcuts (more on configuring those two in later posts).

First a sidenote on starting I3: as with many minimalistic WMs, to get it to start you simply have to execute it from your **.xinitrc**. While I am at it I set it up to also launch my browser at the same time, since this is the very first thing I generally do anyway: 

```
firefox &
exec /usr/bin/i3
exit 0
```

With the startup out of the way, we can focus on the actual configuration. The default location for the configuration file is **~/.config/i3/config** at the time of this writing.

I personally use the *Alt* prefix for a lot of tmux commands: as a long time vim user the *Alt + ijkl* to swap between panes appeals to me for instance. I then decided to change I3's default modifier to the *Win* (or *Super*, as you prefer) key. Note that *Alt* is referred to as *Mod1* and *Win* as *Mod4*: this is because of how [keypresses are handled](http://unix.stackexchange.com/questions/119212/mod-meta-super-keys) by the OS. 

```
set $mod Mod4 # from Mod1
```

As a former [dwm](http://dwm.suckless.org/) user I still want to preserve some WM shortcuts on the Alt key however, which leads me to alter them manually at other points in the file:

```
bindsym Mod1+1 workspace 1
bindsym Mod1+2 workspace 2
bindsym Mod1+3 workspace 3
bindsym Mod1+4 workspace 4
bindsym Mod1+5 workspace 5
bindsym Mod1+6 workspace 6
bindsym Mod1+7 workspace 7
bindsym Mod1+8 workspace 8
bindsym Mod1+9 workspace 9
bindsym Mod1+0 workspace 10

bindsym Mod1+Return exec "st -f \\"Droid Sans Mono:size=14\\""
bindsym Mod1+Shift+q kill
bindsym Mod1+p exec "dmenu_run -fn \\"Droid Sans Mono:size=10\\""
```

Which brings us to another point: this is the place you should specify your terminal as well as various font parameters if you are so inclined. By default I3 uses a special `i3-sensible-terminal` command to determine which terminal to use, but as its name implies all it guarantees is a sensible terminal, not a practical or beautiful one.

One last quirk to sort out for my personal configuration was getting the *Alt+Tab* combination working for virtual desktop switching. The I3 developers are [opposed to including it in the default configuration](https://www.reddit.com/r/i3wm/comments/3o2bdn/i3_how_to_quickly_switch_between_current_and/), and you even have to script the WM if you want window switching instead of desktop switching (you can find the developer-recommended script [here](https://github.com/acrisci/i3ipc-python/blob/master/examples/focus-last.py)). Anyway for our purpose what seems to be a legacy binding suffices:

```
bindsym Mod1+Tab workspace back_and_forth
```

Let us hope it does not end up removed in future versions.

Adjusting your personal environment should be done seldom and decisively: [XKCD](http://xkcd.com/1205/) has a nice graphic representing the cut-off point for time spent optimizing effectively given a task's length. Not that you should always strive for efficiency, aesthetic concerns are valid too when it comes to desktop configuration for instance, but then it is important to be honest with yourself and recognize you are not actually trying improve your productivity.

On that note, you can find my full configuration file in my [GitHub config repository](https://github.com/theodz/config/blob/master/.config/i3/config). Have fun customizing your tools!
