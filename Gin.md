# Gin後端開發筆記
## c *gin.Context
在 Gin 裡，每一個 HTTP 請求（Request）都會對應到一個 gin.Context 物件
 - c *gin.Context 的 c 是一個 指標變數，指向當前請求的 Context
 - 包含了這個請求所有相關資訊
   - Request  : HTTP 方法、URL、Header、Query、Body
   - Response : 用來寫回應（JSON、HTML、字串等）
   - Params   : Path 參數（:id、:name）
   - Keys     : Context 內部資料（可跨中間件傳值）
   - Errors   : 記錄中間件或 Handler 的錯誤
 - 使用指標原因
   - 為了讓 Handler 可以直接修改 Context 內的資料
   - 如果不是指標，Handler 裡的修改不會反映回 Gin 的底層框架
   - 也避免每次呼叫都複製整個 Context 物件，省記憶體和效能  
***

## Get方法取參數
 - ### Query 參數
   - > http://localhost:8080/user?name=Tom&age=18
    -  ```golang
        //用Query()取得參數 
        r.GET("/user", func(c *gin.Context) {
            name := c.Query("name")       // Tom
            age := c.Query("age")         // "18"，都是字串
            c.JSON(http.StatusOK, gin.H{
                "name": name,
                "age":  age,
            })
        })   
        ```

 - ### URL Path參數
    - >http://localhost:8080/user/123
    - ```golang
        //用Param()取得參數
        r.GET("/user/:id", func(c *gin.Context) {
	        id := c.Param("id") // "123"
	        c.JSON(http.StatusOK, gin.H{"user_id": id})
        })
        ```    
***

## Status Code
 - ### 2xx
   #### 成功
   - 201 Created : ，不僅是這次的 Http 請求成功，還額外表示「有個新的資源成功的被創建」

 - ### 3xx 
   #### 重新導向 – 需要採取進一步的措施以完成請求
   - 301 所代表的，這個 url **永久性** 的搬家了 ， 通常會將新的 url 放在 response header 中的 Location 中，告訴我們新的搬家地址在哪

 - ### 4xx
   #### 前端請求錯誤 – 包含語法錯誤或無法完成
   - 400 bad request : 明顯的客戶端錯誤（例如，格式錯誤的請求語法，太大的大小，無效的請求訊息或欺騙性路由請求），伺服器不能或不會處理該請求

   - 401 Unauthorized : 使用者沒有必要的憑據，該狀態碼表示當前請求需要使用者驗證

 - ### 5xx 
   #### 後端處理有問題 – 伺服器未能完成明顯有效的請求
   - 500 Internal Server Error : 後端在處理這次請求時，發生了錯誤，錯誤有可能是因為後端程式出了 bug、或是其他的原因造成的  

   - 504 Gateway Timeout : 表示請求超時
***
   

## 關於Struct tag
  - ### Json Tag
    - #### 控制 JSON 序列化/反序列化的 **key** 名稱
      - > 序列化 : 將資料結構轉換成可 **儲存** 或 **傳輸** 的格式
      - > 反序列化 : 將序列化後的資料轉成可操作的資料結構

    - #### Go 預設用欄位名稱原樣當 JSON key
        ``` golang
        TakeStaffCode string  // → JSON key 變成 "TakeStaffCode"
        ```
        ``` golang
        TakeStaffCode string `json:"take_staff_code"`  // → JSON key 變成 "take_staff_code"
        ```  
    - #### 範例
        ``` golang
        type MissionData struct {
	        TotalMission      int64   `json:"total_mission"`
        }
        
        c.Json(200 , MissionData{
          TotalMission:totalvalue   
        })//雖然這裡是TotalMission 但因為有指定json tag 所以前端會收到 "total_mission" : 5
        ```
  - ### Xorm Tag
    - #### 控制 struct 欄位與 DB 欄位的對應關係

      - 當這個 struct 要被 xorm 做欄位對應時，只要 xorm 需要把 DB 欄位塞進 struct，就要加 xorm : "'欄位名'"

    - #### 不需要加Xorm tag的情況
      - Count() 只回傳數字、 Exist() 只回傳 bool...等等不需要將DB 欄位塞進 struct

    - #### 範例
      ``` golang
      type MedicTakerData struct {
          A string `json:"task_id" xorm:"'task_id'"`
      } //將DB中的task_id欄位，放進A裡面
      ```
    - #### 欄位名與SQL關鍵字衝突
      若欄位名稱剛好跟SQL關鍵字一樣 ( 例如 : order、where、table等等 ) ，DB 會把它當成指令而不是欄位名，所以需要額外加上 **單引號** 區分  

      例如欄位名叫 order：  
      SELECT order FROM task   -- ❌ DB 看到 ORDER 以為是 ORDER BY  
      SELECT 'order' FROM task -- ✅ 加引號告訴 DB 這是欄位名  
      > MSSQL不區分大小寫 包括關鍵字
  - ### 補充
    Go struct 欄位首字母要大寫，才可以被外部套件存取，( 例如 : JSON、XORM 等套件 )
***      

## 交易 (Transaction)
-  交易是一組資料庫操作的集合，要麼全部成功，要麼全部回滾（rollback），常用於需要跨資料庫、資料表的操作。例如：轉帳。  交易。只能用於 **關聯式** 資料庫，非關聯式資料庫則不能使用。

- ### 交易特性

  - **原子性 (Atomicity)**：交易中的所有操作被視為一個整體，要麼全部成功，要麼全部失敗。
  - **一致性 (Consistency)**：交易執行前後，資料庫必須保持一致性。
  - **隔離性 (Isolation)**：交易的執行不應受到其他交易的干擾。
  - **持久性 (Durability)**：交易提交後，其結果會永久保存。
  > 簡稱為 ACID 特性。

- ### XORM 使用交易
  - #### XORM 提供兩種方式來使用交易：
    1. **手動 Session 交易**  
    2. **Engine.Transaction 方法**
  1. #### 手動 Session 交易
      特點：

      - 需要自己建立 session、開始交易、提交或回滾。  
      - 必須手動管理 session 的關閉 (`defer session.Close()`)，否則可能造成連線洩漏。

      範例：

        ```golang
        session := engine.NewSession()
        defer session.Close()  // 必須手動關閉 session

        err := session.Begin() // 開始交易
        if err != nil {
            panic(err)
        }

        // 執行資料庫操作
        _, err = session.Insert(&User{Name: "Alice"})
        if err != nil {
            session.Rollback() // 發生錯誤回滾
            panic(err)
        }

        _, err = session.Where("id = ?", 1).Update(map[string]interface{}{"balance": 200})
        if err != nil {
            session.Rollback() // 發生錯誤回滾
            panic(err)
        }

        // 提交交易
        err = session.Commit()
        if err != nil {
            session.Rollback() // 提交失敗回滾
            panic(err)
        }
        ```
  2. #### Engine.Transaction方法

      特點 : 
      - 自動close、commit、rollback

      範例 :
      ```go 
      err := engine.Transaction(func(session *xorm.Session) error {
          _, err := session.Insert(&User{Name: "Alice"})
          if err != nil {
              return err   // 返回錯誤 → 自動 rollback
      }