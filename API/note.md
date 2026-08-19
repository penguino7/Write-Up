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
