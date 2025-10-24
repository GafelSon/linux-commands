# ۱) چطور برنامه‌هامون رو مث آدم اجرا کنیم
دیگه nohup و tmux کنار - با systemd راحت باش!

---

## چرا اینو یاد بگیرم؟

خب ببین، اگه تا الان برنامه‌ت رو با `nohup` یا داخل `tmux` اجرا میکردی، خبر خوب دارم برات! systemd همه این کارا رو خیلی بهتر انجام میده:
- برنامه کرش کرد؟ **خودش دوباره اجراش میکنه**
- لاگ‌ها؟ **همه یه جا جمعه** با `journalctl`
- میخوای رم یا CPU محدود کنی؟ **خیلی راحته**
- نیازی به root نداری!

---

## چند تا چیز ساده که باید بدونی

- **User unit:** یه سرویس که با یوزر خودت اجرا میشه (نه با root)
- **Linger:** حتی بعد از لاگ‌اوت هم برنامه کار میکنه
- **Socket:** یه پورت رو باز میکنه قبل از اینکه برنامه رو اجرا کنه

---

## ۱. اول systemd رو واسه یوزر خودت فعال کن

```bash
systemctl --user status
sudo loginctl enable-linger $USER
```

این کار میکنه که سرویس‌هات حتی بعد از ریستارت هم خودشون بالا بیان.

---

## ۲. بیا اولین سرویسمون رو بسازیم!

یه فایل بساز اینجا: `~/.config/systemd/user/myapp.service`

```ini
[Unit]
Description=برنامه من

[Service]
WorkingDirectory=%h/projects/myapp
ExecStart=/usr/bin/python3 server.py
Restart=always
RestartSec=2s

# بگو حداکثر چقد رم بخوره
MemoryMax=512M
CPUQuota=80%

# لاگ‌ها رو ذخیره کن
StandardOutput=journal
StandardError=inherit

[Install]
WantedBy=default.target
```

**حالا فعالش کن:**

```bash
systemctl --user daemon-reload
systemctl --user enable --now myapp.service
```

**لاگ‌هاش رو ببین:**

```bash
journalctl --user -u myapp -f
```

---

## دستوراتی که همش ازشون استفاده میکنی

### کارای اصلی
```bash
# روشنش کن
systemctl --user start myapp

# خاموشش کن
systemctl --user stop myapp

# ریستارتش کن
systemctl --user restart myapp

# ببین چخبره
systemctl --user status myapp
```

### لاگ‌ها رو چک کن
```bash
# لاگ زنده ببین
journalctl --user -u myapp -f

# ۵۰ خط آخر
journalctl --user -u myapp -n 50

# لاگ‌های امروز
journalctl --user -u myapp --since today
```

### خاموشش کن برای همیشه
```bash
systemctl --user disable --now myapp
```

---

## یه مثال واقعی: یه وب‌سرور ساده

**قدم اول:** یه فایل پایتون بساز (`~/projects/web/server.py`):

```python
from http.server import HTTPServer, SimpleHTTPRequestHandler

server = HTTPServer(('0.0.0.0', 8080), SimpleHTTPRequestHandler)
print("Server is running on 8080...")
server.serve_forever()
```

**قدم دوم:** فایل سرویس بساز (`~/.config/systemd/user/webserver.service`):

```ini
[Unit]
Description=وب‌سرور ساده من

[Service]
WorkingDirectory=%h/projects/web
ExecStart=/usr/bin/python3 server.py
Restart=always

[Install]
WantedBy=default.target
```

**قدم سوم:** بزن بریم!

```bash
systemctl --user daemon-reload
systemctl --user enable --now webserver
curl http://localhost:8080
```

---

## چند تا نکته خیلی مهم

 **هر وقت فایل service رو عوض کردی، حتماً `daemon-reload` بزن**

 **واسه سرویس‌های شخصیت همیشه `--user` رو یادت باشه**

 **میخوای بعد بوت خودش بیاد بالا؟ `enable` رو فراموش نکن**

 **اصلاً نیازی به `sudo` نیست!**

---

## مشکل پیش اومد؟ بیا حلش کنیم

اگه سرویس بالا نیومد:

```bash
# ببین خطا چیه
systemctl --user status myapp

# لاگ کامل رو ببین
journalctl --user -u myapp -n 100

# خود برنامه رو دستی تست کن
cd ~/projects/myapp
python3 server.py
```

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `systemctl --user status` | ببین کلاً چخبره |
| `systemctl --user start myapp` | روشنش کن |
| `systemctl --user stop myapp` | خاموشش کن |
| `systemctl --user restart myapp` | ریستارت بزن |
| `systemctl --user enable myapp` | خودکار بیاد بالا |
| `systemctl --user disable myapp` | دیگه خودکار نیاد بالا |
| `journalctl --user -u myapp -f` | لاگ زنده ببین |
| `systemctl --user daemon-reload` | فایل‌های تنظیمات رو دوباره بخون |
