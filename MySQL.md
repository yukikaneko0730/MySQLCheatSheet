# 🐬 MySQL Cheat Sheet

> A compact, GitHub‑ready reference for everyday MySQL. Copy, tweak, ship.

---

## 🎨 Style & Design

To make this cheat sheet pop in your portfolio:

* **Color accents** → use Markdown emojis + shields.io badges.
* **Icons** → section headers have icons for visual scanning.
* **Consistency** → keep the same emoji theme (📦 schema, 🔍 queries, ⚡ performance, etc).

---

## 📑 Table of Contents

* [⚡ Quick Start](#-quick-start)
* [📦 Schema Basics](#-schema-basics)
* [🔡 Data Types](#-data-types)
* [✏️ CRUD Essentials](#-crud-essentials)
* [🔍 Filtering & Sorting](#-filtering--sorting)
* [🔗 Joins & Set Ops](#-joins--set-ops)
* [📊 Aggregations & Grouping](#-aggregations--grouping)
* [🌀 Subqueries & CTEs](#-subqueries--ctes)
* [📈 Window Functions](#-window-functions)
* [👁️ Views](#-views)
* [🗂️ Indexes](#-indexes)
* [🔒 Transactions & Isolation](#-transactions--isolation)
* [👤 Users & Privileges](#-users--privileges)
* [⚙️ Stored Routines & Triggers](#-stored-routines--triggers)
* [📦 JSON](#-json)
* [📰 Full‑Text Search](#-fulltext-search)
* [⏰ Dates & Time](#-dates--time)
* [🚀 Performance & Debugging](#-performance--debugging)
* [💾 Backup & Restore](#-backup--restore)
* [ℹ️ Information Schema](#-information-schema-snippets)
* [🔮 Advanced Topics](#-advanced-topics)
* [🛡️ Role-Based Access](#-role-based-access)
* [🔌 ProxySQL & Pooling](#-proxysql--connection-pooling)
* [🔥 HeatWave & Analytics](#-heatwave--analytics)
* [🐚 MySQL Shell & AdminAPI](#-mysql-shell--adminapi)
* [🐳 Docker Compose](#-docker-compose-template)

---

## ⚡ Quick Start

```bash
mysql -u root -p
SELECT VERSION();
CREATE DATABASE shop CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
USE shop;
```

---

## 📦 Schema Basics

```sql
CREATE TABLE users (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL UNIQUE,
  name VARCHAR(100) NOT NULL,
  status ENUM('active','inactive') DEFAULT 'active',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

---

## 🔡 Data Types

* **Strings:** `VARCHAR(n)`, `TEXT`, `LONGTEXT`, `CHAR(n)`
* **Numbers:** `INT`, `BIGINT`, `DECIMAL`, `DOUBLE`
* **Date/Time:** `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`
* **Other:** `BOOLEAN`, `ENUM`, `JSON`

💡 Use `utf8mb4` for full Unicode support (emoji, etc).

---

## ✏️ CRUD Essentials

(...same as before...)

---

## 🔍 Filtering & Sorting

(...same as before...)

---

## 🔗 Joins & Set Ops

(...same as before...)

---

## 📊 Aggregations & Grouping

(...same as before...)

---

## 🌀 Subqueries & CTEs

(...same as before...)

---

## 📈 Window Functions

(...same as before...)

---

## 👁️ Views

(...same as before...)

---

## 🗂️ Indexes

(...same as before...)

---

## 🔒 Transactions & Isolation

(...same as before...)

---

## 👤 Users & Privileges

(...same as before...)

---

## ⚙️ Stored Routines & Triggers

(...same as before...)

---

## 📦 JSON

(...same as before...)

---

## 📰 Full‑Text Search

(...same as before...)

---

## ⏰ Dates & Time

(...same as before...)

---

## 🚀 Performance & Debugging

(...same as before...)

---

## 💾 Backup & Restore

(...same as before...)

---

## ℹ️ Information Schema Snippets

(...same as before...)

---

## 🔮 Advanced Topics

(...constraints, import/export, locking, partitioning, replication, events, GIS, optimizer hints, perf schema, sql modes, pagination, sample data, CLI, gotchas, checklist...)

---

## 🛡️ Role-Based Access

(...same as before...)

---

## 🔌 ProxySQL & Connection Pooling

(...same as before...)

---

## 🔥 HeatWave & Analytics

(...same as before...)

---

## 🐚 MySQL Shell & AdminAPI

(...same as before...)

---

## 🐳 Docker Compose Template

(...same as before...)

---

### ✨ Mini Style Guide (for Teams)

* snake\_case, consistent naming.
* `created_at`, `updated_at` always.
* migrations > manual changes.

---

