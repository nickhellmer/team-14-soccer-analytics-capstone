# Possession Logic – Overview

This document defines what a *possession* is in the context of this project and describes how possession IDs are assigned to events in the StatsBomb dataset.

The goal is to group individual on-ball events (passes, carries, shots, etc.) into meaningful possession sequences so we can analyze:
- possession length and duration
- shots and xG buildup per possession
- momentum and game flow within matches

This logic supports downstream analytics and dashboard visualizations.

---

## What Is a Possession?

At a high level, a **possession** represents a continuous period during which one team controls the ball.

A possession:
- **starts** when a team gains control of the ball
- **ends** when control is lost, play stops, or a terminal action occurs

Because possession is not perfectly defined in raw event data, there are multiple reasonable ways to operationalize it.

---

## Possession Definition Options Considered

### Option 1: Simple / Conservative (Initial Implementation)

A new possession starts when:
- the team in control changes (e.g., interception, turnover, tackle)
- play restarts (kickoff, free kick, throw-in, corner)
- the previous possession ends in:
  - a shot
  - a foul
  - the ball going out of play

**Pros**
- Easy to implement and reason about
- Commonly used in exploratory soccer analytics
- Low implementation risk

**Cons**
- Does not perfectly capture very short stoppages or rebounds

---

### Option 2: Event-Based with Time Gaps

A new possession starts when:
- the team changes **OR**
- there is a stoppage longer than a fixed time threshold **OR**
- the ball is dead (out of play, foul, offside)

**Pros**
- More granular and realistic in theory
- Better for detailed tactical analysis

**Cons**
- Requires careful timestamp handling
- Higher complexity and more edge cases

---

### Option 3: StatsBomb-Style Approximation

A new possession starts based on:
- team changes
- duel outcomes
- loose-ball recovery logic
- rebound handling

**Pros**
- Closest to professional analytics pipelines
- Most faithful to how advanced providers define possession

**Cons**
- Hardest to implement
- Requires validation and iteration

---

## Chosen Definition (Current)

**The project currently uses Option 1 (Simple / Conservative).**

This provides a clean and stable baseline that supports early analytics and visualization. The logic can be refined later without breaking downstream components.

---

## Implementation Summary

Using the StatsBomb `events` table:
- Events are sorted by `match_id` and event timestamp
- A new `possession_id` is created whenever possession-ending conditions are met
- Each event is assigned exactly one `possession_id`
- Each possession belongs to a single match and a single team

---

## Output Columns

After processing, the events dataset includes:
- `possession_id`: unique identifier for each possession sequence
- (optional future fields)
  - `possession_team`
  - `possession_start_time`
  - `possession_end_time`
  - `possession_duration`

---

## How Possessions Are Used

Possession IDs are used to:
- aggregate possession-level statistics
- compute xG flow and momentum over time
- enable match-level drilldowns in the dashboard
- support advanced metrics such as PPDA and field tilt

---

## Future Improvements

Potential enhancements include:
- merging very short possessions
- handling rebounds and blocked shots more intelligently
- adding time-based possession splitting
- validating results against known high-possession teams

This document will evolve as the possession logic matures.
