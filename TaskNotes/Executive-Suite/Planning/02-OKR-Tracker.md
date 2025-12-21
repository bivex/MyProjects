# 🎯 OKR TRACKER | Objectives & Key Results

> **Strategic Goal Alignment System** | Quarter: `$= "Q" + Math.ceil(date(now).month / 3) + " " + date(now).year`

---

## 📈 COMPANY-WIDE OBJECTIVES

### 🏆 Quarterly Goals Overview

#### **O1: OPERATIONAL EXCELLENCE**
**Goal:** Achieve world-class project delivery and team efficiency

**Key Results:**
```dataview
TABLE WITHOUT ID
  file.link AS "Key Result",
  choice(formula.statusProgressBar, formula.statusProgressBar, "░░░░░░░░░░") AS "Progress",
  choice(formula.progressIndicator, formula.progressIndicator + "%", "0%") AS "Complete",
  choice(assignee, assignee, "Unassigned") AS "Owner",
  choice(due, date(due).format("MMM DD"), "No target") AS "Target"
FROM #task
WHERE contains(file.tags, "okr") AND contains(file.tags, "operational-excellence") AND status != "done"
SORT formula.progressIndicator DESC
```

**Health Status:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr") && p.file.tags.includes("operational-excellence") && p.status == "done").length > 0 ? "🟢 On Track" : "🟡 Needs Attention"`

---

#### **O2: STRATEGIC GROWTH**
**Goal:** Expand market presence and accelerate innovation

**Key Results:**
```dataview
TABLE WITHOUT ID
  file.link AS "Key Result",
  choice(formula.statusProgressBar, formula.statusProgressBar, "░░░░░░░░░░") AS "Progress",
  choice(formula.progressIndicator, formula.progressIndicator + "%", "0%") AS "Complete",
  choice(assignee, assignee, "Unassigned") AS "Owner",
  choice(due, date(due).format("MMM DD"), "No target") AS "Target"
FROM #task
WHERE contains(file.tags, "okr") AND contains(file.tags, "strategic-growth") AND status != "done"
SORT formula.progressIndicator DESC
```

**Health Status:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr") && p.file.tags.includes("strategic-growth") && p.status == "done").length > 0 ? "🟢 On Track" : "🟡 Needs Attention"`

---

#### **O3: CUSTOMER SUCCESS**
**Goal:** Deliver exceptional value and customer satisfaction

**Key Results:**
```dataview
TABLE WITHOUT ID
  file.link AS "Key Result",
  choice(formula.statusProgressBar, formula.statusProgressBar, "░░░░░░░░░░") AS "Progress",
  choice(formula.progressIndicator, formula.progressIndicator + "%", "0%") AS "Complete",
  choice(assignee, assignee, "Unassigned") AS "Owner",
  choice(due, date(due).format("MMM DD"), "No target") AS "Target"
FROM #task
WHERE contains(file.tags, "okr") AND contains(file.tags, "customer-success") AND status != "done"
SORT formula.progressIndicator DESC
```

**Health Status:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr") && p.file.tags.includes("customer-success") && p.status == "done").length > 0 ? "🟢 On Track" : "🟡 Needs Attention"`

---

#### **O4: TEAM DEVELOPMENT**
**Goal:** Build world-class talent and organizational capability

**Key Results:**
```dataview
TABLE WITHOUT ID
  file.link AS "Key Result",
  choice(formula.statusProgressBar, formula.statusProgressBar, "░░░░░░░░░░") AS "Progress",
  choice(formula.progressIndicator, formula.progressIndicator + "%", "0%") AS "Complete",
  choice(assignee, assignee, "Unassigned") AS "Owner",
  choice(due, date(due).format("MMM DD"), "No target") AS "Target"
FROM #task
WHERE contains(file.tags, "okr") AND contains(file.tags, "team-development") AND status != "done"
SORT formula.progressIndicator DESC
```

**Health Status:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr") && p.file.tags.includes("team-development") && p.status == "done").length > 0 ? "🟢 On Track" : "🟡 Needs Attention"`

---

## 📊 OVERALL OKR HEALTH

### 🎯 Aggregate Performance
```dataview
TABLE WITHOUT ID
  choice(file.tags, file.tags[1], "Uncategorized") AS "Objective Area",
  length(rows) AS "Total KRs",
  length(rows.where(r => r.status = "done")) AS "✅ Completed",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ In Progress",
  round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) + "%" AS "Completion %",
  choice(round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) >= 70, "🟢 Excellent", choice(round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) >= 40, "🟡 On Track", "🔴 At Risk")) AS "Status"
FROM #task
WHERE contains(file.tags, "okr")
GROUP BY file.tags
SORT round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) DESC
```

---

## 🎖️ TOP PERFORMERS

### ⭐ Leading Key Results
```dataview
TABLE WITHOUT ID
  "⭐" AS "",
  file.link AS "Key Result",
  choice(formula.performanceGrade, formula.performanceGrade, "—") AS "Grade",
  choice(formula.velocityIndicator, formula.velocityIndicator, "—") AS "Velocity",
  choice(assignee, assignee, "Team") AS "Owner"
FROM #task
WHERE contains(file.tags, "okr") AND status = "done" AND date(file.mtime) >= date(today) - dur(30 days)
SORT file.mtime DESC
LIMIT 10
```

---

## 🚨 ATTENTION NEEDED

### ⚠️ At-Risk Key Results
```dataview
TABLE WITHOUT ID
  "🚨" AS "",
  file.link AS "Key Result",
  formula.riskAssessment AS "Issue",
  choice(assignee, assignee, "❌ No Owner") AS "Owner",
  choice(due, "Due: " + date(due).format("MMM DD"), "⚠️ No deadline") AS "Timeline"
FROM #task
WHERE contains(file.tags, "okr") AND (formula.isOverdue OR formula.isBlocked OR !assignee) AND status != "done"
SORT formula.executiveScore DESC
```

---

## 📅 TIMELINE & MILESTONES

### 🗓️ Quarter Roadmap
```dataview
CALENDAR due
FROM #task
WHERE contains(file.tags, "okr") AND due AND date(due) >= date(today)
```

---

## 💡 OKR BEST PRACTICES

### ✅ Success Criteria Checklist
- [ ] All objectives have 3-5 measurable key results
- [ ] Each key result has a clear owner assigned
- [ ] Timelines are realistic and aligned with quarter end
- [ ] Progress is updated weekly
- [ ] 70% achievement = success (Google standard)
- [ ] Regular check-ins scheduled

### 📊 Scoring Guide
- **🟢 0.7 - 1.0:** Excellent performance
- **🟡 0.4 - 0.6:** On track, needs attention
- **🔴 0.0 - 0.3:** At risk, requires intervention

---

## 📈 QUARTERLY METRICS

- **🎯 Total OKRs This Quarter:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr")).length`
- **✅ Completed KRs:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr") && p.status == "done").length`
- **⚡ In Progress:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr") && p.status == "in-progress").length`
- **📊 Overall Achievement:** `$= Math.round((dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr") && p.status == "done").length / (dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("okr")).length || 1)) * 100)`%

---

> **🎯 OKR Tracker** | *Aligning strategy with execution* | Inspired by Google's OKR methodology
