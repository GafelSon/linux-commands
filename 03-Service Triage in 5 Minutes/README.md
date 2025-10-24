# ۳) تو ۵ دقیقه بفهم سرویست چش شده!
برنامه کار نمیکنه؟ یاد بگیر سریع مشکل رو پیدا کنی

---

## چرا مهمه؟

خب فرض کن سرویست کرش کرده، کار نمیکنه، یا کنده. باید سریع بفهمی مشکل از کجاست:
- **برنامه خودش** مشکل داره? (کرش کرده، گیر کرده، حلقه افتاده)
- **شبکه** مشکل داره؟ (پورت بسته، اتصالات زیاد)
- یا **سیستم** مشکل داره؟ (رم تموم شده، CPU نداری، دیسک پره)

با این ابزارا تو ترمینال میتونی سریع بفهمی.

---

## نقشه راه - کدوم ابزار واسه چیه؟

| موضوع | میخوای چی بفهمی | دستوراتش |
|-------|------|----------------|
| پروسه | چی داره کار میکنه | `pgrep`, `ps`, `top` |
| شبکه | پورتا باز شدن یا نه | `ss`, `curl`, `netstat` |
| لاگ | خطاها چین | `journalctl`, `tail -f` |
| سیستم | منابع کافیه یا نه | `free`, `df`, `vmstat` |

---

## ۱. پیدا کردن برنامه

**برنامه رو پیدا کن:**

```bash
# پیداش کن
pgrep -a myapp

# جزئیاتش
ps -aux | grep myapp

# ببین کدوم برنامه پدرشه
pstree -ap $(pgrep -n myapp)
```

**اگه چندتا نمونه اجرا شده:**

```bash
ps -ef | grep myapp | grep -v grep
```

---

## ۲. چک کردن شبکه و پورتا

**ببین پورت باز شده:**

```bash
# پورتایی که گوش میدن
ss -ltnp | grep myapp

# اتصالات فعال
ss -tnp | grep myapp

# کی از پورت ۸۰۸۰ استفاده میکنه؟
sudo lsof -i :8080
```

**تست کن ببین جواب میده:**

```bash
# از localhost تست کن
curl -v http://localhost:8080

# فقط ببین جواب داد یا نه
curl -I http://localhost:8080
```

---

## ۳. خوندن لاگ‌ها

**واسه سرویس‌های systemd:**

```bash
# لاگ زنده ببین
journalctl -u myapp -f

# لاگ کاربر
journalctl --user -u myapp -f

# ۵۰ خط آخر
journalctl -u myapp -n 50

# لاگ امروز
journalctl -u myapp --since today
```

**فایل‌های لاگ معمولی:**

```bash
# زنده ببین
tail -f /var/log/myapp/app.log

# دنبال ERROR بگرد
grep ERROR /var/log/myapp/*.log

# ۱۰۰ خط آخر
tail -n 100 /var/log/myapp/app.log
```

---

## ۴. چک کردن منابع

**یه نگاه سریع:**

```bash
# CPU و رم
top

# بهتر از top
htop

# مرتب بر اساس رم
top -o %MEM
```

**چک کردن رم:**

```bash
# چقد رم داری
free -h

# چقد دیسک داری
df -h

# کدوم برنامه رم زیاد میخوره؟
ps aux --sort=-%mem | head
```

**چک کردن دیسک:**

```bash
# ببین دیسک داره چیکار میکنه
iostat -x 2

# کدوم برنامه دیسک میخونه؟
sudo iotop
```

---

## ۵. یه دستور تند که همه چیو چک کنه!

```bash
# همه چیو یه جا ببین
pgrep -a myapp && ss -ltnp | grep myapp && journalctl -u myapp -n 20
```

این یه خط بهت میگه:
- برنامه داره کار میکنه؟
- روی چه پورتی گوش میده؟
- آخرین لاگ‌ها چی میگن؟

---

## ۶. مشکل داری؟ بیا قدم به قدم حلش کنیم

### مشکل: سرویس اصلاً بالا نمیاد

```bash
# ۱. وضعیتش رو ببین
systemctl --user status myapp

# ۲. خطاها رو ببین
journalctl --user -u myapp -n 50

# ۳. دستی تستش کن
cd /path/to/app
./myapp
```

### مشکل: داره کار میکنه ولی جواب نمیده

```bash
# ۱. پورت باز شده؟
ss -ltnp | grep myapp

# ۲. تست کن
curl -v http://localhost:8080

# ۳. خطای شبکه ببین
journalctl -u myapp | grep -i error
```

### مشکل: خیلی کنده

```bash
# ۱. CPU چطوره
top -o %CPU

# ۲. رم چطوره
free -h

# ۳. دیسک چطوره
iostat -x 2
```

---

## ۷. ریستارت یا Kill کردن

**ریستارت عادی:**

```bash
systemctl --user restart myapp
```

**زور بیار killش کن:**

```bash
# با PID
kill $(pgrep myapp)

# زور زیاد!
kill -9 $(pgrep myapp)
```

**یه لحظه نگهش دار:**

```bash
# وایسا
kill -STOP $(pgrep myapp)

# برو بابا ادامه بده
kill -CONT $(pgrep myapp)
```

---

## ۸. چند تا ابزار مفید دیگه

```bash
# اتصالات شبکه
netstat -tulpn

# ببین کی چه فایلی باز کرده
lsof -p $(pgrep myapp)

# خطاهای کرنل
dmesg | tail -20

# نمودار منابع
vmstat 1 5
```

---

## چند تا نکته مهم

 **همیشه اول لاگ‌ها رو ببین**

 **از `journalctl -f` واسه دیدن زنده استفاده کن**

 **قبل از kill کردن، سعی کن بفهمی مشکل چیه**

 **`kill -9` آخرین کار باشه - اول restart معمولی امتحان کن**

---

## چک‌لیست عیب‌یابی سریع

```bash
# ۱. پروسه اوکیه؟
pgrep -a myapp

# ۲. پورت بازه؟
ss -ltnp | grep myapp

# ۳. لاگ چی میگه؟
journalctl -u myapp -n 20

# ۴. منابع کافیه؟
free -h && df -h

# ۵. تست مستقیم
curl http://localhost:8080
```

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `pgrep -a myapp` | پیداش کن |
| `ps aux \| grep myapp` | جزئیاتش رو ببین |
| `ss -ltnp \| grep myapp` | پورت رو چک کن |
| `journalctl -u myapp -f` | لاگ زنده |
| `top` | CPU و رم |
| `free -h` | چقد رم داری |
| `df -h` | چقد دیسک داری |
| `curl http://localhost:8080` | تستش کن |
| `kill $(pgrep myapp)` | خاموشش کن |
| `systemctl restart myapp` | ریستارت |
