# Contributing to Qavo

Thank you for your interest in contributing to Qavo. This guide explains how to participate effectively and what is expected of contributors.

---

## Code of conduct

By participating in this project you agree to abide by the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). Please be respectful and constructive in all interactions.

---

## Ways to contribute

- **Bug reports** — open an issue describing the problem, steps to reproduce, and expected vs actual behavior.
- **Feature requests** — open an issue describing the use case and the proposed solution.
- **Documentation improvements** — fix typos, clarify explanations, add examples.
- **Code contributions** — bug fixes, new features, or new plugins (see below).

---

## Before you start

For anything beyond a trivial fix, **open an issue first** and discuss the change. This avoids duplicate work and ensures the contribution aligns with the project direction before you invest time coding.

---

## Development setup

Each repository (`qavo-be`, `qavo-fe`) has its own setup instructions in its own `CONTRIBUTING.md`. This document covers the general principles that apply to all Qavo repositories.

---

## Pull request process

1. Fork the repository and create a branch from `main` with a descriptive name (e.g. `fix/auth-token-refresh`, `feat/registration-plugin`).
2. Make your changes, following the code style and conventions of the existing codebase.
3. Add or update tests to cover your changes. All existing tests must pass.
4. Update the relevant documentation if your change affects behavior or public APIs.
5. Open a pull request against `main` with a clear description of what changes and why.
6. Address review feedback. A maintainer will merge once the PR is approved.

All CI checks must pass before a PR can be merged.

---

## AI-assisted contributions

Contributors may use AI tools (code generation, review assistance, etc.) to help write contributions. If you do, the same standards apply as for any other contribution:

- You must understand the code you are submitting.
- You must test it and own it — AI-generated code is a draft, not a finished product.
- You must ensure it does not introduce code of unclear provenance or license.
- You are responsible for the correctness and security of what you submit.

Submitting AI-generated code you do not understand is not acceptable.

---

## Security vulnerabilities

Do not open public issues for security vulnerabilities. Follow the process described in [SECURITY.md](security.md).

---

## License

By contributing to Qavo, you agree that your contributions will be licensed under the [MIT License](../LICENSE) that covers the project.
