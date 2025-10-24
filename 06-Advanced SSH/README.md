# ۶) کارای باحال با SSH!
راهنمای کاربردی واسه اتصال امن، انتقال فایل و تونل‌سازی

---

## چرا مهمه؟

اکثر آدما فقط `ssh user@host` میزنن، ولی SSH خیلی بیشتره:
- **اتصال سریع** بدون اینکه هی آدرس بزنی
- **انتقال فایل امن**
- **تونل پورت** واسه دسترسی به سرویس‌های remote
- **اجرای دستور از راه دور**

---

## چند تا چیز ساده که باید بدونی

| مفهوم | مثال | یعنی چی |
|-------|------|--------|
| **Config فایل** | `~/.ssh/config` | تنظیمات ذخیره میشه |
| **کلید SSH** | `id_ed25519` | ورود بدون رمز |
| **Port Forwarding** | `-L 8080:localhost:80` | انتقال پورت |
| **ProxyJump** | `-J bastion` | از یه سرور رد شو برو اون یکی |

---

## ۱. ساخت کلید SSH

```bash
# بساز یه کلید (توصیه میشه)
ssh-keygen -t ed25519 -C "your_email@example.com"

# جاش اینجاست
~/.ssh/id_ed25519      # کلید خصوصی
~/.ssh/id_ed25519.pub  # کلید عمومی
```

**بفرستش به سرور:**

```bash
ssh-copy-id user@server
```

حالا دیگه رمز نمیخواد!

---

## ۲. فایل پیکربندی SSH

بساز فایل: `~/.ssh/config`

```sshconfig
# سرور اصلی
Host myserver
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# سرور محل کار
Host work
    HostName work.example.com
    User developer
    IdentityFile ~/.ssh/work_key
```

حالا به جای:
```bash
ssh admin@192.168.1.100
```

فقط بزن:
```bash
ssh myserver
```

---

## ۳. دستورات اصلی

### اتصال ساده

```bash
# اتصال عادی
ssh user@hostname

# با پورت خاص
ssh -p 2222 user@hostname

# با کلید خاص
ssh -i ~/.ssh/mykey user@hostname
```

### اجرای دستور از راه دور

```bash
# یه دستور
ssh myserver "ls -la"

# چندتا دستور
ssh myserver "cd /var/www && ls"

# ذخیره خروجی
ssh myserver "df -h" > disk-usage.txt
```

---

## ۴. انتقال فایل با scp

### کپی به سرور

```bash
# یه فایل
scp file.txt myserver:/home/user/

# چندتا فایل
scp file1.txt file2.txt myserver:/backup/

# یه پوشه (همه چیش)
scp -r mydir/ myserver:/home/user/
```

### کپی از سرور

```bash
# بگیر یه فایل
scp myserver:/path/to/file.txt ./

# بگیر یه پوشه
scp -r myserver:/var/www/html ./backup/
```

---

## ۵. همگام‌سازی با rsync (بهتر از scp)

```bash
# همگام‌سازی پوشه
rsync -avz ./local-dir/ myserver:/remote-dir/

# با نمایش پیشرفت
rsync -avz --progress ./files/ myserver:/backup/

# فایل‌های اضافی رو پاک کن
rsync -avz --delete ./src/ myserver:/dst/
```

**پارامترها:**
- `-a` = حالت آرشیو (مجوزها رو نگه میداره)
- `-v` = جزئیات نشون بده
- `-z` = فشرده کن

---

## ۶. Port Forwarding (تونل)

### Local Forward (دسترسی به سرویس remote)

```bash
# پورت ۸۰ سرور رو بیار روی ۸۰۸۰ لوکال
ssh -L 8080:localhost:80 myserver
```

حالا برو به: `http://localhost:8080`

**مثال واقعی:**
```bash
# دسترسی به دیتابیس remote
ssh -L 3306:localhost:3306 dbserver
mysql -h 127.0.0.1 -P 3306
```

### Remote Forward (اشتراک سرویس محلی)

```bash
# پورت ۳۰۰۰ محلی رو برو سرور در دسترس بذار
ssh -R 9000:localhost:3000 myserver
```

کاربرد: تست webhook یا نمایش پروژه به تیم

---

## ۷. اتصال از طریق Bastion (ProxyJump)

اگه باید از یه سرور رد شی تا به اون یکی برسی:

**روش قدیمی:**
```bash
ssh bastion
ssh internal-server
```

**روش جدید (یه مرحله):**
```bash
ssh -J bastion internal-server
```

**تو فایل config:**
```sshconfig
Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump bastion
```

بعد فقط:
```bash
ssh internal
```

---

## ۸. نگه داشتن اتصال

اضافه کن به `~/.ssh/config`:

```sshconfig
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

اینطوری اتصال قطع نمیشه.

---

## ۹. مشکل داری؟ بیا حلش کنیم

### Permission denied

```bash
# مجوزها رو درست کن
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# تست اتصال
ssh -v user@host
```

### کلید قبول نمیکنه

```bash
# کلید رو اضافه کن
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# لیست کلیدها
ssh-add -l
```

### اتصال خیلی کنده

```bash
# DNS lookup رو خاموش کن
echo "UseDNS no" | sudo tee -a /etc/ssh/sshd_config
sudo systemctl restart sshd
```

---

## ۱۰. امنیت SSH

### تنظیمات سرور (`/etc/ssh/sshd_config`)

```bash
# خاموش کن ورود با رمز
PasswordAuthentication no

# خاموش کن ورود root
PermitRootLogin no

# فقط با کلید SSH
PubkeyAuthentication yes
```

بعد از تغییر:
```bash
sudo systemctl restart sshd
```

### عوض کن پورت SSH

```bash
# ویرایش کن
sudo nano /etc/ssh/sshd_config

# تغییر بده خط
Port 2222

# ریستارت
sudo systemctl restart sshd
```

اتصال:
```bash
ssh -p 2222 user@host
```

---

## چند تا نکته مهم

 **همیشه از کلید SSH استفاده کن، رمز نه**

 **از فایل config استفاده کن واسه سرعت**

 **rsync بهتر از scp هست**

 **کلید خصوصی رو share نکن**

 **مجوزهای پوشه .ssh مهمه (700)**

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `ssh user@host` | وصل شو |
| `ssh-keygen -t ed25519` | بساز کلید |
| `ssh-copy-id user@host` | کلید رو بفرست |
| `scp file.txt host:/path/` | کپی فایل |
| `rsync -avz dir/ host:/dir/` | همگام‌سازی |
| `ssh -L 8080:localhost:80 host` | تونل محلی |
| `ssh -R 9000:localhost:3000 host` | تونل معکوس |
| `ssh -J bastion server` | رد شو از bastion |
| `ssh host "command"` | اجرا دستور |
