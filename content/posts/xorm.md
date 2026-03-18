---
title: "XORM 操作筆記"
date: 2024-01-04
draft: false
tags: ["Go", "XORM", "資料庫"]
categories: ["Go"]
description: "XORM ORM 函式庫的常用操作方法"
---

## Find — 查詢多筆資料

`Find()` 用於執行 SELECT 多筆，並將結果放入 slice。

```go
var users []User
err := engine.Find(&users)
// 等同於：SELECT * FROM user
```

**帶條件查詢：**

```go
var users []User
err := engine.Where("age > ?", 18).Find(&users)
// 等同於：SELECT * FROM user WHERE age > 18
```

> 結果一定要傳入 slice 的指標（`&users`），XORM 才能將多筆資料填入。
