# Contributing to whatweb-mcp

Thanks for your interest in improving the WhatWeb MCP server.

## Ways to contribute

- Bug reports and feature ideas: open an issue with steps to reproduce.
- Tool output bugs: include the raw WhatWeb JSON log (redacted) and the parsed summary.
- Documentation: wording, examples, and registry listings all count.

## Development setup

```bash
git clone https://github.com/devkindhq/whatweb-mcp.git
cd whatweb-mcp
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt pytest
.venv/bin/python -m pytest
```

The test suite covers target validation, WhatWeb JSON parsing, the in-memory
result store, and a live MCP stdio JSON-RPC handshake. The tests do not need
the WhatWeb binary. CI runs them on Python 3.12 and builds the Docker image
for `linux/amd64`. Both jobs must pass before a PR is merged.

## Ground rules

- Keep changes minimal and focused; one concern per PR.
- New behavior needs a test first (TDD).
- Never commit client names, client domains, or secrets. Use `example.com` in
  every example.
- Scan only targets you own or are authorized to test.

## Pull requests

1. Fork, branch, and commit with a
   [Conventional Commits](https://www.conventionalcommits.org/) message, for
   example `fix: reject empty targets`.
2. Make sure `pytest` passes.
3. Open the PR describing what changed and why.

## Releasing

1. In one PR, set the new version `X.Y.Z` in all of these places:
   - `server.json`: `version` and the image tag in `packages[0].identifier`
     (`docker.io/devkind/whatweb-mcp:X.Y.Z`).
   - `.claude-plugin/plugin.json`: `version`.
   - `.mcp.json`: the image `devkind/whatweb-mcp:X.Y.Z`.
2. Merge the PR to `main`.
3. Push the tag `vX.Y.Z` from `main`.

The tag runs `.github/workflows/release.yml`. The workflow stops if the tag
does not match the `server.json` version and image tag. It then builds and
pushes the multi-arch image to Docker Hub (`X.Y.Z` and `latest`), and
publishes `server.json` to the Official MCP Registry with GitHub OIDC.

The workflow needs the `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` repository
secrets.

## License

By contributing you agree your work is released under the MIT License.
