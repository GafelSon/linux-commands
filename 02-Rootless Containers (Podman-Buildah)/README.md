# ۲) کار با کانتینر بدون داکر (و بدون root!)
داکر خوبه، ولی Podman بهتره چون root نمیخواد!

---

## چرا Podman؟

خب ببین، همه از داکر استفاده میکنن درسته، ولی یه مشکل بزرگ داره - باید root باشی! **Podman** اومده این مشکل رو حل کنه:
- **امن‌تره:** بدون daemon که root باشه
- **همون دستورات داکر:** فقط جای docker بنویس podman
- **با systemd کار میکنه:** خودکار بالا میاد

واسه کار روزمره، تست، یا حتی سرور مناسبه

---

## چند تا چیز ساده که باید بدونی

- **Rootless:** کانتینر با یوزر خودت اجرا میشه نه root
- **Podman:** جایگزین داکر، بدون daemon
- **Buildah:** واسه ساخت image
- **OCI:** همون استانداردی که داکر هم ازش استفاده میکنه

---

## ۱. اول نصبش کن

```bash
# Arch Linux
sudo pacman -S podman buildah

# Ubuntu / Debian
sudo apt install -y podman

# Fedora / RHEL
sudo dnf install -y podman
```

**ببین نصب شد:**

```bash
podman --version
podman info | grep rootless
```

باید بگه `rootless: true`

---

## ۲. اولین کانتینرت رو بساز!

**بزن nginx رو اجرا کنیم:**

```bash
podman run -d --name webserver -p 8080:80 nginx
```

**چک کن ببین اوکیه:**

```bash
# لیست کانتینرها
podman ps

# لاگ ببین
podman logs webserver

# تست کن
curl http://localhost:8080
```

**خاموشش کن و پاکش کن:**

```bash
podman stop webserver
podman rm webserver
```

---

## ۳. دستوراتی که همش باهاشون سر و کار داری

### کارای اصلی کانتینر

```bash
# بساز و اجرا کن
podman run -d --name myapp nginx

# خاموشش کن
podman stop myapp

# دوباره روشنش کن
podman start myapp

# پاکش کن
podman rm myapp

# زور بیار پاکش کن حتی اگه روشنه
podman rm -f myapp
```

### ببین چی داری

```bash
# لیست کانتینرها
podman ps

# همشون رو نشون بده (حتی خاموشا)
podman ps -a

# جزئیات
podman inspect myapp

# لاگ زنده
podman logs -f myapp

# ببین چقد رم و CPU میخوره
podman stats
```

### کار با imageها

```bash
# لیست imageها
podman images

# دانلود کن یه image
podman pull nginx:latest

# پاک کن
podman rmi nginx

# همه چیو پاک کن
podman system prune -a
```

---

## ۴. بریم یه برنامه پایتون اجرا کنیم

**قدم اول:** فایل برنامه (`app.py`):

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'سلام از Flask!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**قدم دوم:** Dockerfile بساز:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
CMD ["python", "app.py"]
```

**قدم سوم:** بساز و اجرا کن:

```bash
# image رو بساز
podman build -t myflask .

# اجراش کن
podman run -d --name flaskapp -p 5000:5000 myflask

# تستش کن
curl http://localhost:5000
```

---

## ۵. بندازش به systemd که خودکار بیاد بالا

**ساخت فایل systemd:**

```bash
# خودش یه فایل systemd میسازه واست
podman generate systemd --name flaskapp --files --new
```

**نصبش کن:**

```bash
mkdir -p ~/.config/systemd/user
mv container-flaskapp.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now container-flaskapp
```

**چکش کن:**

```bash
systemctl --user status container-flaskapp
journalctl --user -u container-flaskapp -f
```

حالا کانتینرت با بوت سیستم خودش میاد بالا!

---

## ۶. کار با Volume (ذخیره فایل)

```bash
# یه پوشه رو mount کن
podman run -d --name webapp \
  -v $(pwd)/html:/usr/share/nginx/html:Z \
  -p 8080:80 \
  nginx

# یه volume بساز
podman volume create mydata

# استفاده ازش
podman run -d -v mydata:/data myapp

# لیست volumeها
podman volume ls

# پاکش کن
podman volume rm mydata
```

**نکته:** اون `:Z` واسه SELinux لازمه.

---

## ۷. داکر بلدی؟ همونه!

| داکر | Podman |
|--------|--------|
| `docker run` | `podman run` |
| `docker ps` | `podman ps` |
| `docker build` | `podman build` |
| `docker pull` | `podman pull` |
| `docker logs` | `podman logs` |

تقریباً همه دستورا یکین!

---

## ۸. مشکل داری؟ بیا حلش کنیم

### کانتینر بالا نمیاد

```bash
# ببین خطا چیه
podman logs myapp

# خودت بهش وصل شو ببین مشکل چیه
podman run -it myimage /bin/sh
```

### پورت کار نمیکنه

```bash
# ببین پورت درسته
podman port myapp

# از داخل کانتینر تست کن
podman exec myapp curl localhost:80
```

### همه چیو پاک کن

```bash
# همه کانتینرها
podman rm -f $(podman ps -aq)

# همه imageها
podman rmi -a

# همه چی حتی volumeها
podman system prune -a --volumes
```

---

## چند تا نکته مهم

 **Podman daemon نداره - امن‌تر از داکره**

 **میتونی یه alias بسازی:**
```bash
alias docker=podman
```
حالا هر جا docker بزنی podman اجرا میشه!

 **واسه پورت‌های زیر ۱۰۲۴ یه سری تنظیم اضافه لازمه**

 **نیازی به sudo نداری!**

---

## خلاصه دستورات مهم

| دستور | کارش چیه |
|-------|---------|
| `podman run -d --name myapp image` | بساز و اجرا کن |
| `podman ps` | ببین چی داری |
| `podman logs -f myapp` | لاگ زنده |
| `podman stop/start myapp` | خاموش/روشن |
| `podman rm myapp` | پاک کن |
| `podman images` | لیست imageها |
| `podman build -t name .` | بساز image |
| `podman exec -it myapp bash` | برو تو کانتینر |
| `podman system prune -a` | پاک‌سازی کامل |
