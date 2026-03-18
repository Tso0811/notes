# 資料庫操作筆記

## 刪除 
- ### DELETE
    - 可以加 WHERE
    - 一筆一筆刪除
    - 會記錄完整 transaction log
    - identity **不會重置**
    - 可以用在有外鍵的表
     ``` sql
    DELETE FROM users
    WHERE age < 18; 
    ```
    >只刪符合條件資料、表結構保留
    
- ### TRUNCATE
    - 不能使用 WHERE
    - 速度**比 DELETE 快**
    - identity **會重置**
    - **表結構保留**
    - 不能用在有外鍵的表
    ``` sql
    TRUNCATE TABLE users;
    ```
    >所有資料清空、id 從 1 重新開始
    
- ### DROP 
    - **表消失**
    - **欄位消失**
    - **資料全部消失**
    ``` sql
    DROP TABLE users;
    ```
    >整張表刪除

***