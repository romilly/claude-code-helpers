# claude-code-helpers

Commands, Skills and other tools for augmenting Python development with Claude Code.

This is under rapid development, but I hope you'll find it useful as it is now.

It's opinionated. I follow much of the process described in [GOOS](https://www.amazon.co.uk/Growing-Object-Oriented-Software-Guided-Signature/dp/0321503627), but I have adapted the ideas for use with Python. I use TDD, Ports and adapters, and Alistair Cockburn's use cases.

It also contains material from Paul Hammond's [dotfiles project](https://github.com/citypaul/.dotfiles) and from [HTTP4py](https://www.http4py.org/). 


I use it in conjunction with a meta-project that creates new projects. I plan to make that public Real Soon Now(tm).

## Installation

### 1. Clone this repository

```bash
git clone https://github.com/romilly/claude-code-helpers.git
```

### 2. Copy the global CLAUDE.md

**Or merge it** with your existing version.

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
  - `hexagonal.md` - Hexagonal architecture and walking skeleton guide
  - `matchers.md` - PyHamcrest custom matchers guide
  - `builders.md` - Test data builders guide
  - `refactoring.md` - Python refactoring patterns
  - `fastapi-app.md` - FastAPI web application guide (hexagonal architecture, htmx, Playwright E2E testing)

You'll more information about how I develop with Claude, other useful projects, and what *not* to do with GenAI, on [My Ghost Site](https://the-python-programmers-path-to-software-engineering.ghost.io)

