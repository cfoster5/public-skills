# public-skills

Claude Code skills. Clone and symlink any folder into `~/.claude/skills/`:

```sh
git clone https://github.com/cfoster5/public-skills ~/public-skills
ln -s ~/public-skills/review-stack ~/.claude/skills/review-stack
```

## review-stack

`/review-stack <JIRA-KEY> [--mule <path>] [--bruno <path>] [--out <dir>]`

Reviews a stacked set of GitHub PRs with three read-only lenses plus a cross-stack pass and writes draft reviews. Lens A depends on a separate team-standards skill. After downloading, fill in the two placeholders under **Local configuration** at the top of `SKILL.md`:

```sh
sed -i '' -e 's/MULE_APP_PATTERN/(x|p|s)-yourorg-*/g' -e 's/STANDARDS_SKILL/your-standards-skill/g' ~/.claude/skills/review-stack/SKILL.md
```
