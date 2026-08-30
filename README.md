# dotfiles-macos

macOS-specific dotfiles. The cross-platform half lives in
[dotfiles-common](https://github.com/lechl1/dotfiles-common), wired in here as a
submodule at `common/`.

```
dotfiles-macos/                the stow directory
├── common/                    submodule -> dotfiles-common (flat: it IS the package)
└── macos/                     the macOS package
    ├── .Brewfile              brew bundle manifest
    ├── .paneru.toml
    ├── .config/aerospace/     tiling WM
    ├── .local/bin/            default-terminal, terminal-to-ghostty
    └── Library/LaunchAgents/  local.terminal-to-ghostty.plist
```

Both packages sit directly in the repo root, which is what GNU Stow expects, so
either installer works.

## Install

```sh
git clone --recurse-submodules https://github.com/lechl1/dotfiles-macos.git ~/.dotfiles-macos
cd ~/.dotfiles-macos

stow -t ~ common macos      # GNU Stow
./common/stow.sh            # or the bundled installer

brew bundle --file ~/.Brewfile
```

`--recurse-submodules` matters: `macos/Library/Fonts/CodeNewRoman.otf` is a link
into `common/fonts/`, and an empty `common/` leaves it dangling.

The bundled `common/stow.sh` is not a reimplementation for its own sake — it
links file by file instead of folding directories, copies fonts into
`~/Library/Fonts` (CoreText ignores symlinked fonts), and absorbs pre-existing
real files rather than refusing. Its `--relink` flag repoints links that already
exist but aim elsewhere, which is what you want when an older checkout is being
replaced. See the dotfiles-common README for the details.

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
