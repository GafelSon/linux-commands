# مدیریت و نگهداری سیستم‌های لینوکس
تمرین کلاسی درس آزمایشگاه سیستم عامل ۱

## درباره این پروژه
این مخزن شامل تمرینات و راهنماهای عملی برای یادگیری مدیریت پیشرفته سیستم‌های لینوکس است که در درس آزمایشگاه سیستم عامل پوشش داده می‌شود.

## فهرست مطالب
۱. [مدیریت سرویس‌ها با Systemd User Units](#۱--مدیریت-سرویسها-با-systemd-user-units)
۲. [کانتینرها بدون دسترسی Root](#۲--کانتینرها-بدون-دسترسی-root)
۳. [عیب‌یابی سریع سرویس‌ها](#۳--عیبیابی-سریع-سرویسها)
۴. [بررسی عملکرد سیستم](#۴--بررسی-عملکرد-سیستم)
۵. [مدیریت گواهی‌های TLS و HTTPS](#۵--مدیریت-گواهیهای-tls-و-https)
۶. [قابلیت‌های پیشرفته SSH](#۶--قابلیتهای-پیشرفته-ssh)
۷. [محدودسازی منابع و امن‌سازی](#۷--محدودسازی-منابع-و-امنسازی)
۸. [اشکال‌زدایی Core Dump](#۸--اشکالزدایی-core-dump)
۹. [بسته‌بندی نرم‌افزار](#۹--بستهبندی-نرمافزار)
۱۰. [شبیه‌سازی شبکه](#۱۰--شبیهسازی-شبکه)

---

## ۱ – مدیریت سرویس‌ها با Systemd User Units
[مشاهده راهنما](./01-Systemd%20User%20Units%20%26%20Sockets/README.md)

مدیریت سرویس‌ها بدون نیاز به دسترسی root با استفاده از systemd user units و socket activation.

**دستورات کلیدی:**
- `systemctl --user status` - بررسی وضعیت سرویس
- `systemctl --user start myapp` - راه‌اندازی سرویس
- `journalctl --user -f` - مشاهده لاگ‌ها

---

## ۲ – کانتینرها بدون دسترسی Root
[مشاهده راهنما](./02-Rootless%20Containers%20%28Podman-Buildah%29/README.md)

کار با کانتینرها با استفاده از Podman و Buildah بدون نیاز به دسترسی root.

**دستورات کلیدی:**
- `podman run -d nginx` - اجرای کانتینر
- `podman ps` - مشاهده کانتینرهای در حال اجرا
- `podman logs myapp` - مشاهده لاگ‌های کانتینر

---

## ۳ – عیب‌یابی سریع سرویس‌ها
[مشاهده راهنما](./03-Service%20Triage%20in%205%20Minutes/README.md)

روش‌های سیستماتیک برای شناسایی سریع مشکلات سرویس‌ها.

**دستورات کلیدی:**
- `pgrep -a myapp` - بررسی وضعیت پروسه
- `ss -ltnp` - بررسی پورت‌های باز
- `journalctl -u myapp -f` - مشاهده لاگ‌های سرویس

---

## ۴ – بررسی عملکرد سیستم
[مشاهده راهنما](./04-Performance%20Profiling%20%28perf-eBPF%29/README.md)

تحلیل عملکرد سیستم و شناسایی گلوگاه‌ها با استفاده از ابزارهای perf و eBPF.

**دستورات کلیدی:**
- `top` - مانیتورینگ مصرف منابع
- `free -h` - بررسی وضعیت حافظه
- `perf top` - پروفایلینگ CPU

---

## ۵ – مدیریت گواهی‌های TLS و HTTPS
[مشاهده راهنما](./05-TLS%20%26%20Certificates/README.md)

ایجاد و مدیریت گواهی‌های SSL/TLS برای محیط‌های توسعه و تولید.

**دستورات کلیدی:**
- `mkcert localhost` - ایجاد گواهی محلی
- `openssl x509 -in cert.pem -text` - بررسی گواهی
- `certbot --nginx` - دریافت گواهی Let's Encrypt

---

## ۶ – قابلیت‌های پیشرفته SSH
[مشاهده راهنما](./06-Advanced%20SSH/README.md)

کار با قابلیت‌های پیشرفته SSH شامل port forwarding، tunneling و key management.

**دستورات کلیدی:**
- `ssh user@host` - اتصال به سرور
- `scp file.txt host:/path/` - انتقال فایل
- `ssh -L 8080:localhost:80 host` - ایجاد تونل

---

## ۷ – محدودسازی منابع و امن‌سازی
[مشاهده راهنما](./07-Resource%20Limits%20%26%20Hardening/README.md)

محدود کردن مصرف منابع برنامه‌ها و افزایش امنیت سیستم.

**دستورات کلیدی:**
- `systemctl show -p MemoryMax myapp` - بررسی محدودیت حافظه
- `systemd-analyze security myapp` - تحلیل امنیتی سرویس
- `ulimit -a` - مشاهده محدودیت‌ها

---

## ۸ – اشکال‌زدایی Core Dump
[مشاهده راهنما](./08-Crash%20%26%20Core%20Dump%20Debugging/README.md)

تحلیل و رفع خرابی‌های برنامه با استفاده از core dumps.

**دستورات کلیدی:**
- `coredumpctl list` - لیست core dumpها
- `coredumpctl debug` - اشکال‌زدایی
- `ulimit -c unlimited` - فعال‌سازی core dump

---

## ۹ – بسته‌بندی نرم‌افزار
[مشاهده راهنما](./09-Reproducible%20Packaging/README.md)

ایجاد بسته‌های قابل نصب (.deb، .rpm) برای توزیع نرم‌افزار.

**دستورات کلیدی:**
- `fpm -s dir -t deb -n myapp` - ایجاد بسته
- `dpkg -i myapp.deb` - نصب بسته
- `apt remove myapp` - حذف بسته

---

## ۱۰ – شبیه‌سازی شبکه
[مشاهده راهنما](./10-Network%20Simulation%20Labs/README.md)

شبیه‌سازی شرایط مختلف شبکه برای تست برنامه‌ها.

**دستورات کلیدی:**
- `ip netns add myns` - ایجاد network namespace
- `tc qdisc add dev eth0 root netem delay 100ms` - شبیه‌سازی تاخیر
- `ip netns exec myns ping` - اجرای دستور در namespace

---

## ساختار پروژه

```
lpic-uni/
├── 01-Systemd User Units & Sockets/
├── 02-Rootless Containers (Podman-Buildah)/
├── 03-Service Triage in 5 Minutes/
├── 04-Performance Profiling (perf-eBPF)/
├── 05-TLS & Certificates/
├── 06-Advanced SSH/
├── 07-Resource Limits & Hardening/
├── 08-Crash & Core Dump Debugging/
├── 09-Reproducible Packaging/
└── 10-Network Simulation Labs/
```

---

## نصب ابزارهای مورد نیاز

### Arch Linux

```bash
# ابزارهای containerization
sudo pacman -S podman buildah

# ابزارهای performance profiling
sudo pacman -S perf

# ابزارهای TLS/SSL
sudo pacman -S mkcert certbot certbot-nginx

# ابزارهای SSH و شبکه
sudo pacman -S openssh iproute2 iputils

# ابزارهای debugging
sudo pacman -S gdb

# ابزارهای packaging
sudo pacman -S ruby rubygems gcc make
sudo gem install fpm

# ابزارهای monitoring
sudo pacman -S htop iotop iftop
```

---

## اطلاعات درس

- **نام درس:** آزمایشگاه سیستم عامل ۱
- **نوع پروژه:** تمرین کلاسی
- **محیط توسعه:** Arch Linux

---

## منابع و مراجع

- [Systemd Documentation](https://www.freedesktop.org/wiki/Software/systemd/)
- [Podman Documentation](https://docs.podman.io/)
- [Linux Performance](http://www.brendangregg.com/linuxperf.html)
- [OpenSSL Documentation](https://www.openssl.org/docs/)
