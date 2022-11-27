---
layout: post
title:  "Explorer shortcut not working in Visual Code"
date:   2022-05-28 13:20:00 +0100
categories: development linux
excerpt: This was driving me mad for a while. On some systems the Explorer shortcut CTRL+SHIFT+e is not working properly. The shortcut is supposed to open the file Explorer in Visual Code's side bar and jump to the location of the file. When using the keyboard shortcut while the cursor is in a text field, it would not do anything and the subsequent keyboard input is bogged.
---

![Visual Code Logo]({{ site.baseurl }} /images/Visual-Studio-Code-Crack-497510271.png "Visual Code")


This was driving me mad for a while. On some systems the Explorer shortcut `CTRL+SHIFT+e` is not working properly. The shortcut is supposed to open the file Explorer in Visual Code's side bar and jump to the location of the file. When using the keyboard shortcut while the cursor is in a text field, it would not do anything and the subsequent keyboard input is bogged.

Turns out that this shortcut is being used to insert emojis in GTK3 applications. More detail in [this stackoverflow thread](https://stackoverflow.com/questions/52032340/ctrlshifte-inserts-special-characters-into-file-instead-of-showing-explorer-pa).

## How to fix the Explorer Shortcut
The fix is simple. We remove the keyboard mapping in GTK for the emoji tool. For this matter the `gsettings` command can be used, once we know where the setting is hiding.

The aforementioned stackoverflow noted, that it's a feature of the Intelligent Input Bus (IBus). See the always excellent [Arch Wiki](https://wiki.archlinux.org/title/IBus) for more details.

From the Arch Wiki I gathered that IBus is part of the freedesktop package. Sure enough I found the setting with the following command

```shell
$ gsettings list-recursively org.freedesktop.ibus.panel.emoji

org.freedesktop.ibus.panel.emoji favorite-annotations @as []
org.freedesktop.ibus.panel.emoji favorites @as []
org.freedesktop.ibus.panel.emoji font 'Monospace 16'
org.freedesktop.ibus.panel.emoji has-partial-match false
org.freedesktop.ibus.panel.emoji hotkey ['<Control><Shift>e']
org.freedesktop.ibus.panel.emoji lang 'en'
org.freedesktop.ibus.panel.emoji load-emoji-at-startup true
org.freedesktop.ibus.panel.emoji load-unicode-at-startup false
org.freedesktop.ibus.panel.emoji partial-match-condition 0
org.freedesktop.ibus.panel.emoji partial-match-length 3
org.freedesktop.ibus.panel.emoji unicode-hotkey ['<Control><Shift>u']
```

The following command removes the problematic **CTRL+SHIFT+e** button mapping:

```shell
gsettings set org.freedesktop.ibus.panel.emoji hotkey '[]'
```

It's also possible to achieve the same result by opening the GUI with `ibus-setup` and then deleting the button mapping in the "Emoji" tab. I prefer the programatic solution with `gsettings`, as it can be added to a setup script.