
# 🔒 Kill Switch برای macOS جهت جلوگیری از لو رفتن آی‌پی در هنگام قطع VPN

این پروژه یک Kill Switch ساده اما کاربردی برای کاربران macOS است که می‌خواهند در صورت قطع شدن اتصال VPN، از لو رفتن آی‌پی واقعی خود جلوگیری کنند. این تنظیمات با استفاده از سیستم فایروال macOS به نام **Packet Filter (pf)** انجام می‌شود.

## 📌 پیش‌نیازها

- سیستم‌عامل: macOS (تست شده روی macOS Catalina به بالا)
- دسترسی Administrator
- ویرایشگر متن مثل nano یا vscode
- اتصال VPN فعال که از interfaceهایی مثل `utun0`, `utun4` استفاده می‌کند

## 🧰 مراحل راه‌اندازی Kill Switch

### 1. یافتن Interface مربوط به VPN

```bash
ifconfig
```

در خروجی دنبال interfaceهایی مثل `utun0`, `utun2`, `utun4` بگردید. معمولاً VPNها یکی از این‌ها را فعال می‌کنند. در این مثال فرض می‌کنیم `utun4` است.

### 2. ایجاد فایل تنظیمات فایروال

```bash
sudo nano /etc/pf.anchors/com.myvpn.killswitch
```

محتوا:

```pf
# Block all by default
block all

# Allow DNS requests (for domain resolution)
pass out quick on en0 proto udp from any to any port 53 keep state
pass out quick on en0 proto tcp from any to any port 53 keep state

# Allow outgoing traffic to VPN server (replace with your VPN server's IP and port)
pass out quick on en0 proto tcp from any to 123.123.123.123 port 80 keep state
pass out quick on en0 proto udp from any to 123.123.123.123 port 80 keep state

# Allow ICMP traffic only through VPN (optional but recommended)
pass out quick on utun4 inet proto icmp from any to any keep state

# Allow all traffic through VPN interface
pass out quick on utun4 from any to any keep state
```

🔁 نکته: `en0` معمولاً interface مربوط به Wi-Fi است. اگر از Ethernet استفاده می‌کنید، آن را جایگزین کنید.

### 3. افزودن به pf.conf

```bash
sudo nano /etc/pf.conf
```

اضافه کردن:

```pf
anchor "com.myvpn.killswitch"
load anchor "com.myvpn.killswitch" from "/etc/pf.anchors/com.myvpn.killswitch"
```

### 4. بررسی صحت پیکربندی

```bash
sudo pfctl -n -f /etc/pf.conf
```

### 5. فعال‌سازی فایروال

```bash
sudo pfctl -f /etc/pf.conf
sudo pfctl -e
```

### 6. تست Kill Switch

1. اتصال VPN را برقرار کنید
2. VPN را قطع کنید
3. تست با پینگ:

```bash
ping google.com
```

✅ اگر تنظیمات درست باشد، اتصال انجام نمی‌شود.

## 🚫 غیرفعال‌سازی Kill Switch

```bash
sudo pfctl -d
```

## ⚠️ هشدار امنیتی

- آی‌پی VPN را دقیق وارد کنید.
- Interface VPN را به درستی تشخیص دهید.
- این Kill Switch فقط برای خروجی‌های سیستم کار می‌کند.
