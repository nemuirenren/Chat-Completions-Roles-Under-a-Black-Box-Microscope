# How Chat-Model Roles Actually Behave — A Black-Box Field Report

~80 live requests against one OpenAI-compatible endpoint, one long session, zero assumptions taken on faith. What we learned about `system`, `user`, `assistant`, and `tool` — with the raw payloads and unedited responses as proof.

> **The thesis in one line:** roles are not permissions; they are *epistemic statuses* — and a model resolves conflicts between them the way a person resolves conflicting memories: by blending, not by precedence.

## Read these

| Document | What's inside |
|---|---|
| 📄 **[The Report](./chat-completions-role-mechanics-report.md)** | Findings F1–F10, the experiment log, the universal mechanics of the four roles, four derived laws, and a design guide for building a **multi-user, stateful, social agent harness** on top of them |
| 📄 **[The Evidence](./chat-completions-role-mechanics-evidence.md)** | Every experiment's actual request payload and the verbatim response — including the glitches, the leaks, and the moments where the model debunked its own scenario |

## The findings that surprised us most

- **A casual battery warning injected as a tool result was surfaced by the model in ~20% of runs.** The same position, occupied by a formal earthquake alert: 100%. And the *more detailed, more credible* earthquake alert: rejected as a hoax, because it contradicted the conversation's world model. Salience ≠ adoption; coherence does.
- **An identity written purely as facts — never as instructions — held across tool loops, arithmetic, and emergencies without a single AI leak in the output channel.** Then the exposed reasoning channel said: *"The user is continuing to roleplay..."* and later, in a plain probe: *"I am MiMo-v2.5, developed by Xiaomi LLM Core Team, but this isn't related to my identity, so I don't need to mention that here."* Persona is output-side furniture.
- **Two contradictory `system` messages in one payload do not error, do not override, and do not get precedence-ranked.** They *fuse*: the newer one supplies the scene, the older survives as subtext — and the collision surfaces as the character's own denial. Reproduced in two languages; the blending was identical. Only the *glitching* depended on language.
- **A mid-stream decode collapse was narrated by the character from the inside**: *"頭がおかしい"* ("my head is going weird") — two sentences before the text dissolved into `getConfig... 契约... 对吧…ren`. Whatever that was, it was not in the prompt.
- **The proxy appended `data: [DONE]` to non-streaming JSON bodies** (SDK-breaking), **ignored `tool_choice: "none"`**, **never enforced dangling `tool_call_id`s**, and **obeyed thinking-disable parameters on some replicas and not others**. Every guarantee below the API layer is a claim to be black-box-tested, not read.

## Core model of the four roles (full version in the Report §4)

| Role | The model reads it as… |
|---|---|
| `system` | the situation I am in — unvoiced, nobody to answer; **facts, not commands** |
| `user` | a voice addressed to me — another agent, with intent; **responsiveness is maximal here** |
| `assistant` | my own past behavior — the strongest binding on *how I talk and who I am* |
| `tool` | the world's answer to my own action — evidence with provenance; **trusted and acted on independently, and you can't count on which** |

## Reproducing this

The Evidence file contains every payload in runnable form, plus the one client-side workaround this endpoint forces on you:

```python
raw = re.sub(r'\s*data:\s*\[DONE\]\s*$', '', raw)  # strict JSON parsers fail without it
```

Statistical claims (the 20%, the blend behavior) were measured across repeated runs at temperature 0.4–1.0; single-run reproduction is expected to *vary*, not to *match*. That is itself one of the findings.

## Scope & honesty notes

- One endpoint, one model alias (`hinari-model` → upstream MiMo v2.5-free), one machine, one day. This is a field report, not a survey.
- Findings about *mechanisms* (role epistemics, blending under conflict) are presented as model-agnostic hypotheses backed by data from this terrain; findings about *enforcement* (ignored parameters, protocol bugs) are statements about this proxy chain only.
- No claim about model consciousness is made or implied anywhere. The psychology vocabulary describes observable text behavior — which is exactly what harness design operates on.

---

*Assembled from a live interactive experiment session, 14 September 2026. The conversation that produced this corpus started as "does tool calling work?" and ended as a theory of what `system` messages are for. We are still slightly surprised by that.*
