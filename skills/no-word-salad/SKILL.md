---
name: no-word-salad
description: >
  Strip word salad from authored text and code comments — verbose filler, stock AI phrasing,
  hedging, and comments that narrate what the code already says. Use as a final self-review pass
  before emitting authored prose (specs, PR descriptions, messages, docs) or as a lens when
  reviewing a diff for over-explanation, and whenever the user asks to cut word salad, de-slop,
  tighten, or trim writing. Referenced by `develop:dlc` at self-review; also invokable directly.
---

# No word salad

Write the minimum that does the job, in plain words — no AI stock-phrases, hedging, or filler. Comment code only where intent isn't obvious from the code itself — never to narrate what it does.
