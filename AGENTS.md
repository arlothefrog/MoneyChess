# AGENTS.md — Instructions for AI coding agents

This file tells AI coding agents (and humans) how to work safely and correctly on this repository.

## The project

A single-page browser game called **Money Chess** — a chess variant with money, a shop, and spells.
It lives entirely in one file: `money-chess.html` (HTML + CSS + JavaScript, no build step).

Files:
- `money-chess.html` — the game. Open it in a browser to play/test.
- `Readme.html` — the **authoritative rules & implementation specification** (for humans and agents).
- `AGENTS.md` — this file.

## Mandatory workflow for coding agents

1. **Read this file first.**
2. **Read `Readme.html` before making any change.** It documents the game rules, all
   piece/spell/mechanic behaviour, key function names, and known issues. Treat it as the spec:
   do not guess the rules from memory.
3. After making a change that affects rules, behaviour, or UI text:
   - **Update `Readme.html`** so it stays in sync with the code (mechanics, tables, key-functions list).
   - If you changed an intentional rule, note it explicitly there.
4. If you find a bug or an inconsistency between the spec and the code that you are **not fixing now**,
   **add it to the "Known issues, bugs & inconsistencies" section of `Readme.html`** so it is not lost.
   See that section (currently items 1–11) for the existing backlog; you may also fix items marked safe.
5. Do **not** edit `Readme.html` or `AGENTS.md` for cosmetic reasons; only when game content/spec changes.

## Ground rules

- **Single file:** keep all game code inside `money-chess.html`. Do not split into multiple JS/CSS files.
- **AI engine block:** the AI lives in its own `<script id="aiEngine">` block, before `<script id="mainScript">`.
  It also runs in a Web Worker built from the page's own source (`aiWorkerSource()`), so it must not touch the DOM
  at load time. Main-script code the AI calls must be a top-level `function` declaration or a top-level `const`/`let`
  holding plain data (those are copied into the worker by name). After changing the AI, check that the worker still
  starts (no "AI worker unavailable" warning in the console).
- **No build step and no package manager.** Test by refreshing the page in a browser. Expect no unit-test framework.
- **Determinism matters:** all in-game randomness goes through the seeded PRNG `gRng()` seeded by `SEED`
  (see Readme §11). When you add randomness, route it through `gRng()` so online games stay in sync
  (both clients share the seed and RNG counter). Never use `Math.random()` for in-game outcomes.
- **Online play is host-authoritative** (PeerJS): the host runs all mutations and broadcasts snapshots;
  the guest sends commands. If you change a rule mechanic, make sure a mutation still flows through the
  wrapped functions (`makeMove`, `buyPiece`, `placePiece`, `castSpell`, `finishSpell`,
  `skipFollowUpSpell`, `spellEndTurnSkip`, `cancelPlacement`, `handleTargetClick`) so networking keeps working.
- **Reuse existing patterns:** pieces are objects `{type, color, ...}`; effect lists live on `state`
  (`blocked`, `sleeping`, `confused`, `piratify`, `thief`, `cursed`, `shielded`, `swapLocked`); money is `state.money.w` / `state.money.b`.
  Match the existing code style (spaces around operators, descriptive names) even though formatting varies.
- **Do not add new emoji to the UI** unless asked. (Chess/effect glyphs that are already part of the game data are fine.)
- **No secrets or credentials** belong in this repo (there are none; keep it that way).

## Verifying changes

1. Open `money-chess.html` in a browser and exercise the changed behaviour.
2. Check Readme §14 (key functions) to find where your change lives, and confirm the spec is updated.
3. Remember the main known trouble spots from Readme §15 before you "fix" something blindly —
   some behaviours (e.g. castling availability, buying after moving) may be revisiting open design questions.

## Commit style

When asked to commit, keep the message a single descriptive sentence in the style of the existing log
(e.g. "Add online 2-player multiplayer via PeerJS (host=White authoritative...)"). Never commit without being asked.