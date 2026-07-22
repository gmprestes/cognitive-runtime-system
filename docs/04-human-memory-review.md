# Human Memory Review

**English** · **[Português (Brasil)](pt-BR/04-human-memory-review.md)**

## Thesis

> Humans should not review *every answer* from the swarm.  
> They should review **what the organization is allowed to remember and reuse**.

That concentrates human attention at the leverage point: **cold memory**.

## Why this strengthens the CRS thesis

| Approach | Effect |
|---|---|
| Human approves every tool call | slow, expensive, abandoned |
| Human only looks at the final result | rotten memory was already written |
| Human reviews **candidate memory** | fast swarm + learning with an owner |

The swarm stays autonomous on the hot path.  
The company governs what becomes operational capital.

## Role: Memory Reviewer

Not a “prompt engineer.”  
Closer to:

- living runbook editor  
- curator of operational truth  
- auditor of postmortem lessons  

Responsibilities:

1. accept / edit / reject warm→cold proposals  
2. restrict scope (this repo, this team, staging only…)  
3. resolve conflicts between beliefs  
4. force retract when the world changed  
5. (optional) sign high-risk gates  

## Minimum UX (cards, not a scary graph)

Reviewers do not need to “draw a graph.” They see cards:

```text
MEMORY PROPOSAL
Type: procedure
Claim: "After schema changes, run migrate + typecheck before the PR"
Scope: repo veluacode
Based on: 3 episodes (2 ok, 1 failure when skipped)
Evidence: CI #4821, #4829
Estimated impact: less rework on schema PRs

[ Accept ]  [ Accept with edits ]  [ Reject ]  [ Defer ]
```

Under the hood, the runtime writes nodes, `REVIEWED_BY` edges, and cold status.

## Human decision states

| Decision | Effect |
|---|---|
| `accept` | promote to cold; lesson/belief `active` |
| `accept_with_edits` | content/scope changed; provenance keeps origin + amendment |
| `reject` | do not promote; reason stays in the graph |
| `defer` | remains warm with an SLA |
| `retract` | remove from active; `SUPERSEDES` or `retired` |

## Authority matrix (example)

| Memory type | Verifier only | Human required | Dual control |
|---|---|---|---|
| Local fact confirmed by test | ✅ | | |
| Dev procedure | ✅ or human | preferably human the first time | |
| Production constraint | | | ✅ |
| Product/style preference | | ✅ | |
| Security / privacy | | | ✅ |

Each company instantiates the matrix. CRS requires that the matrix **exists** and is enforced in the runtime.

## Full feedback loop

```text
swarm works
  → evidence
  → memory proposal (warm)
  → human/verifier review
  → MemoryGraph commit (cold)
  → retrieve on next orchestration
  → HELPED / HURT / UNUSED
  → supersede / retire / re-review
```

## Metrics that matter for the reviewer

- human time / week on memory review (goal: low and stable)  
- reject rate (bad writer vs healthy rigor)  
- repetition rate of errors already covered by a lesson  
- % of cold memory with incomplete evidence (should trend to 0)  

## Anti-patterns

- turning review into “chat with the agent again”  
- accepting everything to clear the queue  
- rejecting everything without reason (breaks the writer)  
- a single human bottleneck without scope delegation  
- mixing code review and memory review without distinction  

## Product line

**Code review** improves today’s artifact.  
**Memory review** improves tomorrow’s organization.
