# ⚡ EV Charging Station SQL Analytics

This project analyzes electric vehicle (EV) charging behavior at **shared charging stations** using SQL. It answers key usage questions to help optimize charging infrastructure.

---

## 📊 Objectives & Queries

### 1️⃣ Unique Users per Garage
Counts the number of distinct users who have used shared charging stations in each garage.
- Output columns: `garage_id`, `num_unique_users`
- Sorted from most → least users
- Saved as: **unique_users_per_garage**

```sql
SELECT
    garage_id,
    COUNT(DISTINCT user_id) AS num_unique_users
FROM charging_sessions
WHERE is_shared = TRUE
GROUP BY garage_id
ORDER BY num_unique_users DESC;
````

---

### 2️⃣ Top 10 Most Popular Charging Start Times

Finds the most common start times (weekday + hour) for shared charging sessions.

* Output columns: `weekdays_plugin`, `start_plugin_hour`, `num_charging_sessions`
* Sorted from most → least sessions
* Limited to top 10
* Saved as: **most_popular_shared_start_times**

```sql
SELECT
    EXTRACT(DOW FROM plugin_start_time) AS weekdays_plugin,
    EXTRACT(HOUR FROM plugin_start_time) AS start_plugin_hour,
    COUNT(*) AS num_charging_sessions
FROM charging_sessions
WHERE is_shared = TRUE
GROUP BY weekdays_plugin, start_plugin_hour
ORDER BY num_charging_sessions DESC
LIMIT 10;
```

---

### 3️⃣ Long Duration Shared Users

Identifies users whose average shared charging session lasts > 10 hours.

* Output columns: `user_id`, `avg_charging_duration`
* Sorted from longest → shortest durations
* Saved as: **long_duration_shared_users**

```sql
SELECT
    user_id,
    AVG(charging_duration_hrs) AS avg_charging_duration
FROM charging_sessions
WHERE is_shared = TRUE
GROUP BY user_id
HAVING AVG(charging_duration_hrs) > 10
ORDER BY avg_charging_duration DESC;
```

---

## 🧠 Skills Demonstrated

* SQL Aggregation & Grouping
* Time-based Analysis (weekday & hour extraction)
* Filtering and Ranking Insights
* Practical EV usage analytics

---

## 📁 Output DataFrames

| DataFrame Name                  | Description                           |
| ------------------------------- | ------------------------------------- |
| unique_users_per_garage         | Distinct users per garage             |
| most_popular_shared_start_times | Top 10 busy times by weekday/hour     |
| long_duration_shared_users      | Users who charge >10 hours on average |

---

🚀 This SQL analysis helps identify demand patterns and improve resource planning for EV communities.

