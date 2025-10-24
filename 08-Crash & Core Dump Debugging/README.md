# ۸) برنامه کرش کرد؟ بیا ببینیم چش بود!
راهنمای ساده واسه پیدا کردن علت کرش

---

## چرا مهمه؟

برنامه کرش کرد و نمیدونی چرا؟ بدون core dump یا لاگ نمیتونی بفهمی چی شده. با یادگیری این:
- **علت دقیق** کرش رو میفهمی
- میتونی **خط کد مشکل‌دار** رو پیدا کنی
- **تکرار مشکل** آسون‌تر میشه

---

## چند تا چیز ساده که باید بدونی

| مفهوم | یعنی چی |
|-------|--------|
| **Core dump** | عکس حافظه برنامه وقتی کرش کرد |
| **Segmentation fault** | خطای دسترسی به حافظه |
| **coredumpctl** | ابزار systemd واسه مدیریت core dump |
| **gdb** | دیباگر واسه بررسی core dump |

---

## ۱. فعال کن Core Dump رو

### ببین فعاله یا نه

```bash
ulimit -c
```

اگه `0` بود، غیرفعاله.

### فعال کن موقتی

```bash
ulimit -c unlimited
```

### فعال کن دائمی

ویرایش کن: `/etc/security/limits.conf`

```
*  soft  core  unlimited
```

یا واسه یه سرویس systemd:

```ini
[Service]
LimitCORE=infinity
```

---

## ۲. کجا ذخیره میشه Core Dump؟

```bash
cat /proc/sys/kernel/core_pattern
```

**خروجی معمولی:**
```
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %e
```

یعنی systemd داره مدیریتش میکنه.

---

## ۳. ببین Core Dumpها رو با coredumpctl

### لیست کرش‌ها

```bash
coredumpctl list
```

خروجی مثال:
```
TIME                        PID  UID  GID SIG COREFILE EXE
Mon 2024-01-15 10:30:00    1234 1000 1000  11 present /usr/bin/python3
```

### جزئیات آخرین کرش

```bash
coredumpctl info
```

یا واسه PID خاص:

```bash
coredumpctl info 1234
```

### ببین لاگ

```bash
journalctl -t systemd-coredump
```

---

## ۴. عیب‌یابی با gdb

### باز کن آخرین core dump رو

```bash
coredumpctl debug
```

یا:

```bash
coredumpctl gdb
```

### دستورات مفید تو gdb

```gdb
# نشون بده stack trace
(gdb) bt

# stack trace کامل با متغیرها
(gdb) bt full

# ببین کد
(gdb) list

# ببین متغیرهای محلی
(gdb) info locals

# خروج
(gdb) quit
```

---

## ۵. مثال واقعی: بیا عمداً کرش کنیم!

### برنامه پایتون ساده که کرش میکنه

```python
# crash_test.py
import os

def buggy_function():
    print("قبل از کرش...")
    os.abort()  # باعث کرش میشه

buggy_function()
```

### اجرا و ببین کرش

```bash
# فعال کن core dump
ulimit -c unlimited

# اجرا
python3 crash_test.py
```

### چک کن کرش

```bash
# ببین آخرین کرش
coredumpctl list

# جزئیات
coredumpctl info

# لاگ
journalctl -t systemd-coredump -n 20
```

---

## ۶. ذخیره کن Core Dump رو واسه بعد

```bash
# استخراج کن core dump
coredumpctl dump -o myapp.core 1234

# یا آخرین کرش
coredumpctl dump -o crash.core

# فشرده کن
gzip crash.core
```

حالا میتونی نگهش داری یا بفرستی به تیم.

---

## ۷. تست کرش برنامه C

### برنامه ساده C

```c
// crash.c
#include <stdio.h>

int main() {
    int *p = NULL;
    printf("قبل از کرش...\n");
    *p = 42;  // Segmentation fault
    return 0;
}
```

### کامپایل و اجرا

```bash
gcc -g crash.c -o crash
ulimit -c unlimited
./crash
```

### عیب‌یابی

```bash
coredumpctl gdb crash
```

تو gdb:
```gdb
(gdb) bt
(gdb) list
```

---

## ۸. محدود کن تعداد Core Dumpها رو

systemd خودش core dumpها رو نگه میداره.

### پاک کن قدیمیا

```bash
# پاک کن قدیمی‌تر از ۳۰ روز
sudo journalctl --vacuum-time=30d

# پاک کن همه core dumpها
sudo rm -rf /var/lib/systemd/coredump/*
```

### محدود کن فضا

```bash
# حداکثر فضا (تو /etc/systemd/coredump.conf)
[Coredump]
MaxUse=1G
KeepFree=2G
```

---

## ۹. Core Dump واسه سرویس systemd

اضافه کن به service file:

```ini
[Service]
ExecStart=/usr/bin/myapp
LimitCORE=infinity
```

وقتی برنامه کرش کرد:

```bash
# ببین کرش
journalctl -u myapp | grep core

# چک کن با coredumpctl
coredumpctl list /usr/bin/myapp
coredumpctl debug /usr/bin/myapp
```

---

## ۱۰. عیب‌یابی برنامه‌های مختلف

### Python

```bash
# فعال کن fault handler
export PYTHONFAULTHANDLER=1
python3 myapp.py
```

خطاها تو stderr نشون داده میشن.

### Node.js

```bash
ulimit -c unlimited
node --abort-on-uncaught-exception server.js
```

### Go

```bash
export GOTRACEBACK=crash
./myapp
```

---

## چند تا نکته مهم

 **همیشه `ulimit -c unlimited` رو فعال کن**

 **از `coredumpctl` استفاده کن واسه مدیریت آسون**

 **Core dump میتونه خیلی بزرگ باشه - فضای کافی داشته باش**

 **Core dump حاوی اطلاعات حساس هست - احتیاط کن**

 **تو production، core dump رو محدود کن**

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `ulimit -c unlimited` | فعال کن core dump |
| `coredumpctl list` | لیست کرش‌ها |
| `coredumpctl info` | جزئیات آخرین کرش |
| `coredumpctl debug` | عیب‌یابی با gdb |
| `coredumpctl dump -o file` | استخراج core dump |
| `journalctl -t systemd-coredump` | لاگ کرش‌ها |
| `gdb program core` | باز کن دستی تو gdb |

---

## دستورات مفید gdb

| دستور | کارش چیه |
|-------|---------|
| `bt` | نشون بده stack trace |
| `bt full` | stack trace با جزئیات |
| `list` | نشون بده کد |
| `info locals` | نشون بده متغیرهای محلی |
| `frame N` | برو به frame شماره N |
| `print variable` | نشون بده مقدار متغیر |
| `quit` | خروج |
