# PyHamcrest Custom Matchers Guide

## When to Create a Custom Matcher

Create a custom matcher when:
- **Multiple tests** check the same object's properties
- You want **clearer failure messages** than plain assertions provide

Don't create a matcher for a one-off assertion - plain `assert` is fine.

## Spotting Opportunities

**Test smell**: Length assertion followed by individual item checks:

```python
# Before - verbose and redundant length check
assert len(tasks_node.children) == 2
assert tasks_node.children[0].id == "ID_1"
assert tasks_node.children[0].title == "Task A"
assert tasks_node.children[1].id == "ID_2"
assert tasks_node.children[1].title == "Task B"

# After - expressive, no length check needed
assert_that(tasks_node.children, contains_exactly(
    node(id="ID_1", title="Task A"),
    node(id="ID_2", title="Task B"),
))
```

The `contains_exactly` matcher handles "exactly these items, no more, no less" - eliminating the redundant length assertion.

## Naming Convention

Name matcher functions after the type they match, without an `is_` prefix:
- `node(...)` not `is_node(...)`
- `link(...)` not `is_link(...)`

This reads naturally with `contains_exactly`: "contains exactly node with id X, node with id Y".

## The Pattern

```python
"""Custom PyHamcrest matchers for the test suite."""

from hamcrest.core.base_matcher import BaseMatcher
from hamcrest.core.description import Description

from myproject.domain import MyObject


class IsMyObjectMatcher(BaseMatcher):
    """Matcher for MyObject with optional property checks."""

    def __init__(self, name: str | None = None, value: int | None = None):
        self.expected_name = name
        self.expected_value = value

    def _matches(self, item) -> bool:
        if not isinstance(item, MyObject):
            return False
        if self.expected_name is not None and item.name != self.expected_name:
            return False
        if self.expected_value is not None and item.value != self.expected_value:
            return False
        return True

    def describe_to(self, description: Description) -> None:
        parts = ["a MyObject"]
        if self.expected_name is not None:
            parts.append(f"with name={self.expected_name!r}")
        if self.expected_value is not None:
            parts.append(f"with value={self.expected_value}")
        description.append_text(" ".join(parts))

    def describe_mismatch(self, item, mismatch_description: Description) -> None:
        if not isinstance(item, MyObject):
            mismatch_description.append_text(f"was {type(item).__name__}")
        elif self.expected_name is not None and item.name != self.expected_name:
            mismatch_description.append_text(f"had name={item.name!r}")
        elif self.expected_value is not None and item.value != self.expected_value:
            mismatch_description.append_text(f"had value={item.value}")


def my_object(name: str | None = None, value: int | None = None):
    """Match a MyObject with optional property checks."""
    return IsMyObjectMatcher(name=name, value=value)
```

## Usage in Tests

```python
from hamcrest import assert_that, contains_exactly, has_item
from tests.matchers import my_object

def test_creates_objects():
    result = create_objects()

    # Check exact list contents
    assert_that(result, contains_exactly(
        my_object(name="first", value=1),
        my_object(name="second", value=2),
    ))

    # Or check list contains an item
    assert_that(result, has_item(my_object(name="first")))
```

## File Location

Place custom matchers in `tests/matchers.py` at the project root.

## Key Methods

| Method | Purpose |
|--------|---------|
| `_matches(item)` | Return `True` if item matches, `False` otherwise |
| `describe_to(description)` | Describe what you expected (for failure messages) |
| `describe_mismatch(item, desc)` | Describe what you actually got (for failure messages) |

## Collection Wrapper Types

When you have a domain collection type (not a raw `list`), create a matcher for it too. This avoids pyright type errors with `contains_inanyorder` and keeps your domain model clean.

```python
# Domain model
class FunctionList:
    """A collection of FunctionInfo objects."""

    def __init__(self, functions: list[FunctionInfo]):
        self._functions = functions

    def __iter__(self):
        return iter(self._functions)

    def __len__(self):
        return len(self._functions)


# Matcher for the collection
class IsFunctionListMatcher(BaseMatcher):
    """Matcher for FunctionList containing specific functions (any order)."""

    def __init__(self, *matchers: IsFunctionInfoMatcher):
        self.matchers = matchers

    def _matches(self, item) -> bool:
        if not isinstance(item, FunctionList):
            return False
        functions = list(item)
        if len(functions) != len(self.matchers):
            return False
        # Each matcher must match exactly one function
        unmatched = functions.copy()
        for matcher in self.matchers:
            found = None
            for func in unmatched:
                if matcher._matches(func):
                    found = func
                    break
            if found is None:
                return False
            unmatched.remove(found)
        return True

    def describe_to(self, description: Description) -> None:
        description.append_text("a FunctionList containing ")
        for i, matcher in enumerate(self.matchers):
            if i > 0:
                description.append_text(", ")
            matcher.describe_to(description)

    def describe_mismatch(self, item, mismatch_description: Description) -> None:
        if not isinstance(item, FunctionList):
            mismatch_description.append_text(f"was {type(item).__name__}")
        else:
            mismatch_description.append_text(f"was FunctionList with {list(item)}")


def function_list(*matchers: IsFunctionInfoMatcher):
    """Match a FunctionList containing the specified functions (any order)."""
    return IsFunctionListMatcher(*matchers)
```

Usage:

```python
result = extract_function_infos(source)
assert_that(result, function_list(
    function_info(name="foo", calls=["bar", "baz"]),
    function_info(name="bar", calls=[]),
))
```

## Useful Hamcrest Matchers to Combine With

- `contains_exactly(...)` - list has exactly these items in order
- `contains_inanyorder(...)` - list has exactly these items, any order
- `has_item(...)` - list contains at least this item
- `has_length(n)` - collection has n items
- `empty()` - collection is empty
- `equal_to({...})` - for set comparisons, use with a set literal

## Comparing Sets

When comparing sets, use `equal_to` with a set literal rather than `contains_inanyorder`:

```python
# Good - clear and pyright-friendly
assert_that(names, equal_to({"foo", "MyClass", "MyClass.run"}))

# Avoid - causes pyright type errors with sets
assert_that(names, contains_inanyorder("foo", "MyClass", "MyClass.run"))
```

The `contains_inanyorder` matcher is designed for sequences; `equal_to` with a set literal is the correct choice for set equality.
