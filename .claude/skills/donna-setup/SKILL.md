---
name: donna-setup
description: Set up Donna (pre-PR design review) in the current repo. Use when the user asks to set up, configure, or onboard Donna, convert ADRs into Donna checks, or start Donna rules from scratch.
---

# Set up Donna in this repo

Follow `SETUP.md` from the Donna repository step by step. If it isn't in this repo, fetch it from https://github.com/anshita-arya/donna/blob/main/SETUP.md.

Rules while you work:

- Everything you change is a proposal. Show each diff and wait for the user to accept it before writing.
- Never edit an existing ADR's prose, never mark a decision `accepted` yourself, and never invent owners. Owners come only from CODEOWNERS or the user.
- Every glob you write must match at least one real file. List two matches per glob when you propose it.
- If a decision doesn't fit any of the six constraint kinds, leave it unchecked and say why. An honest "unchecked" beats a rule that misfires.
- Finish with a summary: rules added, ADRs left unchecked and why, and anything skipped. Don't commit or open a PR unless asked.
