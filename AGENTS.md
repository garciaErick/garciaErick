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
It uses **`garciaErick/Github-Language-Stats@main`** — a fork of
[`StefVuck/Github-Language-Stats`](https://github.com/StefVuck/Github-Language-Stats)
(local clone: `~/2-repos/Github-Language-Stats`) patched to add an
`extra_repos` input, since upstream only analyzes repos with
`affiliation='owner'` (personal repos — organization repos are invisible
to it). The fork adds: `extra_repos`, which fetches extra repos by
`owner/repo` full name and appends them to the analysis set.

The workflow config:

- `extra_repos: "Ashfall-Software/brews-n-battles"` — the owner's Godot
  game (private org repo, primary language GDScript)
- `exclude_repos` — Obsidian vaults (`tsunderelkasten`,
  `tsunderelkasten-pipboy`, whose committed `.obsidian/plugins/*/main.js`
  bundles previously flooded the stats with ~50MB of third-party JS),
  template repos (`ci-skeleton-3` with 1.8MB of PHP, `Sails.js-template`),
  and TypeScript-heavy site repos (`quartz-tsunderick-themes`, ~80%
  vendored Quartz framework source; `blog.tsunderick.space`, ~400KB of
  generated test fixtures counted as code)

The action then:

1. Query the GitHub API for every repository owned by the token owner
   plus `extra_repos` (public + private; forks and HTML/CSS excluded)
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

## Maintenance: PAT Rotation

The workflow authenticates via the `STATS_TOKEN` repository secret — a
**classic Personal Access Token with `repo` scope and no expiration**.

No scheduled rotation is required. Rotate only if the token is compromised
or revoked. Note that a token's expiration cannot be edited after creation
— "rotating" always means creating a new PAT and updating the secret.

Rotation procedure:

1. Create a fresh classic PAT at <https://github.com/settings/tokens>
   (scope: `repo`)
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

If the chart on the profile page looks stale, check for a failed workflow
run first — an expired or revoked `STATS_TOKEN` is the most likely cause.

Security note: a never-expiring classic PAT with `repo` scope grants
standing read/write access to every repo the account owns. Treat the
token value accordingly.

## Conventions

- Commit directly to `main`; no PRs for this repo.
- Do **not** add a `push` trigger to the stats workflow — the action
  commits to `main` itself, and a push trigger can cause re-run loops.
- Keep `github-stats/` out of `.gitignore` — the action will not publish
  images from an ignored path.
- GitHub auto-disables scheduled workflows after 60 days of repo
  inactivity; the bot's own daily commits keep this one alive.
