# CCDV-F Study Notes: Domain by Domain

Each domain: **Why it exists → How it works → Failure modes → Exam traps.** Numbers (prices, limits, minimums) change, so confirm against Anthropic docs before the exam. Domain weights come from a third-party source; check them against the official exam guide.

---

## 1. Applications & Integration (33.1%)

**Mental model: the API is stateless.** Claude remembers nothing between calls. *Why:* simple, horizontally scalable. *Consequence:* you resend the full history every turn, so cost and latency grow with conversation length. You own state, truncation, and summarization.

**Request:** `model`, `max_tokens` (required), `messages` (alternating user/assistant), `system` (a top-level param, not a message), optional `tools`, `temperature`, `stop_sequences`.
**Response:** `content` blocks (text / tool_use / thinking), `stop_reason`, `usage` (input, output, cache read/write tokens).

**stop_reason: your code's branching signal**
| Value | Meaning | What you do |
|---|---|---|
| end_turn | Finished naturally | Use the output |
| max_tokens | Truncated | Output may be broken (esp. JSON). Raise the limit, continue, or fail safe |
| tool_use | Wants a tool run | Execute, return tool_result, call again |
| stop_sequence | Hit your stop string | Normal, handle per design |
| refusal / pause_turn | Declined / long turn paused | Handle explicitly, don't parse as normal output |

**Errors: know the action, not just the code**
| Code | Meaning | Action |
|---|---|---|
| 400 | Malformed request | Fix the code. Never retry as-is |
| 401 / 403 | Bad key / no permission | Fix config. Alert. No retry |
| 404 | Wrong model/resource | Fix config |
| 413 | Request too large | Shrink input, chunk |
| 429 | Rate limited | Backoff, honor `retry-after`, smooth traffic |
| 500 / 529 | Server error / overloaded | Retry with backoff, then fallback or queue |

**Retry design:** exponential backoff + jitter (jitter stops synchronized retry storms), a max attempt cap, and total timeout. Retry only transient errors (429/5xx/529). The SDKs retry a couple of times by default, so don't stack your own retries on top blindly. Decide the downstream outcome: the user sees a graceful message, the job goes to a dead-letter queue, or you fall back to another model.

**Streaming (SSE)**
- *Why:* perceived latency (first token fast), and long generations avoid HTTP timeouts.
- *Events:* `message_start → content_block_start → content_block_delta* → content_block_stop → message_delta → message_stop`.
- *Gotchas:* tool-call arguments arrive as partial JSON, so parse only after `content_block_stop`. **Errors can occur after a 200 response, mid-stream**, so handle partial output.
- **Streaming ≠ Batch.** Streaming = real-time delivery of one request. Batch = asynchronous bulk processing at about 50% lower price, results within 24h (often much sooner), up to ~100k requests per batch. Results return **by `custom_id`, not in order**, so resubmit only the failed ones. Use batch when nobody is waiting (evals, backfills, overnight summarization). Use streaming/real-time for interactive UX.

**Structured outputs: layers, from weakest to strongest**
1. Prompt for JSON: unreliable.
2. Tool/JSON schema with forced `tool_choice`: strong shape guarantee.
3. Native structured outputs / strict tool use (schema-constrained decoding): schema-valid in the normal case. Verify current availability in docs.
- **Schema-valid ≠ correct.** It guarantees shape, not truth. Still validate semantics (Pydantic/Zod, range checks, business rules).
- **Failure modes:** `max_tokens` truncation → invalid JSON; refusal → no schema-shaped output; wrong-but-well-formed values.
- **Handling:** validate at the boundary → on failure retry once with the validation error fed back → then fallback / human review / dead-letter. Never write unvalidated model output into a DB or trigger actions from it. Bad output propagates silently, so *you* own the guard.
- **When NOT to use it:** open-ended reasoning, creative text, or cases where a rigid schema hurts quality. If you need reasoning, give a `reasoning` field *before* the answer field.

**Multimodal:** images and PDFs go in as content blocks (base64, URL, or Files API). Respect size/page limits. Claude reads scanned PDFs natively, so no OCR step is needed. Put documents before the question.

**Architecture practices:** isolate Claude behind one service/interface (swap models, add caching, centralize retries); externalize model names, prompts, and params in per-environment config; version prompts; keys in a secret manager; log usage tokens per request; use queues for async work; budget for rate limits.

**Traps:** thinking Claude keeps session memory; retrying 400s; parsing JSON without checking `stop_reason`; assuming batch results are ordered; treating streaming as a cost saver (it isn't).

---

## 2. Model Selection & Optimization (16.8%)

**Tiers:** Haiku = fastest/cheapest (classification, routing, extraction, high-volume). Sonnet = balanced default (most production apps, coding, agents). Opus = most capable/slowest/priciest (hard reasoning, long agentic tasks, high-stakes analysis). Reason by *tier*, not version number.

**Decision method:** start from the cost of being wrong → run the cheapest plausible model against your eval set → escalate only where it fails. **Cascade/route:** small model handles the bulk and escalates only the hard minority (e.g., Haiku classifies 1M tickets, Opus drafts the 2% complex ones).

**Latency vs cost vs quality:** you rarely win all three. Real-time chat → latency dominates (smaller model, streaming, caching). Offline bulk → cost dominates (batch + smaller model). Legal/medical analysis → quality dominates.

**Optimization levers (least quality risk first):** trim prompt and cap output → **prompt caching** → **batch** (if async) → right-size the model → use extended thinking only where reasoning is the bottleneck.

**Prompt caching: know the economics**
- Cache order: tools → system → messages. Mark with `cache_control`. Everything *before* the breakpoint must be byte-identical.
- Pricing multipliers: write ≈ **1.25×** base input (5-min TTL; ≈2× for the 1-hour TTL), read ≈ **0.1×**.
- **Break-even:** one use = 1.25 vs 1.0, so it costs *more*. Two uses = 1.25+0.1 = 1.35 vs 2.0, so it saves. Caching pays only when the prefix is reused within the TTL.
- Zero hit rate usually means: dynamic content (timestamp/user ID) placed *before* the breakpoint, prefix below the model's minimum cacheable size, TTL expired, or content changed.
- Best fit: large stable system prompts, long documents, tool definitions, multi-turn chats, few-shot blocks.

**Extended thinking:** the model reasons before answering. Thinking tokens bill as **output** tokens and add latency. It has a token budget (min ~1,024, must be below `max_tokens`) and restrictions on forcing tool_choice. Use for multi-step math/logic/planning and hard debugging. Skip for simple lookups, classification, and latency-critical paths.

**Batch:** ~50% off, async, combinable with caching. Not for anything interactive.

**Traps:** "always pick the biggest model"; caching everything regardless of reuse; enabling thinking on trivial tasks; ignoring output tokens (they cost more than input).

---

## 3. Agents & Workflows (14.7%)

**Definitions (Anthropic's framing):** **Workflow** = LLMs and tools orchestrated by *predefined code paths*. **Agent** = the LLM *dynamically decides* its own steps and tool use in a loop. *Why it matters:* workflows are predictable, cheaper, and easier to test. Agents handle open-ended problems but cost more, are slower, and can go wrong. **Start simple; add autonomy only when the path can't be predetermined.**

**Workflow patterns**
| Pattern | Use when |
|---|---|
| Prompt chaining | Fixed sequential steps (extract → validate → format) |
| Routing | Classify input, send to a specialized handler |
| Parallelization | Independent subtasks (sectioning) or multiple attempts (voting) |
| Orchestrator-workers | Subtasks unknown in advance; a central LLM delegates and synthesizes |
| Evaluator-optimizer | Iterate against clear criteria (generate → critique → refine) |

**The agent loop:** gather context → act (tool call) → observe result → verify → repeat until done. Non-negotiables: **max-iteration cap, loop detection, informative tool errors, clear stop conditions, human approval for risky steps.**

**Agent orchestration (multi-agent):** a lead agent spawns subagents with **isolated context windows** that return condensed results. *Why:* parallel breadth and protection of the main context. *Cost:* many more tokens than a single agent. Best for parallelizable research-style tasks, poor for tightly coupled work.

**Long-running context:** compaction (summarize old turns, keep decisions and open items), external notes/memory files, subagents returning summaries, clearing stale tool results.

**Traps:** using an agent where a chain suffices; no iteration cap; deleting context instead of summarizing; assuming more agents = better.

---

## 4. Prompt & Context Engineering (11.0%)

**Core techniques (and why they work):** be clear and direct with context on *why* (the model infers intent); **XML tags** separate instructions/data/examples and reduce misinterpretation; **3-5 diverse few-shot examples** pin down format and style, useful when instructions alone are ambiguous; **system prompt** = persistent role, rules, tone; let the model **think step by step** for reasoning tasks; specify output format explicitly.
**Prefill** (starting the assistant turn with `{`) forces format, but may be unsupported on the newest models, so prefer tools or structured outputs.
**Long context:** documents first, question last; ask Claude to quote the relevant passages before answering; more context is not always better (relevant signal gets diluted).
**Context engineering:** load the smallest set of high-signal tokens; retrieve just-in-time instead of dumping everything; compact and summarize.
**Traps:** contradictory instructions; shouting in caps; examples that are all alike (model overfits); stuffing irrelevant context.

---

## 5. Tools & MCP (10.6%)

**Tool use: the exact loop order**
1. You send the request with `tools` definitions.
2. Claude replies `stop_reason: tool_use` with `tool_use` blocks (`id`, `name`, `input`).
3. **Your code** executes the tool. Claude never runs client tools itself.
4. You send a *user* message with `tool_result` blocks (matching `tool_use_id`; all parallel results in one message, results first in content).
5. Claude continues, and the loop repeats until `end_turn`.
- On failure return the `tool_result` with `is_error: true` and an actionable message so Claude can adjust or retry. Don't drop the result.
- **Descriptions are the model's only guide** to when and how to call a tool. Vague or overlapping tools cause wrong or unnecessary calls.
- **tool_choice:** `auto` (model decides), `any` (must use some tool), a named tool (forced), `none`. Independent calls can run in parallel.
- Keep results small: paginate, truncate, or return IDs. Huge payloads burn context.

**MCP (Model Context Protocol):** an open standard so any client can connect to any tool/data server, instead of custom integrations per pair. JSON-RPC 2.0 underneath.
- **Primitives:** *Tools* (model-controlled actions), *Resources* (app-controlled data), *Prompts* (user-controlled templates).
- **Lifecycle:** `initialize` (version + capability negotiation) → `initialized` → discover (`tools/list`) → `tools/call` → shutdown.
- **Transports:** **stdio** = client launches the server as a local subprocess and talks over stdin/stdout. Simple, no network, one client, and the server must never print non-protocol output to stdout (log to stderr). **Streamable HTTP** = remote server, many clients, needs real auth (OAuth), fits shared/hosted deployments. Rule: local/single-user/dev → stdio; remote/multi-user/production → HTTP.
- Server-side quality: precise names and descriptions, tight input schemas (enums, filters, pagination), concise structured outputs, actionable errors.

**Skills vs MCP:** **Skills** = folders with `SKILL.md` (name + description in frontmatter, plus scripts/resources) loaded on demand via progressive disclosure. They teach Claude *how* to do a procedure or apply domain expertise. **MCP** = live connection to external systems, data, and actions (*access*). Rule: procedure/expertise → Skill; connectivity/live data → MCP. They combine well.

**Traps:** thinking Claude executes client tools; omitting tool_result; too many overlapping tools; picking stdio for a multi-user hosted service.

---

## 6. Security & Safety (8.1%)

**Prompt injection:** *direct* (user tries to override instructions) and **indirect** (malicious instructions hidden in content Claude reads: web pages, emails, documents, tool results). *Why it's hard:* the model can't perfectly separate data from instructions. No prompt-only fix exists, so use defense in depth.
**Defenses:** least-privilege tool scopes; treat all external content as *untrusted data*; human approval for irreversible or external actions (send, delete, pay); rate limits and audit logs; sandbox code execution; validate and sanitize model output before rendering or executing it (XSS, SQL injection, shell). Be most careful when one agent has **private data + untrusted content + a way to send data out**.
**Secrets:** never put credentials or confidential rules in prompts. "Never reveal this" is not enforcement. Enforce access in code and permissions.
**Also:** minimize and redact PII, mind data retention, use monitoring, and never treat model safety behavior as your security boundary.
**Traps:** "add 'ignore malicious instructions' to the system prompt" as the only fix; trusting tool output like user input; full permissions "so it's never blocked".

---

## 7. Claude Code (3.1%, but cheap to cover)

- **CLAUDE.md:** persistent project context (conventions, commands, architecture) loaded each session. Project-level committed to the repo (shared), user-level in `~/.claude/`. Keep it concise.
- **Settings / permissions:** `.claude/settings.json` (shared) vs local overrides; allow/deny rules and permission modes control what runs unprompted.
- **Hooks:** deterministic scripts at lifecycle events (`PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SessionStart`...). *Why:* prompts are suggestions and hooks are guarantees. Use them to block dangerous commands (PreToolUse, exit code 2 blocks), auto-format/lint after edits, or log. If behavior *must* happen, use a hook, not an instruction.
- **Subagents:** separate context window, own prompt and tool limits, so noisy work doesn't pollute the main context.
- **Headless mode:** `claude -p "prompt"` with `--output-format json|stream-json`, `--allowedTools`, `--max-turns`, `--permission-mode`, `--resume`. *Why:* CI/CD, scripts, pipes, batch automation, with no human at the terminal. There's nobody to approve prompts, so **pre-authorize tools narrowly**. Interactive = exploration and iteration with human oversight.
- **Agent SDK:** the same agent loop, tools, and context management as Claude Code, as a Python/TS library. You configure tools, MCP servers, hooks, subagents, permissions, and sessions in code to build your own agents.

---

## 8. Eval, Testing & Debugging (2.6%)

- **Build an eval set first:** representative real inputs, edge cases, known failures. Without it you can't tell if a change helped.
- **Grading:** code-based (exact/regex/schema), LLM-as-judge (needs a clear rubric; validate against humans), human review for high stakes.
- **Change one variable at a time** (prompt, model, params). Re-run the full set on every prompt or model change to catch regressions.
- **Debug order:** check `stop_reason` and `usage` → inspect the raw request actually sent → isolate whether the failure is prompt, tool description, retrieved context, or schema → reproduce → fix → re-run evals. Outputs aren't perfectly deterministic even at temperature 0.
- **Traps:** judging by "it feels better"; testing on one example; changing several things at once.

---

## Confusable pairs cheat sheet
- Streaming (real-time delivery) vs Batch (async, ~50% cheaper)
- Schema-valid vs correct
- Workflow (fixed code path) vs Agent (model-directed loop)
- Skills (how to do things) vs MCP (connect to things)
- stdio (local subprocess) vs Streamable HTTP (remote, multi-client)
- Hook (guaranteed) vs prompt instruction (best-effort)
- Cache write (1.25×) vs read (0.1×): reuse count decides
- 4xx (fix your request) vs 429/5xx (retry with backoff)
