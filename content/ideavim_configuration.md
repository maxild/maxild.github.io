+++
title = "Idea VIM plugin configuration"
date = 2020-06-25
+++

The plugin can be configured via an `~/.config/ideavim/ideavimrc` run config
file. It is not that straight-forward becuase the plugin does not have a genuine
vimscript parser, and uses hacky regex parsing. Therefore it is a good idea to
write a dedicated vimrc configuration for the plugin, and not use the more
general vimrc (vim.init) file used by vim/nvim.

The commands `set`, `map` etc are supported.

A list of [set commands](https://github.com/JetBrains/ideavim/blob/master/doc/set-commands.md)
is listed in the documentation.

The `map` command can be used to resolve key binding conflicts. It supports two
new constructs

* `actionlist {pattern}`
* `action {name}`

A list of supported IDE commands can be seen in the IDE after typing
`:actionlist`. The list is also here that [this](https://gist.github.com/zchee/9c78f91cc5ad771c1f5d)
gist.

A list of default keybindings can be seen [here](https://github.com/JetBrains/ideavim/blob/master/src/com/maddyhome/idea/vim/package-info.java).
## Builtin plugins

* vim-surround
* vim-commentary
* vim-multiple-cursors

Add the following set commands to your ideavimrc

```vim
set surround
set multiple-cursors
```

## Troubleshooting

Just use vim/nvim!

Okay I like Rider.

You can setup nvim as an external tool, and make a shortcut in the IDE to open
nvim in the current file on the current line. This [guide](https://www.jetbrains.com/help/ruby/using-emacs-as-an-external-editor.html#external-editor)
uses this trick. This works because we can start neovim on a specific line using

```bash
nvim +{linenumber} {path}
```

### Structural Code Selections not working

Selecting content by some action, such as the EditorSelectWord
(aka. Extend Selection) does not mean it is visually selected. That means
that triggering the EditorSelectWord action and then performing something
like c or d on it will not do anything. The issue is well documented [here](https://stackoverflow.com/questions/38126290/intellij-idea-selection-to-ideavim-selection)
and [here](https://youtrack.jetbrains.com/issue/VIM-510). To work around this
issue, you have to enter visual mode after triggering the action - which, granted, is a bit of a hassle.

## Keymap and action names

Keymaps can be printed from within the IDE using the `keymap exporter` plugin.
This is not that useful IMO. A better (searchable) keymap can be found at the
[link](https://github.com/JetBrains/intellij-community/tree/master/platform/platform-resources/src/keymaps),
where all the keymaps are XML encoded. A keymap has a parent, and this represent
inheritance in the OO kind of way. Another great way to interactively inspect
the current keymap in the IDE is using the VimIdea plugin. If you type
`:actionlist showintent` then all action names that matches the pattern
`showintent` will be listed together with any keybindings.

```
--- Actions ---
ShowIntentionActions                               <A-CR>
ShowIntentionsGroup
```

This corresponds with the `Show Context Actions` mapped to `Alt+Enter` in the
default keymap. The output uses the VIM conventions for describing keys

* `C`: Control
* `A`: Alt
* `S`: Shift
* `CR`: Enter
* add more...

Note: `M` (meta) is something we try to avoid because this key works differently
cross OS. Super (linux), Cmd (darwin) and Windows (windows) are keys we try to
avoid using.

We can use the IdeaVim plugin again writing `:action
ShowIntentionActions<CR>` in the IDE. This way we can ensure that we are
talking about the correct command in the IDE.

The goal is 2-fold:

* Define VIM keybindings
* Learn (redefine) IDE keybindings

To make it all less complicated, I have decided to have any conflicts between
VIM and IDE keybindings be defined to use VIM keybindings. Even if this means
redefining standard VIM mappings in the `ideavimrc` file (`<C-C>` etc).

The hope is that we can just use the standard default Intellij Keymap in all IDE
producs (Rider, Goland etc) and platforms (Win, Linux, Darwin). This is partly
because it doesn't seem like sync settings work cross platform and product in
the IDEs.
