# Test Data Builders Guide

## When to Create Builders

Create test data builders when:
- **Verbose setup code** - Tests have multi-line object construction that obscures intent
- **Nested hierarchies** - Objects contain other objects several levels deep
- **Repeated boilerplate** - Same structure appears across many tests with small variations

Don't create builders for simple dataclasses with 2-3 fields.

## Spotting Opportunities

**Test smell**: Deeply nested construction that buries the test's intent:

```python
# Before - 12 lines to set up one task
mm = MindMap(root=Node(
    id="ID_ROOT", title="Project",
    children=[Node(id="ID_TASKS", title="tasks", children=[
        Node(id="ID_LANE", title="Team", children=[
            Node(id="ID_1", title="Task A"),
        ])
    ])]
))

# After - intent is clear
mm = simple_mindmap(task("ID_1", "Task A"))
```

## The Pattern

### 1. Builder Class with Fluent API

```python
class NodeBuilder:
    """Fluent builder for Node objects."""

    def __init__(self, id: str, title: str):
        self._id = id
        self._title = title
        self._description: str | None = None
        self._children: list[Node] = []

    def with_description(self, description: str) -> "NodeBuilder":
        self._description = description
        return self

    def with_children(self, *children: "NodeBuilder | Node") -> "NodeBuilder":
        for child in children:
            if isinstance(child, NodeBuilder):
                self._children.append(child.build())
            else:
                self._children.append(child)
        return self

    def build(self) -> Node:
        return Node(
            id=self._id,
            title=self._title,
            description=self._description,
            children=self._children,
        )
```

### 2. Factory Function (Entry Point)

```python
def task(id: str, title: str) -> NodeBuilder:
    """Factory to start building a task node."""
    return NodeBuilder(id, title)
```

### 3. Convenience Function for Common Cases

```python
def simple_mindmap(*tasks: NodeBuilder | Node) -> MindMap:
    """Build a MindMap with default hierarchy, just supply the tasks."""
    return mindmap().with_lane("Team", *tasks).build()
```

### 4. Semantic Methods for Common Patterns

```python
def complete(self) -> "NodeBuilder":
    """Mark as complete (semantic alias for setting icon)."""
    self._icon = "button_ok"
    return self

def linking_to(self, target_id: str) -> "NodeBuilder":
    """Add dependency to another node."""
    self._links.append(Link(source_id=self._id, target_id=target_id))
    return self
```

## Usage in Tests

```python
from tests.builders import simple_mindmap, task

def test_child_depends_on_parent():
    mm = simple_mindmap(
        task("ID_1", "Parent").with_children(
            task("ID_2", "Child").complete()
        )
    )

    result = process(mm)

    assert result.has_dependency("ID_2", "ID_1")
```

## File Organization

```
tests/
    builders.py      # Builder classes and factory functions
    conftest.py      # Pytest fixtures using builders
    matchers.py      # Custom hamcrest matchers (separate concern)
```

## Pytest Fixtures

Use fixtures for test data that appears in multiple tests:

```python
# tests/conftest.py
import pytest
from tests.builders import simple_mindmap, task

@pytest.fixture
def single_task_mindmap():
    """A MindMap with one simple task."""
    return simple_mindmap(task("ID_1", "Task A"))

@pytest.fixture
def linked_tasks_mindmap():
    """Two tasks with a dependency."""
    return simple_mindmap(
        task("ID_1", "First").linking_to("ID_2"),
        task("ID_2", "Second"),
    )
```

## Design Principles

| Principle | Example |
|-----------|---------|
| **Sensible defaults** | Builder pre-fills required hierarchy/boilerplate |
| **Reveal intent** | `task().complete()` not `task().with_icon("button_ok")` |
| **Accept builders or objects** | `with_children(task(...))` or `with_children(Node(...))` |
| **Terminal build()** | Explicit conversion to domain object |
| **Convenience functions** | `simple_mindmap()` for 80% of cases |

## When Tests Need Different Representations

If some tests need the object and others need a serialized form (XML, JSON):

```python
def to_xml(mm: MindMap) -> str:
    """Convert MindMap to XML string for tests that need it."""
    # Generate XML from the domain object
    ...

# Usage
xml = to_xml(simple_mindmap(task("ID_1", "Task A")))
```

This keeps the builder focused on domain objects while supporting integration tests.
