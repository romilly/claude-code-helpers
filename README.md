# claude-code-helpers

Commands, Skills and other tools for augmenting Python development with Claude Code.

I use it in conjunction with a project that creates projects. I plan to make that public Real Soon Now(tm).

## Installation

### 1. Clone this repository

```bash
git clone https://github.com/romilly/claude-code-helpers.git
```

### 2. Copy the global CLAUDE.md

```bash
cp claude-code-helpers/resources/root-CLAUDE.md ~/.claude/CLAUDE.md
```

### 3. Copy the commands

```bash
cp -r claude-code-helpers/resources/commands ~/.claude/commands
```

## What's Included

- `resources/root-CLAUDE.md` - Global Claude Code configuration with TDD practices
- `resources/commands/` - Slash commands:
  - `matchers.md` - PyHamcrest custom matchers guide
  - `builders.md` - Test data builders guide
  - `refactoring.md` - Python refactoring patterns