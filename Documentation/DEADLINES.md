# Deadlines Page — Technical Documentation

**File:** `deadlines.html`  
**Project:** Luna Onboarding Assistant — Baruch College  
**Purpose:** Displays homework due dates, project milestones, and event registration windows for all first-year graduate courses, organized by subject and grouped into three category tabs.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Page Structure](#2-page-structure)
3. [Tab System](#3-tab-system)
4. [Subject Data (`subjects` array)](#4-subject-data-subjects-array)
5. [Deadline Data (`deadlines` object)](#5-deadline-data-deadlines-object)
6. [Accordion Rows](#6-accordion-rows)
7. [Dynamic List Builder (`buildList`)](#7-dynamic-list-builder-buildlist)
8. [Styling & Theme](#8-styling--theme)
9. [Navigation & Linking](#9-navigation--linking)
10. [How to Update Deadlines](#10-how-to-update-deadlines)
11. [How to Add a New Subject](#11-how-to-add-a-new-subject)

---

## 1. Overview

The Deadlines page is a **single HTML file** (`deadlines.html`) with no external dependencies. All data, logic, and styling are self-contained. It is reachable from the homepage (`index.html`) via the **Deadlines** button.

The page organizes deadline information across **40 graduate subjects** drawn from 10 schools and departments at Baruch College (CUNY). Each subject's deadline information is split into three categories:

| Category | Tab Label | Data Key |
|---|---|---|
| Homework due dates | Homework | `hw` |
| Project milestones | Projects | `proj` |
| Event registration windows | Event Registration | `evt` |

---

## 2. Page Structure

```
deadlines.html
├── <nav>                        Navigation bar (logo + back link)
├── <div class="page-content">
│   ├── .breadcrumb              Home / Deadlines trail
│   ├── .page-title              "📅 Deadlines"
│   ├── .page-sub                Subtitle description
│   ├── .tab-bar                 Three tab buttons (Homework / Projects / Event Registration)
│   ├── #panel-homework          Tab panel — rendered by buildList()
│   ├── #panel-projects          Tab panel — rendered by buildList()
│   └── #panel-events            Tab panel — rendered by buildList()
├── <div class="ai-bar-wrapper"> Luna chat bar at the bottom
└── <script>                     All data + logic (subjects, deadlines, buildList, tab switching)
```

---

## 3. Tab System

Each tab button carries a `data-tab` attribute that maps to a panel ID:

```html
<button class="tab-btn active" data-tab="homework"> → #panel-homework
<button class="tab-btn"        data-tab="projects"> → #panel-projects
<button class="tab-btn"        data-tab="events">   → #panel-events
```

**Switching logic (JavaScript):**

```js
document.querySelectorAll(".tab-btn").forEach(btn => {
  btn.addEventListener("click", () => {
    // Remove active from all buttons and panels
    document.querySelectorAll(".tab-btn").forEach(b => b.classList.remove("active"));
    document.querySelectorAll(".tab-panel").forEach(p => p.classList.remove("active"));
    // Activate clicked button and its matching panel
    btn.classList.add("active");
    document.getElementById("panel-" + btn.dataset.tab).classList.add("active");
  });
});
```

**CSS visibility rule:**
```css
.tab-panel         { display: none; }
.tab-panel.active  { display: block; }
```

Only one panel is visible at a time. The `Homework` tab is active by default on page load.

---

## 4. Subject Data (`subjects` array)

The `subjects` array is defined inside the `<script>` block. It mirrors the subject list from `subjects.html` and contains **40 entries** across 10 departments. Each entry has three fields:

```js
{ code: "CIS 9010", name: "Database Management Systems", dept: "Dept. of Computer Information Systems" }
```

| Field | Type | Description |
|---|---|---|
| `code` | `string` | Course code (e.g. `"CIS 9010"`). Used as the key to look up deadline data. |
| `name` | `string` | Full subject name displayed on the row. |
| `dept` | `string` | Department label shown in muted text on the right of the row header. |

**Departments covered:**

| Department | Sample Codes |
|---|---|
| Marxe School of Public & International Affairs | PAF 9100–9140 |
| Baruch School of Business (Zicklin) | MBA 7000, ACC 7000, FIN 7100, MKT 7100 |
| Dept. of Computer Information Systems | CIS 9001–9490 |
| Dept. of Economics and Finance | ECO 9710–9800, FIN 9770–9780 |
| Weissman School — Humanities | ENG 7001, COM 9100–9110 |
| Dept. of Management | MGT 9100–9420 |
| Dept. of Operations Management | OPM 9100–9300 |
| Dept. of Marketing & International Business | MKT 9100–9300, IBS 9100 |
| Weissman School — Social Sciences | SOC 9100, PSY 9100, POL 9100 |
| Dept. of Technology and Innovation | TEC 9100–9300 |

---

## 5. Deadline Data (`deadlines` object)

The `deadlines` object maps each course code to an object with three keys — `hw`, `proj`, and `evt` — each holding a plain-English deadline message string.

```js
const deadlines = {
  "CIS 9010": {
    hw:   "SQL assignments due every Wednesday.",
    proj: "Database design and implementation project due Apr 7.",
    evt:  "Oracle workshop registration closes Feb 4. Data management symposium RSVP due Mar 24."
  },
  // ... all 40 subjects
};
```

**Fallback:** If a course code is not found in `deadlines`, the message defaults to:
> `"Deadline information will be posted by your instructor on Blackboard."`

---

## 6. Accordion Rows

Each subject is rendered as a collapsible **accordion row**. By default all rows are **closed**. Clicking a row's header toggles it open or closed.

**HTML structure of one row:**
```html
<div class="subject-row" id="list-homework-CIS-9010">
  <div class="subject-header" onclick="toggleRow(this)">
    <span class="subject-code-badge">CIS 9010</span>
    <span class="subject-name-label">Database Management Systems</span>
    <span class="subject-dept">Dept. of Computer Information Systems</span>
    <svg class="chevron">...</svg>          <!-- rotates 180° when open -->
  </div>
  <div class="subject-body">               <!-- hidden until open -->
    <div class="deadline-info">
      <div class="dl-icon hw">...</div>    <!-- color-coded icon -->
      <div class="dl-text">
        <div class="dl-label">Homework Deadlines</div>
        <div class="dl-message">📌 SQL assignments due every Wednesday.</div>
      </div>
    </div>
  </div>
</div>
```

**Toggle function:**
```js
function toggleRow(header) {
  const row = header.closest(".subject-row");
  row.classList.toggle("open");
}
```

When `.open` is added to `.subject-row`:
- `.subject-body` becomes visible (`display: block`)
- `.chevron` rotates 180° via CSS transition

**Icon color coding per category:**

| Category | CSS class | Background | Icon color |
|---|---|---|---|
| Homework | `.dl-icon.hw` | `#ede9fe` (lavender) | `#7c3aed` (purple) |
| Projects | `.dl-icon.proj` | `#fef3c7` (yellow) | `#d97706` (amber) |
| Event Registration | `.dl-icon.evt` | `#dcfce7` (green) | `#16a34a` (green) |

---

## 7. Dynamic List Builder (`buildList`)

All three panels are populated by a single reusable function called at page load:

```js
buildList("list-homework", "hw");
buildList("list-projects", "proj");
buildList("list-events",   "evt");
```

The function:
1. Iterates over every entry in the `subjects` array
2. Looks up the subject code in the `deadlines` object
3. Selects the correct message string (`hw`, `proj`, or `evt`)
4. Builds an accordion row HTML string
5. Injects the full HTML into the target container via `innerHTML`

This means the HTML for all 40 rows in each panel is generated **at runtime** — the HTML file itself contains no static subject rows.

---

## 8. Styling & Theme

The page follows the same design system as the rest of the Luna Onboarding Assistant:

| Token | Value | Usage |
|---|---|---|
| Primary purple | `#7c3aed` | Active tabs, badges, borders on hover, icons |
| Light purple bg | `#ede9fe` / `#faf5ff` | Icon backgrounds, active tab bg |
| Page background | `#f3f4f6` | Body background |
| Card background | `#ffffff` | Accordion rows |
| Border default | `#e5e7eb` | Row borders |
| Border hover | `#c4b5fd` | Row hover state |
| Text primary | `#0f172a` | Headings, subject names |
| Text muted | `#57606a` | Descriptions, department labels |
| Font stack | `-apple-system, "Segoe UI", system-ui, sans-serif` | All text |

---

## 9. Navigation & Linking

- **Entry point:** The **Deadlines** button on `index.html` links to `deadlines.html`
  ```html
  <a href="deadlines.html"><button class="btn-secondary">Deadlines</button></a>
  ```
- **Back navigation:** The nav bar contains a `← Back to Home` link pointing to `index.html`
- **Breadcrumb:** `Home / Deadlines` with `Home` linking back to `index.html`

---

## 10. How to Update Deadlines

To change a deadline message, find the subject code key in the `deadlines` object inside the `<script>` tag and edit the relevant field:

```js
// Before
"FIN 7100": { hw: "Financial problem sets due every Thursday.", ... }

// After — updated date
"FIN 7100": { hw: "Financial problem sets due every Thursday by 11:59 PM. First set due Feb 6.", ... }
```

No other changes are needed. The page re-renders the accordion content dynamically on load.

---

## 11. How to Add a New Subject

**Step 1 — Add to the `subjects` array:**
```js
{ code: "NEW 9999", name: "My New Course", dept: "Dept. of Example" },
```

**Step 2 — Add to the `deadlines` object:**
```js
"NEW 9999": {
  hw:   "Assignments due every Monday.",
  proj: "Final project due Apr 30.",
  evt:  "Workshop registration closes Feb 1."
},
```

The subject will automatically appear in all three tabs (Homework, Projects, Event Registration) on the next page load. No HTML changes are required.

---

*Documentation last updated: November 2026*
