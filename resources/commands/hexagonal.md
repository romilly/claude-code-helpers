# Hexagonal Architecture and Walking Skeleton

## Architecture Principles

Use hexagonal (ports and adapters) architecture:

- **Domain logic**: Completely independent of I/O, persistence, external systems
- **Ports**: Abstract interfaces (Python ABCs) defining boundaries
- **Adapters**: Implement ports for specific technologies
- Dependencies flow inward: adapters depend on ports, never reverse

Structure:
```
src/project_name/
├── domain/           # Pure business logic, no I/O
├── ports/            # Abstract interfaces (ABCs)
└── adapters/         # Implementations (fake, real)
```

## Walking Skeleton

1. Define the use case (Cockburn style: system, actor, goal, main scenario, extensions)
2. Build minimal end-to-end with fake adapters returning hardcoded results
3. Verify with end-to-end tests in clean environment
4. Iterate with TDD, replacing fakes incrementally

The skeleton proves the architecture works before investing in real functionality.

## Testing: Fake Adapters, Not Monkeypatching

**Never monkeypatch** (pyfakefs, unittest.mock.patch for dependencies):
- Creates invisible dependencies
- Tests can pass without verifying anything meaningful
- "Action at a distance" — setup disconnected from calls

**Use explicit dependency injection:**
- Ports as ABCs with explicit signatures
- Fake adapters implementing the port interface
- Constructor injection so dependencies are visible
- Contract tests verifying fakes and real adapters both satisfy ports

Fake adapters must be fully functional, just using in-memory storage.
