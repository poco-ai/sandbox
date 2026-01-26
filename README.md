# sandbox

An open-source sandbox container environment for agent execution.

This project is inspired by [AIO Sandbox](https://github.com/agent-infra/sandbox). We aim to provide a fully open-source alternative that the community can freely customize and extend.

## Docker Images

We provide two build targets in a single Dockerfile:

- `full` (default): desktop + VNC/noVNC + Chrome + Playwright
- `lite`: only Python + Node.js (smaller image), no desktop/browser/code-server services

Published image tags:

- `:full` and `:lite` (default branch only)
- `:full-sha-<short-sha>` and `:lite-sha-<short-sha>`

Build locally (from repo root):

```bash
# full (default target)
docker build -f docker/Dockerfile -t sandbox:full docker

# lite
docker build -f docker/Dockerfile --target lite -t sandbox:lite docker
```
