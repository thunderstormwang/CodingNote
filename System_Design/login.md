# 登入流程

## 登入

### Sequence Diagram

```mermaid
sequenceDiagram
participant User
participant FE
participant BE
participant Redis
participant DataBase

User ->>+ FE: 點選登入
FE ->>+ User: 顯示登入畫面
User ->>+ FE: 輸入統編、帳密
FE ->>+ BE: /api/1.0/auth/login
BE ->>+ BE: 檢查帳密<br/>檢查帳號狀態<br/>產生 Login Temp Ticket
BE ->>+ Redis: 儲存 Login Temp Ticket
Redis ->>+ BE: 
BE ->>+ DataBase: 更新帳號資料
DataBase ->>+ BE: 
BE ->>+ FE: 回傳 Login Temp Ticket<br/>NeedSetPassword<br/>LoginInPast

alt 首次登入，需要設定密碼
  FE ->>+ User: 顯示首次設定密碼畫面
  User ->>+ FE: 輸入統編、帳密
  FE ->>+ BE: 加上 Login Temp Ticket<br/>/api/1.0/auth/set_password
  BE ->>+ Redis: 以 Login Temp Ticket 取回帳號
  Redis ->>+ BE: 
  BE ->>+ BE: 檢查新密碼不得與預設密碼相同<br/>檢查帳號狀態<br/>發通知信至會員信箱
  BE ->>+ DataBase: 刪除 Login Temp Ticket<br/>更新帳號資料
  DataBase ->>+ BE: 
  BE ->>+ FE: 回傳更改密碼成功
  FE ->>+ User: 導回登入頁
else 不是首次登入
  FE ->>+ User: 顯示輸入驗證碼畫面
  User ->>+ FE: 輸入 驗證碼
  FE ->>+ BE: 加上  Login Temp Ticket<br/>/api/1.0/auth/login_check_verification_code
  BE ->>+ Redis: 以 Login Temp Ticket 取回帳號
  Redis ->>+ BE: 
  BE ->>+ BE: 檢查驗證碼<br/>檢查帳號狀態<br/>產生 jwt token
  BE ->>+ Redis: 刪除 Login Temp Ticket
  Redis ->>+ BE: 
  BE ->>+ DataBase: 更新帳號資料
  DataBase ->>+ BE: 
  BE ->>+ FE: 回傳 jwt token
  FE ->>+ User: 顯示登入成功訊息
end
```

### Flow Chart

```mermaid
flowchart TD
A[登入畫面] --> B{是否有帳號資料} 
B --> |是| C{密碼是否正確}
C --> |是| D{是否為預設系統密碼}
D --> |否| D1[驗整碼頁]
B --> |否| Z[無法登入<br/>提示請與管理員聯繫] --> Z4[結束]
C --> |否| M{登入失敗 >= 5次}
M --> |是| N[密碼錯誤提示] --> Z4[結束]
M --> |否| A 
D1 --> D2{登入臨時token是否失效}
D2 --> |是| D3[提示登入逾時請重新登入流程]--> Z4[結束]
D2 --> |否| E{驗證碼是否通過}
E --> |否| O{驗證失敗 >= 5次}
O --> |是| P[無法登入<br/>提示請與管理員聯繫] --> Z4[結束]
E --> |是| F[驗證成功<br/>驗證錯誤次數歸零]
F --> G[登入成功<br/>登入錯誤次數歸零<br/>紀錄登入時間<br/>回事件標示啟用]
G --> Y[SCM後台首頁] --> Z4[結束]
D --> |是| I{是否有效日期後的72小時內}
I --> |是| H[設定密碼頁]
H --> J[更新新密碼]
J --> L[寄信通知更新成功] --> Z4[結束]
I --> |否| K[提示請與管理員聯繫] --> Z4[結束]
```

---

## 申請重發 QrCpoe

### Sequence Diagram

```mermaid
sequenceDiagram
participant User
participant FE
participant BE
participant Redis
participant DataBase

User ->>+ FE: 在輸入驗證碼畫面點選申請補發 Qrcode
FE ->>+ User: 顯示申請補發 Qrcode 畫面
User ->>+ FE: 輸入統編、帳號、手機
FE ->>+ BE: 加上 Login Temp Ticket<br/>/api/1.0/auth/apply_reissue_qrcode
BE ->>+ Redis: 以 Login Temp Ticket 取回帳號
Redis ->>+ BE: 
BE ->>+ BE: 檢查手機號碼是否與原手機相同<br/>檢查帳號狀態<br/>取回原 Qrcode<br/>發重發 Qrcode 信至會員信箱
BE ->>+ DataBase: 更新帳號資料
DataBase ->>+ BE: 
BE ->>+ FE: 回傳申請補發 Qrcode 成功
FE ->>+ User: 提示申請補發成功並導回輸入驗證碼畫面
User ->>+ User: 收信掃 Qrcode
User ->>+ FE: 輸入 驗證碼
Note over FE, BE: 如上登入流程
```

### Flow Chart

```mermaid
flowchart TD
A[QR Code 驗證問題頁]
A --> B{是否有登入成功過}
B --> |否| C(無申請按鈕) --> Z[結束]
B --> |是| D(有申請按鈕)
D --> E[補發申請頁面]
E --> E1{聯絡人電話是否正確}
E1 --> |是| E2(寄申請成功信\n含補發QrCode) --> Z[結束]
E1 --> |否| E3(寄補發失敗通知信) --> Z[結束]
```

---

## 申請更新 QrCpoe

### Sequence Diagram

```mermaid
sequenceDiagram
participant User
participant FE
participant BE
participant Redis
participant DataBase

User ->>+ FE: 在輸入驗證碼畫面點選申請更新 Qrcode
FE ->>+ User: 顯示申請更新 Qrcode 畫面
User ->>+ FE: 輸入統編、帳號、手機
FE ->>+ BE: 加上 Login Temp Ticket<br/>/api/1.0/auth/apply_update_qrcode
BE ->>+ Redis: 以 Login Temp Ticket 取回帳號
Redis ->>+ BE: 
BE ->>+ BE: 檢查手機號碼是否與原手機相同<br/>檢查帳號狀態<br/>產生新 Qrcode<br/>發更新 Qrcode 信至會員信箱
BE ->>+ DataBase: 更新帳號資料
DataBase ->>+ BE: 
BE ->>+ FE: 回傳申請更新 Qrcode 成功
FE ->>+ User: 提示申請更新成功並導回輸入驗證碼畫面
User ->>+ User: 收信掃 Qrcode
User ->>+ FE: 輸入 驗證碼
Note over FE, BE: 如上登入流程
```

### Flow Chart

```mermaid
flowchart TD
A[QR Code 驗證問題頁]
A --> B{是否有登入成功過}
B --> |否| C[無申請按鈕] --> Z[結束]
B --> |是| D[有申請按鈕]
D --> F[更新申請頁面]
F --> F1{聯絡人電話是否正確}
F1 --> |是| F2[寄申請成功信<br/>含更新QrCode] --> Z[結束]
F1 --> |否| F3[寄更新失敗通知信] --> Z[結束]
```