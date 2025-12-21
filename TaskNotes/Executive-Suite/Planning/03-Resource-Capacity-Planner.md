# 👥 RESOURCE & CAPACITY PLANNER

> **Strategic Workforce Optimization** | Updated: `$= date(now).format("YYYY-MM-DD HH:mm")`

---

## 📊 TEAM CAPACITY OVERVIEW

### 👤 Individual Workload Distribution
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows) AS "📋 Total Tasks",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ Active",
  length(rows.where(r => r.formula.isHighPriority)) AS "🔥 High Priority",
  length(rows.where(r => r.formula.isOverdue)) AS "⚠️ Overdue",
  choice(sum(rows.timeEstimate) >= 2400, "🔴 Overloaded", choice(sum(rows.timeEstimate) >= 1600, "🟡 At Capacity", "🟢 Available")) AS "Capacity Status",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60) + "h", "—") AS "Total Hours"
FROM #task
WHERE assignee AND status != "done"
GROUP BY assignee
SORT sum(rows.timeEstimate) DESC
```

---

## 🎯 WORKLOAD ANALYSIS

### 📈 Task Distribution by Priority
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows.where(r => r.priority = "high")) AS "🔴 High",
  length(rows.where(r => r.priority = "normal")) AS "🟡 Normal",
  length(rows.where(r => r.priority = "low")) AS "🟢 Low",
  length(rows.where(r => !r.priority OR r.priority = "none")) AS "⚪ None",
  choice(length(rows.where(r => r.priority = "high")) > 5, "🚨 Critical", choice(length(rows.where(r => r.priority = "high")) > 2, "⚠️ High Load", "✅ Balanced")) AS "Priority Balance"
FROM #task
WHERE assignee AND status != "done"
GROUP BY assignee
SORT length(rows.where(r => r.priority = "high")) DESC
```

---

## ⏰ TIME TRACKING & ESTIMATES

### 🕐 Estimated vs Actual Time
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60, 1) + "h", "0h") AS "📊 Estimated",
  choice(sum(rows.where(r => r.timeEntries).map((r) => sum(list(r.timeEntries).map((t) => (number(date(t.endTime)) - number(date(t.startTime))) / 3600000)))), round(sum(rows.where(r => r.timeEntries).map((r) => sum(list(r.timeEntries).map((t) => (number(date(t.endTime)) - number(date(t.startTime))) / 3600000)))), 1) + "h", "0h") AS "⚡ Tracked",
  choice(sum(rows.timeEstimate) > 0 AND sum(rows.where(r => r.timeEntries).map((r) => sum(list(r.timeEntries).map((t) => (number(date(t.endTime)) - number(date(t.startTime))) / 60000)))) > 0, round((sum(rows.where(r => r.timeEntries).map((r) => sum(list(r.timeEntries).map((t) => (number(date(t.endTime)) - number(date(t.startTime))) / 60000))))) / sum(rows.timeEstimate) * 100) + "%", "—") AS "Efficiency",
  choice(formula.performanceGrade, formula.performanceGrade, "—") AS "Grade"
FROM #task
WHERE assignee AND (status = "done" OR status = "in-progress")
GROUP BY assignee
SORT sum(rows.timeEstimate) DESC
```

---

## 📅 AVAILABILITY FORECAST

### 🗓️ Weekly Capacity Planning
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows.where(r => r.formula.isDueThisWeek)) AS "📅 Due This Week",
  length(rows.where(r => r.formula.isDueToday)) AS "🔥 Due Today",
  choice(sum(rows.where(r => r.formula.isDueThisWeek).timeEstimate), round(sum(rows.where(r => r.formula.isDueThisWeek).timeEstimate) / 60, 1) + "h", "0h") AS "Hours Needed",
  choice(sum(rows.where(r => r.formula.isDueThisWeek).timeEstimate) > 2400, "🔴 Overcommitted", choice(sum(rows.where(r => r.formula.isDueThisWeek).timeEstimate) > 1600, "🟡 Full", "🟢 Capacity Available")) AS "Week Status"
FROM #task
WHERE assignee AND status != "done"
GROUP BY assignee
SORT sum(rows.where(r => r.formula.isDueThisWeek).timeEstimate) DESC
```

---

## 🎨 PROJECT ALLOCATION

### 📁 Team Members by Project
```dataview
TABLE WITHOUT ID
  projects AS "Project",
  length(rows.assignee.distinct()) AS "👥 Team Size",
  list(rows.assignee.distinct()) AS "Team Members",
  length(rows) AS "Total Tasks",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ Active",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60) + "h", "—") AS "Total Effort"
FROM #task
WHERE projects AND assignee AND status != "done"
GROUP BY projects
SORT length(rows.assignee.distinct()) DESC
```

---

## 🚨 RESOURCE ALERTS

### ⚠️ Capacity Issues & Bottlenecks
```dataview
TABLE WITHOUT ID
  "🚨" AS "",
  assignee AS "Team Member",
  choice(length(rows.where(r => r.formula.isOverdue)) > 0, "⏰ " + length(rows.where(r => r.formula.isOverdue)) + " overdue tasks", choice(sum(rows.timeEstimate) >= 2400, "🔴 Overloaded (" + round(sum(rows.timeEstimate) / 60) + "h)", choice(length(rows.where(r => r.formula.isBlocked)) > 0, "🛑 " + length(rows.where(r => r.formula.isBlocked)) + " blocked", "—"))) AS "Issue",
  length(rows) AS "Total Load",
  choice(projects, projects, "Multiple") AS "Project Focus"
FROM #task
WHERE assignee AND status != "done" AND (formula.isOverdue OR formula.isBlocked OR sum(rows.timeEstimate) >= 2400)
GROUP BY assignee
SORT length(rows.where(r => r.formula.isOverdue)) DESC, sum(rows.timeEstimate) DESC
```

---

## 💡 OPTIMIZATION RECOMMENDATIONS

### 🎯 Suggested Actions
```dataview
TABLE WITHOUT ID
  "💡" AS "",
  assignee AS "For Team Member",
  choice(sum(rows.timeEstimate) >= 2400, "🔄 Redistribute high-priority tasks", choice(length(rows.where(r => r.formula.isOverdue)) > 2, "⚡ Focus on overdue items first", choice(!assignee, "👤 Assign owner immediately", "✅ Well balanced"))) AS "Recommendation",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60) + "h load", "—") AS "Current Load"
FROM #task
WHERE status != "done"
GROUP BY assignee
SORT sum(rows.timeEstimate) DESC
LIMIT 10
```

---

## 📊 UTILIZATION METRICS

### 🎖️ Performance Rankings
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(7 days))) AS "✅ Completed (7d)",
  length(rows.where(r => r.status = "done" AND r.formula.velocityIndicator = "🚀 Быстро")) AS "🚀 Fast",
  choice(formula.productivityRating, formula.productivityRating, "—") AS "Rating",
  choice(length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(7 days))) >= 10, "⭐⭐⭐", choice(length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(7 days))) >= 5, "⭐⭐", "⭐")) AS "Performance"
FROM #task
WHERE assignee
GROUP BY assignee
SORT length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(7 days))) DESC
LIMIT 10
```

---

## 🎯 CAPACITY PLANNING INSIGHTS

- **👥 Total Team Members:** `$= dv.pages("#task").where(p => p.assignee).map(p => p.assignee).distinct().length`
- **📋 Total Active Tasks:** `$= dv.pages("#task").where(p => p.status != "done").length`
- **⚡ Tasks Per Person (Avg):** `$= Math.round(dv.pages("#task").where(p => p.status != "done").length / (dv.pages("#task").where(p => p.assignee).map(p => p.assignee).distinct().length || 1))`
- **🕐 Total Estimated Hours:** `$= Math.round(dv.pages("#task").where(p => p.status != "done" && p.timeEstimate).map(p => p.timeEstimate).sum() / 60)`h
- **🔴 Overloaded Members:** `$= dv.pages("#task").where(p => p.assignee && p.status != "done").groupBy(p => p.assignee).where(g => g.rows.map(r => r.timeEstimate || 0).sum() >= 2400).length`
- **🟢 Available Capacity:** `$= dv.pages("#task").where(p => p.assignee && p.status != "done").groupBy(p => p.assignee).where(g => g.rows.map(r => r.timeEstimate || 0).sum() < 1600).length` members

---

## 📋 UNASSIGNED WORK

### 🎯 Tasks Needing Owners
```dataview
TABLE WITHOUT ID
  file.link AS "Task",
  choice(priority, choice(priority="high","🔴 Critical",choice(priority="normal","🟡 High","🟢 Normal")), "⚪ Unset") AS "Priority",
  choice(projects, projects, "—") AS "Project",
  choice(due, date(due).format("MMM DD"), "No deadline") AS "Due Date",
  choice(timeEstimate, timeEstimate + "m", "—") AS "Estimate"
FROM #task
WHERE !assignee AND status != "done"
SORT priority DESC, due ASC
LIMIT 15
```

---

> **👥 Resource & Capacity Planner** | *Strategic workforce optimization for maximum efficiency*
