# ⚠️ ENTERPRISE RISK MANAGEMENT DASHBOARD

> **Strategic Risk Intelligence & Mitigation** | Real-time Monitoring

---

## 🚨 CRITICAL RISK OVERVIEW

### 🔴 High-Impact Risks
```dataview
TABLE WITHOUT ID
  "🚨" AS "",
  file.link AS "Risk Item",
  formula.riskAssessment AS "Risk Level",
  formula.impactLevel AS "Business Impact",
  choice(assignee, "👤 " + assignee, "❌ No Owner") AS "Risk Owner",
  choice(due, date(due).format("MMM DD"), "⚠️ No mitigation date") AS "Mitigation Due",
  choice(projects, projects, "—") AS "Affected Project"
FROM #task
WHERE (formula.riskAssessment = "🚨 Критический риск" OR formula.riskAssessment = "⚠️ Заблокировано" OR formula.isOverdue) AND status != "done"
SORT formula.executiveScore DESC, due ASC
```

---

## 📊 RISK CATEGORIES & DISTRIBUTION

### 🎯 Risk by Type
```dataview
TABLE WITHOUT ID
  choice(formula.riskAssessment, formula.riskAssessment, "Low Risk") AS "Risk Category",
  length(rows) AS "Count",
  choice(length(rows.where(r => r.priority = "high")), length(rows.where(r => r.priority = "high")) + " critical", "—") AS "Critical Items",
  list(rows.file.link)[0:5] AS "Examples (Top 5)"
FROM #task
WHERE status != "done"
GROUP BY formula.riskAssessment
SORT length(rows) DESC
```

---

## 🎨 PROJECT RISK HEAT MAP

### 📁 Risk Exposure by Project
```dataview
TABLE WITHOUT ID
  projects AS "Project Portfolio",
  length(rows) AS "Total Tasks",
  length(rows.where(r => r.formula.isOverdue)) AS "⏰ Overdue",
  length(rows.where(r => r.formula.isBlocked)) AS "🛑 Blocked",
  length(rows.where(r => r.priority = "high" AND r.status != "done")) AS "🔥 High Priority",
  choice(
    length(rows.where(r => r.formula.isOverdue)) >= 3 OR length(rows.where(r => r.formula.isBlocked)) >= 2,
    "🔴 Critical Risk",
    choice(
      length(rows.where(r => r.formula.isOverdue)) >= 1 OR length(rows.where(r => r.priority = "high")) >= 3,
      "🟡 Medium Risk",
      "🟢 Low Risk"
    )
  ) AS "Project Health"
FROM #task
WHERE projects AND status != "done"
GROUP BY projects
SORT length(rows.where(r => r.formula.isOverdue)) DESC
```

---

## ⏰ TIME-BASED RISKS

### 📅 Deadline Risk Analysis
```dataview
TABLE WITHOUT ID
  choice(formula.dueDateCategory, formula.dueDateCategory, "No deadline") AS "Timeline Category",
  length(rows) AS "Task Count",
  length(rows.where(r => r.status = "in-progress")) AS "⚡ Active",
  length(rows.where(r => r.priority = "high")) AS "🔴 Critical",
  choice(
    formula.dueDateCategory = "Overdue",
    "🚨 Immediate Action Required",
    choice(
      formula.dueDateCategory = "Today",
      "⚡ Urgent",
      choice(
        formula.dueDateCategory = "Tomorrow",
        "⚠️ High Priority",
        "✅ Monitored"
      )
    )
  ) AS "Risk Status"
FROM #task
WHERE due AND status != "done"
GROUP BY formula.dueDateCategory
SORT length(rows) DESC
```

---

## 🛑 BLOCKERS & DEPENDENCIES

### 🔗 Critical Path Risks
```dataview
TABLE WITHOUT ID
  "🛑" AS "",
  file.link AS "Blocked Task",
  choice(blockedBy, "Blocked by: " + blockedBy, "Status: " + status) AS "Blocker",
  choice(assignee, assignee, "Unassigned") AS "Owner",
  choice(priority, choice(priority="high","🔴 Critical",choice(priority="normal","🟡 High","🟢 Medium")), "—") AS "Priority",
  choice(due, "Due: " + date(due).format("MMM DD"), "No deadline") AS "Target",
  choice(projects, projects, "—") AS "Project"
FROM #task
WHERE (formula.isBlocked OR status = "blocked") AND status != "done"
SORT priority DESC, due ASC
```

---

## 💼 RESOURCE RISKS

### 👥 Capacity & Allocation Risks
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows) AS "Total Load",
  length(rows.where(r => r.formula.isOverdue)) AS "⚠️ Overdue",
  choice(sum(rows.timeEstimate), round(sum(rows.timeEstimate) / 60) + "h", "—") AS "Estimated Hours",
  choice(
    sum(rows.timeEstimate) >= 2400 OR length(rows.where(r => r.formula.isOverdue)) >= 3,
    "🔴 High Risk - Overloaded",
    choice(
      sum(rows.timeEstimate) >= 1600 OR length(rows.where(r => r.formula.isOverdue)) >= 1,
      "🟡 Medium Risk - At Capacity",
      "🟢 Low Risk - Capacity Available"
    )
  ) AS "Resource Risk"
FROM #task
WHERE assignee AND status != "done"
GROUP BY assignee
SORT sum(rows.timeEstimate) DESC
```

---

## 📈 RISK TREND ANALYSIS

### 📊 7-Day Risk Progression
```dataview
TABLE WITHOUT ID
  date(file.ctime).format("MMM DD") AS "Date Created",
  length(rows) AS "New Risks",
  length(rows.where(r => r.status = "done")) AS "✅ Resolved",
  length(rows.where(r => r.formula.isOverdue)) AS "⏰ Overdue",
  length(rows.where(r => r.status = "blocked")) AS "🛑 Blocked"
FROM #task
WHERE date(file.ctime) >= date(today) - dur(7 days)
GROUP BY date(file.ctime)
SORT date(file.ctime) DESC
```

---

## 🎯 MITIGATION TRACKING

### ✅ Risk Resolution Progress
```dataview
TABLE WITHOUT ID
  file.link AS "Risk Item",
  formula.completionTrend AS "Status",
  choice(formula.statusProgressBar, formula.statusProgressBar, "░░░░░░░░░░") AS "Progress",
  choice(assignee, assignee, "Unassigned") AS "Mitigation Owner",
  choice(formula.velocityIndicator, formula.velocityIndicator, "—") AS "Resolution Speed"
FROM #task
WHERE (formula.riskAssessment != "✓ Под контролем" OR formula.isOverdue OR formula.isBlocked) AND status = "in-progress"
SORT formula.executiveScore DESC
LIMIT 15
```

---

## 💡 RISK MITIGATION RECOMMENDATIONS

### 🧠 Smart Risk Insights
```dataview
TABLE WITHOUT ID
  "💡" AS "",
  file.link AS "Risk",
  choice(
    !assignee,
    "❌ Assign risk owner immediately",
    choice(
      formula.isOverdue AND !due,
      "📅 Set mitigation deadline",
      choice(
        formula.isBlocked AND !blockedBy,
        "🔗 Document blocking dependencies",
        choice(
          priority != "high" AND formula.executiveScore >= 30,
          "🔴 Escalate priority level",
          "✅ Continue monitoring"
        )
      )
    )
  ) AS "Recommended Action",
  formula.impactLevel AS "Impact"
FROM #task
WHERE formula.riskAssessment != "✓ Под контролем" AND status != "done"
SORT formula.executiveScore DESC
LIMIT 10
```

---

## 🎖️ RISK CHAMPIONS

### ⭐ Top Risk Resolvers (Last 30 Days)
```dataview
TABLE WITHOUT ID
  assignee AS "Team Member",
  length(rows.where(r => r.status = "done" AND (r.formula.riskAssessment != "✓ Под контролем" OR r.formula.isOverdue))) AS "🏆 Risks Resolved",
  length(rows.where(r => r.formula.velocityIndicator = "🚀 Быстро")) AS "🚀 Fast Resolutions",
  choice(formula.performanceGrade, formula.performanceGrade, "—") AS "Grade"
FROM #task
WHERE assignee AND status = "done" AND date(file.mtime) >= date(today) - dur(30 days)
GROUP BY assignee
SORT length(rows.where(r => r.status = "done" AND r.formula.isOverdue)) DESC
LIMIT 10
```

---

## 📊 ENTERPRISE RISK METRICS

- **🚨 Critical Risks (Active):** `$= dv.pages("#task").where(p => p.status != "done" && (p.formula && p.formula.riskAssessment == "🚨 Критический риск")).length`
- **⚠️ High Risks (Active):** `$= dv.pages("#task").where(p => p.status != "done" && p.formula && (p.formula.isOverdue || p.formula.isBlocked)).length`
- **🛑 Blocked Tasks:** `$= dv.pages("#task").where(p => (p.status == "blocked" || p.formula && p.formula.isBlocked) && p.status != "done").length`
- **⏰ Overdue Items:** `$= dv.pages("#task").where(p => p.due && dv.date(p.due) < dv.date(today) && p.status != "done").length`
- **📈 Risks Resolved (7d):** `$= dv.pages("#task").where(p => p.status == "done" && dv.date(p.file.mtime) >= dv.date(today) - dv.duration("7 days") && p.formula && p.formula.isOverdue).length`
- **🎯 Risk Resolution Rate:** `$= Math.round((dv.pages("#task").where(p => p.status == "done" && dv.date(p.file.mtime) >= dv.date(today) - dv.duration("7 days")).length / (dv.pages("#task").where(p => dv.date(p.file.ctime) >= dv.date(today) - dv.duration("7 days")).length || 1)) * 100)`%

---

## 🗓️ RISK CALENDAR

### 📅 Upcoming Risk Deadlines
```dataview
CALENDAR due
FROM #task
WHERE (formula.isOverdue OR formula.isBlocked OR priority = "high") AND status != "done" AND due
```

---

> **⚠️ Risk Management Dashboard** | *Proactive risk intelligence for enterprise resilience* | Industry best practices
