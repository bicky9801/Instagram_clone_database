# 📸 Instagram Clone: SQL Database Design & Analytics

![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![SQL](https://img.shields.io/badge/Language-SQL-orange)
![Tables](https://img.shields.io/badge/Tables-7-blue)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

A relational database case study that models the core of Instagram (users, photos, likes, comments, follows and hashtags) and answers real business questions with MySQL: **ad-campaign timing, user retention, bot detection, contest winners and hashtag performance.**

<p align="center">
  <img src="images/sql_analytics_insights.png" alt="Instagram Clone SQL Analytics Insights" width="900">
</p>

---

## 📑 Table of Contents

- [Project Overview](#-project-overview)
- [Database Schema](#-database-schema)
- [Dataset](#-dataset)
- [Getting Started](#-getting-started)
- [Business Questions & Key Findings](#-business-questions--key-findings)
- [SQL Techniques Used](#-sql-techniques-used)
- [Query Notes](#-query-notes)
- [Recommendations](#-recommendations)
- [Repository Structure](#-repository-structure)
- [Author](#-author)

---

## 🎯 Project Overview

This project has two parts:

1. **Database design:** a normalized schema with 7 tables, foreign keys, and composite primary keys, seeded with sample data.
2. **Analytics:** a set of SQL challenges that turn raw activity data into decisions a product, marketing or trust & safety team could act on.

| Business Area | Question |
|---|---|
| 🎁 Rewards | Who are the 5 oldest users? |
| 📣 Marketing | Which day of the week gets the most sign-ups? |
| 💌 Retention | Which users have never posted a photo? |
| 🏆 Contest | Which photo got the most likes, and who posted it? |
| 💼 Investors | How many photos does the average user post? |
| #️⃣ Brand partnerships | What are the 5 most used hashtags? |
| 🤖 Platform integrity | Which accounts liked every single photo (likely bots)? |
| 👻 Engagement | Which users have never commented? |

---

## 🗂 Database Schema

<p align="center">
  <img src="images/er_diagram.png" alt="Instagram Clone ER Diagram" width="700">
</p>

### Tables

| Table | Purpose | Primary Key | Foreign Keys |
|---|---|---|---|
| `users` | Registered accounts | `id` | none |
| `photos` | Photos uploaded by users | `id` | `user_id` → `users.id` |
| `comments` | Comments left on photos | `id` | `user_id` → `users.id`, `photo_id` → `photos.id` |
| `likes` | A user liking a photo | `(user_id, photo_id)` | `user_id` → `users.id`, `photo_id` → `photos.id` |
| `follows` | Follower / followee relationships | `(follower_id, followee_id)` | both columns → `users.id` |
| `tags` | Unique hashtag names | `id` | none |
| `photo_tags` | Junction table between photos and tags | `(photo_id, tag_id)` | `photo_id` → `photos.id`, `tag_id` → `tags.id` |

### Relationships

| Relationship | Type | How it is modeled |
|---|---|---|
| Users → Photos | 1 : N | `photos.user_id` |
| Users → Comments, Likes | 1 : N | `comments.user_id`, `likes.user_id` |
| Photos → Comments, Likes | 1 : N | `comments.photo_id`, `likes.photo_id` |
| Users ↔ Users (follows) | M : N (self-referencing) | `follows` junction table |
| Photos ↔ Tags | M : N | `photo_tags` junction table |

### Design highlights

- **Composite primary keys** on `likes`, `follows` and `photo_tags` make duplicate rows impossible, so a user can't like the same photo twice.
- **Self-referencing junction table** (`follows`) models the follower graph without a separate "followers" entity.
- **`tags.tag_name` is `UNIQUE`**, so each hashtag is stored once and reused through `photo_tags`.
- **`created_at` defaults to `NOW()`** on every table, giving a consistent audit trail. In `photos` this column is named `created_dat` in the script, so queries must use that spelling.

---

## 📊 Dataset

The seed data is generated sample data (not real accounts).

| Table | Rows |
|---|---:|
| `users` | 100 |
| `photos` | 257 |
| `comments` | 7,488 |
| `likes` | 8,782 |
| `follows` | 7,623 |
| `tags` | 21 |
| `photo_tags` | 501 |

Users registered between **May 2016 and May 2017**.

---

## 🚀 Getting Started

### Prerequisites

- MySQL 8.0 (or compatible) and a SQL client (MySQL CLI, MySQL Workbench, DBeaver, etc.)

### Setup

```bash
# 1. Clone the repo
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>

# 2. Create the database, tables and sample data
mysql -u root -p < sql/01_instagram_clone_database.sql

# 3. Run the analytics queries
mysql -u root -p ig_clone < sql/02_instagram_clone_challenges.sql
```

Or open both files in MySQL Workbench and run them in order. The first script creates a database named `ig_clone`.

> ⚠️ `01_instagram_clone_database.sql` begins with `CREATE DATABASE ig_clone;`. Drop the database first if you are re-running it.

---

## 🔍 Business Questions & Key Findings

### 1. Reward the longest-standing users

```sql
SELECT * FROM users
ORDER BY created_at
LIMIT 5;
```

**Result:** the five earliest sign-ups are `Darby_Herzog`, `Emilio_Bernier52`, `Elenor88`, `Nicole71` and `Jordyn.Jacobson2` (all May 2016).

---

### 2. Best day to schedule an ad campaign

```sql
SELECT DAYNAME(created_at) AS day, COUNT(*) AS total
FROM users
GROUP BY day
ORDER BY total DESC;
```

| Day | Registrations |
|---|---:|
| **Thursday** | **16** |
| **Sunday** | **16** |
| Friday | 15 |
| Tuesday / Monday | 14 each |

**Insight:** Thursday and Sunday are tied for the most sign-ups, so they are the best days to launch acquisition campaigns.

---

### 3. Target inactive users with an email campaign

```sql
SELECT username
FROM users
LEFT JOIN photos ON users.id = photos.user_id
WHERE photos.id IS NULL;
```

**Result:** **26 users (26%)** have never posted a photo. These are prime candidates for onboarding emails and trending-tag prompts.

---

### 4. Contest: the most-liked photo

```sql
SELECT username, photos.id, photos.image_url, COUNT(*) AS total
FROM photos
INNER JOIN likes ON likes.photo_id = photos.id
INNER JOIN users ON photos.user_id = users.id
GROUP BY photos.id
ORDER BY total DESC
LIMIT 1;
```

**Winner:** 🏆 **`Zack_Kemmer93`** with **48 likes** on **photo #145**.

---

### 5. Average posts per user (investor metric)

```sql
SELECT ROUND((SELECT COUNT(*) FROM photos) / (SELECT COUNT(*) FROM users), 2);
```

| Metric | Value |
|---|---:|
| Total photos | 257 |
| Total users | 100 |
| **Average posts per user** | **2.57** |
| Users with at least one post | **74** |

---

### 6. Top 5 hashtags for brand partnerships

```sql
SELECT tag_name, COUNT(tag_name) AS total
FROM tags
JOIN photo_tags ON tags.id = photo_tags.tag_id
GROUP BY tags.id
ORDER BY total DESC;
```

| Rank | Hashtag | Uses |
|---:|---|---:|
| 1 | `#smile` | 59 |
| 2 | `#beach` | 42 |
| 3 | `#party` | 39 |
| 4 | `#fun` | 38 |
| 5 | `#concert` | 24 |

**Insight:** lifestyle and positivity tags dominate, which makes them a good fit for sponsor placements. `#lol` also has 24 uses, so it ties with `#concert` for 5th place.

---

### 7. Bot detection: users who liked every photo

```sql
SELECT users.id, username, COUNT(users.id) AS total_likes_by_user
FROM users
JOIN likes ON users.id = likes.user_id
GROUP BY users.id
HAVING total_likes_by_user = (SELECT COUNT(*) FROM photos);
```

**Result:** **13 accounts** liked all **257** photos. That behavior is a strong automation signal, and removing these accounts cleans up engagement metrics and protects advertiser trust.

---

### 8. Users who have never commented

```sql
SELECT COUNT(*) FROM (
    SELECT username, comment_text
    FROM users
    LEFT JOIN comments ON users.id = comments.user_id
    GROUP BY users.id
    HAVING comment_text IS NULL
) AS users_without_comments;
```

**Result:** **23 users (23%)** have never commented, while **77%** have commented at least once.

---

### 9. Mega challenge: bots and "celebrity" accounts

A combined query (in `02_instagram_clone_challenges.sql`) uses nested subqueries and a `JOIN` of derived tables to report, side by side, the share of users who **never commented** (23%) and the share who **liked every photo** (13%).

---

## 🧠 SQL Techniques Used

| Technique | Where it's used |
|---|---|
| `LEFT JOIN` + `IS NULL` (anti-join) | Inactive users, users who never commented |
| `INNER JOIN` across 3 tables | Contest winner |
| `GROUP BY` + `COUNT()` | Registrations per day, hashtag usage |
| `HAVING` with a scalar subquery | Bot detection |
| Scalar subqueries | Average posts per user |
| Derived tables / nested subqueries | Mega challenge, total post counts |
| Date functions (`DAYNAME`, `DATE_FORMAT`) | Best sign-up day |
| Composite keys & junction tables | Schema design for M:N relationships |

---

## 📝 Query Notes

- **Contest winner: use the `photos.user_id` join.** The challenge file contains two versions of this query. The first joins `users` on `likes.user_id`, which returns the user who *liked* the photo, not the one who *posted* it. The second joins on `photos.user_id` and returns the correct creator (`Zack_Kemmer93`). The version shown above is the correct one.
- **`ONLY_FULL_GROUP_BY` (MySQL 8 default).** A few queries select columns that are not in the `GROUP BY` (for example `comment_text` in the "never commented" query). Depending on your `sql_mode`, MySQL 8 may reject them with error 1055. The portable alternative is `LEFT JOIN ... WHERE comments.id IS NULL`, or `SET SESSION sql_mode = ''` for a quick test.
- **Ties exist in two results** (Thursday/Sunday and `#concert`/`#lol`), so add a secondary `ORDER BY` if you need a deterministic ranking.

---

## 💡 Recommendations

| Initiative | Action |
|---|---|
| 📅 **Schedule optimization** | Concentrate paid acquisition on Thursdays and Sundays. |
| 🤖 **Bot purge workflow** | Automate SQL triggers or scheduled jobs that flag users whose like count equals the total photo count. |
| 💌 **Nudge campaigns** | Send onboarding emails to the 26% of accounts with zero posts. |
| 🤝 **Brand partnerships** | Pitch sponsors on high-volume lifestyle hashtags like `#smile`, `#beach` and `#party`. |

---

## 📁 Repository Structure

```
.
├── README.md
├── sql/
│   ├── 01_instagram_clone_database.sql      # Schema + sample data
│   └── 02_instagram_clone_challenges.sql    # Analytics queries
├── images/
│   ├── er_diagram.png                       # ER diagram
│   └── sql_analytics_insights.png           # Insights infographic
└── docs/
    └── Instagram_DB_Analytics.pptx          # Presentation deck
```

---

## 👤 Author

**[Your Name]**
🔗 [LinkedIn](https://www.linkedin.com/in/your-profile) · 💻 [GitHub](https://github.com/your-username)

If you found this project useful, give it a ⭐ and connect with me on LinkedIn for more **SQL & Data Engineering case studies**.
