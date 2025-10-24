# ۵) یه گواهی HTTPS بگیر واسه سایتت!
راهنمای ساده واسه کار با گواهی‌های امنیتی

---

## چرا مهمه؟

گواهی منقضی شده یا اشتباه یکی از مشکلات معمول سرورهاست. یاد بگیر:
- واسه **لوکال** گواهی بسازی که مرورگر قبول کنه
- گواهی‌های موجود رو **چک کنی**
- گواهی **رایگان** بگیری واسه سرورت
- **تمدید خودکار** تنظیم کنی

---

## چند تا چیز ساده که باید بدونی

| اصطلاح | یعنی چی |
|--------|------|
| **CA** | جایی که گواهی میده |
| **Certificate** | گواهی امنیتی واسه HTTPS |
| **Private Key** | کلید خصوصی (خیلی مهمه!) |
| **PEM/CRT** | فرمت فایل گواهی |
| **Let's Encrypt** | گواهی رایگان واسه سایت |

---

## ۱. گواهی محلی واسه توسعه (mkcert)

### نصبش کن

```bash
# Arch Linux
sudo pacman -S mkcert

# Ubuntu/Debian
sudo apt install mkcert libnss3-tools

# macOS
brew install mkcert
```

### یه CA محلی بساز که مرورگر قبولش کنه

```bash
mkcert -install
```

این یه CA محلی میسازه که مرورگرت بهش اعتماد میکنه.

### گواهی بساز واسه localhost

```bash
mkcert localhost 127.0.0.1 ::1
```

دوتا فایل میسازه:
```
localhost+2.pem       # گواهی
localhost+2-key.pem   # کلید خصوصی
```

### استفاده کن با پایتون

```bash
python3 -m http.server 8443 \
  --bind 127.0.0.1 \
  --certfile localhost+2.pem \
  --keyfile localhost+2-key.pem
```

حالا برو به: `https://localhost:8443`

 مرورگر قفل سبز نشون میده!

---

## ۲. بررسی گواهی با openssl

### اطلاعات گواهی رو ببین

```bash
openssl x509 -in cert.pem -text -noout
```

چیزای مهم:
```
Not Before: از کی اعتباره
Not After:  کی منقضی میشه
Subject:    واسه کدوم دامنه
```

### تاریخ انقضا رو ببین

```bash
openssl x509 -in cert.pem -noout -dates
```

### گواهی یه سایت رو چک کن

```bash
echo | openssl s_client -connect google.com:443 2>/dev/null | \
  openssl x509 -noout -dates
```

### فقط تاریخ انقضا

```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | \
  openssl x509 -noout -enddate
```

---

## ۳. گواهی رایگان واسه سرور (Let's Encrypt)

### نصب certbot

```bash
# Arch Linux
sudo pacman -S certbot certbot-nginx

# Ubuntu/Debian
sudo apt install certbot python3-certbot-nginx

# Fedora/RHEL
sudo dnf install certbot
```

### بگیر گواهی (با Nginx)

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot خودش همه کارا رو میکنه:
- گواهی میگیره
- Nginx رو تنظیم میکنه
- HTTPS رو فعال میکنه

### دستی بگیر (بدون وب‌سرور)

```bash
sudo certbot certonly --standalone -d api.example.com
```

گواهی‌ها اینجا ذخیره میشن:
```
/etc/letsencrypt/live/example.com/
├── cert.pem          # گواهی
├── privkey.pem       # کلید خصوصی
├── chain.pem         # زنجیره CA
└── fullchain.pem     # گواهی + زنجیره
```

---

## ۴. تمدید خودکار

### ببین تایمر هست یا نه

```bash
sudo systemctl list-timers | grep certbot
```

### تست کن ببین کار میکنه

```bash
sudo certbot renew --dry-run
```

اگه خطا نداد، یعنی تمدید خودکار کار میکنه!

### دستی تمدید کن

```bash
sudo certbot renew
```

---

## ۵. استفاده تو وب‌سرور

### Nginx

فایل تنظیمات رو ویرایش کن:

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
    }
}
```

Nginx رو ریستارت کن:

```bash
sudo systemctl restart nginx
```

---

## ۶. مشکل داری؟ بیا حلش کنیم

### چرا HTTPS کار نمیکنه؟

```bash
# ۱. گواهی معتبره؟
sudo certbot certificates

# ۲. فایل‌ها هستن؟
ls -la /etc/letsencrypt/live/example.com/

# ۳. تست کن
curl -vI https://example.com

# ۴. خطای SSL ببین
openssl s_client -connect example.com:443
```

### گواهی منقضی شده

```bash
# فوری تمدید کن
sudo certbot renew --force-renewal

# وب‌سرور رو ریستارت کن
sudo systemctl restart nginx
```

---

## ۷. مثال کامل: راه‌اندازی HTTPS

**قدم اول:** نصب Nginx و Certbot

```bash
sudo apt update
sudo apt install nginx certbot python3-certbot-nginx
```

**قدم دوم:** DNS رو تنظیم کن

```
A    example.com     -> IP سرورت
```

**قدم سوم:** بگیر گواهی

```bash
sudo certbot --nginx -d example.com
```

**قدم چهارم:** تست کن

```bash
curl https://example.com
```

 حالا سایتت HTTPS داره!

---

## ۸. نکات امنیتی

### کلید خصوصی رو محافظت کن

```bash
# فقط root بتونه بخونه
sudo chmod 600 /etc/letsencrypt/live/example.com/privkey.pem
```

### امنیت SSL رو چک کن

برو به: https://www.ssllabs.com/ssltest/

دامنه‌ت رو تست کن.

---

## چند تا نکته مهم

 **واسه لوکال از mkcert استفاده کن**

 **واسه سرور واقعی از Let's Encrypt (رایگان!)**

 **تمدید خودکار رو فراموش نکن**

 **کلید خصوصی رو هیچ وقت share نکن**

 **گواهی هر ۹۰ روز باید تمدید شه**

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `mkcert localhost` | گواهی محلی بساز |
| `openssl x509 -in cert.pem -text` | گواهی رو ببین |
| `certbot --nginx -d example.com` | بگیر گواهی |
| `certbot certificates` | لیست گواهی‌ها |
| `certbot renew` | تمدید کن |
| `certbot renew --dry-run` | تست تمدید |
| `openssl s_client -connect site:443` | تست HTTPS |
