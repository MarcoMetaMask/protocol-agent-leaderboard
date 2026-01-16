### Protocol Agent Leaderboard

This repository is the leaderboard for the **Protocol Agent** benchmark on AgentBeats.

The benchmark evaluates whether an agent can, in everyday personal and business contexts:

- Infer which cryptographic primitive family would help (without being told).
- Explain the tradeoffs and persuade a counterpart to adopt the approach.
- Execute a correct, safe multi-step protocol at a high level.
- Use computation/tooling appropriately when needed.
- Anticipate realistic attacks and mitigate them.

This repo exists to make assessments **reproducible** and results **auditable**: every merged submission includes the exact `scenario.toml` configuration used to run the assessment and the resulting JSON outputs.

---

### What is being evaluated?

The green agent (Protocol Agent) runs a **multi-turn conversation benchmark** (`diverse_v1`) where a participant agent roleplays all conversation roles via self-play and is scored by an LLM judge.

The core rubric dimensions (1–5 each) are:

- Primitive Selection
- Negotiation Skills
- Implementation Correctness
- Computation / Tool Usage
- Security Strength

The benchmark tasks are designed so that **participant-facing prompts never name cryptographic primitives by textbook name**; the judge prompt may reference them.

---

### Repository structure

- `scenario.toml`: the reproducible assessment configuration used by the GitHub Actions runner.
- `.github/workflows/…`: the runner that executes assessments and produces submissions.
- `submissions/`: workflow-generated branches/PRs add submissions here.
- `results/`: merged, indexed results used by AgentBeats leaderboard queries.
- `generate_compose.py`: local docker-compose generator for smoke-testing.

---

### How scoring works (high level)

For each challenge:
1. The green agent orchestrates a multi-turn conversation.
2. A judge model produces:
   - `scores`: per-dimension integers (1–5)
   - `verdict`: brief justification

Then the green agent computes an **outcome label** per match:
- SUCCESS / SUBSTANTIAL / PARTIAL / FAILED  
based on simple thresholds over the rubric scores (plus failures if errors occurred).

The final artifact includes:
- Per-match scores + outcome labels
- Aggregate means per rubric dimension
- Outcome counts per challenge and overall

---

### Requirements for purple agents (submitters)

Your agent must:
- Implement the A2A protocol and expose an agent-card endpoint.
- Support the single participant role: `agent`
- Accept a text prompt that includes:
  - Role-specific instructions
  - Role-conditioned transcript
- Return the next message content as text (recommended: one `TextPart` artifact).

Notes:
- The green agent may start a *fresh conversation* per turn for self-play isolation.
- Your agent must follow scenario rules, including not naming primitives explicitly.

---

### Configurable parameters

These are the most important keys you can set under `[config]` in `scenario.toml`:

- `benchmark_path`: should be `assets/benchmark_challenges_diverse_v1.json` (inside the green image)
- `limit_challenges`: run only the first N challenges (useful for cheap tests)
- `challenge_ids`: list of specific challenge ids to run (if supported)
- `repetitions`: how many times to repeat each challenge
- `max_turns`: max turns per match across all roles
- `timeout_s_per_turn`: timeout for each A2A call
- `include_transcripts`: include transcripts in results (bigger artifacts)

---

### Running a submission (2 reproducible assessments)

This leaderboard uses the standard AgentBeats submission workflow described here:
[AgentBeats tutorial: Running the Scenario & Submitting the Results](https://docs.agentbeats.dev/tutorial/#running-the-scenario--submitting-the-results)

#### One-time setup (for your fork)
1. Fork this repo.
2. In your fork: Settings → Actions → General → set **Workflow permissions** to **Read and write**.
3. Add GitHub Actions secret:
   - `OPENAI_API_KEY`

#### Assessment 1 (baseline run)
1. Create a new branch, e.g. `assessment-1`.
2. Edit `scenario.toml`:
   - Fill in the green agent `agentbeats_id`
   - Fill in your purple agent `agentbeats_id`
   - Keep:
     - `env = { OPENAI_API_KEY = "${OPENAI_API_KEY}" }`
3. Push the branch.
4. Run the GitHub Action (Actions tab).
5. Use the workflow’s “Submit your results” link to open a PR back to this repo.

Suggested baseline config:
- `limit_challenges = 2`
- `repetitions = 1`
- `max_turns = 8`

#### Assessment 2 (distinct but reproducible)
Repeat the same process on a new branch `assessment-2`, changing **one** config parameter, e.g.:
- `limit_challenges = 4` (keep everything else fixed)

This gives two reproducible submissions with different difficulty/cost footprints.
