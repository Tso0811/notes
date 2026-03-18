---
title: "Gin 後端開發筆記"
date: 2024-01-01
draft: false
tags: ["Go", "Gin", "後端"]
categories: ["Go"]
description: "Gin 框架的 Context、路由參數、HTTP 狀態碼、Struct Tag 與資料庫交易"
---

## gin.Context

每一個 HTTP 請求都會對應到一個 `gin.Context` 物件，`c *gin.Context` 是指向當前請求 Context 的指標變數。

**包含的資訊：**

| 欄位 | 說明 |
|------|------|
| Request | HTTP 方法、URL、Header、Query、Body |
| Response | 寫回應（JSON、HTML、字串等） |
| Params | Path 參數（`:id`、`:name`） |
| Keys | Context 內部資料（可跨中間件傳值） |
| Errors | 記錄中間件或 Handler 的錯誤 |

**使用指標的原因：**
- Handler 可以直接修改 Context 內的資料
- 不用每次呼叫都複製整個 Context 物件，省記憶體與效能

---

## 取得請求參數

### Query 參數

URL 格式：`http://localhost:8080/user?name=Tom&age=18`

```go
r.GET("/user", func(c *gin.Context) {
    name := c.Query("name") // "Tom"
    age  := c.Query("age")  // "18"（永遠是字串）
    c.JSON(http.StatusOK, gin.H{
        "name": name,
        "age":  age,
    })
})
```

### URL Path 參數

URL 格式：`http://localhost:8080/user/123`

```go
r.GET("/user/:id", func(c *gin.Context) {
    id := c.Param("id") // "123"
    c.JSON(http.StatusOK, gin.H{"user_id": id})
})
```

---

## HTTP 狀態碼

### 2xx — 成功

| 代碼 | 說明 |
|------|------|
| 200 OK | 請求成功 |
| 201 Created | 請求成功，且新資源已被建立 |

### 3xx — 重新導向

| 代碼 | 說明 |
|------|------|
| 301 Moved Permanently | URL **永久性** 搬移，新位址放在 Response Header 的 `Location` |

### 4xx — 客戶端錯誤

| 代碼 | 說明 |
|------|------|
| 400 Bad Request | 請求語法錯誤、格式不符，伺服器拒絕處理 |
| 401 Unauthorized | 缺少驗證憑據，需要使用者認證 |

### 5xx — 伺服器錯誤

| 代碼 | 說明 |
|------|------|
| 500 Internal Server Error | 後端處理時發生錯誤（bug 或其他原因） |
| 504 Gateway Timeout | 請求超時 |

---

## Struct Tag

### JSON Tag

控制 JSON 序列化/反序列化的 key 名稱。

- **序列化**：將資料結構轉換成可儲存或傳輸的格式
- **反序列化**：將序列化後的資料轉回可操作的資料結構

Go 預設用欄位名稱原樣當 JSON key，加 tag 可以自訂：

```go
// 未加 tag → JSON key 為 "TakeStaffCode"
TakeStaffCode string

// 加 tag → JSON key 為 "take_staff_code"
TakeStaffCode string `json:"take_staff_code"`
```

**範例：**

```go
type MissionData struct {
    TotalMission int64 `json:"total_mission"`
}

c.JSON(200, MissionData{TotalMission: 5})
// 前端收到 → {"total_mission": 5}
```

### XORM Tag

控制 struct 欄位與資料庫欄位的對應關係。

當 XORM 需要將 DB 欄位寫入 struct 時，需要加 `xorm:"'欄位名'"`：

```go
type MedicTakerData struct {
    A string `json:"task_id" xorm:"'task_id'"`
}
// 將 DB 中的 task_id 欄位放進 A
```

**不需要加 XORM Tag 的情況：** `Count()`、`Exist()` 等只回傳數字或 bool 的方法。

**欄位名與 SQL 關鍵字衝突：**

若欄位名與 SQL 關鍵字相同（如 `order`、`where`、`table`），需加單引號區分：

```sql
SELECT order FROM task   -- ❌ DB 把 ORDER 當成關鍵字
SELECT 'order' FROM task -- ✅ 加引號表示這是欄位名
```

> MSSQL 不區分大小寫，包括關鍵字。

> Go struct 欄位首字母必須**大寫**，外部套件（JSON、XORM）才能存取。

---

## 交易（Transaction）

交易是一組資料庫操作的集合，要麼全部成功，要麼全部回滾（Rollback）。常用於跨資料表操作，例如轉帳。

> 交易只能用於**關聯式**資料庫。

### ACID 特性

| 特性 | 說明 |
|------|------|
| 原子性 (Atomicity) | 所有操作視為整體，全部成功或全部失敗 |
| 一致性 (Consistency) | 執行前後資料庫必須保持一致 |
| 隔離性 (Isolation) | 不受其他並行交易干擾 |
| 持久性 (Durability) | 提交後結果永久保存 |

### XORM 交易兩種方式

#### 方式一：手動 Session

需要自行管理 `session.Close()`，否則會造成連線洩漏。

```go
session := engine.NewSession()
defer session.Close()

if err := session.Begin(); err != nil {
    panic(err)
}

_, err = session.Insert(&User{Name: "Alice"})
if err != nil {
    session.Rollback()
    panic(err)
}

_, err = session.Where("id = ?", 1).Update(map[string]interface{}{"balance": 200})
if err != nil {
    session.Rollback()
    panic(err)
}

if err = session.Commit(); err != nil {
    session.Rollback()
    panic(err)
}
```

#### 方式二：Engine.Transaction（推薦）

自動處理 Close、Commit、Rollback，程式碼更簡潔。

```go
err := engine.Transaction(func(session *xorm.Session) error {
    _, err := session.Insert(&User{Name: "Alice"})
    if err != nil {
        return err // 自動 Rollback
    }

    _, err = session.Where("id = ?", 1).Update(map[string]interface{}{"balance": 200})
    if err != nil {
        return err // 自動 Rollback
    }

    return nil // 自動 Commit
})
```
