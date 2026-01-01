# Journal Page — Functional Specification

## Purpose & Philosophy

The **Journal page** is a **reflection surface**, not a tracking surface.

Its purpose is to:

* Capture subjective experience, context, and meaning
* Support emotional regulation, insight, and narrative continuity
* Complement habits and goals **without influencing progress**

**Core principle:**

> Journals add *context*, never *truth*.
> Deleting all journals must not change any habit completion, streak, momentum, or goal progress.

---

## Primary User Goals

On the Journal page, a user should be able to:

1. Reflect on their day or a specific experience
2. Use optional structure to reduce friction (templates)
3. Write freely when structure is not desired
4. Review past reflections by day or category
5. Edit or delete journal entries safely
6. Reference habits or goals **for context only**

---

## Core Objects in Scope

* **JournalEntry** — the reflective record
* **JournalTemplate** — optional structured scaffolding
* **Category** — optional organizational grouping
* **DayKey** — time anchoring (shared with habits)

The Journal page **never writes**:

* HabitEntry
* Goal progress
* Derived metrics

---

## Page Structure Overview

### 1. Header

**Displays**

* Page title: `Journal`
* Optional subtitle (contextual, non-evaluative):

  * “Reflect on your day”
  * “Capture thoughts, feelings, or insights”

**Actions**

* `+ New Entry` (primary action)

---

### 2. Entry Creation Flow

#### 2.1 Create New Entry

User may start a journal entry in one of three ways:

1. **Free-form entry**
2. **Template-based entry**
3. **Contextual entry** (launched from another page, e.g. Today View)

All three create a **JournalEntry** object.

---

#### 2.2 Free-Form Entry

**User Inputs**

* Optional title
* Free-form text (multiline)

**System Behavior**

* Assigns `timestampUtc` and `dayKey`
* No required fields
* No validation beyond basic text limits

---

#### 2.3 Template-Based Entry

**Template Selection**

* Templates may be grouped by Category
* System and user-created templates are allowed

**Template Fields**

* Text
* Number
* Scale (e.g. 1–5)
* Choice / Multi-choice
* Time (optional)

**Rules**

* Templates guide input only
* Templates do **not** enforce completion
* Required fields apply only *within* the template, not across days

---

#### 2.4 Contextual Entry (Optional)

A journal entry may be created with **pre-filled context**, such as:

* Referencing a habit (“Today’s run felt…”)
* Referencing a goal (“Progress toward consistency…”)

**Important**

* `referencedHabitIds` and `referencedGoalIds` are **context only**
* They do not affect completion, streaks, or goals

---

### 3. Journal Timeline / List View

#### 3.1 Default View

* Chronological list (most recent first)
* Grouped implicitly by `dayKey`
* Each entry shows:

  * Title (or first line of text)
  * Day / date
  * Template badge (if applicable)
  * Snippet preview

---

#### 3.2 Filters & Organization (Optional, Non-Critical)

* Filter by:

  * Category
  * Template
  * Date range
* Search by text content

These are **read-only affordances**.

---

### 4. Journal Entry Detail View

Selecting an entry opens a detail screen.

**Displays**

* Full content (structured + free text)
* Day and timestamp
* Template fields (if any)
* Referenced habits/goals (display only)

**Actions**

* Edit
* Delete

---

## Editing & Deletion Rules

### 5.1 Editing a JournalEntry

**Editable Fields**

* Title
* Text
* Template responses
* Template association
* Referenced habits/goals
* Timestamp / DayKey (with confirmation)

**Guardrails**

* Confirmation required if DayKey changes
* No behavioral impact preview needed (journals are non-authoritative)

---

### 5.2 Deleting a JournalEntry

**Deletion Model**

* Soft delete via `deletedAt`

**Effects**

* Entry disappears from journal views
* No change to:

  * Habit completion
  * Streaks
  * Momentum
  * Goals
  * Skills

---

## Relationship to Other Pages

### Journals vs Habits

| Journals    | Habits             |
| ----------- | ------------------ |
| Reflective  | Behavioral         |
| Optional    | Intentional        |
| No progress | Source of progress |
| Narrative   | Factual            |

A day with only journals is **behaviorally empty**, by design.

---

### Journals vs Goals

* Journals may **reference** goals
* Journals never advance or regress goals
* Journals provide meaning, not measurement

---

### Journals vs Routines

* Starting or completing a routine does not create a journal
* Journaling about a routine does not imply success or completion

---

## Time Semantics

* Every JournalEntry stores:

  * `timestampUtc`
  * `dayKey` (user-timezone normalized)
* Aggregation and grouping use **DayKey**
* Editing DayKey repositions the entry in the timeline

No journaling streaks or penalties exist by default.

---

## Explicit Non-Goals (Hard Rules)

The Journal page must **never**:

* Mark habits complete
* Create HabitEntries
* Increment streaks or momentum
* Affect goal progress
* Show overdue or missed states
* Imply obligation or failure

If journaling changes progress, it is a bug.

---

## Mental Model (For Engineers & Designers)

> **Habits answer “what happened.”**
> **Journals answer “what it meant.”**

Both are valuable.
Only one is truth.

---

## One-Sentence Summary

**The Journal page is a non-punitive reflection space that captures subjective context and meaning, fully isolated from behavioral truth, progress, and goals.**

---
