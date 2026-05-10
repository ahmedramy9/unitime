# Impact Analysis Report — SearchResultCounter

**Change Request (CR):** Display the count of class search results ("Found N classes") above the results table on the Class Search page.  
**Branch:** `SearchResultCounter` → `main`  
**Analysis date:** 2026-05-09  
**Analyst:** Ahmed  
**Method:** Based on IEEE/EIA 1219 Change Mini-Cycle Model and formal impact analysis process (Tripathy & Naik, Ch. 6)

---

## 1. Change Request Description

The CR requests that after a user performs a class search, the total number of matching classes is shown on-screen before the results table. This is a purely additive enhancement to the existing Class Search feature — no existing behaviour is removed or altered.

---

## 2. Identifying the Starting Impact Set (SIS)

The SIS is the initial set of software lifecycle objects (SLOs) presumed to be impacted, identified by analysing the CR specification, documentation, and source code.

The CR concept is "show a count on the search results page." Mapping this concept onto source code components (the concept assignment problem):

| SLO | Reason for inclusion in SIS |
|-----|----------------------------|
| `ClassSearchAction.java` | The Struts 2 action controller that handles the search — clearly the entry point for any search-result change |
| `classSearch.jsp` | The JSP view that renders the results page — must display the new count |

**SIS = { ClassSearchAction, classSearch.jsp }** → |SIS| = 2

---

## 3. Traceability Analysis

### Internal traceability (within source code)
Tracing the dependency chain inside the codebase from the SIS objects:

- `ClassSearchAction.execute()` → calls `getClasses(form, proxy)` → returns `Collection classes`
- `ClassSearchAction.performAction()` → same call path
- `classSearch.jsp` → reads request attributes → calls `CourseMessages` for i18n messages
- `ClassListForm` → holds the `classes` collection; `size()` is derived from it
- `CourseMessages` → interface used by both the action and the JSP for all user-facing strings

### External traceability (across work products)
Tracing from CR → Design → Code → Test:

| Requirements (CR) | Design | Code | Test |
|---|---|---|---|
| Show count of results | Add `resultCount` to request scope; add `foundClasses()` i18n key | `ClassSearchAction` sets attribute; `CourseMessages` defines message; `classSearch.jsp` renders it | Search with results → count shown; search with no results → no count |

---

## 4. Identifying the Candidate Impact Set (CIS)

Starting from the SIS, the SIS is augmented with SLOs likely to change because of changes in SIS elements. Both direct and indirect impacts are considered.

**Direct impact** — objects with a fan-in / fan-out relation to SIS members:
- `CourseMessages.java` — `ClassSearchAction` and `classSearch.jsp` both fan-out to this interface for all user-visible strings; a new string key is needed
- `ClassListForm.java` — `ClassSearchAction` reads `form.getClasses()` to get the collection; may need modification if count is stored on the form

**Indirect impact** — objects reachable via the direct impact chain:
- None identified beyond the direct layer; the change is shallow and confined to the presentation layer

**CIS = { ClassSearchAction, classSearch.jsp, CourseMessages, ClassListForm }** → |CIS| = 4

### SLO dependency graph (CIS scope)

```
ClassSearchAction ──uses──► CourseMessages
ClassSearchAction ──uses──► ClassListForm
ClassSearchAction ──renders►classSearch.jsp
classSearch.jsp   ──uses──► CourseMessages
```

---

## 5. Implementing the Change — Actual Impact Set (AIS)

After implementing the CR, the set of objects actually changed:

| SLO | Change made |
|-----|-------------|
| `ClassSearchAction.java` | Added `request.setAttribute("resultCount", classes.size())` in `execute()` (line 200) and `performAction()` (line 306) |
| `classSearch.jsp` | Added conditional `<s:if>` block rendering `<div>Found N classes</div>` (lines 341–345) |
| `CourseMessages.java` | Added `@DefaultMessage("Found {0} classes") String foundClasses()` (line 1948) |

**AIS = { ClassSearchAction, classSearch.jsp, CourseMessages }** → |AIS| = 3

`ClassListForm` was in CIS but required **no changes** — `classes.size()` is called directly on the collection returned by `getClasses()`, not via the form.

---

## 6. Discovered Impact Set (DIS) and False Positive Impact Set (FPIS)

**DIS** — new objects not in CIS discovered to be impacted during implementation:

No new objects were discovered. The implementation required no changes beyond CIS.

**DIS = { }** → |DIS| = 0

**FPIS** — objects estimated in CIS but not actually changed:

**FPIS = (CIS ∪ DIS) − AIS = { ClassListForm }** → |FPIS| = 1

`ClassListForm` was a false positive — included in CIS as a precaution because it holds the classes collection, but the count was read directly without modifying the form class.

---

## 7. Impact Analysis Process Metrics

### 7.1 Recall
Measures the fraction of actual impacts (AIS) that were predicted (in CIS). Recall = 1 when DIS is empty.

```
Recall = |CIS ∩ AIS| / |AIS|
       = |{ClassSearchAction, classSearch.jsp, CourseMessages}| / 3
       = 3 / 3
       = 1.0
```

**Recall = 1.0** — all actually changed objects were correctly predicted. DIS is empty, confirming perfect recall.

### 7.2 Precision
Measures the fraction of candidate impacts (CIS) that were truly impacted.

```
Precision = |CIS ∩ AIS| / |CIS|
          = 3 / 4
          = 0.75
```

**Precision = 0.75** — one false positive (ClassListForm) was included in CIS but not changed. FPIS = {ClassListForm} is non-empty.

### 7.3 Adequacy (Inclusiveness)
Adequacy is the ability to identify all affected elements. Ideally AIS ⊆ CIS.

```
Inclusiveness = 1  if AIS ⊆ CIS
              = 0  otherwise
```

AIS = {ClassSearchAction, classSearch.jsp, CourseMessages} ⊆ CIS = {ClassSearchAction, classSearch.jsp, CourseMessages, ClassListForm} ✓

**Inclusiveness = 1** — the approach is adequate; no actual change was missed.

---

## 8. Effectiveness Metrics

### 8.1 Ripple-Sensitivity (Amplification)

**DISO** (directly impacted set of objects) — objects directly affected by the CR:

**DISO = { ClassSearchAction, classSearch.jsp, CourseMessages }** → |DISO| = 3

**IISO** (indirectly impacted set of objects) — objects indirectly impacted via ripple:
- No existing callers of `foundClasses()` exist (new method).
- No subclasses of `ClassSearchAction` exist in the codebase.
- No other JSPs include or delegate to `classSearch.jsp`.
- `request.setAttribute()` is a standard servlet API call — no ripple propagation.

**IISO = { }** → |IISO| = 0

```
Amplification = |IISO| / |DISO| = 0 / 3 ≈ 0
```

**Amplification ≈ 0** — the change has no ripple effect. It is fully contained within the 3 directly modified objects.

### 8.2 Sharpness (ChangeRate)
Sharpness measures the ability to avoid including unnecessary objects in CIS. High sharpness requires ChangeRate ≪ 1.

The total UniTime system contains approximately **2,500 SLOs** (Java source files + JSP views, from codebase metrics).

```
ChangeRate = |CIS| / |System| = 4 / 2500 = 0.0016
```

**ChangeRate = 0.0016** — far below 1. The analysis is highly sharp; only 0.16% of system objects were considered candidates.

### 8.3 Adherence (S-Ratio)
Adherence measures how close CIS is to AIS. Ideally S-Ratio = 1.

```
S-Ratio = |AIS| / |CIS| = 3 / 4 = 0.75
```

**S-Ratio = 0.75** — one candidate (ClassListForm) was not actually changed. While not ideal (1.0), this is acceptable; the false positive was a reasonable conservative precaution and its inclusion caused no wasted implementation effort.

---

## 9. Ripple Effect Analysis

**Stability** reflects the resistance to ripple effect. The SearchResultCounter change exhibits high stability:

- The change adds a new HTTP request attribute (`resultCount`) — a write-only side effect with no feedback loop into existing logic.
- The new `foundClasses()` method in `CourseMessages` is only called from the new JSP block — no existing code is affected.
- Between the `main` branch (before) and the `SearchResultCounter` branch (after), the software's complexity has not increased: no new classes, no new relationships, no new call-graph edges between existing components.
- The addition of `foundClasses()` to `CourseMessages` adds one new node to the system's SLO graph, with in-degree = 1 (called only from classSearch.jsp) and out-degree = 0.

**Error flow analysis:** The only variable involved in the change is `classes.size()` (an `int` derived from the existing `Collection`). Its value propagates only to `request.setAttribute()` and then to the JSP rendering layer — it does not feed back into any computation, query, or decision in `ClassSearchAction`. Inconsistency cannot propagate further. Error propagation terminates at the JSP display layer.

---

## 10. Change Propagation Model (Hassan & Holt, 2006)

| Step | Entity | Action |
|------|--------|--------|
| 1 | `ClassSearchAction` | Initial entity — add `setAttribute("resultCount", ...)` |
| 2 | `CourseMessages` | Suggested entity — add `foundClasses()` for i18n |
| 3 | `classSearch.jsp` | Suggested entity — render count using `foundClasses()` |
| 4 | — | No further entities to change — propagation ends |

The change set is **{ ClassSearchAction, CourseMessages, classSearch.jsp }**. All entities in the change set were changed. No "Consult Guru" step was required.

---

## 11. Dependency-Based Impact Analysis (Call Graph)

A call graph of the new behaviour introduced by the CR:

```
ClassSearchAction.execute()
  └─► getClasses(form, proxy) ──► Hibernate DB
  └─► request.setAttribute("resultCount", classes.size())  [NEW]

ClassSearchAction.performAction()
  └─► getClasses(form, proxy) ──► Hibernate DB
  └─► request.setAttribute("resultCount", classes.size())  [NEW]

classSearch.jsp
  └─► #request.resultCount  [NEW — reads attribute]
  └─► CourseMessages.foundClasses()  [NEW]
       └─► renders "Found N classes"
```

Applying the call-graph assumption: a change in a procedure `p` has the potential to impact all nodes reachable from `p`. The new nodes (`setAttribute`, `foundClasses`) have **no outgoing edges** to existing system procedures — they are leaf nodes. Therefore the call-graph-based CIS for the new code = { classSearch.jsp, CourseMessages } — fully contained and consistent with the AIS computed above.

---

## 12. Metrics Summary Table

| Metric | Formula | Value | Interpretation |
|--------|---------|-------|----------------|
| \|SIS\| | — | 2 | 2 objects initially presumed impacted |
| \|CIS\| | — | 4 | 4 candidate objects traced |
| \|AIS\| | — | 3 | 3 objects actually changed |
| \|DIS\| | — | 0 | No surprises during implementation |
| \|FPIS\| | (CIS∪DIS)−AIS | 1 | 1 false positive (ClassListForm) |
| Recall | \|CIS∩AIS\| / \|AIS\| | **1.0** | Perfect — all real impacts predicted |
| Precision | \|CIS∩AIS\| / \|CIS\| | **0.75** | One false positive |
| Inclusiveness | AIS⊆CIS ? 1 : 0 | **1** | Approach is adequate |
| Amplification | \|IISO\| / \|DISO\| | **0** | No ripple effect |
| ChangeRate | \|CIS\| / \|System\| | **0.0016** | Highly sharp (≪ 1) |
| S-Ratio | \|AIS\| / \|CIS\| | **0.75** | Acceptable adherence |

---

## 13. Regression Testing Scope

Per the impact analysis, the portions of the software requiring regression testing are the AIS:

| Component | Test focus |
|-----------|-----------|
| `ClassSearchAction` | Both `execute()` (auto-search path) and `performAction()` (manual search path) must set `resultCount` correctly |
| `classSearch.jsp` | Count banner appears when `showTable=true` and `resultCount≠null`; hidden when no results |
| `CourseMessages` | `foundClasses()` returns the parameterized string correctly |

Unaffected subsystems (Student Scheduling, Exam Timetabling, Event Management, GWT UI, REST API, Solver) require **no regression testing** — they are outside the AIS and the change propagation stopped at IISO = ∅.
