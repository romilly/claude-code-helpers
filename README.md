# claude-code-helpers

Commands, Skills and other tools for augmenting Claude Code.

## Project Structure

This project contains shared resources for Claude Code:

- **Tools**: Python tools and utilities are located in `src/`
- **Skills**: Reusable skills are stored in `resources/skills/`
- **Commands**: Custom commands are stored in `resources/commands/`

## Installation

```bash
pip install -e .
```

## Development

```bash
pip install -e .[test]
pytest
```