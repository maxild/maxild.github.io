+++
title = "SynthWave '84 - VS Code theme"
date = 2021-04-10
[taxonomies]
tags = ["vscode"]
+++

I don't want to install vscode in user mode (i.e. under `~/`) on my linux machine.
This makes getting updates easier. But I also want to use this cool glowing theme

```
ext install RobbOwen.synthwave-vscode
```

After install of the theme, if you open the command palette with `Ctrl + Shift + P` and choose "Enable Neon Dreams", you will fail, because this requires root/admin privileges.

Therefore re-open vscode as root (not recommended...BTW)

```bash
$ sudo code --user-data-dir="~/.vscode-root"
```

and install the theme and enable the glowing theme.

Finally launch VS Code as root, and `Ctrl + Shift + P`,

```
ext install lehni.vscode-fix-checksums
```
and press enter.

NOTE: To run as root, you must specify an alternate user data directory with the `--user-data-dir` argument.

Launch VS Code as non-root, and `Ctrl + Shidt + P`

```
ext install lehni.vscode-fix-checksums
```

and press enter. This will remove `[Unsupported]` from the title bar.

You now have the theme installed. Use at your own risk.
