# Security Fixes — Supriya Digitals

## 1. Move credentials OUT of web root (CRITICAL)

Create `/volume1/private/config.php` (outside webroot):
```php
<?php
define('DB_HOST',     'localhost');
define('DB_PORT',     '3306');
define('DB_NAME',     'studio_db');
define('DB_USER',     'studio_user');      // NOT root
define('DB_PASS',     'your-strong-password');
define('ADMIN_TOKEN', 'your-new-secret-token');
```

Then in `db_config.php`, replace the hardcoded values with:
```php
require_once '/volume1/private/config.php';
$servername = DB_HOST . ':' . DB_PORT;
$username   = DB_USER;
$password   = DB_PASS;
$dbname     = DB_NAME;
```

## 2. Create a dedicated MySQL user (not root)
```sql
CREATE USER 'studio_user'@'localhost' IDENTIFIED BY 'strong-password-here';
GRANT SELECT, INSERT, UPDATE, DELETE ON studio_db.* TO 'studio_user'@'localhost';
FLUSH PRIVILEGES;
```

## 3. download.php — use the fixed version
Replace `download.php` with `download_fixed.php`. Key fixes:
- Added `require_once 'db_config.php'` + `verifyAdminToken()`
- Fixed double php://input read bug (now uses $GLOBALS['_RAW_INPUT'])

## 4. stream_zip.php — validate paths against DB
Before streaming, confirm the requested file path exists in `media_items` table.
This prevents file path enumeration attacks.

## 5. Admin token in URL params
Remove `admin_token` from GET params in admin.html. 
Send it only via POST JSON body or `Authorization: Bearer TOKEN` header.
