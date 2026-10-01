# omarchy-nix plugins

NixOS port of Omarchy's `bin/omarchy-plugin-*` + `bin/omarchy-git-url-check`
(`./omarchy/bin/`), adapted to this repo's Quickshell desktop
(`./nixos/quickshell/`, `./nixos/plugins/default.nix`).

## Why not use Omarchy's scripts directly?

They assume Arch Linux + the `omarchy-shell` daemon: `$OMARCHY_PATH/shell/plugins`,
`~/.config/omarchy/plugins`, `~/.config/omarchy/shell.json`,
`omarchy-shell shell listPlugins/enablePlugin/rescanPlugins` IPC, `gum`
prompts, and `pacman`/AUR helpers. None of that exists on NixOS. These ports
keep the upstream flow (stage -> validate -> id-collision check -> install;
fast-forward-only update with rollback; symlink/git/hand-made remove branches)
and swap the backend for files the shell already watches:

- discovery: manifests under `$NIX_QUICKSHELL_SOURCE_DIR`
  (default `/etc/nixos/quickshell/plugins`) + `$NIX_QUICKSHELL_PLUGINS_DIR`
  (default `~/.config/quickshell/plugins`)
- enabled state: `$NIX_QUICKSHELL_LAYOUT_FILE`
  (default `~/.config/quickshell/plugin-layout.json`)
- `gum` prompts -> plain `read` prompts; `notify-send` instead of
  `omarchy-notification-send`

`omarchy-plugin-validate` and `omarchy-git-url-check` are verbatim upstream
logic (shebang only). Per-script deltas are in each file's header comment.

## Push

```bash
cd /etc/omarchy-nix-build-test
git init && git add -A && git commit -m "Port Omarchy plugin lifecycle to NixOS Quickshell"
gh repo create <user>/omarchy-nix-plugins --source=. --push
```

## Declarative use (survives rebuilds)

`add` prints a `plugins/default.nix` entry for the cloned repo. Adding it
there + `nixos-rebuild switch` is the permanent route (`fetchFromGitHub`,
pinned rev+hash); the `manual` registry row these scripts write is
immediate but regenerated away on rebuild -- same caveat as
`nixos/quickshell/scripts/plugin.sh` documents.
