# 13. Business Process Flow

A model-driven app shows data on a form, but it doesn't tell a user *where in the journey* a record is. The business process flow (BPF) adds that — a stage bar across the top of every Onboarding Record that shows, at a glance, whether a hire is in pre-boarding, being prepared, actively onboarding, or done.

The BPF is named **Onboarding Process** (`pex_onboardingprocess`) and runs across the Onboarding Record table.

---

## The Onboarding Process — Four Stages

```mermaid
flowchart LR
    A[Pre-Boarding] --> B[Day 1 Preparation] --> C[Onboarding In Progress] --> D[Completed]
```

| Stage | What it represents |
| --- | --- |
| **Pre-Boarding** | The record exists and tasks have been generated, but work hasn't meaningfully started |
| **Day 1 Preparation** | Onboarding is underway; the team is working through tasks ahead of the start date |
| **Onboarding In Progress** | The hire is Day-1 ready; onboarding continues post-arrival through probation |
| **Completed** | All tasks done, probation outcome recorded, record closed |

The stages map onto the lifecycle the rest of the solution already tracks, so the bar is a visual summary of state the system is maintaining anyway — not a separate thing a user has to update by hand.

---

## How the Stage Is Driven

The defining design choice here is that **the BPF bar is driven by automation, not by the user clicking "Next stage."** A user never advances the bar manually. Instead, the **Sync BPF Stage to Status** flow (Section 12) watches the Onboarding Record and moves the stage to match the record's real state.

The flow reads two signals from the record — **Record Status** and **Day 1 Readiness** — and computes which stage the bar should sit on:

| Stage | Condition |
| --- | --- |
| Pre-Boarding | Record Status is Not Started |
| Day 1 Preparation | Record Status is In Progress **and** Day 1 Readiness is not yet Ready |
| Onboarding In Progress | Day 1 Readiness is Ready |
| Completed | Record Status is Completed |

The middle transition is the interesting one: the move from Day 1 Preparation to Onboarding In Progress is gated on **Day 1 Readiness reaching Ready**, evaluated with a conditional expression on the readiness value. A record can be actively In Progress yet still held at "Day 1 Preparation" until its critical tasks are done and readiness flips to Ready. That keeps the bar honest — it advances on genuine readiness, not just on time passing.

---

## Why At-Risk Records Hold

A record flagged **At Risk** has a Day-1 Readiness that is not Ready. Because the advance to Onboarding In Progress is gated on Ready, an At-Risk record **cannot skip ahead** — it holds at its current stage until the gap is closed. This was verified in testing: records flagged At Risk at intake held at Pre-Boarding rather than drifting forward, exactly as intended. The bar therefore doubles as a risk signal — a record stuck at an early stage close to its start date is visibly behind.

---

## Engineering Notes

Driving a BPF from a flow is not as simple as writing a stage value onto the record, and the build went through three failure iterations before landing on the supported approach. These are recorded because they are genuine, non-obvious platform behaviours:

**The deprecated `stageid` field doesn't work for this.** Writing the target stage onto the record's `stageid` column failed in two distinct ways — it rejects bind syntax, and it also rejects bare GUIDs. It is effectively a dead end for driving a BPF from automation.

**The BPF is advanced via its own instance table.** The working pattern is to read the record's BPF instance with a Get row, then **Update the business process flow instance table**, targeting `businessprocessflowinstanceid` and setting **Active Stage** with a bind expression. This is the supported mechanism and is what the Sync flow uses.

**Raw integer choice values, because FormattedValue isn't on trigger outputs.** The flow compares Record Status and Day 1 Readiness using their stored integer values rather than their text labels, since the human-readable FormattedValue annotation is not available on trigger outputs. The trigger's selected columns were expanded to include both `pex_recordstatus` and `pex_day1readiness` so both signals are present to evaluate.

Together these make the bar reliable: it reflects the record's true state, advances only on real readiness, and never has to be touched by a user.

---

➡️ Next: **[Section 14 — Security Model](../14-Security-Model)**
