# Install shopify-theme-validator

The skill is one folder: `shopify-theme-validator`. Copy it into your agent's skills folder and the agent loads it whenever a task matches its description.

## Claude Code

```bash
mkdir -p ~/.claude/skills
cp -r shopify-theme-validator ~/.claude/skills/
```

Restart Claude Code, or run `/skills` to confirm it is listed.

## Codex CLI

```bash
mkdir -p ~/.codex/skills
cp -r shopify-theme-validator ~/.codex/skills/
```

## Cursor

```bash
mkdir -p .cursor/skills
cp -r shopify-theme-validator .cursor/skills/
```

Cursor reads project skills from `.cursor/skills/` in the project root. For a global install, use `~/.cursor/skills/` instead.

## Claude.ai

Go to Settings, then Capabilities, then Skills, and upload the `shopify-theme-validator` folder.

## Check it works

Ask your agent: "Validate this theme with the shopify-theme-validator skill."

The result should be a per-file report with errors, warnings, or a clean pass. If the agent reports a block name over 25 characters, shorten the name in the section schema and run it again.

## Updating

The free skill is updated on GitHub. Replace the folder with the new version. Your own themes and project rules live in your project, not in the skill, so an update will not overwrite them.
