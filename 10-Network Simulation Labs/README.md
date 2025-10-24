# ۱۰) شبیه‌سازی مشکلات شبکه!
یاد بگیر چطور شرایط بد شبکه رو تست کنی

---

## چرا مهمه؟

برنامه‌ت رو لوکال کامل کار میکنه، ولی تو شبکه واقعی:
- **تاخیر** (latency)
- **بسته از دست رفته** (packet loss)
- **پهنای باند محدود**

ممکنه باگ‌هایی رو آشکار کنه. با این ابزارا میتونی شرایط واقعی رو شبیه‌سازی کنی.

---

## چند تا چیز ساده که باید بدونی

| مفهوم | یعنی چی |
|-------|--------|
| **Network namespace** | یه شبکه مجازی جدا |
| **veth** | کابل شبکه مجازی |
| **tc (traffic control)** | کنترل ترافیک شبکه |
| **netem** | شبیه‌سازی مشکلات شبکه |
| **latency** | تاخیر شبکه |

---

## ۱. بساز یه آزمایشگاه شبکه ساده

### بساز دو namespace (مث دوتا کامپیوتر مجازی)

```bash
# بساز دو فضای شبکه
sudo ip netns add server
sudo ip netns add client

# بساز کابل مجازی
sudo ip link add veth-server type veth peer name veth-client

# وصل کن کابل به namespaceها
sudo ip link set veth-server netns server
sudo ip link set veth-client netns client

# بده IP
sudo ip netns exec server ip addr add 10.0.0.1/24 dev veth-server
sudo ip netns exec client ip addr add 10.0.0.2/24 dev veth-client

# فعال کن
sudo ip netns exec server ip link set veth-server up
sudo ip netns exec client ip link set veth-client up
```

### تست کن اتصال

```bash
sudo ip netns exec client ping -c 3 10.0.0.1
```

 باید ping موفق باشه!

---

## ۲. اجرا سرویس تو namespace

### بیار یه وب‌سرور تو server

```bash
sudo ip netns exec server python3 -m http.server 8080
```

### وصل شو از client

```bash
# تو ترمینال جدید
sudo ip netns exec client curl http://10.0.0.1:8080
```

---

## ۳. اضافه کن تاخیر (Latency)

شبیه‌سازی تاخیر ۱۰۰ میلی‌ثانیه:

```bash
sudo ip netns exec server tc qdisc add dev veth-server root netem delay 100ms
```

**تست کن:**

```bash
sudo ip netns exec client ping -c 5 10.0.0.1
```

حالا زمان ping حدود ۱۰۰ms هست!

### حذف کن تاخیر

```bash
sudo ip netns exec server tc qdisc del dev veth-server root
```

---

## ۴. شبیه‌سازی از دست رفتن بسته (Packet Loss)

### اضافه کن ۵٪ packet loss

```bash
sudo ip netns exec server tc qdisc add dev veth-server root netem loss 5%
```

**تست کن:**

```bash
sudo ip netns exec client ping -c 20 10.0.0.1
```

تعدادی از pingها شکست میخورن.

---

## ۵. ترکیب تاخیر و packet loss

شبکه واقعی با مشکل:

```bash
sudo ip netns exec server tc qdisc add dev veth-server root netem \
  delay 100ms loss 2%
```

**ببین تنظیمات:**

```bash
sudo ip netns exec server tc qdisc show dev veth-server
```

---

## ۶. محدود کن پهنای باند

محدود کن سرعت به ۱ مگابیت:

```bash
sudo ip netns exec server tc qdisc add dev veth-server root tbf \
  rate 1mbit burst 32kbit latency 400ms
```

**تست کن:**

```bash
# بساز فایل تست
dd if=/dev/zero of=/tmp/testfile bs=1M count=10

# انتقال و اندازه‌گیری سرعت
sudo ip netns exec client curl -o /dev/null http://10.0.0.1:8080/testfile
```

سرعت دانلود به حدود ۱ مگابیت محدود شده.

---

## ۷. شبیه‌سازی شبکه موبایل

### 3G (تاخیر بالا، سرعت پایین)

```bash
sudo ip netns exec server tc qdisc add dev veth-server root handle 1: tbf \
  rate 1mbit burst 32kbit latency 400ms

sudo ip netns exec server tc qdisc add dev veth-server parent 1:1 handle 10: netem \
  delay 200ms loss 1%
```

### 4G (بهتر ولی هنوز محدود)

```bash
sudo ip netns exec server tc qdisc add dev veth-server root handle 1: tbf \
  rate 10mbit burst 128kbit latency 100ms

sudo ip netns exec server tc qdisc add dev veth-server parent 1:1 handle 10: netem \
  delay 50ms loss 0.5%
```

---

## ۸. اسکریپت آماده واسه تست

```bash
#!/bin/bash
# setup-test-network.sh

echo "داره میسازه شبکه تست..."

# بساز namespaceها
sudo ip netns add server 2>/dev/null
sudo ip netns add client 2>/dev/null

# بساز و وصل کن veth
sudo ip link add veth-server type veth peer name veth-client 2>/dev/null
sudo ip link set veth-server netns server
sudo ip link set veth-client netns client

# تنظیم کن IP
sudo ip netns exec server ip addr add 10.0.0.1/24 dev veth-server
sudo ip netns exec client ip addr add 10.0.0.2/24 dev veth-client

# فعال کن
sudo ip netns exec server ip link set veth-server up
sudo ip netns exec server ip link set lo up
sudo ip netns exec client ip link set veth-client up
sudo ip netns exec client ip link set lo up

# اضافه کن تاخیر و packet loss
sudo ip netns exec server tc qdisc add dev veth-server root netem \
  delay 100ms loss 1%

echo "شبکه آمادست!"
echo "سرور: sudo ip netns exec server python3 -m http.server 8080"
echo "کلاینت: sudo ip netns exec client curl http://10.0.0.1:8080"
```

**اجرا:**

```bash
chmod +x setup-test-network.sh
./setup-test-network.sh
```

---

## ۹. پاک‌سازی

```bash
#!/bin/bash
# cleanup-network.sh

echo "داره پاک‌سازی میکنه شبکه تست..."

sudo ip netns delete server 2>/dev/null
sudo ip netns delete client 2>/dev/null

echo "پاک‌سازی تموم شد!"
```

---

## ۱۰. ببین برنامه چطور عمل میکنه

### اندازه‌گیری زمان پاسخ

```bash
sudo ip netns exec client curl -w "@-" -o /dev/null -s http://10.0.0.1:8080 <<'EOF'
زمان کل: %{time_total}s
زمان اتصال: %{time_connect}s
زمان انتقال: %{time_starttransfer}s
EOF
```

### تست بار (Load Test)

```bash
# نیاز به نصب ab (Apache Bench)
# Arch Linux
sudo pacman -S apache

# Ubuntu/Debian
sudo apt install apache2-utils

# تست
sudo ip netns exec client ab -n 100 -c 10 http://10.0.0.1:8080/
```

---

## ۱۱. سناریوهای تست

### تست timeout

```bash
# تاخیر خیلی زیاد
sudo ip netns exec server tc qdisc change dev veth-server root netem delay 5s

# برنامه‌ت باید timeout رو handle کنه
sudo ip netns exec client curl --max-time 3 http://10.0.0.1:8080
```

### تست retry logic

```bash
# packet loss بالا
sudo ip netns exec server tc qdisc change dev veth-server root netem loss 50%

# برنامه باید retry کنه
```

---

## ۱۲. نمایش وضعیت شبکه

```bash
# لیست namespaceها
sudo ip netns list

# اطلاعات کامل
sudo ip netns exec server ip addr
sudo ip netns exec server ip route

# آمار شبکه
sudo ip netns exec server ip -s link show veth-server
```

---

## چند تا نکته مهم

 **برنامه رو تو شرایط واقعی تست کن**

 **timeout و retry رو پیاده‌سازی کن**

 **عملکرد با latency بالا رو چک کن**

 **namespaceها بعد ریستارت پاک میشن**

 **نیاز به sudo داری**

---

## خلاصه دستورات

| دستور | کارش چیه |
|-------|---------|
| `ip netns add NAME` | بساز namespace |
| `ip netns list` | لیست namespaceها |
| `ip netns delete NAME` | پاک کن namespace |
| `ip netns exec NAME COMMAND` | اجرا کن دستور تو namespace |
| `tc qdisc add ... netem delay 100ms` | اضافه کن تاخیر |
| `tc qdisc add ... netem loss 5%` | packet loss |
| `tc qdisc del dev eth0 root` | پاک کن تنظیمات |
| `tc qdisc show` | نشون بده تنظیمات |

---

## پارامترهای netem

| پارامتر | مثال | یعنی چی |
|---------|------|--------|
| `delay` | `delay 100ms` | تاخیر |
| `loss` | `loss 5%` | از دست رفتن بسته |
| `duplicate` | `duplicate 1%` | تکرار بسته |
| `corrupt` | `corrupt 0.1%` | خرابی داده |
| `reorder` | `reorder 25%` | تغییر ترتیب |