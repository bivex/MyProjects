# 💎 EXECUTIVE COMMAND CENTER

> **Strategic Overview Dashboard** | Last Updated: `$= date(now).format("YYYY-MM-DD HH:mm")`

---

## 📊 CRITICAL METRICS AT A GLANCE

### 🎯 Strategic Objectives Status
```dataview
TABLE WITHOUT ID
  choice(priority="high", "🔴 Critical", choice(priority="normal", "🟡 High", choice(priority="low", "🟢 Medium", "⚪ Low"))) AS "Priority",
  file.link AS "Initiative",
  choice(status="done","✅ Complete",choice(status="in-progress","⚡ Active",choice(status="review","👀 Review",choice(status="blocked","🚫 Blocked","📋 Queued")))) AS "Status",
  choice(due, choice(date(due) < date(today), "🚨 Overdue", choice(date(due) = date(today), "⚡ TODAY", choice(date(due) <= date(today) + dur(7 days), "📅 This Week", date(due).format("MMM DD")))), "—") AS "Timeline",
  choice(projects, projects, "—") AS "Portfolio"
FROM #task
WHERE status != "done" AND priority = "high"
SORT priority DESC, due ASC
LIMIT 15
```

---

## 🔥 URGENT ATTENTION REQUIRED

### ⚠️ Critical Issues & Blockers
```dataview
TABLE WITHOUT ID
  "🚨" AS "",
  file.link AS "Issue",
  choice(due AND date(due) < date(today) AND priority="high", "❌ Critical Risk", choice(status="blocked", "🛑 Blocked", choice(due AND date(due) < date(today), "⏰ Overdue", "⚡ Action Needed"))) AS "Risk Level",
  choice(assignee, assignee, "Unassigned") AS "Owner",
  choice(due, date(due).format("MMM DD"), "No deadline") AS "Due"
FROM #task
WHERE status != "done" AND (status = "blocked" OR (due AND date(due) < date(today)))
SORT priority DESC, due ASC
```

---

## 📈 PORTFOLIO PERFORMANCE

### 💼 Active Projects Health
```dataview
TABLE WITHOUT ID
  projects AS "Project",
  length(rows) AS "Total Tasks",
  length(rows.where(r => r.status = "done")) AS "✅ Done",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ Active",
  length(rows.where(r => r.status = "blocked")) AS "🚫 Blocked",
  choice(length(rows.where(r => r.due AND date(r.due) < date(today) AND r.status != "done")) > 0, "🔴 At Risk", choice(length(rows.where(r => r.status = "blocked")) > 0, "🟡 Warning", "🟢 On Track")) AS "Health"
FROM #task
WHERE projects AND status != "done"
GROUP BY projects
SORT length(rows) DESC
```

---

## ⚡ VELOCITY & CAPACITY

### 📊 Team Performance Metrics
```dataview
TABLE WITHOUT ID
  "Last 7 Days" AS "Period",
  length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(7 days))) AS "✅ Completed",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ In Progress",
  length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(7 days) AND r.due AND date(r.due) >= date(r.file.mtime))) AS "🎯 On Time"
FROM #task
```

### 🎯 Weekly Completion Trend
```dataview
TABLE WITHOUT ID
  choice(status="done","✅","⚡") AS "",
  file.link AS "Task",
  choice(priority, choice(priority="high","🔴",choice(priority="normal","🟡","🟢")), "—") AS "Priority",
  choice(assignee, assignee, "—") AS "Owner",
  choice(timeEstimate, choice(timeEstimate >= 60, round(timeEstimate / 60, 1) + "h", timeEstimate + "m"), "—") AS "Time"
FROM #task
WHERE status = "done" AND date(file.mtime) >= date(today) - dur(7 days)
SORT file.mtime DESC
LIMIT 10
```

---

## 🎯 STRATEGIC FOCUS AREAS

### 📅 This Week's Priorities
```dataview
TABLE WITHOUT ID
  choice(priority="high", "🔴 ", choice(priority="normal", "🟡 ", "🟢 ")) + file.link AS "Priority Task",
  choice(assignee, "👤 " + assignee, "👥 Unassigned") AS "Owner",
  choice(timeEstimate, choice(timeEstimate >= 120, "🔥 Long", choice(timeEstimate >= 30, "⚡ Medium", "✅ Quick")), "—") AS "Effort",
  choice(due, date(due).format("ddd, MMM DD"), "No deadline") AS "Target Date"
FROM #task
WHERE status != "done" AND (priority = "high" OR (due AND date(due) >= date(today) AND date(due) <= date(today) + dur(7 days)))
SORT priority DESC, due ASC
LIMIT 12
```

---

## 📆 CALENDAR OVERVIEW

### 🗓️ Upcoming Milestones
```dataview
CALENDAR due
FROM #task
WHERE due AND status != "done" AND date(due) >= date(today) AND date(due) <= date(today) + dur(30 days)
```

---

## 💡 INSIGHTS & RECOMMENDATIONS

### 🧠 Smart Alerts
```dataview
TABLE WITHOUT ID
  choice(due AND date(due) < date(today) AND priority="high", "🚨", choice(status="blocked", "⚠️", choice(due AND date(due) < date(today), "⏰", "ℹ️"))) AS "",
  file.link AS "Task",
  choice(due AND date(due) < date(today) AND priority="high", "Critical Risk", choice(status="blocked", "Blocked", choice(due AND date(due) < date(today), "Overdue", "Attention Needed"))) AS "Alert",
  choice(priority="high", "🔴 High Impact", choice(priority="normal", "🟡 Medium Impact", "🟢 Low Impact")) AS "Impact",
  choice(assignee, assignee, "No owner assigned") AS "Action Required By"
FROM #task
WHERE status != "done" AND (status = "blocked" OR (due AND date(due) < date(today)) OR priority = "high")
SORT priority DESC, due ASC
LIMIT 8
```

---

## 📊 KEY PERFORMANCE INDICATORS

- **🎯 Active Initiatives:** `$= dv.pages("#task").where(p => p.status != "done").length || 0`
- **✅ Completed This Month:** `$= dv.pages("#task").where(p => p.status == "done" && dv.date(p.file.mtime) >= dv.date(today) - dv.duration("30 days")).length || 0`
- **🔥 High Priority Open:** `$= dv.pages("#task").where(p => p.priority == "high" && p.status != "done").length || 0`
- **⚠️ At Risk Tasks:** `$= dv.pages("#task").where(p => p.status != "done" && p.due && dv.date(p.due) < dv.date(today)).length || 0`
- **📈 Completion Rate:** `$= Math.round((dv.pages("#task").where(p => p.status == "done" && dv.date(p.file.mtime) >= dv.date(today) - dv.duration("7 days")).length / Math.max(dv.pages("#task").where(p => dv.date(p.file.ctime) >= dv.date(today) - dv.duration("7 days")).length, 1)) * 100)`%

---

## 👥 TEAM WORKLOAD

### 📊 Active Tasks by Team Member
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows) AS "📋 Total",
  length(rows.where(r => r.priority = "high")) AS "🔴 High Priority",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ In Progress",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60, 1) + "h", "—") AS "Estimated Time"
FROM #task
WHERE assignee AND status != "done"
GROUP BY assignee
SORT length(rows.where(r => r.priority = "high")) DESC
LIMIT 10
```

---

## 🚨 IMMEDIATE ACTION REQUIRED

### ⚡ Tasks Due Today
```dataview
LIST
FROM #task
WHERE due AND date(due) = date(today) AND status != "done"
SORT priority DESC
```

### 🛑 Blocked Tasks Needing Resolution
```dataview
LIST
FROM #task
WHERE status = "blocked"
SORT priority DESC
```

---

> **💎 Executive Command Center** | *Real-time strategic oversight for decision makers*
