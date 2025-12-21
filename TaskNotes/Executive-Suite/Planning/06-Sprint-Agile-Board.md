# 🏃‍♂️ AGILE SPRINT BOARD

> **Sprint Planning & Execution** | Sprint: `$= "Sprint " + Math.ceil((date(now) - date("2024-01-01")) / (14 * 24 * 60 * 60 * 1000))`

---

## 🎯 CURRENT SPRINT OVERVIEW

### 📊 Sprint Metrics Dashboard

| 📈 Metric             | 📊 Value                                                                                                                                                                                                                       | 🎯 Target | 🏆 Status                                                                                           |       |                                                                                                                                                                                                                     |     |                                                                                                                                                                                                                                     |     |                                                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- | --------------------------------------------------------------------------------------------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ---------------------------------------------------------------------------------------------------------------------- |
| **Sprint Progress**   | `$= Math.round((dv.pages("#task").where(p => p.status == "done" && p.file.tags && p.file.tags.includes("current-sprint")).length / (dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint")).length |           | 1)) * 100)`%                                                                                        | 100%  | `$= (dv.pages("#task").where(p => p.status == "done" && p.file.tags && p.file.tags.includes("current-sprint")).length / (dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint")).length |     | 1)) >= 0.8 ? "🟢" : (dv.pages("#task").where(p => p.status == "done" && p.file.tags && p.file.tags.includes("current-sprint")).length / (dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint")).length |     | 1)) >= 0.5 ? "🟡" : "🔴"`                                                                                              |
| **Story Points Done** | `$= dv.pages("#task").where(p => p.status == "done" && p.storyPoints && p.file.tags && p.file.tags.includes("current-sprint")).map(p => p.storyPoints).sum()                                                                   |           | 0`                                                                                                  | 40    | `$= (dv.pages("#task").where(p => p.status == "done" && p.storyPoints && p.file.tags && p.file.tags.includes("current-sprint")).map(p => p.storyPoints).sum()                                                       |     | 0) >= 40 ? "🟢" : (dv.pages("#task").where(p => p.status == "done" && p.storyPoints && p.file.tags && p.file.tags.includes("current-sprint")).map(p => p.storyPoints).sum()                                                         |     | 0) >= 25 ? "🟡" : "🔴"`                                                                                                |
| **Velocity**          | `$= dv.pages("#task").where(p => p.status == "done" && p.storyPoints && date(p.file.mtime) >= date(today) - dur(14 days)).map(p => p.storyPoints).sum()                                                                        |           | 0` pts/sprint                                                                                       | 35-45 | 🟢                                                                                                                                                                                                                  |     |                                                                                                                                                                                                                                     |     |                                                                                                                        |
| **Team Capacity**     | `$= dv.pages("#task").where(p => p.assignee && p.file.tags && p.file.tags.includes("current-sprint") && p.status != "done").map(p => p.assignee).distinct().length` people                                                     | 5-7       | 🟢                                                                                                  |       |                                                                                                                                                                                                                     |     |                                                                                                                                                                                                                                     |     |                                                                                                                        |
| **Blockers**          | `$= dv.pages("#task").where(p => (p.status == "blocked"                                                                                                                                                                        |           | p.formula && p.formula.isBlocked) && p.file.tags && p.file.tags.includes("current-sprint")).length` | 0     | `$= dv.pages("#task").where(p => (p.status == "blocked"                                                                                                                                                             |     | p.formula && p.formula.isBlocked) && p.file.tags && p.file.tags.includes("current-sprint")).length == 0 ? "🟢" : dv.pages("#task").where(p => (p.status == "blocked"                                                                |     | p.formula && p.formula.isBlocked) && p.file.tags && p.file.tags.includes("current-sprint")).length <= 2 ? "🟡" : "🔴"` |

---

## 🏃‍♂️ SPRINT BACKLOG

### 📋 Current Sprint Tasks
```dataview
TABLE WITHOUT ID
  choice(formula.statusEmoji, formula.statusEmoji + " ", "") + file.link AS "User Story",
  choice(storyPoints, "⚡ " + storyPoints + " pts", "—") AS "Points",
  choice(assignee, "👤 " + assignee, "👥 Unassigned") AS "Owner",
  choice(status, choice(status="done","✅ Done",choice(status="in-progress","⚡ In Progress",choice(status="review","👀 Review","📋 Todo"))), "—") AS "Status",
  choice(formula.statusProgressBar, formula.statusProgressBar, "░░░░░░░░░░") AS "Progress"
FROM #task
WHERE contains(file.tags, "current-sprint")
SORT status ASC, storyPoints DESC
```

---

## 🔥 DAILY STANDUP VIEW

### ⚡ Today's Focus
```dataview
TABLE WITHOUT ID
  "🎯" AS "",
  file.link AS "Task",
  choice(assignee, assignee, "Unassigned") AS "Who",
  choice(formula.timeEstimateRussian, formula.timeEstimateRussian, "—") AS "Time",
  choice(formula.isBlocked, "🛑 BLOCKED", choice(status="in-progress", "⚡ Working", "📋 Planned")) AS "Status"
FROM #task
WHERE (formula.isDueToday OR formula.isScheduledToday OR (contains(file.tags, "current-sprint") AND status = "in-progress")) AND status != "done"
SORT formula.isBlocked DESC, assignee ASC
```

---

## 📊 SPRINT BURNDOWN

### 🔥 Remaining Work by Day
```dataview
TABLE WITHOUT ID
  date(due).format("ddd, MMM DD") AS "Date",
  length(rows) AS "Tasks Remaining",
  choice(sum(rows.storyPoints), sum(rows.storyPoints) + " pts", "—") AS "Story Points",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60, 1) + "h", "—") AS "Hours Left"
FROM #task
WHERE contains(file.tags, "current-sprint") AND status != "done" AND due
GROUP BY date(due)
SORT date(due) ASC
```

---

## 👥 TEAM ALLOCATION

### 🎯 Sprint Capacity by Team Member
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows) AS "📋 Tasks",
  choice(sum(rows.storyPoints), sum(rows.storyPoints) + " pts", "—") AS "Story Points",
  length(rows.where(r => r.status = "done")) AS "✅ Done",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ Active",
  round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) + "%" AS "Progress"
FROM #task
WHERE contains(file.tags, "current-sprint") AND assignee
GROUP BY assignee
SORT sum(rows.storyPoints) DESC
```

---

## 🚧 IMPEDIMENTS & BLOCKERS

### 🛑 Sprint Blockers
```dataview
TABLE WITHOUT ID
  "🚨" AS "",
  file.link AS "Blocked Item",
  choice(blockedBy, blockedBy, "See description") AS "Blocker",
  choice(assignee, assignee, "Unassigned") AS "Owner",
  choice(storyPoints, storyPoints + " pts at risk", "—") AS "Impact"
FROM #task
WHERE contains(file.tags, "current-sprint") AND (status = "blocked" OR formula.isBlocked) AND status != "done"
SORT storyPoints DESC
```

---

## 👀 CODE REVIEW QUEUE

### 🔍 Items Ready for Review
```dataview
TABLE WITHOUT ID
  "👀" AS "",
  file.link AS "Pull Request / Task",
  choice(assignee, assignee, "—") AS "Developer",
  choice(storyPoints, storyPoints + " pts", "—") AS "Points",
  choice(file.mtime, "Updated: " + date(file.mtime).format("MMM DD HH:mm"), "—") AS "Last Update"
FROM #task
WHERE contains(file.tags, "current-sprint") AND status = "review"
SORT file.mtime ASC
```

---

## 🎯 SPRINT GOALS

### 🏆 Sprint Objectives & Key Results

**Sprint Goal:** _[Define your sprint goal here]_

**Definition of Done:**
- [ ] All story points committed are completed
- [ ] Code reviewed and merged to main
- [ ] Tests written and passing
- [ ] Documentation updated
- [ ] Demo prepared for stakeholders

**Key Deliverables:**
```dataview
TABLE WITHOUT ID
  file.link AS "Deliverable",
  choice(storyPoints, storyPoints + " pts", "—") AS "Points",
  choice(status="done", "✅ Complete", choice(status="in-progress", "⚡ " + formula.progressIndicator + "%", "📋 Pending")) AS "Status",
  choice(priority, choice(priority="high","🔴 Critical",choice(priority="normal","🟡 High","🟢 Normal")), "—") AS "Priority"
FROM #task
WHERE contains(file.tags, "current-sprint") AND contains(file.tags, "deliverable")
SORT priority DESC, storyPoints DESC
```

---

## 📈 VELOCITY TRACKING

### 🚀 Historical Sprint Performance
```dataview
TABLE WITHOUT ID
  "Sprint " + sprint AS "Sprint #",
  choice(sum(rows.storyPoints), sum(rows.storyPoints), 0) AS "Points Completed",
  length(rows) AS "Tasks Done",
  choice(
    sum(rows.storyPoints) >= 40,
    "🟢 Above Target",
    choice(
      sum(rows.storyPoints) >= 30,
      "🟡 On Target",
      "🔴 Below Target"
    )
  ) AS "Performance"
FROM #task
WHERE sprint AND status = "done"
GROUP BY sprint
SORT sprint DESC
LIMIT 6
```

---

## 💡 SPRINT RETROSPECTIVE

### 📝 What Went Well
- [Add retrospective items here]

### 🔧 What Needs Improvement
- [Add improvement items here]

### 🎯 Action Items for Next Sprint
```dataview
LIST
FROM #task
WHERE contains(file.tags, "retrospective-action")
SORT priority DESC
```

---

## 📅 SPRINT CALENDAR

### 🗓️ Sprint Timeline
```dataview
CALENDAR due
FROM #task
WHERE contains(file.tags, "current-sprint") AND due
```

---

## 🎖️ SPRINT ACHIEVEMENTS

### ⭐ Top Contributors This Sprint
```dataview
TABLE WITHOUT ID
  "🏆" AS "",
  assignee AS "Team Member",
  choice(sum(rows.storyPoints), sum(rows.storyPoints) + " pts", "—") AS "Points Delivered",
  length(rows) AS "Tasks Completed",
  length(rows.where(r => r.formula.velocityIndicator = "🚀 Быстро")) AS "🚀 Fast Completions"
FROM #task
WHERE contains(file.tags, "current-sprint") AND status = "done" AND assignee
GROUP BY assignee
SORT sum(rows.storyPoints) DESC
LIMIT 5
```

---

## 🔄 SPRINT CEREMONIES

### 📆 Upcoming Events
- **Daily Standup:** Every day @ 9:30 AM
- **Sprint Planning:** [Date] @ [Time]
- **Sprint Review:** [Date] @ [Time]
- **Sprint Retrospective:** [Date] @ [Time]

---

## 📊 SPRINT HEALTH METRICS

- **🎯 Sprint Commitment:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint")).length` tasks
- **✅ Completed Stories:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint") && p.status == "done").length`
- **⚡ In Progress:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint") && p.status == "in-progress").length`
- **📋 Todo:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint") && p.status == "todo").length`
- **🛑 Blocked:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint") && (p.status == "blocked" || p.formula && p.formula.isBlocked)).length`
- **🎯 Story Points Completed:** `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint") && p.status == "done" && p.storyPoints).map(p => p.storyPoints).sum() || 0` / `$= dv.pages("#task").where(p => p.file.tags && p.file.tags.includes("current-sprint") && p.storyPoints).map(p => p.storyPoints).sum() || 0`

---

> **🏃‍♂️ Agile Sprint Board** | *Accelerate delivery with Scrum best practices* | Agile Framework
