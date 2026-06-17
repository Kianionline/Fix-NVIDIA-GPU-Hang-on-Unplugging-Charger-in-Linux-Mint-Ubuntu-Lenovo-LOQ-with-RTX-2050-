# رفع مشکل هنگ کردن سیستم هنگام قطع شارژر در لپ‌تاپ‌های دارای NVIDIA Optimus

---

## 🐛 مشکل
هنگامی که شارژر لپ‌تاپ را از برق می‌کشید، سیستم‌عامل به‌طور کامل قفل می‌کند (هنگ می‌کند) و دیگر هیچ واکنشی نشان نمی‌دهد. این مشکل به‌ویژه در لپ‌تاپ‌های سری **Lenovo LOQ** با کارت گرافیک **NVIDIA GeForce RTX 2050/3050/4050** و توزیع‌های لینوکس مانند لینوکس مینت، اوبونتو و دیگر توزیع‌های مبتنی بر دبیان دیده می‌شود.

---

## 🔍 علت مشکل
هسته لینوکس هنگام تغییر منبع تغذیه (از برق به باتری)، سعی می‌کند مدیریت توان کارت گرافیک NVIDIA را تغییر دهد. این تغییر حالت (Dynamic Power Management) گاهی با درایور NVIDIA یا سخت‌افزار خاص هماهنگ نمی‌شود و منجر به یک **خطای بحرانی در هسته (Kernel Panic)** می‌شود که سیستم را کاملاً قفل می‌کند.

---

## ✅ راه‌حل
با اضافه کردن چند پارامتر به هسته لینوکس از طریق فایل تنظیمات GRUB، مدیریت انرژی پویای NVIDIA را غیرفعال می‌کنیم.

---

### مرحله ۱: ویرایش فایل GRUB
ترمینال را باز کنید و دستور زیر را اجرا کنید:

```bash
sudo nano /etc/default/grub
```

---

### مرحله ۲: اضافه کردن پارامترها
خطی که با `GRUB_CMDLINE_LINUX_DEFAULT` شروع می‌شود را پیدا کنید. آن را به این شکل تغییر دهید:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

سپس پارامترهای زیر را به `GRUB_CMDLINE_LINUX` اضافه کنید:

```
GRUB_CMDLINE_LINUX="nvidia.NVreg_DynamicPowerManagement=0x00 nvidia.NVreg_EnableS0ixPowerManagement=0 nvidia.NVreg_PreserveVideoMemoryAllocations=1"
```

> **نکته:** در برخی سیستم‌ها، اضافه کردن پارامترها به `GRUB_CMDLINE_LINUX` بهتر از `GRUB_CMDLINE_LINUX_DEFAULT` جواب می‌دهد.

فایل را ذخیره کنید (`Ctrl+X`، سپس `Y` و در نهایت `Enter`).

---

### مرحله ۳: به‌روزرسانی GRUB
دستور زیر را اجرا کنید تا تغییرات اعمال شوند:

```bash
sudo update-grub
```

---

### مرحله ۴: ری‌استارت کردن
سیستم را مجدداً راه‌اندازی کنید:

```bash
sudo reboot
```

---

### مرحله ۵: بررسی اعمال شدن پارامترها
بعد از بالا آمدن سیستم، دستور زیر را اجرا کنید:

```bash
cat /proc/cmdline
```

خروجی باید شامل پارامترهای اضافه‌شده باشد. چیزی شبیه به این:

```
BOOT_IMAGE=/boot/vmlinuz-... ro nvidia.NVreg_DynamicPowerManagement=0x00 nvidia.NVreg_EnableS0ixPowerManagement=0 nvidia.NVreg_PreserveVideoMemoryAllocations=1 quiet splash
```

---

## 🖥️ محیط تست‌شده
- **لپ‌تاپ:** Lenovo LOQ 15IAX9
- **کارت گرافیک:** NVIDIA GeForce RTX 2050
- **سیستم‌عامل:** لینوکس مینت (هسته 7.0.0-14-generic)
- **نسخه درایور NVIDIA:** 580.159.03

---

## 🔧 راه‌حل‌های جایگزین (اگر راه‌حل بالا کار نکرد)

### گزینه ۱: تغییر حالت به NVIDIA-Only
سیستم را مجبور کنید همیشه از کارت گرافیک NVIDIA استفاده کند:

```bash
sudo prime-select nvidia
sudo reboot
```

### گزینه ۲: استفاده از فایل modprobe
یک فایل پیکربندی جدید ایجاد کنید:

```bash
sudo nano /etc/modprobe.d/nvidia.conf
```

و این سه خط را در آن بنویسید:

```
options nvidia NVreg_DynamicPowerManagement=0x00
options nvidia NVreg_EnableS0ixPowerManagement=0
options nvidia NVreg_PreserveVideoMemoryAllocations=1
```

سپس دستورات زیر را اجرا کنید:

```bash
sudo update-initramfs -u
sudo reboot
```

### گزینه ۳: غیرفعال کردن سرویس nvidia-powerd
اگر هیچکدام از راه‌حل‌های بالا جواب نداد، این سرویس را متوقف کنید:

```bash
sudo systemctl stop nvidia-powerd.service
sudo systemctl disable nvidia-powerd.service
```

---

## ⚠️ نکات مهم
- **مصرف باتری:** غیرفعال کردن مدیریت انرژی پویا ممکن است باعث شود کارت گرافیک کمی بیشتر باتری مصرف کند، چون همیشه در حالت فعال‌تری قرار می‌گیرد.
- **دمای کارت گرافیک:** بعد از اعمال این تنظیمات، دمای کارت گرافیک را کنترل کنید، مخصوصاً اگر به‌طور مداوم از آن استفاده می‌کنید.

---

## 📚 منابع
- [مستندات درایور NVIDIA لینوکس](https://download.nvidia.com/XFree86/Linux-x86_64/535.154.05/README/dynamicpowermanagement.html)
- [ویکی آرچ لینوکس: NVIDIA Optimus](https://wiki.archlinux.org/title/NVIDIA_Optimus)
- [انجمن لینوکس مینت](https://forums.linuxmint.com/)

---

## 🤝 مشارکت
اگر این راه‌حل را روی سخت‌افزار دیگری تست کردید و جواب گرفت، خوشحال می‌شوم که تجربه خود را به اشتراک بگذارید. می‌توانید یک Issue جدید باز کنید یا Pull Request بفرستید.

---

**آخرین به‌روزرسانی:** خرداد ۱۴۰۵ 
