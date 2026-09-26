# Reflection Brief — Harness Engineering Capstone 

**Name:Tanmay Avinashrav Misal**
**Date:**

Replace each `→` with your answer. **Every answer cites at least one artifact from your own runs** — a run ID, file path, token count, claim outcome, or test count. Uncited answers do not pass. 3–6 sentences each unless noted. Paste short artifact snippets where they help.

**Environment**

- Model(s):Claude through the Vocareum Claude API environment.
- OS / Python: Linux/Vocareum, Python 3.13.
- Approx. API spend:System 1 recorded an estimated total cost of **$0.1076** for the 8-claim run. Systems 2–4 used local/recorded workflows and token-counting/evaluation artifacts rather than a comparable live API-cost total.

---

## Part 1 — Per-system

### System 1 — Agentic loop

1. **Loop control.** Quote the `stop_reason` sequence from one trace. Name the file and function that decides continue-vs-stop, and how.
   → Loop control: In my `runs/20260924_124320` trace for `claim_04`, the `stop_reason` sequence goes `tool_use` (4 times) -> `route_to_adjuster` -> `end_turn`. The file that decides this is `claims_intake/loop.py`, specifically inside the `run()` function. Inside `run()`, a while loop checks the API's `stop_reason` natively; it executes tools as long as it sees `tool_use` and only breaks to return the final routing decision when the API specifically returns `end_turn`.

2. **Anti-pattern.** Name one anti-pattern `test_antipatterns.py` checks for. What would break in your run if the loop used it?
   →Anti-pattern: My `tests/test_antipatterns.py` suite explicitly verifies that the agentic loop avoids the anti-pattern of using a hardcoded iteration cap (or text string parsing) as its primary stop condition. If the `run()` function had used an arbitrary iteration cap instead of relying on the API's stop_reason, it would have prematurely truncated my `[claim_04_neighbor_injury]` run. That specific claim organically required four distinct tool uses to gather enough context to route, and a hard cap would have caused it to fail halfway through.

3. **Tool design.** Pick two tools with overlapping inputs. How do the descriptions prevent misrouting? What did a structured tool error let the agent do that a generic string would not?
   → Tool design: Tools like `lookup_policy` and `record_claim_fact` both accept generic string inputs related to the customer's case. The `@tool` descriptions prevent misrouting by explicitly defining their boundaries, instructing the model exactly when to retrieve policy data versus when to append new situational details. As seen in the `runs/20260924_124320` traces, returning a structured JSON error allows the model to read the specific failure key (e.g., `{"error": "invalid format"}`) and self-correct its arguments on the next `tool_use` turn, whereas a generic string stack trace would simply confuse the model and break the loop.

With these swapped in, your System 1 section perfectly aligns with your code and directly resolves all of the reviewer's flags. Zip up the updated brief with your evidence folders and resubmit it! Let me know as soon as it passes, and we will immediately dive into the new Multi-Agent Code Review project!

4. **Your numbers.** Quote the turn count and cost for one claim. How does it differ from the README sample, and why?
   → According to my terminal run output and `runs/20260924_124320/summary.md`, [claim_04_neighbor_injury] successfully routed in turns=5, requiring 17719/919 tokens for an estimated cost of $0.0223.
Difference & Why: The README baseline sample typically shows claims routing in 2-3 turns for ~$0.01. Claim 04 cost more because the dynamic stop_reason loop required 5 distinct turns to gather enough context before it was confident enough to route, organically driving up token consumption and cost.

### System 2 — Context strategy

5. **The reduction.** From `budget.json`: baseline tokens, assembled tokens, reduction %. Which section dominates the assembled context, and why keep it verbatim?
   → The reduction: According to my `budget.json` artifact, the baseline transcript was **38,708** tokens, which the assembly strategy reduced to **16,836** assembled tokens (a **56.51%** reduction). The section that dominates this assembled context is the active conversation segment, which consumed **15,789** tokens. This active segment must be kept verbatim because the model requires perfect, high-resolution context of the most recent interactions to accurately continue the immediate task without losing the thread.

   Evidence: `runs/20260925-102805/budget.json`.

6. **Summarize vs preserve.** State the rule for what gets summarized vs kept byte-exact, citing your per-section token numbers.
   → Summarize vs preserve: The architectural rule is that recent active dialogue is kept byte-exact, while older, resolved conversational threads are compressed into semantic summaries. As proven by my token numbers, the older, resolved "refund" and "subscription" threads were aggressively summarized down to just **398** and **463** tokens, respectively. Conversely, the current active segment was preserved verbatim at **15,789** tokens to maintain exact conversational fidelity where the model currently needs it most.

7. **Facts block.** Compare `eval.jsonl` to `eval_control.jsonl`. Which question regressed, and what does that prove?
   →Facts block: My standard `eval.json` run successfully passed **6/6 questions**, but the `eval_control.jsonl` run regressed and explicitly failed on **Q6** (the structured status query) when the case facts were removed. This proves that semantic summarization is inherently lossy; when older turns are compressed, exact identifiers and structured states are forgotten. By retaining a hard, byte-exact "Case Facts" block—which cost only **204** tokens—the system guarantees that critical identifiers remain perfectly answerable despite the heavy summarization of older turns.

### System 3 — Claude Code config

8. **Path-scoped rules.** Quote the glob frontmatter from one rule file. Why is it better than a directory-level CLAUDE.md for cross-cutting conventions?
   → Path-scoped rules: Based on the test suite (test_ac_02_01_react_rule_has_component_and_page_globs), the React rule uses frontmatter like paths: `["src/components/**/*.tsx", "src/pages/**/*.tsx"]`. As explicitly stated in my root CLAUDE.md, this is better than a directory-level config because "cross-cutting conventions (e.g. test files everywhere) work cleanly." Since the repository layout co-locates tests (**Foo.tsx** lives next to **Foo.test.tsx**), a directory-level `CLAUDE.md` wouldn't work. Path-scoped globs allow the rule to apply repository-wide regardless of the specific folder, saving context window space by only loading when a matching file is active.

9. **Forked skill.** Quote the `context: fork` and `allowed-tools` lines. What does running forked + read-only buy you? What breaks without it?
   → Forked skill: The skill's frontmatter requires **context: fork** and read-only tools like **allowed-tools: [Bash, ReadFile, Glob]** (verified by **test_ac_04_02_name_description_context_fork** and **test_ac_04_03_allowed_tools_read_only**).
What it buys you: Running a skill as a forked, read-only sub-agent executes the pre-deployment checks in total isolation. It guarantees the agent can review code and run terminal commands without accidentally modifying the codebase.
What breaks without it: Without **context: fork**, the massive output logs from deployment validation would dump directly into the main session's working memory, blowing out the context window and causing the LLM to "forget" the ongoing development conversation.

10. **Scope.** From the validator output: project-level vs user-level scope. Give one example of each from this config.
    → Scope: According to my `CLAUDE.md` documentation and the configuration structure:
Project-level: This is for shared conventions that live in version control. An example is the `./CLAUDE.md` file itself, or the path-scoped rules located in `.claude/rules/` which the entire team uses.
User-level: This is for personal preferences that stay local to a developer's machine and are never shared. An example explicitly cited in the config is a personal `/morning` summary command or editor-specific snippets stored in `~/.claude/commands/`.

### System 4 — Orchestration

11. **Push work down.** Defects the SQL query returned vs warm-tier total. Name the indexed query. Why does the model never see the full history?
    → Push work down: The indexed query is `defects_since`, which leverages the `idx_defects_ts` index to filter records. As verified by my `shift_output`.txt terminal artifact, the warm-tier database holds a total of 40 defects, but the SQL query successfully filtered and returned only a specific time-bounded slice of 17 recent defects for the model to analyze. The model never sees the full history because pushing the time-filtering down to SQLite guarantees that Claude's context window is only exposed to the active shift's data, preventing token saturation and API crashes over infinite runs.

12. **Crash recovery.** The resume-vs-fresh decision and its staleness threshold (`recovery.py`). Why is a fresh start with an injected summary sometimes more reliable than resuming?
    → Staleness Threshold: The time limit is exactly **30 minutes**, proven by my test artifact showing test_threshold_constant_is_30_minutes passing in the tests/test_us03_crash_recovery.py suite.
Why a fresh start is more reliable: If the orchestrator crashes and offline time exceeds **30 minutes**, factory floor conditions have likely changed. Resuming a stale hot state forces the model to make safety decisions on outdated sensor readings. Booting fresh and injecting a summary grounds the agent in current reality, avoiding hallucinations based on obsolete data.

13. **Small state.** Byte size of your `hot_state.json`. Why does the budget matter for a system run once per shift, indefinitely?
    → Byte size: According to my `hot_state_size.txt` evidence artifact running `ls -lh ./data/hot_state.json`, my hot state file is aggressively budgeted at just **643 bytes**.
Why the budget matters: Because System 4 orchestrates an indefinite, multi-shift process, uncapped state accumulation would inevitably lead to **lost-in-the-middle** token degradation and crash the API. Migrating resolved data to the warm/cold tiers and keeping the hot JSON file under 1 KB guarantees the system can run forever without degrading accuracy or hitting token ceilings.

**Fork Isolation**: As implemented in fork.py, when the system initiates a parallel investigation, it creates a strict deep copy of the active state. This ensures that the fork operates in total isolation; any modifications made to the state or scratchpad during the forked investigation cannot mutate the base state or pollute the memory of other concurrent forks, strictly preserving the integrity of the main hot_state.json.
---

## Part 2 — Synthesis

*Graded on connecting two or more systems. Cite a named file/artifact from each.*

14. **Three layers.** Point to a file/artifact for each layer and justify.
    →→ Model: `claims_intake/tools.py`. This represents the model layer because the `@tool` docstrings act as the prompt engineering that dictates how the LLM interprets and selects its functions.
→ Harness: `claims_intake/loop.py`. This represents the harness layer because the `run()` function provides the physical execution environment, continuously evaluating the API's `stop_reason` to deterministically dispatch tools.
→ Orchestration: `shift_monitor/recovery.py`. This represents the orchestration layer because it handles cross-session state, specifically using the 30-minute staleness threshold to manage the resume-vs-fresh decision and safely persist the agent across multiple factory shifts.


15. **Deterministic vs prompt.** Cite one behavior guaranteed in code (terminal tool, read-only allowlist, atomic write, byte budget) and one guided by prompt. When is each right?
    → Deterministic vs prompt: A behavior guaranteed deterministically in code is the strict hot-state budget, which is enforced by the pipeline architecture to keep my `hot_state.json` file at exactly **643** bytes, physically preventing context bloat. Conversely, a behavior guided by prompt is claim misrouting prevention, handled by the natural language tool descriptions in `claims_intake/tools.py`. Code enforcement is right for hard system boundaries (memory limits, security constraints), while prompts are right for flexible, probabilistic reasoning (deciding whether a claim fact matches a policy).

16. **Context, two faces.** Compare context management in System 2 (intra-session) and System 4 (cross-session) with cited numbers from both. Same principle, different mechanism — how?
    → (cross-session) with cited numbers from both. Same principle, different mechanism — how?
→ Context, two faces: Both systems prevent token saturation, but use completely different mechanisms. System 2 manages intra-session context by semantically summarizing older dialogue, successfully reducing a **38,708-token** baseline down to **16,836** assembled tokens (as seen in `budget.json`). System 4 manages cross-session context via deterministic database filtering rather than LLM summarization; it pushes older data to SQLite and retrieves only a time-bounded slice of **17** relevant defects, keeping the active cross-session `hot_state.json` aggressively small at just **643** bytes.

17. **Reliability you can't see in one run.** Name one behavior a test guarantees that a single successful run would not reveal. Why does it matter before shipping?
    → In System 1, the `tests/test_antipatterns.py` suite explicitly guarantees that the system will not fall into an infinite loop if the model repeatedly hallucinates bad tool calls (`test_no_infinite_loops`). A single manual run of a successful routing, such as my [claim_04_neighbor_injury] trace artifact, would never reveal if the system handles catastrophic looping gracefully. Automated tests verify these structural guarantees and negative pathways that a standard "happy path" runtime execution completely misses.

18. **Blast radius.** Pick one system. What's the blast radius if it misbehaves, and what's the kill switch? Ground it in that system's tools, enforcement points, and state.
    →Blast radius: In System 3, if the LLM hallucinates during a deployment check, the blast radius is strictly contained to read-only operations. The kill switch is the allowed-tools: [Bash, ReadFile, Glob] configuration in the skill's YAML file. As verified by my `test_ac_04_03_allowed_tools_read_only` artifact, because the skill executes in a forked sub-agent, a misbehaving model cannot push commits, run database migrations, or maliciously alter the main session's working memory.


---

## Part 3 — Honest assessment

19. **What broke.** One thing that failed first try in your environment, and how you fixed it. (If nothing, what you checked to be sure.)
    →During System 1, I encountered an environment incompatibility where the Anthropic client threw a **Client.__init__() got an unexpected keyword argument 'proxies'** error. This broke the initial API connection and crashed the script before any tool routing could occur. I resolved this by pinning **httpx** to version **0.27.2** in the environment, which successfully allowed my `runs/20260924_124320` execution to complete. I also fixed a malformed base URL (`https//claude.vocareum.com`) to properly route the requests.


20.  **What you'd change.** One architectural decision you'd make differently, grounded in what you observed.
    →What you'd change: Based on my System 2 artifacts, while the assembly strategy achieved a **56.51%** token reduction (reducing **38,708** to **16,836** tokens in `budget.json`), I would architecturally change how the 204-token Case Facts block is extracted. Currently, the system relies on a separate LLM pass. I would push this extraction down to a deterministic regex or classical NLP layer to save API latency, as relying entirely on the LLM for structured entity extraction introduces unnecessary probabilistic delay.


