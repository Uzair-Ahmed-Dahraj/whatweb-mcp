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
result store, and a live MCP stdio JSON-RPC handshake. All tests must pass
before a PR is merged.

## Ground rules

- Keep changes minimal and focused; one concern per PR.
- New behavior needs a test first (TDD).
- Never commit client names, client domains, or secrets. Use `example.com` in
  every example.
- Scan only targets you own or are authorized to test.

## Pull requests

1. Fork, branch, commit with a clear message.
2. Ensure `pytest` passes.
3. Open the PR describing what changed and why.

## Releasing

Pushing a tag `vX.Y.Z` (matching `version` in `server.json` and the image tag in
`packages[0].identifier`) runs `.github/workflows/release.yml`. It builds and
pushes the multi-arch image to Docker Hub (`X.Y.Z` and `latest`), then
publishes `server.json` to the Official MCP Registry. Maintainers need the
`DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` repository secrets.

The Claude Code plugin runs a pinned image, and installs from `main` get each
change at once. So update the plugin in a second PR, after the release run
publishes the image (`docker manifest inspect devkind/whatweb-mcp:X.Y.Z`
succeeds): set the image tag in `.mcp.json` to `X.Y.Z` and raise `version` in
`.claude-plugin/plugin.json`. Without the `version` change, existing installs
keep the old image.

## License

By contributing you agree your work is released under the MIT License.
