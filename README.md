# Latch contracts

The contracts a **Latch** is built against. A Latch is a hook contract attached to a pool on
Latch Protocol's core.

You do not need to clone this repository by hand. The scaffolder fetches it for you:

```bash
npx @latchprotocol/create-hook my-latch --template dynamic-fee
cd my-latch
forge test -vv
```

## What is here

| Path | What it is |
|---|---|
| `packages/core/src` | The settlement layer: `Vault`, the CL and Bin pool managers, libraries and types |
| `packages/core/test` | Its tests, and the test routers your own tests can import |
| `packages/core/lib` | `forge-std`, `openzeppelin-contracts`, `solmate`, as git submodules |
| `packages/hooks/src/base` | `BaseCLHook` and `BaseBinHook`: what a Latch extends |

## Two build profiles

Chains with EIP-1153 use transient storage; chains without it use a storage backend with the
same interface. One source, two profiles:

```bash
forge build                          # default: EIP-1153, evm_version cancun
FOUNDRY_PROFILE=legacy forge build   # legacy:  storage backend, evm_version shanghai
```

Test a Latch under the profile of the chain you deploy to.

## Rules a Latch lives by

- **A Latch's address is part of its pool's identity.** A pool names its Latch in its key, so a
  Latch cannot be upgraded or swapped: a new version means new pools.
- **A Latch works from any address.** Its permissions come from
  `getHooksRegistrationBitmap()`, which the pool manager checks against the pool key's
  `parameters` when the pool is created. There is no address to mine.
- **A Latch acts only on pools that name it**, and only inside the callbacks its bitmap declares.
- **Fee-on-transfer and rebasing tokens are not supported** by the Vault's accounting.

## Licence

GPL-2.0-or-later. `packages/core` is a derivative of
[pancakeswap/infinity-core](https://github.com/pancakeswap/infinity-core); see `NOTICE`.
A contract that imports these sources is a derivative work under the same licence.
The SDK ([latch-sdk](https://github.com/Latch-Protocol-Team/latch-sdk)) is MIT and talks to
deployed contracts through their ABI.

## This repository is published, not edited

It is written from Latch's source repository by a script. Open an issue here; a pull request
against these files would be overwritten by the next publication.

[latches.fun](https://latches.fun) · [docs](https://docs.latches.fun) · [X](https://x.com/Latchesdotfun)
