# herdr config

My [herdr](https://herdr.dev) setup for running several coding agents at once, each on its own branch of a Rails app.

- `config.toml`: tmux-style keys on an `alt+space` prefix, a wide sidebar for ticket titles, desktop notifications when an agent is blocked or done, and `alt` shortcuts for `wt`.
- `bin/wt`: gives every branch its own worktree, herdr workspace, databases, and browser URL.

## What `wt` does

`wt new` (alt+t) asks for a branch, a Linear issue (`ABC-1234`), or a PR (`pr:1234`), then:

1. creates the worktree and a herdr workspace for it;
2. opens a **code** tab: the agent (Claude, or Codex with `WT_AGENT=codex`) on the left, an editor on the right, a terminal along the bottom;
3. opens a **server** tab that runs `wt setup`, then `wt serve`.

For a Linear issue the agent starts by reading the issue. For a PR it starts by reading the PR.

`wt setup` gives the worktree:

- its own dev database, cloned from the main one (near instant on btrfs);
- its own test databases, one per CPU for parallel specs;
- symlinked credentials keys, and copies of `node_modules` and built assets;
- optionally, a dev-only initializer that signs the browser straight in as one user.

Each worktree's databases are named in its `.env_overrides.rb`.

| Command | Key | What it does |
|---|---|---|
| `wt new [target] [base]` | alt+t | New worktree workspace (alt+shift+t uses Codex) |
| `wt ls` | alt+w | Every worktree with its PR, CI, uncommitted files, commits behind and ahead of main, unpushed commits, and age |
| `wt prune` | alt+shift+w | Removes clean worktrees whose PR merged and deletes their branches; asks about ones whose PR closed or that sat idle; deletes merged branches without a worktree |
| `wt gc [--dry-run]` | | Drops every database no checkout names, however the checkout was removed |
| `wt rm [--force] <branch>` | | Closes the workspace, removes the worktree, drops its databases |
| `wt open` | alt+o | Opens this checkout's URL in the browser |
| `wt review [base]` | alt+r | Starts a read-only Codex review of this branch beside the focused pane |
| `wt scratch` | alt+s | A workspace on the main checkout with no branch, for questions |
| `wt snapshot` | | Refreshes the idle dev-database copy new worktrees clone from |

Run `wt` with no arguments for the full help.

## Requirements

- herdr, `git`, `gh` (signed in), `jq`, `psql`, `foreman`
- [portless](https://github.com/vercel-labs/portless) for the per-branch `https://<branch>.<app>.localhost` URLs
- PostgreSQL 18 on btrfs for instant database clones; elsewhere the clone is a full copy
- The Claude Code CLI with the Linear connector, for `wt new ABC-1234`
- A Rails app that:
  - loads a git-ignored `.env_overrides.rb` before its env defaults;
  - reads its database names from `DATABASE_NAME` and `DATABASE_TEST_NAME` (plus `TEST_ENV_NUMBER` for the test database);
  - allows `ENV['ADDITIONAL_HOST']` in `config.hosts`.

  The app's main checkout must use `<prefix>_development` and `<prefix>_test`.

## Install

```sh
git clone https://github.com/gavinomelia/herdr-config ~/src/herdr-config
ln -s ~/src/herdr-config/bin/wt ~/.local/bin/wt
cp ~/src/herdr-config/wt.env.example ~/.config/herdr/wt.env   # then fill it in
cp ~/src/herdr-config/config.toml ~/.config/herdr/config.toml # or merge the parts you want
herdr server reload-config
```

`app/Procfile` runs the web server, `yarn build --watch`, and GoodJob. Point `WT_PROCFILE` at your own copy if your app runs something else. `app/worktree_auto_login.rb` assumes a Devise model called `UserIdentity`; change it to match your app, or leave `WT_AUTO_LOGIN_EMAIL` empty.

## Cleaning up

`wt gc` runs inside `wt new`, `wt rm`, and `wt prune`. It keeps the main checkout's databases, the snapshot, and whatever each worktree's `.env_overrides.rb` names, and drops every other database starting with your prefix. Anything else that makes checkouts of the app must write its database names to `.env_overrides.rb` before creating them, or `wt gc` will drop them.
