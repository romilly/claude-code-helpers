# Python Refactoring Guide

## Code Smells and Their Refactorings

### 1. isinstance Chains → Dictionary Dispatch

**Smell**: Multiple `isinstance` checks to handle different types:

```python
# Before - isinstance chain
for node in ast.walk(tree):
    if isinstance(node, ast.ClassDef):
        handle_class(node, defined, graph)
    elif isinstance(node, ast.FunctionDef):
        handle_function(node, defined, graph)
```

**Refactoring**: Use a dictionary mapping types to handler functions:

```python
# After - dictionary dispatch
_node_handlers = {
    ast.ClassDef: _handle_class_def,
    ast.FunctionDef: _handle_function_def,
}

for node in ast.walk(tree):
    handler = _node_handlers.get(type(node))
    if handler:
        handler(node, defined, graph)
```

**Benefits**:
- Open for extension (add new types without modifying dispatch logic)
- Explicit mapping of types to behavior
- Unknown types handled uniformly (silently skipped or raise, your choice)

---

### 2. Complex Boolean Conditions → Named Helper Functions

**Smell**: Long boolean expression that's hard to understand:

```python
# Before - inline complex condition
if (type(func) is ast.Attribute
    and type(func.value) is ast.Name
    and func.value.id == "self"
    and class_name is not None):
    # handle self.method() call
```

**Refactoring**: Extract to a well-named function. Use `TypeGuard` when the condition narrows a type:

```python
# After - named helper with TypeGuard
from typing import TypeGuard

def _is_self_method_call(func: ast.expr, class_name: str | None) -> TypeGuard[ast.Attribute]:
    """Check if func is a self.method() call within a class."""
    return (type(func) is ast.Attribute
            and type(func.value) is ast.Name
            and func.value.id == "self"
            and class_name is not None)

# Usage - type checker knows func is ast.Attribute after this
if _is_self_method_call(func, class_name):
    qualified = f"{class_name}.{func.attr}"  # No assert needed
```

**Benefits**:
- Self-documenting code (the function name explains intent)
- TypeGuard eliminates need for asserts or casts
- Reusable across multiple call sites

---

### 3. Functions with Shared State → Class with Methods

**Smell**: Multiple functions passing the same state through parameters:

```python
# Before - functions passing state around
def build_graph(source: str) -> dict[str, set[str]]:
    defined = get_defined_names(source)
    graph: dict[str, set[str]] = {}
    for node in ast.walk(tree):
        handler = handlers.get(type(node))
        if handler:
            handler(node, defined, graph)  # passing state
    return graph

def _handle_class_def(node, defined, graph):  # receiving state
    # uses defined and graph
    ...
```

**Refactoring**: Create a class with state as fields:

```python
# After - class with state as fields
class GraphBuilder:
    def __init__(self, source: str):
        self._tree = ast.parse(source)
        self._defined = get_defined_names(source)
        self._graph: dict[str, set[str]] = {}

    def build(self) -> dict[str, set[str]]:
        for node in ast.walk(self._tree):
            self._handle_node(node)
        return self._graph

    def _handle_node(self, node: ast.AST) -> None:
        handler = self._node_handlers.get(type(node))
        if handler:
            handler(self, node)

    def _handle_class_def(self, node: ast.ClassDef) -> None:
        # Access self._defined and self._graph directly
        ...
```

**Benefits**:
- Methods have fewer parameters
- State is encapsulated
- Related behavior is grouped together

---

### 4. Accumulator Parameter → Instance Field

**Smell**: Passing a mutable collection through multiple methods to accumulate results:

```python
# Before - passing accumulator as parameter
def _extract_calls(self, node, class_name) -> set[str]:
    calls: set[str] = set()
    for child in ast.walk(node):
        if _is_call(child):
            self._handle_call(child, class_name, calls)  # passing calls
    return calls

def _handle_call(self, call, class_name, calls):  # receiving calls
    calls.add(...)
```

**Refactoring**: Make the accumulator a field, reset at start of operation:

```python
# After - accumulator as field
def __init__(self, source: str):
    ...
    self._calls: set[str] = set()

def _extract_calls(self, node, class_name) -> None:
    self._calls = set()  # reset at start
    for child in ast.walk(node):
        if _is_call(child):
            self._handle_call(child, class_name)

def _handle_call(self, call, class_name) -> None:
    self._calls.add(...)  # access field directly
```

**Note**: This makes the class not thread-safe. Document this if relevant.

---

### 5. Inline Handler Lookup → Named Method

**Smell**: Handler lookup and dispatch logic inline in a loop:

```python
# Before - inline dispatch
for node in ast.walk(tree):
    handler = self._handlers.get(type(node))
    if handler:
        handler(self, node)
```

**Refactoring**: Extract to a named method:

```python
# After - named dispatch method
def build(self) -> dict[str, set[str]]:
    for node in ast.walk(self._tree):
        self._handle_node(node)
    return self._graph

def _handle_node(self, node: ast.AST) -> None:
    """Dispatch node to appropriate handler if one exists."""
    handler = self._node_handlers.get(type(node))
    if handler:
        handler(self, node)
```

**Benefits**:
- Main loop is cleaner and more readable
- Dispatch logic is isolated and testable
- Method name documents intent

---

### 6. Inline Type Check → Named Predicate

**Smell**: `type(x) is SomeType` scattered through code:

```python
# Before - inline type checks
if type(child) is ast.FunctionDef:
    ...
if type(child) is ast.Call:
    ...
```

**Refactoring**: Extract to named predicates with TypeGuard:

```python
# After - named predicates
def _is_function_def(node: ast.stmt) -> TypeGuard[ast.FunctionDef]:
    return type(node) is ast.FunctionDef

def _is_call(node: ast.AST) -> TypeGuard[ast.Call]:
    return type(node) is ast.Call

# Usage
if _is_function_def(child):
    ...  # child is now typed as ast.FunctionDef
```

---

## When to Apply These Refactorings

Apply refactorings **after tests pass** (Green phase in TDD). The cycle is:

1. **Red**: Write failing test
2. **Green**: Make test pass with minimal code
3. **Refactor**: Improve code while keeping tests green
4. **Commit**: Commit the working, refactored code

Always run tests after each refactoring step to ensure behavior is preserved.
