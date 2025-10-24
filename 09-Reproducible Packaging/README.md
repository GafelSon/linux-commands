# ۹) بسته‌بندی برنامه واسه توزیع آسون!
یاد بگیر چطور یه بسته نصبی درست کنی

---

## چرا مهمه؟

نصب دستی برنامه روی چندین سرور خسته‌کننده‌ست. با بسته‌بندی:
- **نصب آسون** با یه دستور
- **حذف تمیز** بدون فایل باقی‌مونده
- **نسخه‌گذاری** و **ردیابی** تغییرات
- **توزیع سریع** روی چندتا سرور

---

## چند تا چیز ساده که باید بدونی

| مفهوم | یعنی چی |
|-------|--------|
| **Package** | فایل نصبی (.deb یا .rpm) |
| **.deb** | فرمت بسته واسه Ubuntu/Debian |
| **.rpm** | فرمت بسته واسه Fedora/RHEL |
| **fpm** | ابزار ساده واسه ساخت بسته |
| **Version** | شماره نسخه (مثلاً 1.0.0) |

---

## ۱. نصب کن fpm رو

```bash
# Arch Linux
sudo pacman -S ruby rubygems gcc make

# Ubuntu/Debian
sudo apt install ruby ruby-dev gcc make

# نصب fpm
sudo gem install fpm
```

**چک کن:**

```bash
fpm --version
```

---

## ۲. بساز اولین بسته رو!

فرض کن یه اسکریپت ساده داری:

### قدم اول: بساز ساختار پوشه

```bash
mkdir -p myapp/usr/local/bin
echo '#!/bin/bash' > myapp/usr/local/bin/myapp
echo 'echo "سلام از برنامه من!"' >> myapp/usr/local/bin/myapp
chmod +x myapp/usr/local/bin/myapp
```

### قدم دوم: بساز بسته .deb

```bash
fpm -s dir -t deb \
  -n myapp \
  -v 1.0.0 \
  --description "برنامه ساده من" \
  -C myapp \
  .
```

**خروجی:** `myapp_1.0.0_amd64.deb`

### قدم سوم: نصبش کن

```bash
sudo dpkg -i myapp_1.0.0_amd64.deb
```

### قدم چهارم: تستش کن

```bash
myapp
```

خروجی: `سلام از برنامه من!`

---

## ۳. حذف بسته

```bash
# حذف کن
sudo apt remove myapp

# یا
sudo dpkg -r myapp
```

---

## ۴. بسته‌بندی برنامه پایتون

### ساختار پروژه

```
myproject/
├── app/
│   ├── __init__.py
│   └── server.py
├── requirements.txt
└── start.sh
```

### اسکریپت start.sh

```bash
#!/bin/bash
cd /opt/myproject
python3 -m app.server
```

### بساز بسته

```bash
fpm -s dir -t deb \
  -n myproject \
  -v 1.0.0 \
  --description "پروژه پایتون من" \
  --prefix /opt/myproject \
  --depends python3 \
  --depends python3-pip \
  app=/opt/myproject/app \
  requirements.txt=/opt/myproject/ \
  start.sh=/usr/local/bin/myproject
```

---

## ۵. اضافه کن اسکریپت‌های نصب

### اسکریپت بعد از نصب (postinstall.sh)

```bash
#!/bin/bash
echo "داره نصب میکنه وابستگی‌ها..."
pip3 install -r /opt/myproject/requirements.txt
echo "نصب تموم شد!"
```

### اسکریپت قبل از حذف (preremove.sh)

```bash
#!/bin/bash
echo "داره پاک‌سازی میکنه..."
```

### استفاده تو fpm

```bash
fpm -s dir -t deb \
  -n myproject \
  -v 1.0.0 \
  --after-install postinstall.sh \
  --before-remove preremove.sh \
  ...
```

---

## ۶. بسته با systemd

### قدم اول: بساز service file

```ini
# myapp.service
[Unit]
Description=برنامه من

[Service]
ExecStart=/opt/myapp/start.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

### قدم دوم: بسته‌بندی

```bash
fpm -s dir -t deb \
  -n myapp \
  -v 1.0.0 \
  --deb-systemd myapp.service \
  app=/opt/myapp/
```

بسته خودش سرویس رو ثبت میکنه!

---

## ۷. بساز بسته RPM (واسه Fedora/RHEL)

همون دستورات، فقط `-t rpm`:

```bash
fpm -s dir -t rpm \
  -n myapp \
  -v 1.0.0 \
  --description "برنامه من" \
  -C myapp \
  .
```

**نصب RPM:**

```bash
sudo rpm -ivh myapp-1.0.0-1.x86_64.rpm
```

**حذف:**

```bash
sudo rpm -e myapp
```

---

## ۸. نسخه‌گذاری درست

### استفاده از Git tag

```bash
# بساز یه برچسب نسخه
git tag v1.0.0
git push --tags

# استفاده تو ساخت بسته
VERSION=$(git describe --tags --always)
fpm -s dir -t deb -n myapp -v $VERSION ...
```

### Semantic Versioning

```
1.0.0  →  اولین نسخه
1.0.1  →  رفع باگ
1.1.0  →  قابلیت جدید
2.0.0  →  تغییر بزرگ (breaking change)
```

---

## ۹. مدیریت وابستگی‌ها

```bash
fpm -s dir -t deb \
  -n myapp \
  --depends python3 \
  --depends python3-pip \
  --depends nginx \
  ...
```

نصب بسته، وابستگی‌ها رو هم نصب میکنه.

---

## ۱۰. ببین تو بسته چی هست

### واسه .deb

```bash
# لیست فایل‌ها
dpkg -c myapp_1.0.0_amd64.deb

# اطلاعات بسته
dpkg -I myapp_1.0.0_amd64.deb
```

### واسه .rpm

```bash
# لیست فایل‌ها
rpm -qlp myapp-1.0.0-1.x86_64.rpm

# اطلاعات بسته
rpm -qip myapp-1.0.0-1.x86_64.rpm
```

---

## ۱۱. استقرار خودکار

### با Ansible

```yaml
- name: نصب برنامه
  apt:
    deb: /path/to/myapp_1.0.0_amd64.deb
```

### با اسکریپت Bash

```bash
#!/bin/bash
SERVERS="server1 server2 server3"
PACKAGE="myapp_1.0.0_amd64.deb"

for server in $SERVERS; do
  echo "داره نصب میکنه روی $server..."
  scp $PACKAGE $server:/tmp/
  ssh $server "sudo dpkg -i /tmp/$PACKAGE"
done
```

---

## ۱۲. مثال کامل

```bash
#!/bin/bash
# build-package.sh

NAME="mywebapp"
VERSION=$(git describe --tags --always)
DESCRIPTION="وب‌سرور من"

# ساختار پوشه
mkdir -p package/opt/mywebapp
mkdir -p package/etc/systemd/system

# کپی فایل‌ها
cp -r app/* package/opt/mywebapp/
cp myapp.service package/etc/systemd/system/

# بساز بسته
fpm -s dir -t deb \
  -n $NAME \
  -v $VERSION \
  --description "$DESCRIPTION" \
  --depends python3 \
  --deb-systemd myapp.service \
  -C package \
  .

echo "بسته ساخته شد: ${NAME}_${VERSION}_amd64.deb"
```

---

## چند تا نکته مهم

 **همیشه از semantic versioning استفاده کن**

 **وابستگی‌ها رو مشخص کن**

 **بسته رو قبل توزیع تست کن**

 **از Git tag واسه نسخه‌گذاری استفاده کن**

 **بسته رو روی سیستم تمیز تست کن**

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `fpm -s dir -t deb -n name -v ver` | بساز بسته |
| `dpkg -i package.deb` | نصب کن بسته deb |
| `rpm -ivh package.rpm` | نصب کن بسته rpm |
| `dpkg -r package` | پاک کن بسته |
| `dpkg -l \| grep package` | جستجو کن بسته نصب شده |
| `dpkg -c package.deb` | ببین محتویات |
| `git tag v1.0.0` | بساز برچسب نسخه |
