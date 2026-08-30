# dotfiles-macos

macOS-specific dotfiles. The repo is **flat** — its root *is* the macOS package,
so `.Brewfile` installs to `~/.Brewfile`. The shared half lives in
[dotfiles-dev](https://github.com/lechl1/dotfiles-dev) (private), mounted here
as a submodule at `common/`.

```
dotfiles-macos/
├── .Brewfile              brew bundle manifest
├── .paneru.toml
├── .config/aerospace/     tiling WM
├── .local/bin/            default-terminal, terminal-to-ghostty
├── Library/
│   ├── Fonts/             links into common/fonts/ — copied in, not linked
│   └── LaunchAgents/      local.terminal-to-ghostty.plist
├── .stow-os               names this package's OS to common/stow.sh
└── common/                submodule -> dotfiles-dev (flat: it IS the package)
```

## Install

```sh
git clone --recurse-submodules git@github.com:lechl1/dotfiles-macos.git ~/.dotfiles-macos
~/.dotfiles-macos/common/stow.sh
brew bundle --file ~/.Brewfile
```

`stow.sh` installs both packages in one go: the shared config from `common/`,
and this repo because `.stow-os` says `macos` and that matches the machine.

`--recurse-submodules` matters: `Library/Fonts/*` links into `common/fonts/`, so
an empty `common/` leaves the fonts dangling. Since dotfiles-dev is private, the
submodule only populates for accounts that can read it.

### With plain GNU Stow

Both packages are stowable, but they now live in *different* stow directories —
this repo's root is one package, and `common/` inside it is the other — so it
takes two invocations, and both need `--no-folding`:

```sh
cd ~
stow --no-folding -d ~ -t ~ .dotfiles-macos
stow --no-folding -d ~/.dotfiles-macos -t ~ common
```

`--no-folding` is not optional here. Without it Stow folds `~/.config` into a
single symlink to whichever package it stows first, and the second package then
fails with `cannot read directory: ../.dotfiles-macos/.config/.config` — Stow's
unfolding assumes every package shares one stow directory, which a flat repo
plus a nested submodule breaks. `--no-folding` links file by file instead, which
is what `stow.sh` does natively.

## What's here

**`.Brewfile`** — the package manifest, applied with `brew bundle --file ~/.Brewfile`.

**Aerospace** — tiling window manager. `alt`+arrows to focus, `alt-shift`+arrows
to move, `alt`+digit for workspaces.

**Fonts** — the DejaVu Sans Mono Nerd Font faces Ghostty uses, plus
CodeNewRoman. These are links into `common/fonts/`, and `stow.sh` *copies* them
into `~/Library/Fonts` rather than linking, because CoreText silently ignores
symlinked fonts. They are carried in the repo instead of installed by a cask, so
the terminal config doesn't depend on a package outside it.

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
