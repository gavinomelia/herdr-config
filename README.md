# herdr config

My [herdr](https://herdr.dev) setup for running several coding agents at once, each on its own branch.

- `config.toml`: tmux-style keys on an `alt+space` prefix, and a wide sidebar for ticket titles. It sends desktop notifications when an agent is blocked or done, and has `alt` shortcuts for [wt](https://github.com/gavinomelia/wt).
- `wt/`: the hooks that make `wt` open each worktree as a herdr workspace, plus the herdr-only commands `wt scratch`, `wt layout`, and `wt review`.

## Worktrees with wt

[wt](https://github.com/gavinomelia/wt) gives every branch its own git worktree. It's its own package and knows nothing about herdr: the hooks in `wt/` connect the two. `wt new` (alt+t) asks for a branch, a Linear issue (`ABC-1234`), or a PR (`pr:1234`). The hook then opens a workspace with:

1. a **code** tab: the agent (Claude, or Codex with alt+shift+t) on the left, an editor on the right, and a terminal along the bottom;
2. a **server** tab that runs `wt serve`, which sets the worktree up first.

For a Linear issue the agent starts by reading the issue, and for a PR, by reading the PR. A branch that already has a workspace gets it focused. A worktree whose workspace was closed gets a new one, in which Claude continues its last conversation.

| Key | Runs | What it does |
|---|---|---|
| alt+t | `wt new` | New worktree workspace (alt+shift+t uses Codex) |
| alt+w | `wt ls` | Every worktree with its PR, CI, and changes |
| alt+shift+w | `wt prune` | Removes worktrees whose PR merged, and asks about abandoned ones |
| alt+o | `wt open` | Opens this checkout's URL in the browser |
| alt+r | `wt review` | Codex reviews this branch, read-only, beside the focused pane |
| alt+s | `wt scratch` | A scratch tab on this workspace's checkout, for questions on the side |
| alt+shift+s | `wt scratch main` | A scratch workspace on the main checkout, with no branch |

How the app gets its own databases and URL per branch is project setup, not herdr. That's the [rails-postgres](https://github.com/gavinomelia/wt/tree/main/examples/rails-postgres) example in the wt repo.

## Install

```sh
git clone https://github.com/gavinomelia/herdr-config ~/src/herdr-config
cp ~/src/herdr-config/config.toml ~/.config/herdr/config.toml   # or merge the parts you want
herdr server reload-config
```

For the worktree keys, also run:

```sh
npm install -g @gavinomelia/wt
mkdir -p ~/.config/herdr/wt ~/.config/wt
cp ~/src/herdr-config/wt/* ~/.config/herdr/wt/
cat ~/.config/herdr/wt/config >> ~/.config/wt/config
```

Or point your agent at wt's [configure.md](https://github.com/gavinomelia/wt/blob/main/configure.md) and say "configure herdr to work with wt".
