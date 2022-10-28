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
User ->>+ FE: 輸入帳密
FE ->>+ BE: /auth/login
BE ->>+ BE: 檢查帳密<br/>檢查帳號狀態<br/>產生 Login Temp Ticket
BE ->>+ Redis: 儲存 Login Temp Ticket
Redis ->>+ BE: 
BE ->>+ DataBase: 更新帳號資料
DataBase ->>+ BE: 
BE ->>+ FE: 回傳 Login Temp Ticket<br/>NeedSetPassword<br/>LoginInPast

alt 首次登入，需要設定密碼
  FE ->>+ User: 顯示首次設定密碼畫面
  User ->>+ FE: 輸入密碼
  FE ->>+ BE: 加上 Login Temp Ticket<br/>/auth/set_password
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
  FE ->>+ BE: 加上  Login Temp Ticket<br/>/auth/login_check_verification_code
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


#### /auth/login
```mermaid
flowchart TD
A[被呼叫 /auth/login] --> B{是否有帳號資料}
B --> |否| C[回傳帳號或密碼錯誤] --> D[結束]
B --> |是| E{帳號狀態是是否黑名單或刪除}
E --> |是| F[回傳無權限] --> D
E --> |否| G{是否為首次登入}
G --> |是| H{是否有效日期後的72小時內}
H --> |否| I[回傳未在72小時內登入] --> D
H --> |是| J{驗證碼是否通過}
G --> |否| J
J --> |是| K[重設登入失敗次數<br/>建立 Login Temp TicIet<br/>儲存 Login Temp TicIet 且時效 10 分鐘]
K --> L[回傳登入成功] --> D
J --> |否| M[登入失敗次數 + 1]
M --> N{登入失敗 >= 5次}
N --> |否| C
N --> |是| O[將帳號設為黑名單<br/>新增帳號歷程記錄]
O --> P[回傳登入失敗] --> D
```

#### /auth/set_password
```mermaid
flowchart TD
A[被呼叫 /auth/set_password] --> B{Logoin Temp Teicket 是否仍有效且能依 Ticket 找回帳號}
B --> |否| C[回傳臨時 Token 已失效] --> D[結束]
B --> |是| E{帳號狀態是否為黑名單或刪除}
E --> |是| F[回傳無權限] --> L[刪除 Login Temp Ticket] --> D
E --> |否| G{驗證輸入密碼是否與原密碼相同}
G --> |是| H[回傳密碼不得與原密碼相同] --> D
G --> |否| I[設定新密碼<br/>新增帳號歷號記錄<br/>發首次設定密碼成功通知信至會員信箱]
I --> K[回傳成功訊息] --> L
```

#### /auth/login_check_verification_code
```mermaid
flowchart TD
A[被呼叫 /auth/login_check_verification_code] --> B{Logoin Temp Teicket 是否仍有效且能依 Ticket 找回帳號}
B --> |否| C[回傳臨時 Token 已失效] --> D[結束]
B --> |是| E{帳號狀態是否為黑名單或刪除}
E --> |是| F[回傳無權限] --> G[刪除 Login Temp Ticket] --> D
E --> |否| H{驗證輸入的驗證碼是否正確}
H --> |否| I[驗證錯誤次數 + 1]
I --> J{驗證失敗次數 >= 5}
J --> |否| K[回傳驗證失敗] --> D
J --> |是| L[將帳號設為黑名單<br/>新增帳號歷程記錄]
L --> M[回傳驗證錯誤達上限] --> D
H --> |是| N[重設驗證失敗次數<br/>記錄登入時間]
N --> O{帳號狀態為非啓用}
O --> |否| P[產生 jwt token<br/>刪除 Login Temp Ticket]
P --> R[回傳登入成功] --> D
O --> |是| S[將帳號狀態設為啓用<br/>新增帳號歷程記錄] --> P
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
FE ->>+ BE: 加上 Login Temp Ticket<br/>/auth/apply_reissue_qrcode
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
A[被呼叫 /auth/apply_reissue_qrcode]
A --> B{Logoin Temp Teicket 是否仍有效且能依 Ticket 找回帳號}
B --> |否| C[回傳臨時 Token 已失效] --> D[結束]
B --> |是| E[輸入的電話號是否相符]
E --> |否| H[新增帳號歷程記錄]
H --> I[寄補發失敗通知信]  --> D
E --> |是| J[取回原 Qrcode<br/>更新帳號資料<br/>新增帳號歷程記錄]
J --> K[寄送補發成功通知信] --> D
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
FE ->>+ BE: 加上 Login Temp Ticket<br/>/auth/apply_update_qrcode
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
A[被呼叫 /auth/apply_update_qrcode]
A --> B{Logoin Temp Teicket 是否仍有效且能依 Ticket 找回帳號}
B --> |否| C[回傳臨時 Token 已失效] --> D[結束]
B --> |是| G[輸入的電話號是否相符]
G --> |否| H[新增帳號歷程記錄]
H --> I[寄送更新失敗通知信]  --> D
G --> |是| J[產生新的 Qrcode<br/>更新帳號資料<br/>新增帳號歷程記錄]
J --> K[寄送更新成功通知信] --> D
```