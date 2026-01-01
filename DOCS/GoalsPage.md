Below is a **concise, implementation-ready markdown** describing the **Goals page functionality** in HabitFlow.
It is written to be **consistent with the canonical domain rules and the iOS Feature Spec**, and avoids redefining semantics.

---

# Goals Page — Functional Specification

## Purpose

The **Goals page** provides **long-horizon orientation and motivation** by aggregating effort derived from Habits.
It answers: **“Am I moving toward what I care about?”**

It does **not** track behavior, enforce schedules, or store progress.

> Goals are **read-only aggregations** over `HabitEntry`. All progress is derived. 

---

## Core Principles (Non-Negotiable)

1. **Goals never own history**

   * No goal-level logs
   * No goal-level completion flags
   * No stored progress counters

2. **All progress is derived**

   * Computed from `HabitEntry` via `GoalLink`
   * Recomputable at any time

3. **Goals are non-punitive**

   * No overdue states
   * No failure indicators
   * Target dates are informational only

4. **Habits drive goals (one-way)**

   * `HabitEntry → Goal aggregation`
   * Never `Goal → HabitEntry` without explicit user action

---

## Page Structure

### 1. Goal List View (Primary)

**Displays**

* Active goals grouped by **Category**
* Each goal rendered as a **card or row**

**Each Goal Card Shows**

* Goal title
* Optional description
* Progress indicator (derived)
* Target value + unit (if applicable)
* Linked habits count
* Status badge (Active / Paused / Completed — informational)

**Does NOT Show**

* Daily completion states
* Streaks
* Missed-pace warnings

---

### 2. Goal Detail View

Opened by tapping a goal.

**Sections**

#### A. Goal Overview

* Title
* Description
* Goal type (Quantity / Count)
* Target value
* Optional target date (non-punitive)
* Category

#### B. Progress Summary (Derived)

* Current aggregated value
* Percentage toward target
* Visual progress bar or ring

> This view is always computed from HabitEntries at render time. 

#### C. Linked Habits

* List of contributing habits
* Each habit shows:

  * Contribution mode (count / sum)
  * Unit (if numeric)
* Read-only configuration view

#### D. Contribution History (Optional, Read Model)

* Chart or summary derived from entries
* No editing here

---

## Goal Types & Aggregation

### 1. Quantity Goals

Examples:

* “Run 500 miles”
* “Write 100,000 words”

**Aggregation**

* `sum(HabitEntry.value)` across linked habits
* Units must match or be explicitly convertible

---

### 2. Count Goals

Examples:

* “Exercise 100 times”
* “Practice piano 50 sessions”

**Aggregation**

* `count(distinct HabitEntry.id)` across linked habits

---

## User Actions & Allowed Mutations

### 1. Create Goal

User can:

* Define title, description
* Choose goal type
* Set target value and unit
* Assign category
* Link one or more habits

**Creates**

* `Goal`
* One or more `GoalLink` records

**Does NOT**

* Create HabitEntries
* Backfill progress

---

### 2. Edit Goal

User can:

* Edit title, description
* Adjust target value
* Add/remove linked habits
* Pause or resume goal

**Effects**

* Progress recomputes immediately
* No history rewritten

---

### 3. Delete Goal

**Deletes**

* Goal object only

**Must NOT**

* Delete HabitEntries
* Mutate habits

---

### 4. Log Progress from Goal UI (UX Shortcut)

Allowed **only as a helper flow**.

**Rules**

1. User must select a source habit
2. System creates or updates a `HabitEntry`
3. Goal progress updates naturally via aggregation

**Forbidden**

* Updating goal progress directly
* Creating goal-owned logs

---

## Empty & Edge States

### Goal with No Linked Habits

* Show progress = 0
* Prompt user to link habits
* No manual progress entry allowed without habit

---

### Habit Deleted While Linked

* Existing HabitEntries remain
* Goal recomputes with remaining data
* Graceful degradation

---

## Sorting & Organization

* Goals grouped by **Category**
* User-reorderable within category (optional)
* Category order mirrors Habits/Goals shared ordering model

---

## Explicit Anti-Patterns (Hard No)

The Goals page must NEVER:

* Store `goalProgress` as truth
* Create goal-level entries
* Auto-complete habits
* Penalize missed pace
* Show streaks
* Infer completion without `HabitEntry`

---

## Derived-Only Guarantee

If **all derived caches are deleted**, the Goals page must still fully reconstruct:

* Current progress
* Historical contribution
* Completion state

from **HabitEntries + GoalLinks alone**.

---

## One-Sentence Summary

**The Goals page is a read-only, non-punitive aggregation surface that orients users toward long-term commitments by deriving progress entirely from HabitEntries.**

