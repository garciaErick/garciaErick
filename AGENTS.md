# AGENTS.md

Guidance for AI coding agents working in this repository.

## What This Repo Is

This is `garciaErick/garciaErick` — the GitHub **profile repository**. Its
`README.md` renders on the owner's GitHub profile page. There is no
application code, build system, or test suite here.

## Repository Layout

| Path                          | Purpose                          | Hand-edit?         |
| ----------------------------- | -------------------------------- | ------------------ |
| `README.md`                   | Profile page content             | Yes                |
| `sailor_wet.png`              | Profile image asset              | Yes                |
| `.github/workflows/stats.yml` | Language stats automation        | Yes                |
| `github-stats/`               | **Generated** visualization PNGs | **Never**          |

## Language Stats Workflow

`.github/workflows/stats.yml` runs daily (00:00 UTC) and on manual dispatch.
It uses the third-party action
[`StefVuck/Github-Language-Stats@v1.2.0`](https://github.com/StefVuck/Github-Language-Stats)
to:

1. Query the GitHub API for every repository owned by the token owner
   (public + private; forks and HTML/CSS are excluded)
2. Shallow-clone each repo and count real lines of code (`use_loc: true`)
3. Render a dark-mode leaderboard chart into `github-stats/`
4. Commit and push the PNGs back to `main` as `github-actions[bot]`

Output files (all generated; the next run regenerates them):

- `github-stats/leaderboard_by_repos.png` — ranked by repository count
- `github-stats/leaderboard_by_lines.png` — ranked by lines of code
  (**this is the one embedded in the README**)
- `github-stats/leaderboard_by_weighted.png` — balanced ranking

Note: with `use_loc: true` the volume suffix is `lines`. Disabling `use_loc`
renames that file to `leaderboard_by_bytes.png` **and breaks the README
embed** — update `README.md` if that config ever changes.

## Maintenance: PAT Rotation (CRITICAL)

The workflow authenticates via the `STATS_TOKEN` repository secret — a
**classic Personal Access Token with `repo` scope and a 7-day expiration**.

> **Every 7 days the scheduled run starts failing (401/403) until the
> secret is rotated.** A stale chart on the profile page most likely means
> an expired `STATS_TOKEN`.

Rotation procedure:

1. Create a fresh classic PAT at <https://github.com/settings/tokens>
   (scope: `repo`, expiration: 7 days)
2. Update the secret (never write the token to a file):
   ```bash
   gh secret set STATS_TOKEN --repo garciaErick/garciaErick
   # paste the new PAT when prompted
   ```
3. Verify a run passes:
   ```bash
   gh workflow run "Update Language Statistics" --repo garciaErick/garciaErick
   gh run watch --repo garciaErick/garciaErick
   ```

## Conventions

- Commit directly to `main`; no PRs for this repo.
- Do **not** add a `push` trigger to the stats workflow — the action
  commits to `main` itself, and a push trigger can cause re-run loops.
- Keep `github-stats/` out of `.gitignore` — the action will not publish
  images from an ignored path.
- GitHub auto-disables scheduled workflows after 60 days of repo
  inactivity; the bot's own daily commits keep this one alive.
