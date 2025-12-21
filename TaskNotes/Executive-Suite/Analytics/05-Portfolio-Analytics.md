# 📊 PORTFOLIO ANALYTICS & INSIGHTS

> **Data-Driven Project Intelligence** | Real-Time Business Intelligence

---

## 🎯 EXECUTIVE SUMMARY

### 💼 Portfolio Health at a Glance
```dataview
TABLE WITHOUT ID
  projects AS "Project",
  length(rows) AS "📋 Total",
  length(rows.where(r => r.status = "done")) AS "✅ Done",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ Active",
  round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) + "%" AS "Completion",
  choice(
    length(rows.where(r => r.formula.isOverdue)) >= 3,
    "🔴 At Risk",
    choice(
      length(rows.where(r => r.formula.isOverdue)) >= 1 OR length(rows.where(r => r.formula.isBlocked)) >= 1,
      "🟡 Caution",
      choice(
        round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) >= 70,
        "🟢 Healthy",
        "🔵 On Track"
      )
    )
  ) AS "Health",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60) + "h", "—") AS "Total Effort"
FROM #task
WHERE projects
GROUP BY projects
SORT round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) ASC
```

---

## 📈 STRATEGIC METRICS

### 🎖️ Portfolio Performance Indicators
```dataview
TABLE WITHOUT ID
  "Metric" AS "KPI",
  "Value" AS "Current",
  "Target" AS "Goal",
  "Status" AS "Health"
FROM ""
WHERE file = this.file
LIMIT 0
```

| 🎯 KPI | 📊 Current | 🎯 Target | 🏆 Status |
|--------|-----------|----------|-----------|
| **Overall Completion Rate** | `$= Math.round((dv.pages("#task").where(p => p.status == "done").length / dv.pages("#task").length) * 100)`% | 80% | `$= dv.pages("#task").where(p => p.status == "done").length / dv.pages("#task").length >= 0.8 ? "🟢 Exceeding" : dv.pages("#task").where(p => p.status == "done").length / dv.pages("#task").length >= 0.6 ? "🟡 On Track" : "🔴 Below"` |
| **On-Time Delivery** | `$= Math.round((dv.pages("#task").where(p => p.status == "done" && p.due && dv.date(p.file.mtime) <= dv.date(p.due)).length / (dv.pages("#task").where(p => p.status == "done" && p.due).length || 1)) * 100)`% | 90% | `$= (dv.pages("#task").where(p => p.status == "done" && p.due && dv.date(p.file.mtime) <= dv.date(p.due)).length / (dv.pages("#task").where(p => p.status == "done" && p.due).length || 1)) >= 0.9 ? "🟢 Excellent" : "🟡 Improving"` |
| **High Priority Completion** | `$= Math.round((dv.pages("#task").where(p => p.status == "done" && p.priority == "high").length / (dv.pages("#task").where(p => p.priority == "high").length || 1)) * 100)`% | 85% | `$= (dv.pages("#task").where(p => p.status == "done" && p.priority == "high").length / (dv.pages("#task").where(p => p.priority == "high").length || 1)) >= 0.85 ? "🟢 Strong" : "🔴 Focus Needed"` |
| **Active Projects** | `$= dv.pages("#task").where(p => p.projects && p.status != "done").map(p => p.projects).distinct().length` | 5-8 | `$= dv.pages("#task").where(p => p.projects && p.status != "done").map(p => p.projects).distinct().length <= 8 ? "🟢 Optimal" : "🟡 High Load"` |
| **Team Utilization** | `$= Math.round((dv.pages("#task").where(p => p.assignee && p.status == "in-progress").length / (dv.pages("#task").where(p => p.assignee).map(p => p.assignee).distinct().length || 1)))`x | 3-5x | `$= (dv.pages("#task").where(p => p.assignee && p.status == "in-progress").length / (dv.pages("#task").where(p => p.assignee).map(p => p.assignee).distinct().length || 1)) <= 5 ? "🟢 Balanced" : "🔴 Overloaded"` |

---

## 🏆 TOP PERFORMING PROJECTS

### ⭐ Success Stories
```dataview
TABLE WITHOUT ID
  "🏆" AS "",
  projects AS "Project",
  round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) + "%" AS "Completion Rate",
  length(rows.where(r => r.status = "done" AND r.due AND date(r.file.mtime) <= date(r.due))) + "/" + length(rows.where(r => r.due)) AS "On-Time Delivery",
  choice(sum(rows.where(r => r.status = "done").timeEstimate), round(sum(rows.where(r => r.status = "done").timeEstimate) / 60) + "h delivered", "—") AS "Value Delivered"
FROM #task
WHERE projects AND status = "done"
GROUP BY projects
SORT round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) DESC
LIMIT 5
```

---

## 📉 PROJECTS NEEDING ATTENTION

### ⚠️ At-Risk Portfolios
```dataview
TABLE WITHOUT ID
  "⚠️" AS "",
  projects AS "Project",
  length(rows.where(r => r.formula.isOverdue)) AS "⏰ Overdue",
  length(rows.where(r => r.formula.isBlocked)) AS "🛑 Blocked",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ Active",
  choice(
    length(rows.where(r => r.formula.isOverdue)) >= 3,
    "🔴 Critical - Immediate Action",
    choice(
      length(rows.where(r => r.formula.isBlocked)) >= 2,
      "🟡 High - Review Dependencies",
      "🔵 Monitor - Minor Issues"
    )
  ) AS "Action Required"
FROM #task
WHERE projects AND status != "done" AND (formula.isOverdue OR formula.isBlocked)
GROUP BY projects
SORT length(rows.where(r => r.formula.isOverdue)) DESC
```

---

## 📊 WORKLOAD DISTRIBUTION

### 🎯 Effort Allocation Across Portfolio
```dataview
TABLE WITHOUT ID
  projects AS "Project",
  length(rows) AS "Total Tasks",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60, 1) + "h", "Not estimated") AS "Estimated Effort",
  round((sum(rows.timeEstimate) / sum(rows.from.timeEstimate)) * 100) + "%" AS "% of Portfolio",
  choice(length(rows.assignee.distinct()), length(rows.assignee.distinct()) + " people", "—") AS "Team Size"
FROM #task
WHERE projects AND status != "done" AND timeEstimate
GROUP BY projects
SORT sum(rows.timeEstimate) DESC
```

---

## 🎨 PRIORITY DISTRIBUTION

### 🔥 Critical Focus Areas by Project
```dataview
TABLE WITHOUT ID
  projects AS "Project",
  length(rows.where(r => r.priority = "high")) AS "🔴 Critical",
  length(rows.where(r => r.priority = "normal")) AS "🟡 High",
  length(rows.where(r => r.priority = "low")) AS "🟢 Medium",
  choice(
    length(rows.where(r => r.priority = "high")) > length(rows) * 0.5,
    "⚠️ Too many critical items",
    choice(
      length(rows.where(r => r.priority = "high")) > length(rows) * 0.3,
      "🔶 High pressure project",
      "✅ Well balanced"
    )
  ) AS "Priority Balance"
FROM #task
WHERE projects AND status != "done"
GROUP BY projects
SORT length(rows.where(r => r.priority = "high")) DESC
```

---

## 📅 TIMELINE ANALYSIS

### 🗓️ Delivery Schedule Overview
```dataview
TABLE WITHOUT ID
  choice(formula.nextDateMonth, formula.nextDateMonth, "No deadline") AS "Month",
  length(rows) AS "Tasks Due",
  length(rows.where(r => r.priority = "high")) AS "🔴 Critical",
  list(rows.projects.distinct()) AS "Affected Projects",
  choice(
    length(rows.where(r => r.priority = "high")) >= 5,
    "🔴 Heavy month",
    choice(
      length(rows) >= 10,
      "🟡 Busy month",
      "🟢 Normal load"
    )
  ) AS "Capacity Alert"
FROM #task
WHERE due AND status != "done"
GROUP BY formula.nextDateMonth
SORT formula.nextDateMonth ASC
```

---

## 💰 VALUE DELIVERY METRICS

### 📈 Completed Work Analysis (Last 30 Days)
```dataview
TABLE WITHOUT ID
  projects AS "Project",
  length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(30 days))) AS "✅ Delivered",
  choice(sum(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(30 days)).timeEstimate), round(sum(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(30 days)).timeEstimate) / 60, 1) + "h", "—") AS "Effort Hours",
  choice(
    length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(30 days))) >= 10,
    "🚀 High Velocity",
    choice(
      length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(30 days))) >= 5,
      "✅ Good Pace",
      "🔵 Steady Progress"
    )
  ) AS "Momentum"
FROM #task
WHERE projects
GROUP BY projects
SORT length(rows.where(r => r.status = "done" AND date(r.file.mtime) >= date(today) - dur(30 days))) DESC
```

---

## 🎯 STRATEGIC INITIATIVES

### 💎 High-Impact Projects
```dataview
TABLE WITHOUT ID
  projects AS "Strategic Initiative",
  length(rows.where(r => r.formula.executiveScore >= 50)) AS "💎 VIP Tasks",
  length(rows.where(r => r.formula.isHighPriority)) AS "⭐ Premium",
  round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) + "%" AS "Progress",
  choice(assignee, assignee, "Multiple owners") AS "Executive Sponsor"
FROM #task
WHERE projects AND (formula.executiveScore >= 30 OR priority = "high")
GROUP BY projects
SORT length(rows.where(r => r.formula.executiveScore >= 50)) DESC
```

---

## 📊 COMPARATIVE ANALYSIS

### 🏁 Project-to-Project Benchmarking
```dataview
TABLE WITHOUT ID
  projects AS "Project",
  round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) AS "Completion %",
  choice(
    sum(rows.where(r => r.status = "done" AND r.timeEntries).map((r) => list(r.timeEntries).length)) > 0,
    round(sum(rows.where(r => r.timeEstimate).timeEstimate) / (sum(rows.where(r => r.timeEntries).map((r) => sum(list(r.timeEntries).map((t) => (number(date(t.endTime)) - number(date(t.startTime))) / 60000)))) || 1) * 100) + "%",
    "—"
  ) AS "Efficiency",
  length(rows.where(r => r.status = "done" AND r.formula.velocityIndicator = "🚀 Быстро")) AS "🚀 Fast Wins",
  choice(formula.performanceGrade, formula.performanceGrade, "—") AS "Overall Grade"
FROM #task
WHERE projects
GROUP BY projects
SORT round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) DESC
```

---

## 📈 GROWTH & TRENDS

### 📊 30-Day Portfolio Trend
```dataview
TABLE WITHOUT ID
  date(file.ctime).format("Week WW") AS "Week",
  length(rows) AS "📋 Created",
  length(rows.where(r => r.status = "done")) AS "✅ Completed",
  round((length(rows.where(r => r.status = "done")) / length(rows)) * 100) + "%" AS "Completion Rate"
FROM #task
WHERE date(file.ctime) >= date(today) - dur(30 days)
GROUP BY date(file.ctime).weekyear
SORT date(file.ctime).weekyear DESC
```

---

## 🎯 PORTFOLIO INSIGHTS

- **📊 Total Active Projects:** `$= dv.pages("#task").where(p => p.projects && p.status != "done").map(p => p.projects).distinct().length`
- **💼 Total Portfolio Tasks:** `$= dv.pages("#task").where(p => p.projects).length`
- **✅ Overall Completion:** `$= Math.round((dv.pages("#task").where(p => p.status == "done" && p.projects).length / dv.pages("#task").where(p => p.projects).length) * 100)`%
- **🔥 High Priority Projects:** `$= dv.pages("#task").where(p => p.projects && p.priority == "high" && p.status != "done").map(p => p.projects).distinct().length`
- **⚠️ Projects at Risk:** `$= dv.pages("#task").where(p => p.projects && p.status != "done" && (p.formula && p.formula.isOverdue)).map(p => p.projects).distinct().length`
- **🎯 Average Project Size:** `$= Math.round(dv.pages("#task").where(p => p.projects).length / (dv.pages("#task").where(p => p.projects).map(p => p.projects).distinct().length || 1))` tasks

---

> **📊 Portfolio Analytics** | *Data-driven insights for strategic decision making* | Enterprise Intelligence Platform
