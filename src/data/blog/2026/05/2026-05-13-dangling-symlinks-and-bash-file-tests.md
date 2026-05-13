---
title: Dangling Symlinks and Why `[ -e ]` Misses Them
description: How symbolic links break, why bash `[ -e ]` lies about them, and the right test operators for safely managing symlinks in install scripts.
pubDatetime: 2026-05-13T23:43:05Z
tags: [bash, symlinks, shell-scripting, dotfiles, linux]
---

A symbolic link is just a small file containing a path string. The kernel doesn't validate that the path points to anything real — creation and resolution are two separate concerns. This decoupling is what makes symlinks flexible, but it's also what produces a class of bugs that quietly sit in install scripts for years.

This post walks through what a dangling symlink is, why the most obvious bash test operator (`[ -e ]`) fails to detect them, and how to write a symlink-creation script that handles every case correctly.

## What is a dangling symlink?

A **dangling symlink** (also called a broken symlink) is a symlink whose target no longer exists.

```bash
$ echo hello > /tmp/real-file
$ ln -s /tmp/real-file /tmp/my-link
$ cat /tmp/my-link
hello                          # works — symlink resolves

$ rm /tmp/real-file
$ cat /tmp/my-link
cat: /tmp/my-link: No such file or directory   # dangling
$ ls -l /tmp/my-link
lrwxrwxrwx ... /tmp/my-link -> /tmp/real-file  # link still exists
```

The link entry on disk is unchanged. Only the target it points to has disappeared.

Common ways to end up with one:

- The symlink points into a checkout directory that was later deleted.
- The target lived on a USB drive or network mount that isn't currently attached.
- A file was renamed in the source repo, but the old symlink still references the old name.
- A previous install script created the link, then its target was cleaned up.

```mermaid
flowchart LR
    A[/tmp/my-link/] -->|points to| B[/tmp/real-file]
    B -.->|deleted| X((gone))
    A -.->|now dangling| X
```

## The `ln -sf` family and its gotchas

Before talking about how to detect dangling symlinks, it helps to understand how to create symlinks safely — because the same surprises show up here.

`ln -sf TARGET LINK_NAME` behavior depends on what's already at `LINK_NAME`:

| State of LINK_NAME | What `ln -sf` does |
|---|---|
| Does not exist | Creates a new symlink to TARGET |
| Regular file or symlink | Removes it, then creates the new symlink |
| **Directory** | Creates the symlink **inside** the directory, named after TARGET's basename — ❗ probably not what you want |
| **Symlink → directory** | Follows the symlink, then creates the new link inside the target directory — ❗ also probably not what you want |

The last two are real footguns. If you want `ln` to replace a symlink-to-a-directory atomically instead of dereferencing it, use:

```bash
ln -sfn TARGET LINK_NAME      # -n: do not follow if LINK_NAME is a symlink to a dir
# or, on GNU coreutils:
ln -sfT TARGET LINK_NAME      # -T: treat LINK_NAME as a normal file, never a directory
```

One more nuance: `-f` is not atomic. It unlinks `LINK_NAME` first, then creates the new symlink. There's a brief window where `LINK_NAME` does not exist. For truly atomic replacement:

```bash
ln -s TARGET LINK_NAME.tmp && mv -T LINK_NAME.tmp LINK_NAME
```

## Bash file tests: `-e` vs `-L`

Most install scripts begin with something like "is there already a file at this path?" The natural choice is `[ -e PATH ]`. The natural choice is wrong for symlinks.

### `[ -e PATH ]` — "exists"

Returns true if **something exists at PATH that the kernel can resolve to an inode**. **It follows symlinks.** The test is performed on the final target, not the link itself.

- Regular file present → true
- Directory present → true
- Symlink to existing target → true
- Symlink to missing target (dangling) → **false**
- Nothing at PATH → false

### `[ -L PATH ]` — "is a symlink"

Returns true if **PATH itself is a symlink**, regardless of whether its target exists. **It does NOT follow the link.** It inspects the link entry directly.

- Regular file → false
- Directory → false
- Symlink to existing target → true
- Symlink to missing target (dangling) → **true**
- Nothing at PATH → false

### Side-by-side

Setup:

```bash
touch /tmp/realfile
mkdir /tmp/realdir
ln -s /tmp/realfile /tmp/good-link
ln -s /tmp/nonexistent /tmp/bad-link
```

| PATH | `[ -e PATH ]` | `[ -L PATH ]` |
|---|---|---|
| `/tmp/realfile` | ✅ true | ❌ false |
| `/tmp/realdir` | ✅ true | ❌ false |
| `/tmp/good-link` | ✅ true | ✅ true |
| `/tmp/bad-link` | ❌ **false** | ✅ **true** |
| `/tmp/missing` | ❌ false | ❌ false |

The interesting row is `/tmp/bad-link`. `-e` reports "nothing here" because it followed the link and found no target. `-L` correctly reports "yes, a symlink exists here."

### Related operators

All of these follow symlinks by default — the same trap applies:

| Operator | Meaning | Follows symlinks? |
|---|---|---|
| `-e` | exists | yes |
| `-f` | regular file | yes |
| `-d` | directory | yes |
| `-r` / `-w` / `-x` | readable / writable / executable | yes |
| `-L` | is a symlink | **no** |
| `-h` | synonym for `-L` | **no** |

## Putting it together: a safe install script

Here's a common-looking dotfiles install snippet:

```bash
if [ -e "$HOME/.tmux.conf" ]; then
    echo "~/.tmux.conf already exists, please remove it first."
    exit 1
fi
ln -sf "$DOTFILES_DIR/tmux/.tmux.conf" "$HOME/.tmux.conf"
```

This looks defensive, but a dangling `~/.tmux.conf` symlink slips through: `-e` returns false, the guard passes, and `ln -sf` silently overwrites the broken link. Sometimes that's fine. But if your intent was "stop and let me clean up," you have a bug.

Adding `-L` covers the gap:

```bash
if [ -e "$HOME/.tmux.conf" ] || [ -L "$HOME/.tmux.conf" ]; then
    echo "~/.tmux.conf already exists, please remove it first."
    exit 1
fi
ln -sf "$DOTFILES_DIR/tmux/.tmux.conf" "$HOME/.tmux.conf"
```

Now the check catches:

- Regular files (`-e`)
- Directories (`-e`)
- Working symlinks (`-e` and `-L` both true)
- Dangling symlinks (`-L` only)

A friendlier variant — skip instead of aborting, making the script idempotent:

```bash
if [ -e "$HOME/.tmux.conf" ] || [ -L "$HOME/.tmux.conf" ]; then
    echo "~/.tmux.conf already exists, skipping."
else
    ln -sf "$DOTFILES_DIR/tmux/.tmux.conf" "$HOME/.tmux.conf"
    echo "tmux symlink created."
fi
```

## Bash `if` / `else` / `fi` cheat sheet

Easy to get wrong if you're coming from another language:

```bash
if CONDITION; then
    # runs if CONDITION is true
elif OTHER_CONDITION; then
    # runs if first was false, this one true
else
    # runs if all above were false
fi
```

Things to remember:

- Every `if` must be closed with `fi` (it's `if` spelled backwards).
- `if` and `elif` require `then`. `else` does not.
- There is **no** colon after `else` (that's Python). Just `else`.
- The semicolon before `then` is needed only because `then` is on the same line. `then` on its own line works without the semicolon.

## Takeaways

- ✅ Use `[ -L ]` (or `[ -h ]`) to test for symlinks, including broken ones.
- ✅ Combine `[ -e ] || [ -L ]` when you want to detect "anything at this path."
- ✅ Reach for `ln -sfn` when replacing a symlink that might currently point to a directory.
- ❌ Don't trust `[ -e ]` alone for "does anything exist here?" when symlinks are in play.
- ❌ Don't assume `ln -sf` is atomic — it isn't.
