# Fix-NVIDIA-GPU-Hang-on-Unplugging-Charger-in-Linux-Mint-Ubuntu-Lenovo-LOQ-with-RTX-2050-
Fix: System Freezes When Unplugging Charger on Laptops with NVIDIA Optimus 

## 🐛 Problem
When unplugging the AC power on a laptop with hybrid NVIDIA graphics (Optimus), the system completely freezes and becomes unresponsive. This issue is common on **Lenovo LOQ** series laptops with **NVIDIA GeForce RTX 2050/3050/4050** graphics cards running Linux Mint, Ubuntu, or other Debian-based distributions.

## 🔍 Root Cause
The Linux kernel's power management for NVIDIA GPUs attempts to switch power states when the power source changes (AC ↔ Battery). This dynamic power management can cause a kernel panic with certain NVIDIA driver versions and hardware configurations.

## ✅ Solution
Disable NVIDIA's dynamic power management by adding kernel parameters to GRUB.

### Step 1: Edit GRUB Configuration
```bash
sudo nano /etc/default/grub
```

### Step 2: Add Kernel Parameters
Find the line starting with `GRUB_CMDLINE_LINUX_DEFAULT` and change it to:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

Then add the following parameters to `GRUB_CMDLINE_LINUX`:
```
GRUB_CMDLINE_LINUX="nvidia.NVreg_DynamicPowerManagement=0x00 nvidia.NVreg_EnableS0ixPowerManagement=0 nvidia.NVreg_PreserveVideoMemoryAllocations=1"
```

**Or**, if you prefer, add them all to `GRUB_CMDLINE_LINUX_DEFAULT`:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia.NVreg_DynamicPowerManagement=0x00 nvidia.NVreg_EnableS0ixPowerManagement=0 nvidia.NVreg_PreserveVideoMemoryAllocations=1"
```

> **Note:** Using `GRUB_CMDLINE_LINUX` worked more reliably during testing.

### Step 3: Update GRUB
```bash
sudo update-grub
```

### Step 4: Reboot
```bash
sudo reboot
```

### Step 5: Verify
After reboot, check if the parameters are loaded:
```bash
cat /proc/cmdline
```

You should see the parameters in the output.

---

## 🖥️ Tested Environment
- **Laptop:** Lenovo LOQ 15IAX9
- **GPU:** NVIDIA GeForce RTX 2050
- **OS:** Linux Mint (Kernel 7.0.0-14-generic)
- **NVIDIA Driver:** 580.159.03

---

## 🔧 Alternative Solutions (If the above doesn't work)

### Option 1: Switch to NVIDIA-Only Mode
```bash
sudo prime-select nvidia
sudo reboot
```

### Option 2: Use modprobe Configuration
Create a file:
```bash
sudo nano /etc/modprobe.d/nvidia.conf
```

Add these lines:
```
options nvidia NVreg_DynamicPowerManagement=0x00
options nvidia NVreg_EnableS0ixPowerManagement=0
options nvidia NVreg_PreserveVideoMemoryAllocations=1
```

Then update:
```bash
sudo update-initramfs -u
sudo reboot
```

### Option 3: Stop nvidia-powerd Service
```bash
sudo systemctl stop nvidia-powerd.service
sudo systemctl disable nvidia-powerd.service
```

---

## ⚠️ Important Notes
- **Battery Life Impact:** Disabling dynamic power management may increase battery drain slightly since the GPU stays in a more active state.
- **GPU Temperature:** Monitor your GPU temperatures after applying this fix, especially if you're using the NVIDIA card constantly.

---

## 📚 References
- [NVIDIA Linux Driver Documentation](https://download.nvidia.com/XFree86/Linux-x86_64/535.154.05/README/dynamicpowermanagement.html)
- [Arch Linux Wiki: NVIDIA Optimus](https://wiki.archlinux.org/title/NVIDIA_Optimus)
- [Linux Mint Forums: System Freeze on Power State Change](https://forums.linuxmint.com/)

---

## 🤝 Contributing
If you have tested this on other hardware and it worked, feel free to open an issue or pull request with your configuration details.

---

## 📄 License
MIT - Feel free to use and share!

---

**Last Updated:** June 2026
