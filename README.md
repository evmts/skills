# Voltaire Skills

Cross-platform AI agent skills for [Voltaire](https://voltaire.tevm.sh) and [Voltaire-Effect](https://voltaire-effect.tevm.sh).

## Skills

| Skill | Description |
|-------|-------------|
| `voltaire` | Core Ethereum primitives library (Address, Uint256, Hex, ABI, RLP, etc.) |
| `voltaire-effect` | Effect.ts integration with typed errors, services, and contract patterns |

## Installation

### Claude Code

Skills are stored in `~/.claude/skills/` (personal) or `.claude/skills/` (project-level).

```bash
# Personal installation (available in all projects)
git clone https://github.com/evmts/skills ~/.claude/skills/voltaire-skills
```

Or symlink individual skills:
```bash
ln -s ~/.claude/skills/voltaire-skills/voltaire ~/.claude/skills/voltaire
ln -s ~/.claude/skills/voltaire-effect ~/.claude/skills/voltaire-effect
```

For project-level (just this repo):
```bash
git clone https://github.com/evmts/skills .claude/skills/voltaire-skills
```

### OpenAI Codex

Codex uses `AGENTS.md` files for instructions. Add to your project root or `~/.codex/`:

```bash
# Personal installation
git clone https://github.com/evmts/skills ~/.codex/skills/voltaire-skills

# Then reference in your AGENTS.md:
# See ~/.codex/skills/voltaire-skills/voltaire/SKILL.md for Voltaire usage
```

Or copy skill content directly into your `AGENTS.md`:
```bash
cat ~/.codex/skills/voltaire-skills/voltaire/SKILL.md >> AGENTS.md
```

### Amp (Sourcegraph)

```bash
# Project-level
mkdir -p .agents/skills
git clone https://github.com/evmts/voltaire-skills .agents/skills/voltaire-skills
```

### Cursor

```bash
mkdir -p .cursor/rules

# Download MDC rule files
curl -o .cursor/rules/voltaire.mdc \
  https://raw.githubusercontent.com/evmts/voltaire-skills/main/voltaire/voltaire.mdc

curl -o .cursor/rules/voltaire-effect.mdc \
  https://raw.githubusercontent.com/evmts/voltaire-skills/main/voltaire-effect/voltaire-effect.mdc
```

### As Git Submodule (for library maintainers)

```bash
git submodule add https://github.com/evmts/voltaire-skills skills
```

## Usage

### Claude Code
```
/voltaire         # Load Voltaire primitives guidance
/voltaire-effect  # Load Effect.ts integration guidance
```

Skills auto-invoke when Claude detects relevant context.

### Codex
Reference the skill content in your `AGENTS.md` or include via comments:
```
# See voltaire skill for Ethereum primitives patterns
```

### Amp
Skills auto-load based on context.

### Cursor
Rules auto-apply to relevant files.

## Links

- [Voltaire Docs](https://voltaire.tevm.sh)
- [Voltaire-Effect Docs](https://voltaire-effect.tevm.sh)
- [GitHub - Voltaire](https://github.com/evmts/voltaire)
- [Effect.ts](https://effect.website)

## License

MIT
