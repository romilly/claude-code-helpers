# FastAPI Web Application Guide

## Architecture

Use **hexagonal architecture** (ports and adapters):

```
src/project_name/
├── domain/           # Pure business logic, no I/O
├── ports/            # Abstract interfaces (ABCs)
├── adapters/         # Implementations
│   ├── web_app.py    # FastAPI app factory
│   └── *_repository.py
└── templates/        # Jinja2 templates
```

### App Factory Pattern

Create the FastAPI app via a factory function with dependency injection:

```python
from fastapi import FastAPI
from project.ports.some_port import SomePort

def create_app(dependency: SomePort) -> FastAPI:
    app = FastAPI()
    # Define routes that use the injected dependency
    return app
```

This enables testing with fake adapters and production with real adapters.

## Frontend Stack

- **Bootstrap 5** via CDN for styling
- **htmx** via CDN for dynamic interactions
- **Jinja2** templates in `src/project_name/templates/`

Base template includes:
```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
<script src="https://unpkg.com/htmx.org@2.0.4"></script>
```

## Dependencies

In `requirements.txt`:
```
fastapi
uvicorn
jinja2
```

In `requirements-test.txt`:
```
httpx
playwright
pytest-playwright
```

Install Playwright browser: `playwright install chromium`

## Testing Strategy

### 1. Endpoint Tests (TestClient)

Use FastAPI's TestClient with fake adapters:

```python
from fastapi.testclient import TestClient
from project.adapters.fake_adapter import FakeAdapter
from project.adapters.web_app import create_app

@pytest.fixture
def client() -> TestClient:
    adapter = FakeAdapter()
    adapter.add(sample_data)
    app = create_app(adapter)
    return TestClient(app)

def test_endpoint(client: TestClient) -> None:
    response = client.get("/")
    assert response.status_code == 200
```

### 2. E2E Tests (Playwright)

Use a test server utility with dynamic port allocation:

```python
# tests/helpers/test_server.py
import socket
import threading
from contextlib import contextmanager
from typing import Generator

import httpx
import uvicorn
from fastapi import FastAPI


def find_free_port() -> int:
    """Find an available port by letting the OS assign one."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("127.0.0.1", 0))
        return s.getsockname()[1]


def wait_for_server(url: str, timeout: float = 5.0) -> bool:
    """Wait for server to be ready."""
    import time
    start = time.time()
    while time.time() - start < timeout:
        try:
            response = httpx.get(url, timeout=0.5)
            if response.status_code == 200:
                return True
        except httpx.RequestError:
            pass
        time.sleep(0.1)
    return False


@contextmanager
def run_test_server(app: FastAPI) -> Generator[str, None, None]:
    """Context manager to run a test server, yields the base URL."""
    port = find_free_port()
    config = uvicorn.Config(app, host="127.0.0.1", port=port, log_level="error")
    server = uvicorn.Server(config)

    thread = threading.Thread(target=server.run, daemon=True)
    thread.start()

    url = f"http://127.0.0.1:{port}"
    if not wait_for_server(url):
        raise RuntimeError(f"Server failed to start on {url}")

    try:
        yield url
    finally:
        server.should_exit = True
        thread.join(timeout=2)
```

E2E test example:
```python
from playwright.sync_api import Page, expect
from tests.helpers.test_server import run_test_server

@pytest.fixture
def server(sample_data) -> Generator[str, None, None]:
    adapter = FakeAdapter()
    for item in sample_data:
        adapter.add(item)
    app = create_app(adapter)
    with run_test_server(app) as url:
        yield url

def test_page_displays_data(page: Page, server: str) -> None:
    page.goto(server)
    expect(page.locator("table")).to_be_visible()
```

## Port Management

**Critical**: Always use dynamic port allocation for both test and production servers.

```python
def find_free_port() -> int:
    """Find an available port by letting the OS assign one."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("127.0.0.1", 0))
        return s.getsockname()[1]
```

- Never hardcode ports without checking availability
- Test servers and production servers must use different ports
- Let the OS assign ports to avoid conflicts

## Run Script

Create `scripts/run_web.py`:

```python
#!/usr/bin/env python3
import socket
from pathlib import Path
import uvicorn
from project.adapters.web_app import create_app
from project.adapters.real_adapter import RealAdapter

def find_free_port() -> int:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("127.0.0.1", 0))
        return s.getsockname()[1]

def main() -> None:
    port = find_free_port()
    adapter = RealAdapter()
    app = create_app(adapter)
    print(f"Starting server at http://127.0.0.1:{port}")
    uvicorn.run(app, host="127.0.0.1", port=port)

if __name__ == "__main__":
    main()
```

## Checklist

- [ ] Define port (ABC) for external dependency
- [ ] Create fake adapter for testing
- [ ] Create real adapter for production
- [ ] Create `web_app.py` with app factory using dependency injection
- [ ] Create Jinja2 templates with Bootstrap + htmx
- [ ] Write endpoint tests with TestClient + fake adapter
- [ ] Create test server utility with dynamic ports
- [ ] Write E2E tests with Playwright
- [ ] Create run script with dynamic port allocation
