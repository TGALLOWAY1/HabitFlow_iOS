# Dashboard (Today) Page — Functional Spec

## Purpose

The **Dashboard (Today)** page is the primary daily entry point to HabitFlow.

Its role is to:

* Orient the user to *today*
* Enable fast, low-friction action
* Surface supportive structure (routines, focus, reflection)
* Highlight a small set of important habits
* Provide motivation **without evaluation or punishment**

The Dashboard does **not** own truth, scheduling, or progress.
It is an orchestration layer over canonical objects.

---

## Canonical Constraints (Non-Negotiable)

* **HabitEntry is the sole source of behavioral truth**
* All completion indicators on this page are **derived**
* No object on this page stores:

  * `isComplete`
  * streaks
  * progress
* Routines **never** complete habits
* Journals and check-ins **never** imply completion
* Missing actions are never framed as failure

If removing this page does not change any underlying data, the design is correct.

---

## Page Sections (Top → Bottom)

---

## 1. Header / Orientation

### Purpose

Provide temporal and emotional grounding without evaluation.

### Elements

* **Date pill**

  * Format: `Today, Mon Dec 30`
  * Derived from DayKey
* **Primary greeting**

  * Example: `Good morning.`
* **Subtext**

  * Soft, non-instructional framing
  * Example: `Focus on what brings you calm today.`

### Rules

* No performance metrics
* No streaks
* No obligations

---

## 2. Time-Based Routines

### Purpose

Offer optional structure anchors for the day.

### UI

* Horizontal card list
* Section label: `TIME-BASED ROUTINES`
* Default cards:

  * Morning Routine
  * Evening Routine

### Interaction

* Tap routine → **Start Routine**

  * Creates a `RoutineExecution`
  * May generate `HabitPotentialEvidence`
  * Never creates `HabitEntry` automatically

### Rules

* Starting a routine ≠ completion
* No success/failure state
* No requirement to finish

---

## 3. Quick Journal / Intention

### Purpose

Enable lightweight reflection without blocking action.

### UI

* Card with:

  * Title: `Quick Journal / Intention`
  * Text input
  * “Add” button

### Behavior

* Submitting creates a `JournalEntry`
* Entry is optional and skippable
* JournalEntry:

  * Does not affect habits
  * Does not affect goals
  * Does not affect streaks

### Placeholder Copy

> Set your intention or write a short note…

---

## 4. Daily Check-In

### Purpose

Capture subjective state for context and later insight.

### UI

* Card title: `Daily Check-In`
* Three metric buttons:

  * Mood
  * Energy
  * Stress

### Interaction

* Tap metric → value picker (e.g. 1–5)
* Creates or updates a **WellbeingMetricEntry** (if adopted)

### Rules

* Check-ins are **orthogonal** to behavior
* They never imply completion
* They never affect goals or habits

---

## 5. Focus Session

### Purpose

Provide a simple entry point into deep work.

### UI

* Card title: `Focus Session`
* Duration buttons:

  * 30 min
  * 1 hr

### Interaction

* Selecting duration starts a focus session (timer UX)
* No habit completion implied
* Optional future linkage to routines or habits (non-automatic)

---

## 6. Prioritized Habits

### Purpose

Surface a *small, curated* subset of habits that matter today.

This is **not** the full habits list.

### UI

* Section label: `PRIORITIZED HABITS`
* Max 3–5 habits
* Each row contains:

  * Icon
  * Habit name
  * Optional detail (e.g. `(10 min)`)
  * **Square checkbox on the right**

### Completion Indicator

* Checkbox state is **derived**:

  * `checked` iff a `HabitEntry` exists for `(habitId, todayDayKey)`
* No stored completion flags

### Interaction

* Boolean habit:

  * Tap checkbox → create/delete today’s HabitEntry
* Numeric habit:

  * Tap row → numeric input sheet
  * Save → create/update today’s HabitEntry

### Rules

* One entry per habit per day (default)
* Editing affects all derived metrics
* No penalties for uncompleted habits

---

## 7. Goal Anchors

### Purpose

Provide long-horizon motivation without pressure.

### UI

* Section label: `GOAL ANCHORS`
* Horizontal chips/cards
* Example goals:

  * Improve Sleep
  * Reduce Stress
  * Learn Spanish

### Interaction

* Tap goal → Goal detail view
* Goals are read-only aggregations of HabitEntries

### Rules

* Goals never create entries
* Goals never track independently
* No deadlines or “behind” states

---

## Data Dependencies (Read-Only)

The Dashboard may **read** from:

* Habit
* HabitEntry (filtered by today DayKey)
* Routine
* RoutineExecution
* HabitPotentialEvidence
* Goal
* JournalEntry
* WellbeingMetricEntry (optional)
* Persona / Identity (for framing only)

The Dashboard may **write** only via explicit user actions:

* Creating/editing HabitEntries
* Creating RoutineExecutions
* Creating JournalEntries
* Creating WellbeingMetricEntries

---

## Non-Goals

The Dashboard does **not**:

* Replace the Habits page
* Act as a planner or scheduler
* Enforce daily completion
* Display punishment or overdue states
* Store progress, streaks, or metrics

---

## One-Sentence Summary

**The Dashboard (Today) page is a non-punitive, action-oriented home surface that helps users orient to the day, take meaningful action, and reflect lightly—without owning truth, enforcing schedules, or evaluating performance.**

---