# Claude prompt: build a Latch

Paste the block below into [Claude Code](https://claude.com/claude-code) in an empty folder, or
in the project the scaffolder made for you. **Fill in `## My Latch` first.**

A project made by `create-hook` already carries a `CLAUDE.md` with these rules, so Claude Code
follows them there without being told. This prompt is for starting from nothing, and for any
other coding agent.

---

## Before you paste

Four things about Latch Protocol shape everything an agent writes, and an agent that does not
know them writes code that looks right and is not.

**A Latch can never be upgraded.** A pool names its Latch in its key, and the key is the pool's
identity. There is no proxy and no second chance: a fix is a new contract and new pools. So the
work is in getting it right before it is deployed.

**A pool has one Latch.** You cannot add yours to a pool that already has one, and that includes
every launch pool on Latch, whose Latch is the launch guard. A product for existing pools has to
be an ordinary contract, not a Latch.

**A fee you return is ignored on a static-fee pool.** No error, no warning. A Latch that sets
fees needs a pool created with the dynamic fee flag.

**A green test run does not mean it can be deployed.** Foundry's test chain does not enforce the
24,576-byte contract size limit. Read the size out of the build artifact.

---

## The prompt

````text
I am building a Latch on Latch Protocol. A Latch is a hook contract attached to a pool on
Latch Protocol's core. I am NOT deploying or changing the core.

## My Latch

- What it does, in one sentence: ____________________________________   <-- REPLACE
- Pool type: concentrated liquidity (CL)        <-- CL | Bin
- Pool fee: dynamic                              <-- dynamic | a static fee in hundredths of a bip
- Does it ever hold tokens or take a fee? no     <-- no | yes (say whose, and who is paid)
- Is there any role (owner, guardian)? no        <-- no | yes (say what it may change)
- Chain I will deploy to: ____________          <-- decides the build profile you test under

## Ground rules: follow these exactly

1. START FROM THE SCAFFOLDER, NOT FROM A BLANK FILE.

   npx @latchprotocol/create-hook my-latch --template dynamic-fee
   cd my-latch && forge test -vv

   Templates: noop, dynamic-fee, swap-counter. Choose the nearest and change it. The scaffolder
   fetches the contracts into lib/latch-contracts at a pinned commit and writes the registration
   bitmap and the pool key's parameters word TOGETHER. Never edit one without the other, and
   never edit anything under lib/.

   If a CLAUDE.md or AGENTS.md is in the project, read it first and follow it.

2. READ THE BASE CONTRACT BEFORE YOU WRITE.

   lib/latch-contracts/packages/hooks/src/base/BaseCLHook.sol (or BaseBinHook.sol). Its header
   is the security model. Extend it; override only the internal `_before...` / `_after...`
   functions for the callbacks you declare; return the base's selector helpers.

3. DECLARE ONLY WHAT YOU IMPLEMENT.

   `getHooksRegistrationBitmap()` lists your callbacks. A callback you declare and do not
   implement reverts `HookNotImplemented` when the pool calls it. A `*ReturnsDelta` flag needs
   its base callback. Bits 14 and 15 are zero.

4. DESIGN FIRST. Before any code, tell me: the callbacks you will use and why, the state you
   keep, every role and what it can and cannot do, and for every contract what value can reach
   it and how every unit leaves. If my description above leaves something undecided, say which
   assumption you are making. Wait for my answer only if the choice is irreversible.

5. IF IT TOUCHES MONEY:
   - Every unit that can enter must be able to leave. Prefer a contract that cannot receive.
   - No key may move a user's funds or change a term that was fixed.
   - Ownership is two-step and `renounceOwnership` reverts. Prefer no role.
   - Caps are constructor immutables. Whatever a deployer may want to change is a constructor
     argument, never a constant.
   - Durations are block.timestamp seconds, never block numbers.
   - Credit a balance and let its owner claim it. Do not pay out inside a swap.
   - Refuse fee-on-transfer and rebasing tokens. The Vault does not support them.
   - No proxy, no delegatecall to an address that can be set, no selfdestruct.

6. BE DELIBERATE ABOUT WHAT REVERTS. A `beforeSwap` that reverts stops the pool trading. A
   callback on removing liquidity that reverts traps a provider's money: it must never revert on
   a condition the provider cannot clear.

7. TEST UNDER BOTH PROFILES, AND PROVE EACH GUARD.

   forge test
   FOUNDRY_PROFILE=legacy forge test

   Cover the happy path, every refusal, an unauthorised caller, the boundaries, and a fuzz test
   for any arithmetic. For each check you wrote, remove it, confirm a test fails, and put it
   back; run `git diff` afterwards and show me it is clean.

8. CHECK THE SIZE FROM THE ARTIFACT, not from a green run:
   the length of `deployedBytecode.object` must be under 24,576 bytes.

9. NEVER put a private key, a mnemonic or an RPC URL with a key in a file, a test, a comment or
   your output. Deploy scripts read `vm.envUint("PRIVATE_KEY")`. Never commit `.env` or
   `broadcast/`.

10. DO NOT DEPLOY OR SEND ANY TRANSACTION ON A REAL CHAIN. Give me the command and what it will
    do. Rehearse on `anvil` first, calling every external function once to succeed and once to be
    refused.

## What I want back

1. The design (rule 4).
2. The contract, the tests and the deploy script, as complete files.
3. A security review: for each finding, the severity, what it is, the attack, the code and the
   fix. No invented findings. Then what is still assumed.
4. The test output under both profiles, the runtime size, and the registration bitmap with the
   pool key parameters word to use.
5. What I must decide or do myself.

Never tell me the Latch is "safe". Tell me what it is permitted to do and what you did not check.
````

---

## After it is built

- **List it.** A Latch is listed in the on-chain registry with its permissions read from the
  contract. Use the
  [listing form](https://github.com/Latch-Protocol-Team/.github/issues/new/choose).
- **Build the front end** with the MIT SDK:
  [latch-sdk](https://github.com/Latch-Protocol-Team/latch-sdk), which carries prompts for a
  DEX and for a Launchpad.
- **Have it reviewed** by somebody who did not write it before real funds touch it. An agent's
  review is a first pass, not an audit.
