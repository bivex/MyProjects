# 📊 Dataview Сниппеты для TaskNotes

## 🎯 Быстрые Запросы

### Задачи на сегодня
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  priority AS "Приоритет",
  timeEstimate AS "Время",
  projects AS "Проект"
FROM #task
WHERE due = date(today) OR scheduled = date(today)
SORT priority DESC
```

### Просроченные задачи
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  due AS "Дедлайн",
  priority AS "Приоритет",
  projects AS "Проект"
FROM #task
WHERE date(due) < date(today) AND status != "done"
SORT due ASC
```

### Задачи по проекту
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  status AS "Статус",
  priority AS "Приоритет",
  due AS "Дедлайн"
FROM #task
WHERE contains(projects, "Проект A")
SORT status ASC
```

## 📊 Аналитика

### Прогресс по проектам
```dataview
TABLE WITHOUT ID
  projects AS "Проект",
  length(rows) AS "Всего"
FROM #task
WHERE status != "done"
GROUP BY projects
SORT length(rows) DESC
```

*Примечание: Для расчета прогресса используйте отдельные запросы для завершенных и незавершенных задач*

### Эффективность работы
```dataview
TABLE WITHOUT ID
  "Показатель" AS "Метрика",
  length(rows.where(status = "done" AND completed)) AS "Значение"
FROM #task
WHERE completed AND date(completed) >= date(today) - dur(7 days)
```

## 📅 Календари

### Календарь дедлайнов
```dataview
CALENDAR due
FROM #task
WHERE due AND status != "done"
```

### Календарь создания задач
```dataview
CALENDAR file.ctime
FROM #task
WHERE date(file.ctime) >= date(today) - dur(30 days)
```

## 🔄 Автоматизация с Templater

### Шаблон ежедневной заметки
```markdown
# 📅 {{date:YYYY-MM-DD}}

## 🎯 Задачи на сегодня
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  priority AS "Приоритет",
  timeEstimate AS "Время"
FROM #task
WHERE date(due) = date("{{date}}") OR date(scheduled) = date("{{date}}")
SORT priority DESC
```

## 📈 Прогресс
- ✅ Выполнено вчера: `{{yesterday_completed}}`
- 🎯 Цели на сегодня: `{{today_goals}}`
- 📊 Общий прогресс: `{{overall_progress}}`

## 📝 Заметки
```

### Автоматическое обновление прогресса
```javascript
// В Templater функции
function getCompletedTasks(date) {
  // Dataview запрос для подсчета выполненных задач
}

function getTotalTasks() {
  // Общее количество активных задач
}
```

## ⚡ Полезные сниппеты

### Задачи без дедлайнов
```dataview
LIST
FROM #task
WHERE !due AND status != "done"
SORT file.ctime DESC
```

### Задачи с высокой приоритетностью
```dataview
LIST
FROM #task
WHERE priority = "high" AND status != "done"
SORT due ASC
```

### Группировка по статусу
```dataview
TASK
FROM #task
WHERE status != "done"
GROUP BY status
SORT status ASC
```

## 🎨 Кастомизация

### Цветовая кодировка приоритетов
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  choice(priority = "high", "🔴", choice(priority = "normal", "🟡", choice(priority = "low", "🟢", "⚪"))) AS "Приоритет",
  status AS "Статус",
  due AS "Дедлайн"
FROM #task
WHERE status != "done"
SORT priority DESC
```

### Форматирование времени
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  choice(timeEstimate > 60, timeEstimate / 60 + "ч " + timeEstimate % 60 + "м", timeEstimate + "м") AS "Время",
  priority AS "Приоритет"
FROM #task
WHERE timeEstimate AND status != "done"
```

## 🔧 Техническое обслуживание

### Очистка старых задач
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  file.mtime AS "Последнее изменение",
  status AS "Статус"
FROM #task
WHERE date(file.mtime) < date(today) - dur(90 days) AND status = "done"
SORT file.mtime ASC
```

### Задачи без обновлений
```dataview
TABLE WITHOUT ID
  file.link AS "Задача",
  date(file.mtime) AS "Обновлено",
  date(today) - date(file.mtime) AS "Дней без обновления"
FROM #task
WHERE status != "done" AND date(file.mtime) < date(today) - dur(7 days)
SORT date(file.mtime) ASC
```
