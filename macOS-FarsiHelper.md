🔒 Kill Switch برای macOS جهت جلوگیری از لو رفتن آی‌پی در هنگام قطع VPN
این پروژه یک Kill Switch ساده اما کاربردی برای کاربران macOS است که می‌خواهند در صورت قطع شدن اتصال VPN، از لو رفتن آی‌پی واقعی خود جلوگیری کنند. این تنظیمات با استفاده از سیستم فایروال macOS به نام Packet Filter (pf) انجام می‌شود.

📌 پیش‌نیازها
سیستم‌عامل: macOS (تست شده روی macOS Catalina به بالا)

دسترسی Administrator

ویرایشگر متن مثل nano یا vscode

اتصال VPN فعال که از interfaceهایی مثل utun0, utun4 استفاده می‌کند

🧰 مراحل راه‌اندازی Kill Switch
1. یافتن Interface مربوط به VPN
ابتدا باید بفهمید که VPN شما از کدام interface استفاده می‌کند. در ترمینال دستور زیر را بزنید:

bash
Copy
Edit
ifconfig
در خروجی دنبال interfaceهایی مثل utun0, utun2, utun4 بگردید. معمولاً VPNها یکی از این‌ها را فعال می‌کنند. در این مثال فرض می‌کنیم utun4 است.

2. ایجاد فایل تنظیمات فایروال
فایل تنظیمات خود را در مسیر زیر ایجاد یا ویرایش کنید:

bash
Copy
Edit
sudo nano /etc/pf.anchors/com.myvpn.killswitch
و محتوای زیر را وارد کنید (آی‌پی سرور VPN را با آی‌پی واقعی خود جایگزین نکنید، یک آی‌پی فرضی گذاشتیم):

pf
Copy
Edit
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
🔁 نکته: en0 معمولاً interface مربوط به Wi-Fi در مک است. اگر از Ethernet یا Interface دیگر استفاده می‌کنید، آن را جایگزین کنید.

3. افزودن به pf.conf
فایل اصلی تنظیمات pf را ویرایش کنید:

bash
Copy
Edit
sudo nano /etc/pf.conf
و در ابتدای فایل، این خط را اضافه کنید:

pf
Copy
Edit
anchor "com.myvpn.killswitch"
load anchor "com.myvpn.killswitch" from "/etc/pf.anchors/com.myvpn.killswitch"
4. بررسی صحت پیکربندی
قبل از فعال‌سازی فایروال، مطمئن شوید که تنظیمات مشکلی ندارند:

bash
Copy
Edit
sudo pfctl -n -f /etc/pf.conf
اگر خطایی نداشت، به مرحله بعد بروید.

5. فعال‌سازی فایروال
برای اعمال تنظیمات:

bash
Copy
Edit
sudo pfctl -f /etc/pf.conf
sudo pfctl -e
اگر فایروال قبلاً فعال بوده باشد، فقط از دستور -f استفاده کنید.

6. تست Kill Switch
اتصال VPN را برقرار کنید و مطمئن شوید اینترنت دارید.

VPN را قطع کنید.

سعی کنید به سایتی مثل google.com پینگ بزنید:

bash
Copy
Edit
ping google.com
✅ اگر تنظیمات درست باشد، در صورت قطع VPN هیچ داده‌ای از سیستم خارج نمی‌شود.

🚫 غیرفعال‌سازی Kill Switch
برای غیرفعال کردن Kill Switch (مثلاً برای تغییر تنظیمات):

bash
Copy
Edit
sudo pfctl -d
و اگر خواستید کامل حذفش کنید، خط‌های مربوطه در /etc/pf.conf و فایل com.myvpn.killswitch را پاک کنید.

⚠️ هشدار امنیتی
حتماً آی‌پی سرور VPN را دقیق وارد کنید.

مطمئن شوید که interfaceهای مربوط به VPN را درست تشخیص داده‌اید.

این Kill Switch فقط برای خروجی‌های سیستم کار می‌کند و در مقابل مشکلات داخلی VPN محافظت نمی‌کند.
