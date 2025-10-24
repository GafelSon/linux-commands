# ۴) بفهم چی سیستمت رو کند کرده!
چرا کنده؟ کی رم میخوره؟ اینجا میفهمی!

---

## چرا مهمه؟

برنامه‌ت کنده ولی کرش نمیکنه؟ حدس زدن فایده نداره! با این ابزارا دقیقاً میفهمی:
- **کدوم تابع** زمان زیاد میبره
- **CPU** یا **رم** کجا تموم میشه
- **کدوم فراخوانی** کنده

---

## چند تا چیز ساده که باید بدونی

| ابزار | کارش چیه |
|-------|---------|
| `top` | CPU و رم رو نشون میده |
| `htop` | نسخه بهتر top |
| `free` | چقد رم آزاد داری |
| `iostat` | دیسک چطوره |
| `perf` | پروفایلینگ حرفه‌ای CPU |

---

## ۱. ابزارای ساده (از اینجا شروع کن)

### CPU و رم رو ببین

```bash
# نگاه کلی
top

# نسخه بهتر (باید نصب کنی)
htop

# مرتب بر اساس رم
top -o %MEM

# مرتب بر اساس CPU
top -o %CPU
```

**کلیدهای مفید تو top:**
- `P` - مرتب بر اساس CPU
- `M` - مرتب بر اساس رم
- `k` - برنامه رو بکش
- `q` - بیرون

### چک کردن رم

```bash
# چقد رم داری
free -h

# جزئیات بیشتر
cat /proc/meminfo | head -20

# کی رم زیاد میخوره؟
ps aux --sort=-%mem | head -10
```

### چک کردن دیسک

```bash
# چقد جا داری
df -h

# کدوم پوشه حجیمه
du -sh * | sort -h

# دیسک چقد کار میکنه
iostat -x 2
```

---

## ۲. پیدا کردن برنامه‌ای که زیاد میخوره

```bash
# بالاترین CPU
ps aux --sort=-%cpu | head -10

# بالاترین رم
ps aux --sort=-%mem | head -10

# زنده ببین
watch 'ps aux --sort=-%cpu | head -10'
```

---

## ۳. ابزار perf (یکم پیشرفته‌تره)

### نصبش کن

```bash
# Arch Linux
sudo pacman -S perf

# Ubuntu/Debian
sudo apt install linux-tools-common linux-tools-$(uname -r)

# Fedora/RHEL
sudo dnf install perf
```

### استفاده ساده

```bash
# توابع پرمصرف رو زنده ببین
sudo perf top

# آمار سریع واسه ۱۰ ثانیه
sudo perf stat -p $(pgrep myapp) sleep 10
```

خروجی چیزایی نشون میده مثل:
- تعویض context (زیاد باشه = مشکل threading یا I/O)
- دستورالعمل‌ها
- Cache miss

---

## ۴. چک کردن فشار شبکه

```bash
# اتصالات فعال
ss -s

# چند نفر به پورت ۸۰۸۰ وصلن
ss -tn | grep :8080 | wc -l

# ترافیک شبکه (باید نصب کنی)
sudo iftop -i eth0
```

---

## ۵. لاگ‌های سیستم رو ببین

```bash
# خطاهای کرنل
dmesg | tail -50

# ببین رم تموم شده یا نه (OOM)
dmesg | grep -i oom

# خطاهای اخیر
journalctl -p err -n 50
```

---

## ۶. یه چک سریع کامل!

یه اسکریپت ساده که همه چیو چک میکنه:

```bash
#!/bin/bash
echo "=== CPU ==="
ps aux --sort=-%cpu | head -5

echo -e "\n=== رم ==="
free -h

echo -e "\n=== دیسک ==="
df -h | grep -v tmpfs

echo -e "\n=== بار سیستم ==="
uptime

echo -e "\n=== شبکه ==="
ss -s
```

ذخیرش کن به عنوان `check-system.sh` و اجراش کن:

```bash
chmod +x check-system.sh
./check-system.sh
```

---

## ۷. مشکلات معمول و راه حل

### برنامه کنده

```bash
# ۱. CPU کمه؟
top -p $(pgrep myapp)

# ۲. رم کمه؟
free -h

# ۳. دیسک کنده؟
iostat -x 2

# ۴. شبکه مشکل داره؟
ping -c 5 8.8.8.8
```

### رم داره تموم میشه

```bash
# کی میخوره؟
ps aux --sort=-%mem | head -5

# رم leak چک کن
watch 'ps -p $(pgrep myapp) -o %mem,rss'

# ببین کرنل کسی رو نکشته (OOM killer)
dmesg | grep -i "out of memory"
```

### CPU صد درصده

```bash
# کدوم پروسه؟
top -o %CPU

# جزئیات بیشتر
ps -p $(pgrep myapp) -o %cpu,cmd

# اگه یه هسته پره
htop  # و بررسی کن
```

---

## ۸. نمایش زنده

```bash
# CPU زنده
watch -n 1 'ps aux --sort=-%cpu | head -10'

# رم زنده
watch -n 2 'free -h'

# اتصالات زنده
watch -n 1 'ss -s'
```

---

## چند تا نکته مهم

 **همیشه اول `top` و `free -h` رو بزن**

 **از `htop` استفاده کن راحت‌تره**

 **دیسک پر شد؟ با `du -sh *` پیداش کن**

 **قبل از `perf`، اول ابزارای ساده رو امتحان کن**

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `top` | نگاه کلی به منابع |
| `htop` | نسخه بهتر top |
| `free -h` | وضعیت رم |
| `df -h` | فضای دیسک |
| `iostat -x 2` | چک کردن دیسک |
| `ps aux --sort=-%cpu` | پرمصرف‌های CPU |
| `ps aux --sort=-%mem` | پرمصرف‌های رم |
| `sudo perf top` | توابع پرمصرف |
| `dmesg \| tail -50` | خطاهای کرنل |
| `uptime` | بار سیستم |
