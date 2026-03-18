# XORM操作筆記

## Find
用於 **查詢多筆資料(SELECT多筆)** 並把結果放進 **slice**


```golang
var users []User
err := engine.Find(&users)
//同sql
//SELECT * FROM user
```


