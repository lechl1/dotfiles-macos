# dotfiles-macos

macOS-specific dotfiles. The cross-platform half lives in
[dotfiles-common](https://github.com/lechl1/dotfiles-common), wired in here as a
submodule at `shared/`.

```
dotfiles-macos/
├── macos/                     the macOS package
│   ├── .Brewfile              brew bundle manifest
│   ├── .paneru.toml
│   ├── .config/aerospace/     tiling WM
│   ├── .local/bin/            default-terminal, terminal-to-ghostty
│   └── Library/LaunchAgents/  local.terminal-to-ghostty.plist
└── shared/                    submodule -> dotfiles-common (common/ + stow.sh)
```

## Install

```sh
git clone --recurse-submodules https://github.com/lechl1/dotfiles-macos.git ~/.dotfiles-macos
~/.dotfiles-macos/shared/stow.sh
brew bundle --file ~/.Brewfile
```

`stow.sh` installs two packages: `common` from the submodule, and `macos` from
this repo. If the clone already exists without submodule contents,
`git submodule update --init` fills `shared/` in.

Already had these dotfiles installed from a different checkout? Add `--relink`
to repoint the existing symlinks instead of having them reported as conflicts.

## What's here

**`.Brewfile`** — the package manifest, applied with `brew bundle --file ~/.Brewfile`.

**Aerospace** — tiling window manager. `alt`+arrows to focus, `alt-shift`+arrows
to move, `alt`+digit for workspaces.

**`default-terminal`** — points LaunchServices' shell-script and unix-executable
types at Ghostty, so double-clicking a `.sh` or `.command` opens Ghostty rather
than Terminal.app. Run it once: `default-terminal` (or `default-terminal
terminal` to undo). macOS has no global "default terminal" setting; the script's
header documents what is and isn't reachable this way — notably that `ssh://`
and `x-man-page://` can't be redirected, because Ghostty declares no URL
schemes.

**`terminal-to-ghostty`** + its LaunchAgent — covers the rest: a login agent
watches for Terminal.app launching and swaps in a Ghostty window. Terminal.app
itself is on the sealed system volume and can't be modified, replaced or
wrapped, so reacting to the launch is the only avenue.

```sh
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/local.terminal-to-ghostty.plist
```

`touch ~/.terminal-to-ghostty-off` disables the swap without unloading the
agent — worth knowing about when Ghostty is what's broken.
