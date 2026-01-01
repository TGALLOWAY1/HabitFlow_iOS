# **Routines Page — Functional Specification**

**Purpose:**
The Routines page provides **structure and friction-reduction** for behavior without ever becoming a tracking surface.
Routines help users *start* actions more easily, but **never determine completion or progress**.

---

## **1. Primary Goals of the Routines Page**

The Routines page exists to:

1. Help users **organize habits into meaningful sequences**
2. Reduce **task initiation friction**
3. Provide **repeatable structure** (morning, evening, workouts, focus blocks)
4. Encourage consistency **without enforcing it**
5. Surface routines as *support tools*, not obligations

**Critical constraint:**

> No action on the Routines page may directly create progress without an explicit HabitEntry.

---

## **2. Core Page Sections**

### **2.1 Routine List (Primary Surface)**

Displays all routines grouped by **Category**.

Each routine row includes:

* Routine name
* Optional short description
* Category indicator
* List preview (icons or compact labels of linked habits)
* Primary action: **Start Routine**

**Rules:**

* No completion indicators
* No streaks
* No progress bars
* No “done / missed” language

Routines are always neutral and optional.

---

### **2.2 Category Grouping**

Routines are grouped by **Category**, consistent with:

* Habits page
* Goals page

This enables:

* Predictable navigation
* Mental mapping between habits and routines
* Persona-driven emphasis (future enhancement)

Categories are **organizational only** and carry no behavioral meaning.

---

### **2.3 Empty State**

If no routines exist:

* Explain what routines are (structure, not tracking)
* Provide a clear **Create Routine** CTA
* Example copy:

  > “Routines help you group habits into repeatable flows. They don’t track progress — habits do.”

---

## **3. Routine Row Interactions**

### **3.1 Start Routine (Primary Action)**

When the user taps **Start Routine**:

1. A **RoutineExecution** is created
2. The user is navigated into the **Routine Run View**

**Important:**

* Starting a routine does **not**:

  * Complete habits
  * Create HabitEntries
  * Affect streaks, momentum, or goals

It only records **intent to use structure** .

---

### **3.2 Edit Routine**

From the routine row or overflow menu:

* Edit routine name
* Edit description
* Reorder steps
* Add/remove linked habits
* Change category

**Rules:**

* Editing a routine does not affect past executions
* Editing a routine does not affect any HabitEntries

---

### **3.3 Delete / Archive Routine**

* Routine may be archived or deleted
* Past **RoutineExecutions** may be optionally deleted
* **HabitEntries are never deleted or modified**

---

## **4. Routine Run View (Execution Flow)**

### **4.1 Purpose**

The Routine Run View is a **guided checklist-like flow** designed to:

* Reduce cognitive load
* Encourage action
* Surface opportunities to log habits

It is **not a tracker**.

---

### **4.2 Routine Steps**

Each routine consists of **steps**, typically linked to habits.

For each step, the UI may show:

* Habit name
* Habit type (binary / numeric)
* Optional helper text
* Action affordance:

  * “Log”
  * “Add value”
  * “Skip”

**Rules:**

* Skipping has no penalty
* No step is required
* Order is optional and editable

---

### **4.3 Logging From a Routine**

When a user logs a habit from a routine step:

1. A **HabitEntry** is created or updated
2. The entry is marked with:

   * `source = "routine"`
   * `routineId`
   * `routineExecutionId`

This is the **only way** a routine affects progress — indirectly, through explicit habit logging .

---

### **4.4 HabitPotentialEvidence (Optional UX Layer)**

If you support passive confirmation instead of immediate logging:

* A step interaction may create **HabitPotentialEvidence**
* Evidence can later surface on:

  * Today View
  * Habit detail view

Evidence must always require explicit confirmation before becoming a HabitEntry.

---

## **5. RoutineExecution Semantics (Invisible but Critical)**

Every time a routine is started:

* Exactly one **RoutineExecution** is created
* It records:

  * `routineId`
  * `startedAtUtc`
  * `dayKey`

**RoutineExecution:**

* Is append-only
* Has no completion state
* May optionally record `endedAtUtc`
* Exists even if the user quits immediately

This enforces a clean separation between **intent** and **outcome** .

---

## **6. What the Routines Page Must NEVER Do**

The Routines page must not:

* ❌ Show completion state
* ❌ Show streaks or momentum
* ❌ Auto-complete habits
* ❌ Store progress
* ❌ Punish skipping
* ❌ Imply failure
* ❌ Compete with the Habits page

If a routine is deleted, **nothing about behavioral history changes**.

---

## **7. Relationship to Other Pages**

### **7.1 Habits Page**

* Habits are logged and evaluated there
* Routines only *assist* logging

### **7.2 Today View**

* Routine starts may surface as context
* HabitPotentialEvidence may appear there
* Completion still derives only from HabitEntries

### **7.3 Goals Page**

* Goals aggregate HabitEntries
* Routines have no direct relationship to goals

---

## **8. Analytics & Insights (Read-Only)**

Allowed analytics (future):

* “How often do I start routines?”
* “Which routines do I tend to use?”

Disallowed analytics:

* “Did I complete the routine?”
* “Routine success rate”
* “Routine streaks”

Routines are **tools**, not performance metrics.

---

## **9. One-Sentence Summary**

**The Routines page provides optional structure that makes habits easier to perform without ever owning tracking, progress, or evaluation.**

