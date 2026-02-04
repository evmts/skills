# Voltaire Skills

Cross-platform AI agent skills for [Voltaire](https://voltaire.tevm.sh) and [Voltaire-Effect](https://voltaire-effect.tevm.sh).

## Skills

| Skill | Description |
|-------|-------------|
| `voltaire` | Core Ethereum primitives library (Address, Uint256, Hex, ABI, RLP, etc.) |
| `voltaire-effect` | Effect.ts integration with typed errors, services, and contract patterns |

## Installation

### Claude Code

```bash
# User-level installation
git clone https://github.com/evmts/voltaire-skills ~/.claude/skills/voltaire-skills

# Or symlink individual skills
ln -s ~/.claude/skills/voltaire-skills/voltaire ~/.claude/skills/voltaire
ln -s ~/.claude/skills/voltaire-skills/voltaire-effect ~/.claude/skills/voltaire-effect
```

### OpenAI Codex

```bash
git clone https://github.com/evmts/voltaire-skills ~/.codex/skills/voltaire-skills
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

### Codex
```
$voltaire
$voltaire-effect
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
