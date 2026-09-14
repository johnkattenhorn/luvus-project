# luvus-project

A project picker for [luvus](https://github.com/RizRiyz/luvus) on
[Omarchy](https://omarchy.org). One key opens a menu of your projects, and each
one gets its own luvus session and its own window.

```
 curatarr                  ~/Code/curatarr
 billing-api               ~/Code/billing-api
 storefront                ~/Code/storefront
 Open the default session  ~/.luvus · the shared ledger
 Add a project…            pick a git checkout in ~/Code
```

## Why one session per project

luvus scopes its orchestration ledger, path leases and task board to the
**session**, not the workspace. Two projects sharing a session collide in ways
that are easy to miss:

- A path lease stores only its globs. A lane holding `src/**` in one repository
  refuses a lane in an unrelated one.
- `task next` claims the oldest ready task wherever you happen to be standing,
  then creates its worktree in the focused workspace. A lane from one project
  runs inside another project's checkout.
- The board has no project column and `task list` has no filter, so a shared
  board is one flat list across everything.

Reported upstream as [luvus#359](https://github.com/RizRiyz/luvus/issues/359),
with reproductions. Until the model changes, one session per project avoids all
of it, and this makes that the path of least resistance.

## Install

```bash
git clone https://github.com/johnkattenhorn/luvus-project.git
install -m755 luvus-project/luvus-project ~/.local/bin/luvus-project
```

Bind it in `~/.config/hypr/bindings.lua`:

```lua
hl.unbind("SUPER + CTRL + RETURN")
o.bind("SUPER + CTRL + RETURN", "Luvus", "luvus-project")

-- optional: straight to a project, skipping the picker
o.bind("SUPER + CTRL + SHIFT + C", "Luvus: curatarr", "luvus-project curatarr")
```

Each project runs in a window whose app-id is `org.omarchy.luvus.<name>`, so
widen any window rule that pins the plain id:

```lua
o.window({ class = "^org\\.omarchy\\.luvus(\\..+)?$" }, { workspace = "8" })
```

Do not autostart a plain `luvus` alongside this. It recreates the shared
default session the picker exists to avoid.

## Use

```bash
luvus-project            # pick a project
luvus-project curatarr   # skip the picker
luvus-project default    # the shared ~/.luvus session
luvus-project --list
```

**Ordering.** Projects are listed newest first, by when each was last worked
on, so the ones in play sit at the top and `--list` shows the same order.

"Worked on" means session activity, not commits: the ledger, task board and
leases live in the session, so it moves when you *use* a project rather than
only when you commit to it. The date is the newest file anywhere under
`~/.luvus/sessions/<name>`, which keeps a running session with a live agent at
the top. `session.json` alone is not enough — it only moves on a layout
change, so a session busy for hours can still look untouched.

A project with no session yet falls back to its checkout: the last commit, or
for a path that merely groups several checkouts the newest commit in any of
them, so a grouping directory ages with its repositories. No git history at
all falls back to the path's mtime, and a missing path sorts last. Ties keep
their configured order.

The picker always ends with two rows. **Open the default session** reaches
`~/.luvus` and its ledger, which otherwise has no route through the picker.
**Add a project…** lists the git checkouts under `~/Code` that are not
configured yet, records the one you pick, and opens it.

Pressing the key again focuses the project's existing window rather than
opening a second one, because each project has its own app-id.

## Configuration

`~/.config/luvus/projects.conf`, one project per line:

```
# <session-name> <path to the checkout>
curatarr        /home/you/Code/curatarr
billing-api     /home/you/Code/billing-api
```

Without that file, every git checkout directly under `~/Code` is offered.

| variable | default | meaning |
|---|---|---|
| `LUVUS_PROJECTS_CONFIG` | `~/.config/luvus/projects.conf` | the project list |
| `LUVUS_CODE_DIR` | `~/Code` | where **Add a project…** looks |
| `LUVUS_BIN` | `luvus` | the binary a session runs |

A session name must be one word, so **Add a project…** refuses a checkout whose
name has spaces, and skips one whose basename is already a session name.

## Requirements

Omarchy, for `omarchy-menu-select` and `omarchy-launch-or-focus-tui`. luvus on
`PATH`. bash and python3, both of which Omarchy already has.

## See also

[omarchy-luvus](https://github.com/riclib/omarchy-luvus) puts agent status in
the Omarchy bar. It pairs well with this: the bar tells you something is
waiting, the picker takes you to the project it is waiting in.

## License

MIT.
