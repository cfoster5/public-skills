# public-skills

Claude Code skills. Clone and symlink any folder into `~/.claude/skills/`:

```sh
git clone https://github.com/cfoster5/public-skills ~/public-skills
ln -s ~/public-skills/review-stack ~/.claude/skills/review-stack
```

## review-stack

`/review-stack <JIRA-KEY> [--mule <path>] [--bruno <path>] [--out <dir>]`

Reviews a stacked set of GitHub PRs with three read-only lenses plus a cross-stack pass and writes draft reviews. Lens A depends on a separate `onb-integration` skill being installed.
