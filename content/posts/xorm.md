---
title: "XORM 操作筆記"
date: 2024-01-04
tags: ["Go", "XORM", "資料庫"]
---

## Find
用於 **查詢多筆資料(SELECT多筆)** 並把結果放進 **slice**

```golang
var users []User
err := engine.Find(&users)
//同sql
//SELECT * FROM user
```
