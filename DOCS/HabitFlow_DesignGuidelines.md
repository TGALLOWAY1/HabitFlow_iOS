Absolutely — here’s a **Cursor-ready Color & Style Guidelines document** for **HabitFlow iOS**, written so you can paste it directly into your repo (e.g. `STYLE_GUIDE.md` or `.cursorrules`).
It is opinionated, implementation-aware, and aligned with your **non-punitive, calm, high-clarity philosophy** defined in the canonical docs .

---

# HabitFlow — Color & Style Guidelines (Cursor Authoritative)

> **Purpose**
> This document defines the visual language, color system, and interaction style for HabitFlow.
> Cursor must treat this as **authoritative UI guidance** unless explicitly overridden.

---

## 0. Cursor Rules (MANDATORY)

### 0.1 Global Cursor Rules

**Cursor MUST:**

* Prefer **clarity over decoration**
* Prefer **subtle contrast** over harsh color separation
* Avoid visual punishment (no red failure states, no alarming indicators)
* Use **color as information**, not as judgment
* Keep UI calm, grounded, and low-cognitive-load

**Cursor MUST NOT:**

* Introduce bright red “error” or “missed” states for habits
* Use aggressive success colors (neon green, saturated yellow)
* Add celebratory animations without explicit instruction
* Encode completion as a stored state (visuals must reflect derived truth only)
* Introduce skeuomorphic or playful UI elements

---

### 0.2 Semantic Alignment Rule (Critical)

UI must **never imply semantic meaning that does not exist canonically**.

Examples:

* A checked box **means a HabitEntry exists**
* A dimmed row **means no entry exists**
* A glow or emphasis **means focus or recency**, not success or failure

If a visual state cannot be recomputed from canonical data, **it must not exist**.

---

## 1. Design Philosophy

### 1.1 Emotional Tone

HabitFlow UI should feel:

* Calm
* Grounded
* Supportive
* Non-judgmental
* Adult (never gamified by default)

The UI should communicate:

> “You are allowed to be human here.”

---

### 1.2 Visual Hierarchy Principles

1. **Structure first** (layout, spacing)
2. **Typography second**
3. **Color last**

Color should *support* hierarchy, never replace it.

---

## 2. Color System (Dark-First)

HabitFlow is **dark-mode first**, with light mode as a derivative.

### 2.1 Base Palette (Dark)

| Token            | Purpose           | Description               |
| ---------------- | ----------------- | ------------------------- |
| `bg.primary`     | App background    | Near-black, slightly warm |
| `bg.secondary`   | Cards / surfaces  | Soft charcoal             |
| `bg.tertiary`    | Elevated surfaces | Slightly lighter charcoal |
| `divider.subtle` | Separators        | Low-contrast gray         |

**Guidance**

* No pure black (`#000000`)
* Avoid high contrast white on black
* Prefer charcoal + off-white

---

### 2.2 Text Colors

| Token            | Purpose                |
| ---------------- | ---------------------- |
| `text.primary`   | Primary content        |
| `text.secondary` | Labels, metadata       |
| `text.tertiary`  | De-emphasized / helper |
| `text.disabled`  | Inactive states        |

**Rules**

* Never rely on opacity alone to convey meaning
* Disabled ≠ failed

---

### 2.3 Accent Colors (Very Limited)

HabitFlow uses **one primary accent** at a time.

| Token              | Use                         |
| ------------------ | --------------------------- |
| `accent.primary`   | Active focus, selection     |
| `accent.secondary` | Optional secondary emphasis |

**Default Accent Character**

* Muted
* Cool or neutral
* Never neon
* Never urgent

Examples that fit:

* Muted teal
* Soft cyan
* Desaturated indigo

Examples that do **not** fit:

* Bright red
* Neon green
* Hot pink
* High-sat yellow

---

## 3. Semantic Color Usage (Strict)

### 3.1 Habit States

| State          | Visual Treatment                      |
| -------------- | ------------------------------------- |
| No entry       | Neutral text, empty checkbox          |
| Entry exists   | Subtle fill + soft accent             |
| Focused        | Glow or outline (very light)          |
| Non-negotiable | Weight / icon emphasis, **not color** |

**Important**

* No “missed” color
* No red states
* No decay visuals

---

### 3.2 Goals & Progress

* Progress is **informational**
* Use muted fills and thin progress indicators
* Avoid completion fireworks or large green fills

Completion should feel like:

> “This is true now.”
> Not:
> “You won.”

---

### 3.3 Journals & Reflection

* Journals use **neutral tones**
* Never accent colors that resemble completion
* Journals must visually feel *orthogonal* to habits

---

## 4. Typography

### 4.1 Style

* System fonts only (SF Pro on iOS)
* No decorative fonts
* No handwriting styles

### 4.2 Hierarchy

| Level    | Usage           |
| -------- | --------------- |
| Title    | Page titles     |
| Headline | Section headers |
| Body     | Primary content |
| Caption  | Metadata, hints |

**Rules**

* Size changes > weight changes
* Avoid all-caps except for very small labels

---

## 5. Components Style Rules

### 5.1 Habit Rows

* Rectangular rows
* Rounded corners (medium radius)
* Checkbox is a **square**, never circular
* Checkbox appears on the **right**
* Tappable area is generous
* Subtle bottom divider only (no cards by default)

---

### 5.2 Buttons

| Type        | Style                                                 |
| ----------- | ----------------------------------------------------- |
| Primary     | Filled, muted accent                                  |
| Secondary   | Outline or ghost                                      |
| Destructive | Neutral confirmation first, red only on final confirm |

**No impulsive red buttons. Ever.**

---

### 5.3 Modals

* Dark background
* Clear action hierarchy
* Primary action visually obvious
* Cancel always available

---

## 6. Motion & Animation

### 6.1 Allowed

* Subtle fades
* Gentle scale (≤ 1.02)
* Ease-in-out only

### 6.2 Forbidden

* Bounce
* Elastic motion
* Confetti
* Celebration explosions

Motion should feel:

> “Responsive, not excited.”

---

## 7. Accessibility

* Minimum contrast AA for text
* Touch targets ≥ 44pt
* Do not encode meaning with color alone
* Support reduced motion

---

## 8. Anti-Patterns (Hard No)

Cursor must never introduce:

* Red “missed” days
* Angry warnings
* Shame-based copy
* Aggressive streak visuals
* Gamified XP bars by default
* Emoji-heavy UI

---
