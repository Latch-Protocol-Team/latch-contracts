# Latch contracts: guide for Claude Code

The rules for working in this repository, and for any Latch built against it, are in
[AGENTS.md](AGENTS.md). Read that file first and follow it.

@AGENTS.md

## In this repository

- It is PUBLISHED from Latch's source repository by a script. Do not propose edits to files
  here; a change would be overwritten by the next publication. Open an issue instead.
- To build a Latch, do not work here. Scaffold a project
  (`npx @latchprotocol/create-hook my-latch`), which fetches this repository as a dependency,
  and work there. The prompt is in [prompts/build-a-latch.md](prompts/build-a-latch.md).
- To read how the core works, start at `packages/core/src/Vault.sol`,
  `packages/core/src/pool-cl/CLPoolManager.sol` and
  `packages/hooks/src/base/BaseCLHook.sol`.
