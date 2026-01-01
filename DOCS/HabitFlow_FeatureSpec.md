# HABITFLOW iOS FEATURE SPEC (v1)

**Status:** Authoritative for iOS Implementation  
**Audience:** iOS Engineers & Architects  
**Scope:** Defines how HabitFlow behaves on iOS without redefining domain semantics

---

## 0. Authority, Scope, and Guardrails

### 0.1 Canonical Authority

This feature spec is **subordinate** to the following canonical sources:

- `00_NORTHSTAR.md`
- Canonical Domain Rules (Domain Semantics)
- Object Specifications (`01_HABIT.md` → `15_HABIT_ENTRY_REFLECTION.md`)

This document **does not redefine domain meaning**.  
It defines **how the iOS application orchestrates canonical objects**.

---

### 0.2 Global Invariants (Non-Negotiable)

These invariants must hold true across all iOS features:

1. **HabitEntry is the sole source of behavioral truth**
   - All completion, progress, streaks, momentum, quotas, charts are derived from entries.
2. **Completion is derived, never stored**
   - No object may store `isComplete`, `streak`, `progress`, or equivalent as truth.
3. **DayKey is the aggregation boundary**
   - Aggregation never uses timestamps directly.
4. **Routines never complete habits**
   - Routines may suggest, never assert.
5. **Journals never imply completion**
   - Reflection is orthogonal to behavior tracking.
6. **Interpretive layers are deletable**
   - Persona, Skill, Identity, Derived Metrics must be removable without data loss.

Any shortcut that violates these rules is invalid by definition.

---

## 1. Canonical Object Contract (iOS View)

This section defines how iOS is allowed to interact with canonical objects.

### 1.1 Canonical Objects in Scope

| Object | iOS Role |
|---|---|
| Habit | Defines what can be logged |
| HabitEntry | Canonical behavioral truth |
| Routine | Structural grouping |
| RoutineExecution | Intent marker |
| HabitPotentialEvidence | Non-authoritative suggestion |
| Goal | Read-only aggregation |
| GoalLink | Contribution configuration |
| Category | Organizational structure |
| Persona | Focus lens |
| Identity | Narrative context |
| Skill | Interpretive capability |
| JournalTemplate | Reflection scaffolding |
| JournalEntry | Reflection record |
| HabitEntryReflection | Micro-context |

---

### 1.2 Object Responsibility Matrix

| Object | Writes Allowed | Reads Allowed | Never Does |
|---|---:|---:|---|
| Habit | ❌ | ✅ | Track progress |
| HabitEntry | ✅ | ✅ | Derive meaning |
| Routine | ❌ | ✅ | Create entries |
| RoutineExecution | ✅ | ✅ | Assert completion |
| HabitPotentialEvidence | ✅ | ✅ | Count as truth |
| Goal | ❌ | ✅ | Store progress |
| JournalEntry | ✅ | ✅ | Affect behavior |
| Derived Metrics | ❌ | ✅ | Act as truth |

**This table is binding.**

---

## 2. iOS Feature Surfaces (What the App Actually Does)

This section defines the user-visible feature surfaces that iOS implements.

### 2.1 Today View (Primary Surface)

**Purpose**
- Daily habit logging
- Evidence confirmation
- Light reflection
- Orientation (not evaluation)

**User Can**
- Log habits (binary or numeric)
- Edit or delete entries
- Start routines
- Confirm or dismiss evidence
- Add optional reflections

**User Cannot**
- Mark completion without creating a HabitEntry
- Complete habits via routines
- See punitive indicators

---

### 2.2 Habits View

**Purpose**
- Browse habits by category
- Distinguish daily vs weekly habits
- Edit habit definitions

**Rules**
- Completion indicators are derived
- Weekly habits never show empty daily cells
- Bundle parents are never logged directly

---

### 2.3 Routines View

**Purpose**
- Discover and run structured routines
- Improve UX, not tracking

**Rules**
- Starting a routine creates a RoutineExecution
- No completion state exists
- HabitPotentialEvidence may be generated

---

### 2.4 Goals View

**Purpose**
- Show long-horizon progress
- Provide orientation and motivation

**Rules**
- Progress is fully derived
- Logging from goal UI must create HabitEntries
- Goals never own history

---

### 2.5 Journaling View

**Purpose**
- Reflection and subjective context

**Rules**
- Journals never imply completion
- Journals may reference habits/goals
- Journals are optional and non-punitive

---

## 3. User Actions → Canonical Mutations

This is the most important section for engineers.

### 3.1 Action-to-Mutation Table

| User Action | Creates | Updates | Deletes |
|---|---|---|---|
| Log habit (manual) | HabitEntry | — | — |
| Edit habit entry | — | HabitEntry | — |
| Delete habit entry | — | — | HabitEntry (soft) |
| Start routine | RoutineExecution | — | — |
| Confirm evidence | HabitEntry | — | HabitPotentialEvidence |
| Dismiss evidence | — | — | HabitPotentialEvidence |
| Log from goal UI | HabitEntry | — | — |
| Add journal entry | JournalEntry | — | — |

If a user action does not result in a canonical mutation, it must be UI-only.

---

### 3.2 Explicitly Forbidden Flows

- RoutineExecution → HabitEntry (automatic)
- JournalEntry → HabitEntry
- Goal → HabitEntry without user confirmation
- UI checkmark → persistence without entry

---

## 4. Time & Aggregation Semantics (DayKey)

### 4.1 DayKey Rules

Every time-based record stores:
- `timestampUtc`
- `dayKey` (derived once, immutable)

Aggregation uses **only** DayKey:
- Daily completion
- Weekly windows
- Streaks
- Goals
- Momentum

---

### 4.2 iOS Responsibilities

- iOS must derive DayKey using the user’s timezone at creation time
- iOS must never recompute DayKey dynamically
- Editing timestamp/dayKey triggers full recomputation

---

### 4.3 Weekly Semantics

- Weeks are derived windows of DayKeys
- No `weekId` or stored week state exists
- Weekly habits aggregate entries within the window

---

## 5. Derived Metrics (Read Models Only)

### 5.1 Allowed Derived Metrics

- Daily completion
- Weekly progress
- Streaks (daily / weekly)
- Momentum (recency-weighted)
- Goal progress
- Bundle parent completion

All are computed views, never stored as truth.

---

### 5.2 Caching Rules

**Allowed**
- UI-only caches
- Computed summaries that can be invalidated

**Forbidden**
- Stored completion flags
- Stored streak counters
- Goal-owned totals
- Bundle completion stored in DB

If deleting all derived caches changes behavior, the implementation is invalid.

---

## 6. Explicit Anti-Patterns (Hard No’s)

The iOS app must NEVER:
- Store `isComplete` on Habit or Day
- Store streak counters
- Auto-complete habits from routines
- Treat UI state as authoritative
- Mutate historical meaning without recomputation
- Allow journaling to count as behavior
- Let goals track progress independently

If any of these appear in code, it is a design bug, not a feature choice.

---

## 7. Editing, Deletion, and Backfill Semantics

This section defines how historical data may be modified and what must happen when it is.

**Principle:**  
History may be edited or deleted, but meaning is never mutated silently.  
All derived state must recompute from canonical truth.

---

### 7.1 HabitEntry Editing

#### 7.1.1 Editable Fields

A HabitEntry may be edited in the following ways:
- `value` (numeric habits only)
- `timestampUtc`
- `dayKey` (derived from timestamp or explicitly changed)
- `note` / reflection metadata
- (rare) `source` — restricted to import/test repair flows

No other fields may be edited.

---

#### 7.1.2 Edit Guardrails

When editing a HabitEntry:
- The user must explicitly confirm the edit if:
  - The entry is not on the current DayKey
  - The edit changes the DayKey
- The UI should preview affected consequences when possible:
  - Streak changes
  - Weekly progress changes
  - Goal contribution changes

Editing is allowed, but surprise recomputation is not.

---

#### 7.1.3 Required Recomputations

Editing a HabitEntry MUST trigger recomputation of:
- Daily completion
- Weekly progress
- Streaks
- Momentum
- Goal progress
- Bundle parent completion
- Skill interpretation (if applicable)

No derived cache may survive an edit without invalidation.

---

### 7.10 JournalEntry Editing & Deletion

#### 7.10.1 Editable Fields (JournalEntry)

A JournalEntry may be edited in the following ways:
- `timestampUtc`
- `dayKey` (derived from timestamp or explicitly changed)
- `templateId` (optional; if converting between template-based and freeform is supported)
- `responses` (structured fields)
- `text` (free-form narrative)
- `title`
- `referencedHabitIds` / `referencedGoalIds` (context only)

References are context only. They must never change completion/progress.

---

#### 7.10.2 Edit Guardrails (JournalEntry)

When editing a JournalEntry:
- Confirmation required if:
  - DayKey changes
  - Entry is moved to a different day
- No “impact preview” is required for goals/streaks because:
  - Journals do not affect behavioral derivations

---

#### 7.10.3 Deletion Model (JournalEntry)

- JournalEntries are soft-deleted via `deletedAt`
- Deleting a JournalEntry:
  - Must not delete HabitEntries
  - Must not change any derived behavioral state
  - May affect journal-only views (journaling streaks if you ever add them—but careful: avoid punitive framing)

---

### 7.11 Wellbeing Metric Entries Editing & Deletion (If Adopted)

This assumes you introduce a canonical object like:
- **WellbeingMetricEntry** (or **WellbeingEntry**)  
  A time-bound record of subjective or measured wellbeing signals (sleep, energy, mood, hydration, stress, etc.)

#### 7.11.1 Key Rule

Wellbeing entries are truth for subjective state, but:
- They are orthogonal to behavioral progress and must not mutate the completion state.
- They can power coaching, readiness, correlations — but not “completion.”

#### 7.11.2 Editable Fields (WellbeingMetricEntry)

A WellbeingMetricEntry may be edited:
- `timestampUtc`
- `dayKey`
- metric values (e.g., `energy`, `mood`, `sleepQuality`, etc.)
- optional notes
- optional source metadata (manual/import)

#### 7.11.3 Deletion Model (WellbeingMetricEntry)

- Soft-delete via `deletedAt`
- Deleting a wellbeing entry:
  - Must not affect HabitEntry-derived progress
  - Must not affect Goal progress or streaks
  - Must only affect wellbeing charts / insights

#### 7.11.4 Recompute Rules (Wellbeing)

Edits/deletes must recompute:
- wellbeing trend views
- readiness composites (if you implement them)
- correlation analyses (HabitEntry ↔ Wellbeing)

But must never recompute or change:
- Habit completion
- Streaks
- Goal progress

---

### 7.2 HabitEntry Deletion

#### 7.2.1 Deletion Model

HabitEntries are soft-deleted using `deletedAt`. Deleted entries:
- Do not participate in aggregation
- Remain available for audit/debug
- May be restored in the future

#### 7.2.2 Deletion Guardrails

Before deleting an entry:
- The user must confirm deletion
- The UI should communicate that:
  - Progress may change
  - Streaks may break
  - Goals may regress

There is no “undo by counter” — restoration occurs only by un-deleting the entry.

#### 7.2.3 Required Recomputations

Deleting a HabitEntry MUST recompute:
- All derived metrics listed in §7.1.3

Deletion is semantically equivalent to “this fact never occurred.”

---

### 7.3 Backfilling HabitEntries (Past Logging)

#### 7.3.1 Definition

Backfilling means creating a HabitEntry for a DayKey prior to today.  
Backfilled entries are fully real — not second-class data.

#### 7.3.2 Creation Rules

When backfilling:
- `timestampUtc` must reflect the chosen time
- `dayKey` is derived from the user’s timezone at creation
- The entry is treated identically to a real-time entry

There is no “retroactive” flag in canonical state.

#### 7.3.3 UX Guardrails

For backfilled entries:
- The UI must clearly indicate the chosen day
- The user must confirm the action
- Optional preview of affected streaks/goals is recommended

---

### 7.4 One-Entry-Per-Day Constraints

#### 7.4.1 Default Rule

For daily habits, the canonical default is:
- `UNIQUE(habitId, dayKey)`

This ensures:
- Idempotent logging
- Predictable UI
- Clean recomputation

#### 7.4.2 Explicit Exceptions

Multiple entries per day are allowed only if:
- The habit explicitly supports it
- The value type is numeric or session-based
- The UI makes this explicit

If this is enabled in the future, it must be a vocabulary-level change, not a UI accident.

---

### 7.5 RoutineExecution Deletion

#### 7.5.1 Allowed Deletion

RoutineExecutions may be deleted:
- For cleanup
- For testing
- For user correction

#### 7.5.2 Cascade Rules

Deleting a RoutineExecution:
- MUST delete dependent HabitPotentialEvidence
- MUST NOT delete any HabitEntries

RoutineExecutions are never authoritative.

---

### 7.6 HabitPotentialEvidence Lifecycle

#### 7.6.1 Evidence Deletion

HabitPotentialEvidence may be deleted when:
- The user dismisses it
- The day changes
- The associated RoutineExecution is deleted
- The evidence is confirmed and converted into a HabitEntry

Deleting evidence has no behavioral consequences.

#### 7.6.2 Confirmation Semantics

Confirming evidence:
1. Creates or updates a HabitEntry
2. Deletes the evidence
3. Triggers full recomputation

Evidence never persists beyond confirmation.

---

### 7.7 Editing Goals, Habits, and Structure

#### 7.7.1 Editing Habits

Editing a Habit definition:
- Does NOT rewrite past entries
- Does NOT retroactively reinterpret units
- Affects future entries only

Example:
- Changing unit from “miles” to “minutes” does not rewrite history.

#### 7.7.2 Editing Goals

Editing a Goal:
- Never deletes HabitEntries
- Recomputes progress immediately
- May affect historical charts (expected behavior)

#### 7.7.3 Editing Categories, Personas, Identity

Edits to these objects:
- Affect only organization, emphasis, or framing
- Must not affect historical truth
- Must not require recomputation of HabitEntries

---

### 7.8 Non-Editable History (Explicit)

The system does NOT support:
- Silent mutation of historical meaning
- Auto-correction of entries
- Time-shifting entries due to timezone changes
- “Fixing” streaks without editing entries

If the user wants history to change, entries must change.

---

### 7.9 Invariant Validation Checklist (Engineering)

Before merging any change that touches history:
- Are edits explicit and user-confirmed?
- Are all derived caches invalidated?
- Can all views recompute from HabitEntries?
- Are time semantics preserved?
- Is any interpretive layer being treated as truth?

If any answer is “no,” the change must not ship.

---

### Section 7 Summary

History in HabitFlow is editable but never malleable.  
All meaning flows from HabitEntries, and any change to history must transparently recompute everything derived from them.

---

## 8. Offline, Sync, and Conflict Semantics

This section defines how the iOS app behaves when offline, how it syncs with canonical storage, and how conflicts are resolved without violating source-of-truth rules.

**Principle:**  
Offline capability must never introduce alternate truth stores.  
Offline data is provisional until reconciled, but truth semantics do not change.

---

### 8.1 Offline Capability Scope

The HabitFlow iOS app must support offline usage for the following actions:

**Allowed Offline**
- Creating HabitEntries
- Editing HabitEntries
- Deleting HabitEntries (soft delete)
- Creating JournalEntries
- Editing JournalEntries
- Deleting JournalEntries
- Creating WellbeingMetricEntries (if adopted)
- Editing / deleting WellbeingMetricEntries
- Starting RoutineExecutions
- Dismissing or confirming HabitPotentialEvidence
- Viewing cached derived metrics (read-only)

**Not Required Offline**
- Cross-device conflict resolution UI
- Global recomputation previews
- Multi-user collaboration (out of scope)

---

### 8.2 Local Persistence Model (iOS)

#### 8.2.1 Local Store Role

The iOS local store acts as:

> A write-ahead log and temporary cache — not a competing source of truth.

Rules:
- All canonical objects created offline must be stored locally
- Local objects must preserve canonical fields (ids, timestamps, dayKeys)
- No derived truth may be persisted locally as authoritative state

#### 8.2.2 Local IDs and Identity

- Objects created offline must receive stable client-generated IDs
- IDs must survive sync and reconciliation
- The server must not reassign canonical IDs post-sync

This ensures:
- Edit/delete operations remain referentially stable
- No “duplicate entry” collapse occurs silently

---

### 8.3 Sync Semantics (High-Level)

Sync is object-level, not “screen-level.”

For each canonical object type:
1. Upload pending local mutations
2. Receive authoritative state from server
3. Merge according to object-specific rules
4. Invalidate all derived caches
5. Recompute read models from canonical truth

---

### 8.4 Conflict Resolution Rules (By Object)

**Important:** Conflict resolution is semantic, not timestamp-only.

#### 8.4.1 HabitEntry Conflicts (Critical)

HabitEntry is the most sensitive object.

**Case A: Same Entry Edited on Two Devices**
- Resolve using last explicit edit (by `updatedAt`)
- Entire entry is replaced atomically
- Derived state recomputes

**Case B: Entry Deleted on One Device, Edited on Another**
- Deletion wins only if deletion is newer
- Otherwise, restore entry and apply edit
- Never keep “half-deleted” state

**Case C: Duplicate Entries (Same `habitId` + `dayKey`)**
- Apply canonical uniqueness rules
- Prefer the entry that:
  - Has explicit user intent (manual > routine > import)
  - Has a value if numeric
- Surface a repair state if ambiguity remains (future UX)

No automatic merging of values is allowed.

#### 8.4.2 JournalEntry Conflicts
Rules:
- Last edit wins
- No impact on behavioral derivations
- Duplicate journals may coexist if IDs differ

#### 8.4.3 WellbeingMetricEntry Conflicts (If Present)
Rules mirror JournalEntries:
- Last edit wins
- Entries remain independent
- No behavioral impact

#### 8.4.4 RoutineExecution Conflicts
Rules:
- Duplicates are allowed
- Deletions remove only the execution
- No merge semantics required

#### 8.4.5 HabitPotentialEvidence Conflicts
Rules:
- Evidence may be dropped safely
- If confirmed on one device and dismissed on another:
  - Confirmation wins (HabitEntry exists)
  - Evidence is deleted

---

### 8.5 Derived Metrics & Offline Mode

#### 8.5.1 Offline Derived Views
When offline:
- iOS may compute derived metrics from local canonical data
- These metrics are provisional
- UI must tolerate correction after sync

#### 8.5.2 Post-Sync Reconciliation
After any sync event:
- All derived caches must be invalidated
- All metrics must be recomputed from canonical objects
- UI must update immediately if values change

There is no concept of “locking in” offline-derived state.

---

### 8.6 Time Semantics Offline

#### 8.6.1 Timestamp & DayKey Creation
When offline:
- `timestampUtc` is derived from device clock
- `dayKey` is derived using device timezone
- Both are treated as authoritative unless explicitly edited later

Clock drift is tolerated; meaning derives from user intent, not precision.

#### 8.6.2 Timezone Changes
If device timezone changes:
- Previously stored DayKeys are not rewritten
- New entries use the new timezone
- History reflects lived experience, not retroactive correction

---

### 8.7 Sync Failure & Partial Sync

If sync fails mid-process:
- Local data remains intact
- No partial derived state is persisted
- Sync retries must be idempotent
- The user is never shown “half-applied” truth

---

### 8.8 Explicitly Forbidden Offline Patterns

The iOS app must NEVER:
- Invent completion state while offline
- Persist derived metrics as truth
- Auto-resolve ambiguous HabitEntry conflicts silently
- Rewrite DayKeys during sync
- Treat server state as “more real” than explicit user edits

If offline logic changes meaning without user intent, it is incorrect.

---

### 8.9 Offline & Sync Validation Checklist

Before shipping any offline/sync change:
- Are HabitEntries still the sole behavioral truth?
- Are all edits explicit and replayable?
- Can derived state be deleted and recomputed?
- Are conflicts resolved per object semantics?
- Is user intent preserved over timestamps?

If any answer is “no,” the implementation must not ship.

---

### Section 8 Summary

Offline support in HabitFlow preserves user intent without inventing truth.  
Sync reconciles canonical objects first, then recomputes meaning — never the reverse.

---

## 9. Edge Cases, Failure Modes, and Safety Guarantees

This section enumerates known edge cases and failure modes and defines the only acceptable behavior in each case.

**Principle:**  
When something goes wrong, HabitFlow must fail conservatively, preserving canonical truth and user intent over convenience or automation.

---

### 9.1 Duplicate Logging Scenarios

#### 9.1.1 Rapid Repeated Taps (Double Log)

**Scenario**
- User taps “Log” twice rapidly
- Network latency or animation delay causes duplicate actions

**Required Behavior**
- Enforce canonical uniqueness constraints:
  - For daily habits: `UNIQUE(habitId, dayKey)`
- Second action must:
  - Update the existing entry, or
  - Be ignored safely

**Forbidden**
- Creating duplicate HabitEntries
- Storing “completion flags” to suppress duplicates

#### 9.1.2 Multi-Device Same-Day Logging

**Scenario**
- User logs the same habit on two devices on the same DayKey

**Required Behavior**
- Resolve to a single canonical HabitEntry
- Prefer:
  1. Explicit manual entry
  2. Entry with numeric value (if applicable)
  3. Most recently edited entry

**Forbidden**
- Silent summing or merging of values
- Keeping both entries active for daily habits

---

### 9.2 Time Boundary Edge Cases

#### 9.2.1 Late-Night Logging (Crossing Midnight)

**Scenario**
- User logs a habit at 12:05 AM
- Intended for “yesterday”

**Required Behavior**
- Entry belongs to the derived DayKey by default
- User may explicitly change DayKey via edit flow

**Forbidden**
- Automatic “grace windows”
- Heuristic reassignment to yesterday

#### 9.2.2 Daylight Saving Time Transitions

**Scenario**
- DST shift causes day length irregularities

**Required Behavior**
- DayKey derivation remains stable
- Aggregation uses DayKey only
- No special DST logic is allowed

---

### 9.3 Routine & Evidence Edge Cases

#### 9.3.1 Starting a Routine and Quitting Immediately

**Scenario**
- User starts a routine, then abandons it

**Required Behavior**
- RoutineExecution exists
- Zero HabitEntries are created
- HabitPotentialEvidence may exist or not (implementation choice)

**Forbidden**
- Interpreting abandonment as failure
- Auto-cleaning RoutineExecution due to inactivity

#### 9.3.2 Evidence Confirmed Multiple Times

**Scenario**
- Evidence confirmation action is triggered twice

**Required Behavior**
- First confirmation creates HabitEntry
- Evidence is deleted
- Second action is a no-op

**Forbidden**
- Creating multiple HabitEntries
- Recreating evidence

---

### 9.4 Journal & Wellbeing Safety

#### 9.4.1 Journaling Without Habits

**Scenario**
- User journals on a day with zero habits logged

**Required Behavior**
- JournalEntry is created normally
- No habit completion is implied
- Day remains behaviorally empty

#### 9.4.2 Editing or Deleting Journals

**Scenario**
- User edits or deletes a JournalEntry

**Required Behavior**
- Journal views update
- No behavioral or goal recomputation occurs

**Forbidden**
- Triggering streak, momentum, or goal recalculation

#### 9.4.3 Wellbeing Entries Without Habits (If Present)

**Scenario**
- User logs mood, sleep, or energy with no HabitEntries

**Required Behavior**
- WellbeingMetricEntry stands alone
- Coaching insights may reference it
- No behavioral inference occurs

---

### 9.5 Goal & Aggregation Edge Cases

#### 9.5.1 Goal With No Linked Habits

**Scenario**
- Goal exists but no habits are linked

**Required Behavior**
- Goal progress shows zero or “not started”
- UI should prompt to link habits

**Forbidden**
- Allowing manual goal progress without creating HabitEntries

#### 9.5.2 Habit Deleted While Linked to Goal

**Scenario**
- Habit definition is deleted or archived

**Required Behavior**
- HabitEntries remain intact
- Goal recomputes using remaining entries
- Goal degrades gracefully

**Forbidden**
- Deleting HabitEntries
- Breaking goal integrity

---

### 9.6 Bundle & Choice Habit Edge Cases

#### 9.6.1 Multiple Options Logged in a Choice Bundle

**Scenario**
- User attempts to log two options on the same day

**Required Behavior**
- UI prevents second log
- Or second log replaces the first with confirmation

**Forbidden**
- Allowing two active entries
- Implicitly summing options

#### 9.6.2 Editing Option Habits

**Scenario**
- Option habit definition changes

**Required Behavior**
- Past entries retain original meaning
- Parent bundle recomputes accordingly

---

### 9.7 Offline & Sync Failure Modes

#### 9.7.1 Partial Sync Failure

**Scenario**
- Network drops mid-sync

**Required Behavior**
- No partial truth is committed
- Local state remains usable
- Sync retries are idempotent

#### 9.7.2 Clock Drift While Offline

**Scenario**
- Device clock is inaccurate

**Required Behavior**
- User-entered data is preserved
- User may edit timestamps later
- No automatic correction is applied

---

### 9.8 UI Safety Guarantees

#### 9.8.1 UI State Loss

**Scenario**
- App crashes or reloads mid-action

**Required Behavior**
- No canonical mutation without confirmation
- No partial HabitEntries
- No phantom completion

#### 9.8.2 Rendering Without Data

**Scenario**
- Derived metrics fail to compute

**Required Behavior**
- Fallback to safe empty state
- Never invent progress

---

### 9.9 Explicit “Never Do” Scenarios (Reinforced)

Under no circumstances may the system:
- Infer completion from journaling
- Infer completion from routine usage
- Store completion flags as truth
- Rewrite history automatically
- Hide or “smooth over” missing data
- Penalize missed days

If any of these appear, the implementation is incorrect by definition.

---

### 9.10 Edge Case Validation Checklist

Before shipping any feature touching edge cases:
- Does user intent always dominate heuristics?
- Can all meaning be recomputed from entries?
- Are all failures conservative?
- Is any truth inferred instead of asserted?
- Could deleting derived state change behavior?

If any answer is “yes,” the feature must not ship.

---

### Section 9 Summary

HabitFlow treats ambiguity as a signal to slow down, not guess.  
When edge cases arise, the system preserves truth, intent, and recomputability above all else.

---

## 10. Invariant-Based Test Scenarios (Authoritative)

This section defines non-negotiable test scenarios derived directly from canonical invariants.

**Principle:**  
Features are considered correct only if these invariants hold under all tested conditions.  
If a test here fails, the feature is wrong by definition.

These tests apply to:
- Unit tests
- Integration tests
- Manual QA
- PR review checklists

---

### 10.1 Core Truth Invariants

#### 10.1.1 HabitEntry Is the Sole Behavioral Truth

**Scenario**
- Create multiple HabitEntries
- Delete all derived state (completion flags, caches, summaries)

**Expected**
- All completion, progress, streaks, and charts recompute correctly
- UI reflects the same behavioral truth

**Fail If**
- Any behavioral meaning is lost
- Any behavior depends on non-entry state

#### 10.1.2 No Entry → No Completion

**Scenario**
- A habit exists
- No HabitEntry exists for a given DayKey

**Expected**
- Habit is incomplete
- No streak increment
- No goal progress

**Fail If**
- Any completion is inferred
- UI shows a completed state without an entry

---

### 10.2 Editing & Deletion Invariants

#### 10.2.1 Editing a HabitEntry Recomputes Everything

**Scenario**
- Create HabitEntry
- Edit value or DayKey

**Expected**
- Daily completion updates
- Weekly progress updates
- Streaks update
- Goals update
- Bundle parents update
- Skills reinterpret

**Fail If**
- Any derived metric remains stale
- Any cache survives without invalidation

#### 10.2.2 Deleting a HabitEntry Is Equivalent to “Never Happened”

**Scenario**
- Create HabitEntry
- Verify derived progress
- Delete the entry

**Expected**
- All derived progress disappears
- No ghost streaks or totals remain

**Fail If**
- Any progress remains
- Any “undo counter” logic exists

---

### 10.3 DayKey & Time Semantics

#### 10.3.1 Aggregation Uses DayKey, Not Timestamp

**Scenario**
- Create two entries on same DayKey with different timestamps

**Expected**
- Treated as same day for aggregation
- Daily completion counts once (unless explicitly multi-entry)

**Fail If**
- Timestamp order affects aggregation

#### 10.3.2 Timezone Changes Do Not Rewrite History

**Scenario**
- Create entries in Timezone A
- Change device timezone to Timezone B

**Expected**
- Existing DayKeys remain unchanged
- New entries use new timezone

**Fail If**
- Historical DayKeys are rewritten
- Past completion changes without edits

---

### 10.4 Routines & Evidence Invariants

#### 10.4.1 RoutineExecution Does Not Create Progress

**Scenario**
- Start a routine
- Do nothing else

**Expected**
- RoutineExecution exists
- No HabitEntry exists
- No completion or progress occurs

**Fail If**
- Any habit shows progress

#### 10.4.2 Evidence Requires Explicit Confirmation

**Scenario**
- HabitPotentialEvidence is generated
- User dismisses it

**Expected**
- Evidence disappears
- No HabitEntry exists

**Fail If**
- Completion is inferred
- Progress increments

---

### 10.5 Journals & Wellbeing Invariants

#### 10.5.1 Journaling Is Orthogonal to Behavior

**Scenario**
- Create JournalEntry
- No HabitEntries exist

**Expected**
- No habits complete
- No streaks
- No goals advance

**Fail If**
- Journaling affects behavioral metrics

#### 10.5.2 Editing Journals Has No Behavioral Impact

**Scenario**
- Create HabitEntry
- Create JournalEntry
- Edit or delete JournalEntry

**Expected**
- Habit completion unchanged
- Goals unchanged
- Streaks unchanged

**Fail If**
- Any behavioral recomputation occurs

#### 10.5.3 Wellbeing Entries Are Isolated (If Present)

**Scenario**
- Create WellbeingMetricEntry
- Edit or delete it

**Expected**
- Only wellbeing views change
- No habit/goal changes

**Fail If**
- Behavioral progress mutates

---

### 10.6 Goal Aggregation Invariants

#### 10.6.1 Goals Are Fully Recomputable

**Scenario**
- Create HabitEntries
- Create Goals linked to habits
- Delete all goal-side cached state

**Expected**
- Goal progress recomputes identically

**Fail If**
- Goal progress depends on stored counters

#### 10.6.2 Manual Goal Logging Creates HabitEntries

**Scenario**
- Log progress from Goal UI

**Expected**
- HabitEntry is created or updated
- Goal progress updates naturally

**Fail If**
- Goal progress updates without an entry

---

### 10.7 Bundles & Choice Habits

#### 10.7.1 Bundle Parents Never Store Entries

**Scenario**
- Complete a child habit in a bundle

**Expected**
- Child HabitEntry exists
- Parent completion is derived

**Fail If**
- Parent HabitEntry exists
- Parent stores completion

#### 10.7.2 Choice Bundles Enforce Single Option

**Scenario**
- Attempt to log two options on same DayKey

**Expected**
- Second action blocked or replaces first (with confirmation)

**Fail If**
- Two active entries exist

---

### 10.8 Offline & Sync Invariants

#### 10.8.1 Offline Edits Preserve Truth

**Scenario**
- Create/edit/delete entries offline
- Sync later

**Expected**
- Canonical state matches explicit user actions
- Derived state recomputes

**Fail If**
- Offline heuristics override user intent

#### 10.8.2 Sync Does Not Invent Truth

**Scenario**
- Sync with partial data
- Force recomputation

**Expected**
- No new entries appear
- No completion inferred

**Fail If**
- Sync introduces progress

---

### 10.9 UI Safety Invariants

#### 10.9.1 UI State Is Never Truth

**Scenario**
- App crashes mid-interaction

**Expected**
- No partial HabitEntry
- No phantom completion

**Fail If**
- UI state persists as truth

#### 10.9.2 Derived Metric Failure Is Safe

**Scenario**
- Derived computation fails

**Expected**
- Empty or fallback UI
- No invented values

**Fail If**
- Placeholder data is treated as real

---

### 10.10 Release Gate Checklist

Before any feature ships:
- Can all behavior be reconstructed from HabitEntries?
- Are journals and wellbeing isolated?
- Are routines non-authoritative?
- Are DayKeys stable?
- Are derived metrics deletable?
- Does user intent always dominate heuristics?

If any answer is “no,” the feature must not ship.

---

### Section 10 Summary

In HabitFlow, correctness is defined by invariants, not appearances.  
If the invariants hold, the feature is correct — if they don’t, it isn’t.

---

## 11. Canonical Reference Index

This section defines the authoritative source documents that govern HabitFlow’s domain semantics.  
This iOS Feature Spec orchestrates these sources but does not redefine them.

**Rule:**  
If an engineer needs to understand what something means, they must consult the canonical document below — not infer from UI or code.

---

### 11.1 Northstar & Domain Authority

| Document | Purpose |
|---|---|
| `00_NORTHSTAR.md` | Defines HabitEntry as the sole source of behavioral truth |
| Canonical Domain Rules | Global invariants and non-negotiable system rules |

These documents supersede all PRDs, mocks, and implementation details.

---

### 11.2 Core Canonical Objects

| ID | Document | Canonical Responsibility |
|---:|---|---|
| 01 | `01_HABIT.md` | Defines valid repeatable actions |
| 02 | `02_HABIT_ENTRY.md` | Behavioral truth store |
| 03 | `03_ROUTINE.md` | Reusable action structure |
| 04 | `04_ROUTINE_EXECUTION.md` | Intent marker |
| 05 | `05_HABIT_POTENTIAL_EVIDENCE.md` | Non-authoritative suggestion |
| 06 | `06_GOAL.md` | Read-only aggregation |
| 07 | `07_GOAL_LINK.md` | Explicit contribution semantics |
| 08 | `08_CATEGORY.md` | Organizational structure |
| 09 | `09_PERSONA.md` | Focus and visibility lens |
| 10 | `10_SKILL.md` | Interpretive capability layer |
| 11 | `11_TIME_DAYKEY.md` | Time & aggregation semantics |
| 12 | `12_DERIVED_METRICS.md` | Read-only computed views |

---

### 11.3 Journaling & Reflection Objects

| ID | Document | Purpose |
|---:|---|---|
| 13 | `13_JOURNAL_TEMPLATE.md` | Structured reflection scaffolding |
| 14 | `14_JOURNAL_ENTRY.md` | Reflective truth store |
| 15 | `15_HABIT_ENTRY_REFLECTION.md` | Micro-reflection attached to entries |

Journals and reflections are orthogonal to behavioral truth and must never imply completion.

---

### 11.4 Optional / Emerging Canonical Objects

These objects are supported by this feature spec but may be under active iteration.

| Object | Status | Notes |
|---|---|---|
| WellbeingMetricEntry | Optional | Subjective/physiological truth store |
| WellbeingDerivedMetrics | Optional | Interpretive insights only |

If adopted, these objects must obey all invariants defined in Sections 0, 7, 8, and 10.

---

### 11.5 PRDs, Mockups, and UX Documents

PRDs and mockups:
- Explain intent
- Propose experience
- Illustrate flow

They do not define truth semantics.

If a PRD conflicts with:
- Canonical Domain Rules
- Object specifications
- This iOS Feature Spec

Then the PRD is wrong.

---

### 11.6 How Engineers Should Use This Index

When implementing:
- Start with this iOS Feature Spec
- Use this index to resolve semantic questions
- Do not infer meaning from UI behavior alone

When reviewing PRs:
- Ask “Which canonical object owns this?”
- Validate invariants in Section 10
- Reject implementations that invent truth

---

## Final Authority Statement

HabitFlow is defined by its canonical objects and invariants.  
This Feature Spec compiles those rules into an implementable iOS contract.  
If something feels ambiguous, the system is telling you to slow down — not guess.

