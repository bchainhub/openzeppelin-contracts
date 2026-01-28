# AGENTS.md instructions for /home/baron/corecoin/core_openzeppelin

<INSTRUCTIONS>
## Skills
A skill is a set of local instructions to follow that is stored in a `SKILL.md` file. Below is the list of skills that can be used. Each entry includes a name, description, and file path so you can open the source for full instructions when using a specific skill.
### Available skills
- skill-creator: Guide for creating effective skills. This skill should be used when users want to create a new skill (or update an existing skill) that extends Codex's capabilities with specialized knowledge, workflows, or tool integrations. (file: /home/baron/.codex/skills/.system/skill-creator/SKILL.md)
- skill-installer: Install Codex skills into $CODEX_HOME/skills from a curated list or a GitHub repo path. Use when a user asks to list installable skills, install a curated skill, or install a skill from another repo (including private repos). (file: /home/baron/.codex/skills/.system/skill-installer/SKILL.md)
### How to use skills
- Discovery: The list above is the skills available in this session (name + description + file path). Skill bodies live on disk at the listed paths.
- Trigger rules: If the user names a skill (with `$SkillName` or plain text) OR the task clearly matches a skill's description shown above, you must use that skill for that turn. Multiple mentions mean use them all. Do not carry skills across turns unless re-mentioned.
- Missing/blocked: If a named skill isn't in the list or the path can't be read, say so briefly and continue with the best fallback.
- How to use a skill (progressive disclosure):
  1) After deciding to use a skill, open its `SKILL.md`. Read only enough to follow the workflow.
  2) If `SKILL.md` points to extra folders such as `references/`, load only the specific files needed for the request; don't bulk-load everything.
  3) If `scripts/` exist, prefer running or patching them instead of retyping large code blocks.
  4) If `assets/` or templates exist, reuse them instead of recreating from scratch.
- Coordination and sequencing:
  - If multiple skills apply, choose the minimal set that covers the request and state the order you'll use them.
  - Announce which skill(s) you're using and why (one short line). If you skip an obvious skill, say why.
- Context hygiene:
  - Keep context small: summarize long sections instead of pasting them; only load extra files when needed.
  - Avoid deep reference-chasing: prefer opening only files directly linked from `SKILL.md` unless you're blocked.
  - When variants exist (frameworks, providers, domains), pick only the relevant reference file(s) and note that choice.
- Safety and fallback: If a skill can't be applied cleanly (missing files, unclear instructions), state the issue, pick the next-best approach, and continue.
</INSTRUCTIONS>

# Project context

This repo ports OpenZeppelin to a custom blockchain. It uses the custom framework `spart` (a clone of Foundry/Forge). Build with `spark build`, and run tests with `spark test`.

OpenZeppelin source and tests directories must remain unchanged:
- `openzeppelin_contracts` and `openzeppelin_tests` are read-only references.
- All ported logic lives under `src/` and all ported tests live under `test/`.
- Files must mirror the OpenZeppelin folder structure under these destinations.

This project is based on the Ylem compiler rather than Solidity. It may still use legacy names like `pragma solidity` or `keccak256`, but under the hood the compiler differs:

- Pragmas should always be `pragma solidity ^1.1.2`.
- Addresses are 22 bytes.
- `keccak256` is implemented as `sha256` under the hood (exact parameters to be defined later).

When porting tests from `openzeppelin_tests`, preserve the same folder structure under `test/` (create folders as needed).

Cryptography differences vs Ethereum:
- `ecrecover` takes two arguments: `ecrecover(bytes32 hash, bytes signature)`; signature parsing/verification happens inside the chain.
- Failed recovery returns `address(0)` and should be treated as invalid.
- Signature length is fixed to 171 bytes; reject any other length (e.g., `InvalidSignatureLength`).
- No exposed `(v, r, s)` tuple at the Solidity level; signatures are a single blob.
- Signed-message prefix is `"\x19Core Signed Message:\n32"` (not Ethereum’s prefix).
- Typed-data hashing keeps EIP-712 construction: `keccak256("\x19\x01" || domainSeparator || structHash)`.
- Tooling must produce 171-byte signatures, use the Core prefix, and verify via two-arg `ecrecover`.
