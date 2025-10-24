# ۷) محدود کن برنامه‌ها رو که سیستم رو خراب نکنن!
یاد بگیر چطور CPU و رم رو کنترل کنی

---

## چرا مهمه؟

یه برنامه باگ داره و تموم رم رو میخوره؟ یا CPU رو صد درصد گرفته؟ با محدود کردن منابع:
- سیستم **پایدارتر** میشه
- برنامه‌های خراب **بقیه رو خراب نمیکنن**
- **امنیت بیشتر** در برابر حملات

---

## چند تا چیز ساده که باید بدونی

| مفهوم | یعنی چی |
|-------|--------|
| **cgroups** | مکانیزم کرنل واسه محدود کردن منابع |
| **systemd-run** | اجرای پروسه با محدودیت |
| **ulimit** | محدودیت منابع تو shell |
| **MemoryMax** | حداکثر رم مجاز |
| **CPUQuota** | درصد CPU مجاز |

---

## ۱. تست سریع با systemd-run

بزن برنامه رو با محدودیت موقت اجرا کن:

```bash
systemd-run --user --unit=test-app \
  -p MemoryMax=512M \
  -p CPUQuota=50% \
  python3 myapp.py
```

**ببین چطوره:**

```bash
systemctl --user status test-app
```

**ببین چقد میخوره:**

```bash
systemctl --user show test-app -p MemoryCurrent
```

**خاموشش کن:**

```bash
systemctl --user stop test-app
```

---

## ۲. محدودیت تو فایل سرویس

ویرایش کن: `~/.config/systemd/user/myapp.service`

```ini
[Service]
ExecStart=/usr/bin/python3 /home/user/app/server.py

# محدودیت رم
MemoryMax=512M
MemoryHigh=400M

# محدودیت CPU (50% از یه هسته)
CPUQuota=50%

# حداکثر تعداد thread
TasksMax=100

# محدودیت دیسک
IOWeight=100

# حداکثر فایل‌های باز
LimitNOFILE=1024

[Install]
WantedBy=default.target
```

**اعمال کن:**

```bash
systemctl --user daemon-reload
systemctl --user restart myapp
```

---

## ۳. توضیح محدودیت‌ها

### رم

```ini
MemoryMax=512M      # بیشتر از این = kill میشه
MemoryHigh=400M     # هشدار (فشار میاد)
```

### CPU

```ini
CPUQuota=50%        # نصف یه هسته
CPUQuota=200%       # دوتا هسته کامل
```

### Thread و پروسه

```ini
TasksMax=100        # حداکثر ۱۰۰ تا thread
```

### دیسک

```ini
IOWeight=100        # اولویت دیسک (1-1000)
```

---

## ۴. محدودیت با ulimit

واسه session فعلی shell:

```bash
# حداکثر رم (KB)
ulimit -v 524288    # 512MB

# حداکثر فایل باز
ulimit -n 1024

# ببین همه محدودیت‌ها
ulimit -a
```

**دائمیش کن:** ویرایش کن `/etc/security/limits.conf`

```
username  soft  nproc   100
username  hard  nproc   200
username  soft  nofile  1024
username  hard  nofile  2048
```

---

## ۵. تست محدودیت رم

برنامه تست که رم میخوره:

```python
# memory_hog.py
data = []
while True:
    data.append(' ' * 10**6)  # 1MB
    print(f"خورد: {len(data)} MB")
```

اجرا با محدودیت:

```bash
systemd-run --user -p MemoryMax=100M python3 memory_hog.py
```

وقتی به ۱۰۰MB برسه، خودش killش میکنه.

---

## ۶. ببین محدودیت‌ها چین

```bash
# محدودیت‌های سرویس
systemctl show myapp -p MemoryMax -p CPUQuota

# مصرف فعلی
systemctl status myapp

# جزئیات cgroup
cat /sys/fs/cgroup/user.slice/user-1000.slice/user@1000.service/app.slice/myapp.service/memory.max
```

---

## ۷. امنیت بیشتر (Sandboxing)

اضافه کن به service file:

```ini
[Service]
# نمیتونه مجوز بیشتر بگیره
NoNewPrivileges=yes

# فقط خواندن /usr و /etc
ProtectSystem=strict

# مخفی کن /home
ProtectHome=yes

# /tmp جداگانه
PrivateTmp=yes

# مخفی کن /dev
PrivateDevices=yes

# محدود کن شبکه
RestrictAddressFamilies=AF_INET AF_UNIX
```

**چک کن امنیت:**

```bash
systemd-analyze security myapp.service
```

نمره امنیتی میده (0-10، پایین‌تر بهتر)

---

## ۸. مثال واقعی: وب‌سرور محدود شده

```ini
[Unit]
Description=وب‌سرور محدود شده

[Service]
ExecStart=/usr/bin/python3 -m http.server 8080
WorkingDirectory=/var/www/html

# محدودیت منابع
MemoryMax=256M
CPUQuota=50%
TasksMax=50

# امنیت
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
ReadWritePaths=/var/www/html

[Install]
WantedBy=default.target
```

---

## ۹. مشکل داری؟ بیا حلش کنیم

### سرویس kill میشه

```bash
# ببین چرا
journalctl -u myapp | grep -i oom

# افزایش بده محدودیت
systemctl edit myapp

# اضافه کن:
[Service]
MemoryMax=1G
```

### CPU کم میاره

```bash
# ببین چقد میخوره
systemctl status myapp

# افزایش بده CPUQuota
CPUQuota=100%  # یه هسته کامل
```

---

## ۱۰. نظارت بر مصرف

```bash
# نمایش زنده
watch 'systemctl status myapp'

# مصرف دقیق
systemd-cgtop

# یا
systemctl show myapp -p MemoryCurrent -p CPUUsageNSec
```

---

## چند تا نکته مهم

 **شروع کن با محدودیت‌های بالا، بعد کم کن**

 **محدودیت‌ها رو تو محیط تست امتحان کن**

 **از `systemd-analyze security` استفاده کن**

 **محدودیت خیلی پایین ممکنه برنامه رو خراب کنه**

 **OOM killer برنامه رو بدون هشدار میکشه**

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `systemd-run --user -p MemoryMax=512M cmd` | اجرا با محدودیت موقت |
| `systemctl show myapp -p MemoryMax` | ببین محدودیت چیه |
| `systemctl status myapp` | ببین چقد میخوره |
| `systemd-analyze security myapp` | چک کن امنیت |
| `ulimit -a` | محدودیت‌های shell |
| `systemd-cgtop` | نمایش زنده cgroup |
| `journalctl -u myapp \| grep OOM` | ببین OOM kill شده یا نه |
