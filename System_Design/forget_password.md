# 重設密碼

## Sequence Diagram

```mermaid
sequenceDiagram
participant User
participant FE
participant BE
participant DataBase

User ->>+ FE: 點選忘記密碼
FE ->>+ User: 顯示忘記密碼畫面
User ->>+ FE: 輸入統編、帳號
FE ->>+ BE: /api/1.0/auth/check_reset_password_data
BE ->>+ BE: 發送重設密碼信件至會員信箱<br/>產生 Reset Password Ticket
BE ->>+ DataBase: 儲存 Reset Password Ticket
DataBase ->>+ BE: 
BE ->>+ FE: 回傳發送的信箱位置
FE ->>+ User: 提示會員去收信

User ->>+ FE: 點選信中的重設密碼連結
FE ->>+ BE: 帶上重設密碼連結中的 Reset Password Ticket<br/>/api/1.0/auth/check_reset_password_token
BE ->>+ DataBase: 確認 Reset Password Ticket 是否存在
DataBase ->>+ BE: 
BE ->>+ FE: 回傳 Reset Password Ticket 是否仍有效

alt Reset Password Ticket 已失效
  FE ->>+ User: 提示連結失效，請重新申請重設密碼
else 仍有效
  FE ->>+ User: 顯示重設密碼畫面
  User ->>+ FE: 輸入新密碼
  FE ->>+ BE: 加上重設密碼連結中的 Reset Password Ticket<br/>/api/1.0/auth/change_password
end

BE ->>+ DataBase: 以 Reset Password Ticket 取回帳號
DataBase ->>+ BE: 
BE ->>+ DataBase: 刪除 Reset Password Ticket<br/>更新帳號資料
BE ->>+ BE: 發重設密碼成功信至會員信箱
BE ->>+ FE: 回傳更改密碼成功
FE ->>+ User: 提示更改密碼成功
```

## Flow Chart

---