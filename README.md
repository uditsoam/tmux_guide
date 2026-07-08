# tmux — Basic to Advanced Guide (Easiest Way)

> tmux lets you run multiple terminal sessions inside one window, keep them running after you disconnect, and split your screen into panes. Extremely useful for pentesting work — keep a listener running in one pane while you work in another, survive SSH disconnects, etc.

---

## Table of Contents

1. [What is tmux & Why Use It](#1-what-is-tmux--why-use-it)
2. [Installation](#2-installation)
3. [Core Concepts](#3-core-concepts)
4. [Basic Usage — Sessions](#4-basic-usage--sessions)
5. [Windows (Tabs)](#5-windows-tabs)
6. [Panes (Splits)](#6-panes-splits)
7. [Navigating Between Panes/Windows](#7-navigating-between-paneswindows)
8. [Resizing Panes](#8-resizing-panes)
9. [Copy Mode / Scrolling](#9-copy-mode--scrolling)
10. [Detach & Reattach (The Killer Feature)](#10-detach--reattach-the-killer-feature)
11. [Renaming & Organizing](#11-renaming--organizing)
12. [Killing Things](#12-killing-things)
13. [The Config File — Customizing tmux](#13-the-config-file--customizing-tmux)
14. [Advanced — Scripting tmux](#14-advanced--scripting-tmux)
15. [Practical Pentesting Workflow Example](#15-practical-pentesting-workflow-example)
16. [Quick Reference Card](#16-quick-reference-card)

---

## 1. What is tmux & Why Use It

**tmux** (terminal multiplexer) lets you:
- Run multiple terminal "windows" and split-screen "panes" inside ONE terminal
- **Detach** from a session and reattach later — your programs keep running even if you close the terminal or lose SSH connection
- Split your screen to watch a listener, a log file, and run commands all at once

```
Without tmux:  one terminal = one task. Close terminal = lose everything running.
With tmux:     one terminal = many tasks, organized in windows/panes, survives disconnects.
```

---

## 2. Installation

```bash
# Debian/Ubuntu/Kali
sudo apt install tmux

# Verify
tmux -V
```

---

## 3. Core Concepts

```
Session
  └── Window (like a browser tab)
        └── Pane (a split within that window)
```

- **Session** = the whole container, can be detached/reattached, survives disconnects
- **Window** = like a tab — one session can have many windows
- **Pane** = a split section within a window — one window can have many panes

**The prefix key:** almost every tmux command starts by pressing `Ctrl+b` first, releasing, then pressing the actual command key. This guide writes it as `Ctrl+b` then the next key.

---

## 4. Basic Usage — Sessions

```bash
# Start a new tmux session (unnamed)
tmux

# Start a NAMED session (recommended — easier to find later)
tmux new -s mysession

# List all running sessions
tmux ls

# Attach to a session by name
tmux attach -t mysession
tmux a -t mysession        # shorthand

# Attach to the most recent session (if only one exists)
tmux attach

# Kill a specific session entirely
tmux kill-session -t mysession

# Kill ALL tmux sessions
tmux kill-server
```

---

## 5. Windows (Tabs)

| Action | Keys |
|---|---|
| Create new window | `Ctrl+b` then `c` |
| Next window | `Ctrl+b` then `n` |
| Previous window | `Ctrl+b` then `p` |
| Go to window number (0-9) | `Ctrl+b` then `0`-`9` |
| List all windows (interactive picker) | `Ctrl+b` then `w` |
| Rename current window | `Ctrl+b` then `,` |
| Close current window | `Ctrl+b` then `&` (confirms first) |

```bash
# Create a new window directly from command line when starting tmux
tmux new -s mysession -n firstwindow
```

---

## 6. Panes (Splits)

| Action | Keys |
|---|---|
| Split pane vertically (side by side) | `Ctrl+b` then `%` |
| Split pane horizontally (top/bottom) | `Ctrl+b` then `"` |
| Close current pane | `Ctrl+b` then `x` (confirms first), or just type `exit` |
| Toggle pane to fullscreen / back | `Ctrl+b` then `z` |
| Show pane numbers briefly | `Ctrl+b` then `q` |

**Easiest way to remember:** `%` looks like a vertical split (left/right), `"` looks like a horizontal split (top/bottom).

---

## 7. Navigating Between Panes/Windows

| Action | Keys |
|---|---|
| Move to next pane | `Ctrl+b` then arrow key (↑↓←→) |
| Cycle through panes | `Ctrl+b` then `o` |
| Switch to last-used window | `Ctrl+b` then `l` |
| Jump directly to pane by number | `Ctrl+b` then `q` then the number |

---

## 8. Resizing Panes

```
Ctrl+b then Ctrl+arrow key  -> resize the current pane in that direction
(hold Ctrl, tap arrow repeatedly to resize incrementally)
```

| Action | Keys |
|---|---|
| Resize pane | `Ctrl+b` then hold `Ctrl` + arrow key |
| Even out all pane sizes | `Ctrl+b` then `Ctrl+b` then `=`(some configs) — or use `Alt+1`/`Alt+2` layouts below |
| Cycle through preset layouts | `Ctrl+b` then `Space` |

---

## 9. Copy Mode / Scrolling

By default, scrolling your mouse wheel doesn't work the normal way in tmux — you enter a special "copy mode" to scroll back through history.

| Action | Keys |
|---|---|
| Enter copy/scroll mode | `Ctrl+b` then `[` |
| Scroll up/down (inside copy mode) | Arrow keys or `Page Up`/`Page Down` |
| Start text selection (inside copy mode) | `Space` |
| Copy selection | `Enter` |
| Exit copy mode | `q` |
| Paste copied text | `Ctrl+b` then `]` |

**Easiest fix if scrolling feels broken:** enable mouse mode (see config section below) — then your mouse wheel just works normally without needing copy mode at all.

---

## 10. Detach & Reattach (The Killer Feature)

This is the single most useful tmux feature for any long-running task — including listeners, downloads, or long enumeration scripts.

| Action | Keys |
|---|---|
| Detach from session (leave it running in background) | `Ctrl+b` then `d` |

```bash
# After detaching, your session keeps running. Reattach anytime:
tmux attach -t mysession

# Example real workflow:
tmux new -s listener
nc -lvnp 4444
# Ctrl+b, then d   <- detach, listener keeps running
# ... do other things, close your terminal, SSH back in later ...
tmux attach -t listener
# Your nc listener is still sitting there exactly as you left it
```

**Why this matters so much over SSH:** if your SSH connection drops (network issue, laptop sleeps, VPN hiccup), anything running directly in a normal SSH session dies with the connection. Anything running inside tmux survives — you just reconnect and reattach.

---

## 11. Renaming & Organizing

```bash
# Rename a session (from outside, via command line)
tmux rename-session -t oldname newname

# Rename current window (from inside tmux)
Ctrl+b then ,
# type new name, press Enter
```

---

## 12. Killing Things

| Action | Keys / Command |
|---|---|
| Kill current pane | `Ctrl+b` then `x` (confirm with `y`) |
| Kill current window | `Ctrl+b` then `&` (confirm with `y`) |
| Kill a session by name | `tmux kill-session -t name` |
| Kill everything (all sessions) | `tmux kill-server` |

---

## 13. The Config File — Customizing tmux

Create/edit `~/.tmux.conf` to make tmux easier to use long-term.

```bash
nano ~/.tmux.conf
```

**Recommended beginner-friendly config (paste this in):**

```conf
# Enable mouse support — click panes to select, drag to resize, scroll normally
set -g mouse on

# Change prefix from Ctrl+b to Ctrl+a (easier to reach, many people prefer this)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Start window/pane numbering at 1 instead of 0 (matches keyboard number row)
set -g base-index 1
setw -g pane-base-index 1

# Faster pane switching with Alt+arrow (no prefix needed)
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# Easier vertical/horizontal split keys
bind | split-window -h
bind - split-window -v

# Reload config without restarting tmux
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# Increase scrollback history buffer (default is quite small)
set -g history-limit 10000
```

```bash
# After saving, reload it without restarting tmux:
tmux source-file ~/.tmux.conf
# Or if you added the "bind r" line above, just press: Ctrl+b then r
```

---

## 14. Advanced — Scripting tmux

You can fully script a tmux layout so it sets itself up exactly how you want with one command — very useful for pentesting setups you repeat often.

```bash
#!/bin/bash
# save as setup_pentest.sh, chmod +x, then run it

SESSION="pentest"

tmux new-session -d -s $SESSION -n main

# Split into 3 panes: top-left (notes), top-right (listener), bottom (working shell)
tmux split-window -h -t $SESSION
tmux split-window -v -t $SESSION

# Send commands to specific panes (panes are numbered 0,1,2 in creation order)
tmux send-keys -t $SESSION:0.0 'nano notes.txt' C-m
tmux send-keys -t $SESSION:0.1 'nc -lvnp 4444' C-m
tmux send-keys -t $SESSION:0.2 'echo "Ready to work"' C-m

# Attach to the session
tmux attach -t $SESSION
```

```bash
# Run it
./setup_pentest.sh
# Instantly drops you into a 3-pane layout with everything already running
```

**Other useful scripting commands:**

```bash
# Create a detached session and run a command immediately
tmux new-session -d -s background_job 'python3 long_script.py'

# Send a command into an already-running session/pane from outside
tmux send-keys -t mysession 'whoami' C-m

# Capture the current visible pane content to a file (useful for logging)
tmux capture-pane -t mysession -p > output.txt
```

---

## 15. Practical Pentesting Workflow Example

```bash
# Start a named session for the engagement
tmux new -s oscp

# Split: left pane for notes, right pane for active shell
Ctrl+b then %

# Left pane: keep notes open
nano notes.md

# Move to right pane (Ctrl+b then right arrow)
# Right pane: split again horizontally for a listener
Ctrl+b then "

# Top-right: nmap scan
nmap -sV -sC <TARGET_IP>

# Bottom-right: listener ready and waiting
nc -lvnp 4444

# Detach when you need to step away — everything keeps running
Ctrl+b then d

# Reattach later, exactly where you left off
tmux attach -t oscp
```

---

## 16. Quick Reference Card

```
====================================================================
 TMUX — QUICK REFERENCE
====================================================================
 Prefix key = Ctrl+b  (press, release, then press the next key)
====================================================================

[SESSIONS]
  tmux new -s name          start named session
  tmux ls                   list sessions
  tmux attach -t name       reattach
  Ctrl+b d                  detach (keeps running in background)
  tmux kill-session -t name kill a session

[WINDOWS]
  Ctrl+b c                  new window
  Ctrl+b n / p               next / previous window
  Ctrl+b 0-9                 jump to window number
  Ctrl+b ,                   rename window
  Ctrl+b w                   list windows (picker)

[PANES]
  Ctrl+b %                   split vertical (side by side)
  Ctrl+b "                   split horizontal (top/bottom)
  Ctrl+b arrow                move between panes
  Ctrl+b o                    cycle panes
  Ctrl+b z                    zoom pane fullscreen / unzoom
  Ctrl+b x                    kill current pane
  Ctrl+b Ctrl+arrow           resize pane

[COPY MODE / SCROLL]
  Ctrl+b [                    enter copy mode (scroll with arrows)
  Space                       start selection (inside copy mode)
  Enter                       copy selection
  Ctrl+b ]                    paste
  q                            exit copy mode

[EASIEST FIX]
  Add "set -g mouse on" to ~/.tmux.conf -> click/scroll/resize with
  mouse normally, no copy-mode needed

[CONFIG FILE]
  ~/.tmux.conf
  tmux source-file ~/.tmux.conf     reload without restart

[SCRIPTING]
  tmux new-session -d -s name 'command'
  tmux send-keys -t name 'command' C-m
  tmux split-window -h -t name

[WHY USE IT]
  Survives SSH disconnects. Keep listeners/scans running in the
  background, detach, reconnect later -- exactly where you left off.
====================================================================
```

---

*Reference guide for general terminal productivity.*
