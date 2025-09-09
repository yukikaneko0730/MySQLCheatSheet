# 🐬 MySQL Cheat Sheet

> A compact, GitHub‑ready reference for everyday MySQL. Copy, tweak, ship.

---

## 🎨 Style & Design


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

ALTER TABLE users ADD INDEX idx_users_status (status);
ALTER TABLE users MODIFY name VARCHAR(150) NOT NULL;
-- ALTER TABLE users DROP COLUMN status;  -- example

DROP TABLE IF EXISTS users; -- careful
```

---

## 🔡 Data Types

* **Strings:** `VARCHAR(n)`, `TEXT`, `LONGTEXT`, `CHAR(n)`
* **Numbers:** `TINYINT`, `INT`, `BIGINT`, `DECIMAL(p,s)`, `FLOAT`, `DOUBLE`
* **Date/Time:** `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`, `YEAR`
* **Other:** `BOOLEAN` (alias of `TINYINT(1)`), `ENUM`, `SET`, `JSON`

💡 Use `utf8mb4` for full Unicode (emoji). Prefer `DECIMAL` for money.

---

## ✏️ CRUD Essentials

```sql
-- INSERT
INSERT INTO users (email, name) VALUES ('a@x.io','Alex');
INSERT INTO users SET email='b@x.io', name='Bea';

-- UPSERT (ON DUPLICATE KEY)
INSERT INTO users (email, name)
VALUES ('a@x.io','Alex 2')
ON DUPLICATE KEY UPDATE name = VALUES(name);

-- UPDATE
UPDATE users SET name='Alexandra' WHERE email='a@x.io';

-- DELETE
DELETE FROM users WHERE id = 10;

-- Safe delete pattern
SELECT * FROM users WHERE updated_at < NOW() - INTERVAL 1 YEAR; -- preview
DELETE FROM users WHERE updated_at < NOW() - INTERVAL 1 YEAR;   -- execute
```

---

## 🔍 Filtering & Sorting

```sql
SELECT id, email, name
FROM users
WHERE name LIKE 'A%'
  AND email IS NOT NULL
  AND id BETWEEN 100 AND 200
ORDER BY name ASC, id DESC
LIMIT 20 OFFSET 40;  -- page 3 (0‑based), 20 rows/page
```

Common predicates: `=`, `<>`, `>`, `>=`, `<`, `<=`, `BETWEEN`, `IN (...)`, `LIKE`, `REGEXP`, `IS NULL`.

---

## 🔗 Joins & Set Ops

```sql
CREATE TABLE orders (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT UNSIGNED NOT NULL,
  total DECIMAL(10,2) NOT NULL,
  created_at DATETIME NOT NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- INNER JOIN
SELECT u.id, u.name, o.id AS order_id, o.total
FROM users u
JOIN orders o ON o.user_id = u.id;

-- LEFT JOIN
SELECT u.id, u.name, o.id AS order_id
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;

-- SET operations (MySQL 8+)
SELECT email FROM users_2024
UNION DISTINCT
SELECT email FROM users_2025;   -- UNION is DISTINCT by default

SELECT email FROM users_2024
UNION ALL
SELECT email FROM users_2025;   -- keep duplicates
```

---

## 📊 Aggregations & Grouping

```sql
SELECT u.id, u.name, COUNT(o.id) AS order_count, SUM(o.total) AS revenue
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name
HAVING order_count >= 3
ORDER BY revenue DESC;
```

Useful: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP_CONCAT`.

---

## 🌀 Subqueries & CTEs (8+)

```sql
-- Subquery
SELECT *
FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- CTE
WITH high_spenders AS (
  SELECT user_id, SUM(total) AS spend
  FROM orders
  GROUP BY user_id
  HAVING spend > 1000
)
SELECT u.id, u.name, hs.spend
FROM users u
JOIN high_spenders hs ON hs.user_id = u.id;
```

---

## 📈 Window Functions (8+)

```sql
SELECT
  o.id,
  o.user_id,
  o.total,
  SUM(o.total) OVER (PARTITION BY o.user_id ORDER BY o.created_at) AS running_total,
  RANK() OVER (PARTITION BY o.user_id ORDER BY o.total DESC) AS total_rank
FROM orders o;
```

---

## 👁️ Views

```sql
CREATE OR REPLACE VIEW v_user_sales AS
SELECT u.id AS user_id, u.name, SUM(o.total) AS revenue
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;

SELECT * FROM v_user_sales WHERE revenue > 500;
DROP VIEW IF EXISTS v_user_sales;
```

---

## 🗂️ Indexes

```sql
-- Single & composite
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);
DROP INDEX idx_orders_user_created ON orders;

-- Unique
CREATE UNIQUE INDEX idx_users_email ON users (email);

-- Show indexes
SHOW INDEX FROM orders;  -- or INFORMATION_SCHEMA.STATISTICS
```

Guidelines:

* Put most selective columns first in composite indexes.
* Index join keys, `WHERE`, and `ORDER BY` columns.
* Avoid over‑indexing (slows writes).

---

## 🔒 Transactions & Isolation

```sql
START TRANSACTION; -- or BEGIN
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- or ROLLBACK

-- isolation level (session)
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; -- InnoDB default
```

Levels: `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`.

---

## 👤 Users & Privileges

```sql
-- Create user
CREATE USER 'app'@'%' IDENTIFIED BY 'strong_password';

-- Grant least privilege
GRANT SELECT, INSERT, UPDATE, DELETE ON shop.* TO 'app'@'%';
FLUSH PRIVILEGES;  -- often optional in modern MySQL

-- Show grants
SHOW GRANTS FOR 'app'@'%';
```

---

## ⚙️ Stored Routines & Triggers

```sql
DELIMITER $$
CREATE PROCEDURE top_users(IN min_spend DECIMAL(10,2))
BEGIN
  SELECT u.id, u.name, SUM(o.total) AS spend
  FROM users u
  JOIN orders o ON o.user_id = u.id
  GROUP BY u.id, u.name
  HAVING spend >= min_spend
  ORDER BY spend DESC;
END $$
DELIMITER ;

-- Trigger example
DELIMITER $$
CREATE TRIGGER orders_ai
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
  INSERT INTO audit_log(entity, entity_id, action, at)
  VALUES ('orders', NEW.id, 'INSERT', NOW());
END $$
DELIMITER ;
```

---

## 📦 JSON

```sql
-- Store
CREATE TABLE events (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  props JSON NOT NULL,
  INDEX idx_events_user ((CAST(props->>'$.userId' AS UNSIGNED)))  -- functional index
);

-- Query
SELECT JSON_EXTRACT(props, '$.userId') AS userId,
       props->>'$.path' AS path_text
FROM events
WHERE JSON_EXTRACT(props, '$.path') = '/checkout';
```

---

## 📰 Full‑Text Search

```sql
CREATE TABLE articles (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(200),
  body TEXT,
  FULLTEXT KEY ft_title_body (title, body) -- InnoDB supports FULLTEXT
) ENGINE=InnoDB;

SELECT id, MATCH(title, body) AGAINST ('"mysql index" tutorial' IN BOOLEAN MODE) AS score
FROM articles
WHERE MATCH(title, body) AGAINST ('+mysql +index -postgres*' IN BOOLEAN MODE)
ORDER BY score DESC;
```

---

## ⏰ Dates & Time

```sql
SELECT NOW() AS ts,
       CURDATE() AS d,
       DATE_ADD(NOW(), INTERVAL 7 DAY) AS next_week,
       TIMESTAMPDIFF(MINUTE, created_at, NOW()) AS age_min
FROM users;

-- Truncate to date
SELECT DATE(created_at) AS d, COUNT(*)
FROM orders
GROUP BY d;
```

---

## 🚀 Performance & Debugging

```sql
-- Explain plan (8.0.18+: ANALYZE for actuals)
EXPLAIN ANALYZE
SELECT u.id, u.name, SUM(o.total)
FROM users u
JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;

-- Server & session variables
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW STATUS LIKE 'Threads%';
```

Quick tips:

* Always `EXPLAIN` heavy queries; watch `type = ALL` (full scan).
* Use `LIMIT` for exploratory queries.
* Prefer covering indexes and smaller result sets.

---

## 💾 Backup & Restore

```bash
# logical backup (single DB)
mysqldump -u root -p --routines --triggers --events --single-transaction --quick shop > shop.sql

# restore
mysql -u root -p < shop.sql
```

---

## ℹ️ Information Schema Snippets

```sql
-- list tables
SELECT table_name
FROM information_schema.tables
WHERE table_schema = DATABASE();

-- list columns
SELECT column_name, data_type, is_nullable, column_type
FROM information_schema.columns
WHERE table_schema = DATABASE() AND table_name = 'users'
ORDER BY ordinal_position;

-- index usage
SELECT table_name, index_name, seq_in_index, column_name
FROM information_schema.statistics
WHERE table_schema = DATABASE() AND table_name = 'orders'
ORDER BY index_name, seq_in_index;
```

---

## 🔮 Advanced Topics

### Constraints & Foreign Keys

```sql
CREATE TABLE line_items (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  order_id BIGINT UNSIGNED NOT NULL,
  sku VARCHAR(64) NOT NULL,
  qty INT NOT NULL CHECK (qty > 0),               -- MySQL 8.0.16+ (enforced)
  price DECIMAL(10,2) NOT NULL,
  CONSTRAINT fk_li_order
    FOREIGN KEY (order_id) REFERENCES orders(id)
    ON DELETE CASCADE
    ON UPDATE RESTRICT,
  UNIQUE KEY uk_order_sku (order_id, sku)
) ENGINE=InnoDB;
```

Notes:

* `ON DELETE CASCADE` cleans up children automatically.
* Use `CHECK` for business rules (8.0.16+).
* Keep FK columns indexed.

### Import / Export & Bulk Loading

```sql
-- fastest: server-side (needs FILE privilege + secure_file_priv)
LOAD DATA INFILE '/var/lib/mysql-files/users.csv'
INTO TABLE users
FIELDS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(email, name);

-- client-side
LOAD DATA LOCAL INFILE '/path/users.csv' INTO TABLE users
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 LINES;

-- export
SELECT id, email, name
INTO OUTFILE '/var/lib/mysql-files/users_out.csv'
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
FROM users;
```

Tips: For InnoDB, batch inserts; for dev, you may relax `innodb_flush_log_at_trx_commit=2`.

### Locking & Concurrency (InnoDB)

```sql
-- explicit locks
SELECT * FROM orders WHERE id = 1 FOR UPDATE;         -- exclusive
SELECT * FROM orders WHERE id = 1 LOCK IN SHARE MODE;  -- shared (8.0: FOR SHARE)

-- consistent reads
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION; SELECT * FROM orders WHERE user_id=10; -- snapshot
```

Check deadlocks:

```sql
SHOW ENGINE INNODB STATUS;     -- last deadlock
SHOW OPEN TABLES WHERE In_use > 0;
```

### Partitioning (large tables)

```sql
CREATE TABLE events_part (
  id BIGINT PRIMARY KEY,
  created_at DATETIME NOT NULL,
  payload JSON,
  KEY idx_created (created_at)
)
PARTITION BY RANGE COLUMNS(created_at) (
  PARTITION p2025q3 VALUES LESS THAN ('2025-10-01'),
  PARTITION p2025q4 VALUES LESS THAN ('2026-01-01'),
  PARTITION pmax   VALUES LESS THAN (MAXVALUE)
);

-- pruning demo
SELECT COUNT(*) FROM events_part
WHERE created_at >= '2025-10-01' AND created_at < '2026-01-01';
```

Use for time‑series; make sure predicates include the partition key.

### Replication & Binary Log

```sql
SHOW BINARY LOGS;          -- requires privileges
SHOW MASTER STATUS;        -- binlog file & position / GTID set
-- Recommended
-- gtid_mode=ON
-- enforce_gtid_consistency=ON
-- log_bin=ON
-- binlog_format=ROW
```

Monitor lag via `performance_schema.replication_applier_status_by_worker`.

### Event Scheduler

```sql
SET GLOBAL event_scheduler = ON;  -- once per server

CREATE EVENT IF NOT EXISTS ev_purge_old
ON SCHEDULE EVERY 1 DAY STARTS CURRENT_TIMESTAMP + INTERVAL 5 MINUTE
DO
  DELETE FROM audit_log WHERE at < NOW() - INTERVAL 90 DAY;

SHOW EVENTS LIKE 'ev_purge_old';
DROP EVENT IF EXISTS ev_purge_old;
```

### Spatial (GIS)

```sql
CREATE TABLE places (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(120),
  loc POINT NOT NULL SRID 4326,
  SPATIAL INDEX sp_loc (loc)
);

-- nearest 5 within ~5km (approx bbox + precise distance)
SELECT id, name,
       ST_Distance_Sphere(loc, ST_SRID(POINT(13.4050, 52.5200),4326)) AS meters
FROM places
WHERE MBRContains(ST_Buffer(ST_SRID(POINT(13.4050, 52.5200),4326), 0.05), loc)
ORDER BY meters
LIMIT 5;
```

### Optimizer Hints & EXPLAIN

```sql
/*+ INDEX(o idx_orders_user_created) */
SELECT /*+ NO_MERGE() */ u.id, SUM(o.total)
FROM users u
JOIN orders o USE INDEX (idx_orders_user_created) ON o.user_id = u.id
GROUP BY u.id;

EXPLAIN FORMAT=JSON
SELECT u.id, SUM(o.total)
FROM users u JOIN orders o ON o.user_id = u.id
GROUP BY u.id;  -- inspect cost, rows, used indexes
```

### Performance Schema & Slow Query Log

```sql
-- enable slow log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.5; -- seconds

-- quick hotspots
SELECT *
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 20;
```

### Server Config & SQL Modes

```sql
SELECT @@sql_mode;  -- prefer STRICT modes
SET SESSION sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,ONLY_FULL_GROUP_BY';

-- charset/collation
SHOW VARIABLES LIKE 'character_set_server';
SHOW VARIABLES LIKE 'collation_server';
```

Use `utf8mb4_0900_ai_ci` (or `_as_cs` for case‑sensitive) unless you need legacy compat.

### Time Zones

```sql
-- load tz tables once on server
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql

SET time_zone = 'Europe/Berlin';
SELECT CONVERT_TZ(NOW(), 'UTC', 'Europe/Berlin');
```

### Pagination Patterns

```sql
-- classic (offset)
SELECT * FROM orders ORDER BY id DESC LIMIT 20 OFFSET 2000; -- slower at big offsets

-- keyset (seek) pagination
SELECT * FROM orders
WHERE id < 123456
ORDER BY id DESC
LIMIT 20;  -- fast and stable
```

### Sample Data Generation

```sql
-- recursive CTE for sequences (8+)
WITH RECURSIVE seq(n) AS (
  SELECT 1 UNION ALL SELECT n+1 FROM seq WHERE n < 1000
)
INSERT INTO users (email, name)
SELECT CONCAT('user', n, '@example.com'), CONCAT('User ', n)
FROM seq;
```

### CLI One‑Liners

```bash
# run a query and output TSV
mysql -N -B -e "SELECT id, email FROM shop.users LIMIT 5" > users.tsv

# pipe SQL file to server
mysql -u root -p shop < init.sql

# biggest tables
mysql -N -B -e "
SELECT table_schema, table_name, ROUND((data_length+index_length)/1024/1024,1) MB
FROM information_schema.tables
ORDER BY (data_length+index_length) DESC LIMIT 10;" | column -t
```

### Common Gotchas

* `NULL` comparisons: use `IS NULL` / `IS NOT NULL` (not `= NULL`).
* Implicit conversions can break indexes—compare typed columns to *typed* values.
* `ONLY_FULL_GROUP_BY` may fail sloppy `GROUP BY`; fix by selecting aggregated or grouped columns only.
* `VARCHAR(255)` ≠ always right; size by need.

### Checklist for Production

* ✅ `utf8mb4` + sane collation.
* ✅ Proper indexes (cover hot queries).
* ✅ Strict SQL modes.
* ✅ Backups tested (restore drills!).
* ✅ Least‑privilege users.
* ✅ Observability: slow log + performance schema.
* ✅ Binlogs retained (point‑in‑time recovery).

---

## 🛡️ Role-Based Access

```sql
CREATE ROLE reporting_role;
GRANT SELECT ON shop.* TO reporting_role;

CREATE USER 'reporter'@'%' IDENTIFIED BY 'secret';
GRANT reporting_role TO 'reporter'@'%';
SET DEFAULT ROLE reporting_role TO 'reporter'@'%';

SHOW GRANTS FOR 'reporter'@'%';
```

---

## 🔌 ProxySQL & Connection Pooling

**Why**: persistent connections, query routing, failover, caching.

```bash
# proxysql.cnf (snippet)
mysql_servers = (
  { address = "127.0.0.1" , port = 3306 , hostgroup_id = 0 }
)
mysql_users = (
  { username = "app" , password = "pw" , default_hostgroup = 0 }
)
```

---

## 🔥 HeatWave & Analytics

* MySQL HeatWave (Oracle Cloud): in‑memory accelerator for OLAP on MySQL.
* Run analytics + OLTP in one engine; managed service specific.

---

## 🐚 MySQL Shell & AdminAPI

```bash
mysqlsh --uri root@localhost --sql

# dump & load
util.dumpInstance("/backups/shop");
util.loadDump("/backups/shop", {threads:8});

# InnoDB Cluster (simplified)
dba.createCluster("prodCluster");
var cluster = dba.getCluster("prodCluster");
cluster.addInstance("app@node2:3306");
```

---

## 🐳 Docker Compose Template

```yaml
version: '3.8'
services:
  db:
    image: mysql:8.4
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpw
      MYSQL_DATABASE: shop
      MYSQL_USER: app
      MYSQL_PASSWORD: app_pw
    ports:
      - "3306:3306"
    command: ["--character-set-server=utf8mb4", "--collation-server=utf8mb4_0900_ai_ci"]
    volumes:
      - db_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
volumes:
  db_data:
```

Usage: `docker-compose up -d` → init from `init.sql` automatically.

---

### ✨ Mini Style Guide (for Teams)

* Use `snake_case` column names; singular or plural table names consistently.
* Always include `created_at` & `updated_at`.
* Prefer migrations over manual prod changes.

---

