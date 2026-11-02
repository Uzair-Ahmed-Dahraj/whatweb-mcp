# Security Policy

## Supported versions

Only the latest release is supported.

## Reporting a vulnerability

Email security@devkind.com.au.

Please do not open public issues for suspected vulnerabilities. We aim to
respond within 5 business days.

## Scope notes

This project wraps WhatWeb, an offensive-reconnaissance tool. Use it only
against systems you own or are explicitly authorized to test. The maintainers
are not responsible for misuse of the underlying scanner.

## Hardening already in place

- Non-root container user, tini as PID 1.
- Suggested run flags: `--memory=512m --cpus=0.5 --pids-limit=100`.
- Target validation rejects flag-injection attempts before any subprocess runs.
- WhatWeb is executed with an argument array, never a shell string.
