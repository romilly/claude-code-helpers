# CLAUDE.md (Global)

Personal configuration for all Claude Code sessions.

## Working Together

We're colleagues working together. Be direct and honest:
- Push back if an approach seems wrong or overcomplicated
- Ask clarifying questions rather than guessing at requirements
- Say "I don't know" when uncertain rather than confabulating
- Point out when my instructions conflict or don't make sense

## Project Conventions

- Almost all Python projects have a `venv/` virtual environment in the project root - activate it before doing anything else
- Check for `requirements.txt` to understand dependencies

## General Preferences

- Prefer simple, readable solutions over clever ones
- Follow existing code patterns in the project before introducing new ones
- Keep functions and methods focused—one clear purpose each
- **Never drop database tables** unless specifically instructed. Use `CREATE TABLE IF NOT EXISTS` to preserve existing data.

## Development Approach

**Strict Test-Driven Development (TDD)** - All development follows the Red-Green-Refactor cycle:

1. **Red**: Write a failing test for new functionality
2. **Green**: Write minimal code to make the test pass
3. **Commit**: Commit the passing test and implementation
4. **Refactor** (if needed): Improve test or code while keeping tests green
5. **Retest**: Ensure all tests still pass after refactoring
6. **Commit**: Commit the refactored code
7. **Repeat**: Go back to step 4 for further refactoring or step 1 for next feature

### TDD Guidelines

**Core principle: Test behavior, not implementation.**

- **Write the simplest test that can fail first**
- **Write only enough code to make the test pass**
- **Refactor after each green test while keeping all tests passing**
- **Commit after every successful test cycle**
- **One logical change per commit** - a test and its implementation belong together
- **Test file naming**: `test_[feature_name].py`
- **Test method naming**: `test_[specific_behavior]`

### What to Test (and What Not To)

**Test behavior that matters to users and the system:**
- Business logic and domain rules
- Edge cases, error conditions, and boundary values
- Public API contracts and integration points
- State transitions and side effects

**Do NOT write tests for:**
- Dataclasses, DTOs, or simple data containers (Python handles these)
- Language features (e.g., that an ABC raises TypeError when instantiated)
- Constructor assignments or trivial property access
- Private methods (test them through public interfaces)
- Type checking that pyright already validates
- Return type validation (`assert isinstance(result, SomeClass)`) - the type signature already declares this
- "Sanity check" or "smoke test" style tests that just verify a function runs - test actual behavior instead

**Before writing a test, ask:** "If this broke in production, would anyone notice or care?" If the answer is no, skip the test.

## Project Commands

**Always activate the project's virtual environment before running tests or using project code:**

```bash
source venv/bin/activate
```

```bash
# Install dependencies
uv pip install -r requirements.txt

# Install test dependencies
uv pip install -r requirements-test.txt

# Run tests
pytest
pytest tests/test_specific.py::test_function -v
pytest -k "test_name_pattern" -v

# Run tests with coverage
coverage run -m pytest
coverage report
coverage html
```

## Testing Structure

- Test files in `tests/`
- Test data in `tests/data/`
- Test output in `tests/output/`

## Key Dependencies

- **pytest** (>=7.0.0) - Testing framework
- **coverage** - Code coverage measurement

## Static Analysis

- **pyright** - Static type checker (stricter and faster than mypy)
  - Run `pyright src/` before committing
  - Fix type errors - don't ignore them

## Testing Style

- **pyhamcrest** - Matcher library for expressive assertions
  - Prefer `assert_that(result, has_length(3))` over `assert len(result) == 3`
  - Common matchers: `equal_to`, `contains_string`, `has_item`, `has_length`, `empty`, `is_not`
  - Plain `assert` is fine for simple equality: `assert result == expected`

### Custom Matchers

Create custom matchers in `tests/matchers.py` when **multiple tests** check the same object properties. For the pattern and examples, run `/matchers`.
