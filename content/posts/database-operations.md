---
title: "資料庫操作筆記"
date: 2024-01-03
draft: false
tags: ["資料庫", "SQL", "MSSQL"]
categories: ["資料庫"]
description: "DELETE、TRUNCATE、DROP 三種刪除方式的差異與使用時機"
---

## 刪除資料的三種方式

### DELETE

逐筆刪除，可以加 WHERE 條件篩選。

```sql
DELETE FROM users
WHERE age < 18;
-- 只刪符合條件的資料，表結構保留
```

**特性：**
- 可加 WHERE 條件
- 一筆一筆刪除，速度較慢
- 會記錄完整 transaction log
- Identity（自動編號）**不會重置**
- 可用於有外鍵（Foreign Key）的表

---

### TRUNCATE

清空整張表，速度比 DELETE 快。

```sql
TRUNCATE TABLE users;
-- 所有資料清空，id 從 1 重新開始
```

**特性：**
- **不能**加 WHERE 條件
- 速度比 DELETE 快
- Identity **會重置**（從 1 開始）
- 表結構保留
- **不能**用於有外鍵的表

---

### DROP

刪除整張表，包含結構與資料。

```sql
DROP TABLE users;
-- 整張表消失，無法復原
```

**特性：**
- 表結構、欄位、資料**全部刪除**
- 操作不可逆

---

## 三者比較

| 特性 | DELETE | TRUNCATE | DROP |
|------|--------|----------|------|
| WHERE 條件 | ✅ 可以 | ❌ 不行 | ❌ 不行 |
| 速度 | 慢 | 快 | 快 |
| 表結構 | 保留 | 保留 | **刪除** |
| Identity 重置 | ❌ 不會 | ✅ 會 | — |
| Transaction Log | 完整記錄 | 少量記錄 | 少量記錄 |
| 有外鍵的表 | ✅ 可以 | ❌ 不行 | ❌ 不行 |
