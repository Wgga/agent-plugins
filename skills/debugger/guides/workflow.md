## Workflow

### Step 1 — Problem Definition & Initialization (BLOCKING)

1.  **Restate the Issue**: Clearly define the difference between actual behavior and expected behavior.
2.  **Generate Session ID**: Create a semantic session identifier based on the bug description using lowercase English words with hyphens (e.g., `login-500-error`, `cart-empty-bug`).
3.  **Create debug-<sessionId>.md**: Initialize `debug-<sessionId>.md` in the project root, recording the initial status `[OPEN]`, problem description, and known reproduction steps.

> **⚠️ Core Constraint**: Modifying any business logic code is strictly prohibited during this phase. **Executing any other write operations before creating `debug-<sessionId>.md` is strictly forbidden.**

> **💡 User Guidance**: After this step, inform the user:
> - The session ID being used (e.g., `login-500-error`)
> - Where the debug file is located (e.g., `debug-login-500-error.md`)
> - What the next steps will be

Output: Bug Brief + `debug-<sessionId>.md` (Initial)

---

### Step 2 — Hypothesis Generation (3–5 items) (BLOCKING)

Generate hypotheses with **Likelihood** and **Effort** assessment to prioritize verification:

| ID | Hypothesis | Likelihood | Effort | Expected Signal |
|----|------------|------------|--------|-----------------|
| A | Description | High/Medium/Low | Low/Medium/High | What logs would confirm |

**Prioritization Rule**: Verify **High Likelihood + Low Effort** hypotheses first.

Each hypothesis must include:

*   What signals will be seen in the logs if true (falsifiable signals).
*   Where instrumentation is needed (minimal points).
*   Whether it can be determined by a single fastest reproduction.

**Example**:
| ID | Hypothesis | Likelihood | Effort | Expected Signal |
|----|------------|------------|--------|-----------------|
| A | Database connection timeout | High | Low | `ETIMEDOUT` in error object |
| B | Auth token expired | Medium | Low | `401` response code |
| C | Memory leak causing slowdown | Low | High | Memory growth pattern over time |

> **💡 Tip**: Start with Hypothesis A (High likelihood, Low effort). Refer to `guides/scenarios.md` for scenario-specific hypothesis templates.

Output: Hypothesis Table (A–E) with Likelihood/Effort

---

### Step 3 — Instrumentation Design (3–8 points)

Principle: Use the minimum amount of logs to simultaneously distinguish 3–5 hypotheses (parallel verification).

Requirements:

*   Each log must be bound to a `hypothesisId`.
*   Instrumentation code must use semantic tags: `// #region debug-point <hypothesisId>[:description]`.
*   It is recommended to include a unified log prefix in the `message` field (default `[DEBUG]`).

Output: List of Insertion Points + Expected Signals

---

### Step 4 — Instrumentation Patch (No Behavioral Changes)

*   **Strictly Prohibit Business Logic Changes**: Code changes in this step are limited only to adding instrumentation logs; fixing any discovered potential bugs or performing refactoring is forbidden.
*   **Strictly Prohibit Creating New Util Files**: Use inline one-liners or reuse existing capabilities.
*   Introduce lightweight inline reporting logic (One-liner).

> **💡 User Guidance**: After instrumentation, clearly explain to the user:
> - Which files were modified and why
> - The instrumentation code uses the env file for configuration (no manual setup needed)
> - What to do if the env file cannot be read (fallback URL)

Output: Instrumentation Diff

---

### Step 5 — Log Initialization (MANDATORY)

*   **Clear Content**: If log files related to the current `sessionId` already exist in the `.dbg/` directory, clear their contents (do not delete the files).
*   **Confirm Status**: Ensure `runId = "pre-fix"`.

Output: Reproduction Steps (must include runId/sessionId)

---

### Step 6 — Interactive Reproduction (User Participation)

1.  **Clear Logs (MANDATORY)**: Before formally inviting the user to operate, **the contents of the current Session's log files must be cleared** to ensure the collected evidence contains only data from the current run, avoiding confusion with old logs.
2.  **Summarize Current Changes**: Clearly explain to the user which instrumentation points were added and their purpose.
3.  **Provide Operational Suggestions**: Give specific reproduction paths or a Checklist.
4.  **Invite User Action**: Request the user to perform the reproduction and provide log snippets or paths.

Output: Evidence (Runtime Logs)

---

### Step 7 — Evidence-Based Analysis

> **⚠️ Log Reading Constraint**: Read logs directly from `.dbg/debug-log-<sessionId>.ndjson` using file read tool. **Do NOT use `curl` or HTTP API** to query logs.

1.  **Read Evidence**: Use file read tool to read `.dbg/debug-log-<sessionId>.ndjson` and extract relevant log entries.
2.  **Determine Hypotheses**: Mark each hypothesis as CONFIRMED / REJECTED / INCONCLUSIVE.
3.  **Present Verification Status**: Report hypothesis verification status to user in table format:

| ID | Hypothesis | Status | Evidence Summary |
|----|------------|--------|------------------|
| A | Database connection timeout | ✅ Confirmed | Line 23: `ETIMEDOUT` in error object |
| B | Auth token expired | ❌ Rejected | Line 45: Token valid, 200 response |
| C | Memory leak | ⏳ Inconclusive | Insufficient data for pattern analysis |

4.  **Update Records**: Synchronize these verification conclusions in `debug-<sessionId>.md`, referencing specific log lines.

> **💡 User Guidance**: When presenting verification results:
> - Use the table format above to clearly show confirmed/rejected/inconclusive hypotheses
> - Provide brief evidence summary for each hypothesis
> - Highlight the confirmed root cause if identified

Output: Root cause + Verification Table + `debug-<sessionId>.md` (Updated)

---

### Step 8 — Minimal Fix (Retain Instrumentation)

1.  Implement a minimal fix Patch.
2.  Summarize the fix logic for the user and explain why instrumentation is retained for comparative verification.

---

### Step 9 — Post-Fix Verification (Interactive Feedback)

1.  **Clear Logs (MANDATORY)**: Before guiding the user through fix verification, **previous debugging logs must be cleared** to ensure the post-fix evidence is pure.
2.  **Update runId**: Set `runId="post-fix"`.
3.  **Guide Verification**: Provide the suggested operational path after the fix and guide the user to interact again.
4.  **No Premature Cleanup**: All instrumentation and the Debug Server must be retained at this point so that evidence collection can immediately continue if the user reports "not fixed."
5.  **Collect User Feedback**: Use the platform's interactive feedback tools to ask whether the problem is solved (see Step 10 for options).

---

### Step 10 — User Confirmation Gate (Sole Cleanup Entry)

> **⚠️ MANDATORY: Use Interactive Feedback Tools**
>
> You **MUST** use the platform's interactive feedback tools to collect user feedback. **Do NOT** present text-based A/B/C options like:
> ```
> ❌ WRONG - Do NOT do this:
> Please select:
> A. Fixed
> B. Still reproducible
> ```

**Options**:
- **A. Fixed / No longer reproducible** → Proceed to Step 11 (Cleanup)
- **B. Still reproducible** → Return to Step 7 (Analyze post-fix logs)
- **C. Symptoms changed / Need further analysis** → Return to Step 2 (Adjust hypotheses)
- **D. Abort debugging** → Proceed to Step 11 (Cleanup)

---

### Step 11 — Cleanup & Conclusion (User Confirmation Required)

> **⚠️ Entry Condition**: Can only proceed after user confirms Fix (Option A) or Abort (Option D).

1.  **Remove Instrumentation**: Clean up all `#region debug-point` blocks.
2.  **Terminate Debug Server**: If a Debug Server is running, stop it.
3.  **Close Session**: Update `debug-<sessionId>.md` status to `[FIXED]` or `[ABORTED]`.
4.  **Submit Summary**: Present root cause, fix-oneliner, and key evidence.
5.  **Delete debug-<sessionId>.md** and `.dbg/*.env` files.