# OSX-PROXMOX

**Full Proxmox 9.x Support with Automatic QEMU 10.x Compatibility**

Run macOS on Proxmox VE (AMD & Intel) with automatic version detection and compatibility fixes.

[Quick Start](#quick-start) • [Detailed Guide](#detailed-installation-guide) • [Troubleshooting](#troubleshooting) • [FAQ](#faq)

---

## Table of Contents

- [What's New](#whats-new)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Detailed Installation Guide](#detailed-installation-guide)
- [Supported Versions](#supported-versions)
- [Post-Installation](#post-installation)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Performance Optimization](#performance-optimization)
- [Technical Details](#technical-details-proxmox-9x)

---

## What's New

### Latest Updates (January 2026)
- **Sequoia Stability Improvements** - Pinned recovery versions, retry logic, checksum validation
- **Pre-flight Checks** - BCM94360 kernel compatibility, disk space, CDN connectivity
- **Enhanced Error Handling** - 3-attempt retry with exponential backoff, detailed troubleshooting guidance

### Previous Updates (December 2024)
- **Interactive Menu Navigation** - Arrow key selection for all VM options (CPU/RAM/Disk/Storage/Bridge)
- **Security Hardening** - Fixed 5 critical vulnerabilities (command injection, path traversal, network timeouts)
- **Better Error Handling** - ISO validation prevents broken VMs, automatic rollback on failures

### Proxmox 9.x Full Support
- **KVM-Opencore v21** - Uses thenickdude's KVM-specific OpenCore (critical for QEMU 10.x)
- **Automatic CPU vendor detection** - Intel vs AMD host compatibility (vendor=GenuineIntel or AuthenticAMD)
- **qm importdisk method** - Imports OpenCore and recovery ISOs as disk images (QEMU 10.x requirement)
- **ACPI hotplug fixes** - Prevents macOS boot failures on q35 machine type
- **Auto-detects Proxmox version** - Applies appropriate fixes for 7.x, 8.x, and 9.x

### Why This Fork?
Original tutorials for Proxmox 8.4 don't work on 9.x. This version:
- Auto-detects Proxmox version and applies appropriate fixes
- Works on Proxmox 7.x, 8.x, and 9.x
- Interactive menus instead of manual typing
- Security hardened against injection attacks
- Comprehensive documentation explaining what and why

---

## Prerequisites

### Hardware Requirements

| Component | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| **CPU** | Intel/AMD 4 cores | 8+ cores | VT-x/AMD-V required |
| **RAM** | 8GB | 16GB+ | 4GB+ allocated to VM |
| **Storage** | 64GB free | 128GB+ SSD | macOS needs ~64GB minimum |
| **GPU** | Integrated | Discrete (passthrough) | Better performance with passthrough |

### ⚠️ CRITICAL: TSC Check (Required for macOS Monterey+)

Your CPU **must have a stable TSC** or macOS will crash with multiple cores.

**Run this check on your Proxmox host:**
```bash
dmesg | grep -i -e tsc -e clocksource
```

**✅ Good Output** (Safe to proceed):
```
clocksource: Switched to clocksource tsc
```

**❌ Bad Output** (Will cause crashes):
```
tsc: Marking TSC unstable due to check_tsc_sync_source failed
clocksource: Switched to clocksource hpet
```

**🛠 How to Fix:**
1. Enter BIOS and disable:
   - ErP mode
   - All C-state power saving modes
   - Speed step / Cool'n'Quiet
2. Power off completely (not reboot) and restart
3. If still broken, try GRUB override (risky):
   ```bash
   # Edit /etc/default/grub and add to GRUB_CMDLINE_LINUX_DEFAULT:
   clocksource=tsc tsc=reliable
   # Then: update-grub && reboot
   ```

### Software Requirements
- Proxmox VE 7.x, 8.x, or **9.x** (fresh install recommended)
- Internet connection on Proxmox host
- Web browser to access Proxmox console

---

## Quick Start

**For experienced users** — Skip to [Detailed Guide](#-detailed-installation-guide) if you want step-by-step walkthroughs.

### 1. Install Proxmox VE
Download and install from [proxmox.com](https://www.proxmox.com/en/downloads)
- **Proxmox 9.x users**: ✅ Fully supported with automatic workarounds

### 2. Run Installation Script
Open Proxmox web console → Your Node → Shell, then:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/wmehanna/OSX-PROXMOX/main/install.sh)"
```

### 3. Follow Interactive Menu
The script will:
- ✅ Check Proxmox version (auto-configures for 9.x if detected)
- ✅ Download OpenCore bootloader
- ✅ Create macOS recovery images
- ✅ Configure VM with optimal settings

### 4. Start Your VM
Access Proxmox web UI → Select your macOS VM → Start → Console

🎉 **Done!** Boot from OpenCore and install macOS.

---

## Detailed Installation Guide

### Step 1: Prepare Proxmox Host

**1.1 - Install Proxmox VE**
- Download ISO from [proxmox.com/downloads](https://www.proxmox.com/en/downloads)
- Boot from USB and follow installer
- Choose defaults (Next → Next → Finish)

**Expected result:** Access Proxmox web UI at `https://YOUR-SERVER-IP:8006`

**1.2 - Verify Prerequisites**

Run TSC check:
```bash
dmesg | grep -i tsc
```
Expected: `clocksource: Switched to clocksource tsc`

Check virtualization enabled:
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```
Expected: Number > 0 (your CPU core count)

---

### Step 2: Run OSX-PROXMOX Installer

**2.1 - Open Proxmox Shell**
- Navigate to: `Datacenter` → `[Your Node Name]` → `Shell`

**2.2 - Execute Installation Command**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/wmehanna/OSX-PROXMOX/main/install.sh)"
```

**What happens:**
```
Installing git...
Cloning OSX-PROXMOX repository...
Running setup script...
```

**Expected output:**
```
✓ Proxmox 9.x detected - Using enhanced compatibility mode for QEMU 10.x
  Note: Script implements automatic workarounds for media parameter validation

[Interactive menu appears]
```

---

### Step 3: Create macOS VM (Interactive Walkthrough)

The script presents an interactive menu. Here's what each option does:

**Menu Options:**

| Option | What It Does | When to Use |
|--------|--------------|-------------|
| **1** - Create macOS VM | Creates new VM with OpenCore | First time setup |
| **2** - Create Recovery Image | Downloads macOS installer | If you need different macOS version |
| **100** - Setup Prerequisites | Configures Proxmox environment | Run this first on new Proxmox install |
| **101** - Fix macOS stuck at Apple logo | Applies QEMU 6.1+ fix | If VM won't boot past Apple logo |

**3.1 - First Time: Run Option 100**
```
Select: 100 - Setup Prerequisites
```
This will:
- Copy OpenCore ISO to storage
- Configure system aliases
- Update package repositories
- Apply Proxmox 9.x specific configurations

**3.2 - Create Recovery Image (Option 2)**
```
Select: 2 - Download & Create Recovery Image
Choose macOS version: [1-8]
  1 = High Sierra (10.13)
  ...
  8 = Sequoia (15) ← Recommended for latest
```

**What happens:**
- Downloads genuine Apple recovery files (~1-2GB)
- Creates bootable recovery ISO
- Stored in `/var/lib/vz/template/iso/`

**Expected time:** 5-15 minutes depending on internet speed

**3.3 - Create VM (Option 1)**
```
Select: 1 - Create macOS VM - Automatic
```

**Interactive prompts:**

**Prompt 1 - Platform:**
```
Select Platform:
[1] Intel (RECOMMENDED)
[2] AMD

Choice: [1/2]
```
- **Intel**: Host CPU is Intel or you want best compatibility
- **AMD**: Host CPU is AMD (requires additional kernel arguments)

**Prompt 2 - macOS Version:**
```
Select macOS version to install:
[1] High Sierra - 10.13
...
[8] Sequoia - 15 (Latest)

Choice: [1-8]
```
Choose based on your needs (latest = option 8)

**Prompt 3 - VM ID:**
```
Enter VM ID [default: 9000]:
```
- Press Enter for default
- Or choose custom ID (100-999999)

**Prompt 4 - VM Name:**
```
Enter VM Name [default: HACK-macos-sequoia]:
```
- Press Enter for default
- Or customize (alphanumeric, dash, underscore only)

**Prompt 5 - Storage:**
```
Available storages:
  - local-lvm (450.2 GB)

Storage [local-lvm]:
```
Choose where to store VM disk

**Prompt 6 - Disk Size:**
```
Enter disk size in GB [default: 64]:
```
- Minimum: 64GB for macOS
- Recommended: 128GB+

**Prompt 7 - CPU Cores:**
```
Enter number of cores [default: 4]:
```
- Minimum: 2
- Maximum: Your host CPU cores
- Recommended: 4-8

**Prompt 8 - RAM:**
```
Calculated RAM: 8192 MB
Enter RAM in MB [default: 8192]:
```
- Minimum: 4096 (4GB)
- Recommended: 8192+ (8GB+)

**Prompt 9 - Network Bridge:**
```
Available bridges:
  - vmbr0

Bridge [vmbr0]:
```
Press Enter for default

---

### Step 4: Boot and Install macOS

**4.1 - Start VM**
- Proxmox UI → Select your VM (ID 9000) → **Start**
- Click **Console** to view boot process

**4.2 - OpenCore Boot Menu**

**⚠️ IMPORTANT (Proxmox 9.x users):**
- **Press SPACE repeatedly** as soon as the VM starts booting
- This shows the OpenCore picker menu
- Without pressing SPACE, the boot may auto-select and appear stuck

You'll see:
```
[#] macOS Install (DMG)     ← Select this
[ ] UEFI Shell
[ ] Reset NVRAM
```

**Navigation:**
- Arrow keys to select
- Enter to boot

**4.3 - macOS Recovery**

**First Boot Screen:**
```
[Language Selection]
Select: English (or your preference)
```

**macOS Utilities Menu:**
```
[Disk Utility]           ← Choose this first
[Reinstall macOS]
[Time Machine]
[Get Help Online]
```

**4.4 - Format Virtual Disk**

In Disk Utility:
1. Click **View** → **Show All Devices**
2. Select top-level disk (e.g., "64.00 GB QEMU HARDDISK Media")
3. Click **Erase**
   - **Name:** Macintosh HD
   - **Format:** APFS
   - **Scheme:** GUID Partition Map
4. Click **Erase** → Wait for completion
5. Quit Disk Utility

**4.5 - Install macOS**

Back in macOS Utilities:
1. Select **Reinstall macOS** (or "Install macOS Sequoia")
2. Click **Continue**
3. **Agree** to license
4. Select **Macintosh HD**
5. Click **Install**

**Installation Progress:**
- **Phase 1:** Copying files (10-20 min)
  - VM will reboot automatically
- **Phase 2:** Installing system (15-30 min)
  - VM will reboot again
- **Phase 3:** Setup Assistant

**⚠️ Important:** Always select **macOS Installer** in OpenCore menu after each reboot until setup completes

---

### Step 5: macOS Setup Assistant

**5.1 - Language & Region**
Choose your preferences

**5.2 - Create User Account**
```
Full Name: [Your Name]
Account Name: [username]
Password: [secure password]
```

**5.3 - Skip Apple ID** (recommended initially)
- You can add later after EFI configuration

**5.4 - Disable Gatekeeper**

Open Terminal (Utilities → Terminal):
```bash
sudo spctl --master-disable
```
Enter your password when prompted

**Why:** Allows installation of OpenCore configuration tools

---

## Supported Versions

### macOS Versions
| Version | Codename | Status | Notes |
|---------|----------|--------|-------|
| 10.13 | High Sierra | ✅ Tested | May need HTTP recovery workaround |
| 10.14 | Mojave | ✅ Tested | Stable |
| 10.15 | Catalina | ✅ Tested | Stable |
| 11 | Big Sur | ✅ Tested | Requires TSC |
| 12 | Monterey | ✅ Tested | Requires TSC |
| 13 | Ventura | ✅ Tested | Requires TSC, Recommended |
| 14 | Sonoma | ✅ Tested | Requires TSC |
| 15 | Sequoia | ✅ Tested | Latest, Requires TSC |

### Proxmox VE Versions
| Version | Debian Base | QEMU | Status | Notes |
|---------|-------------|------|--------|-------|
| 7.x | Bullseye (11) | 6.2 | ✅ Supported | Legacy |
| 8.x | Bookworm (12) | 8.0 | ✅ Supported | Stable |
| **9.x** | **Trixie (13)** | **10.0** | **✅ Full Support** | **Auto-workarounds** |

### OpenCore Version
- **1.0.4** (April 2025)
  - SIP Enabled
  - Signed DMG only
  - Full security features

---

## Post-Installation

### 1. Install OpenCore to Internal Disk (Optional but Recommended)

**Why:** Boot directly without recovery ISO attached

**How:**
1. Download OpenCore configurator for macOS
2. Mount EFI partition of your Macintosh HD
3. Copy OpenCore files from recovery ISO EFI to disk EFI
4. Update VM boot order in Proxmox

### 2. Enable Apple Services (iCloud, iMessage, etc.)

**For Sequoia users:**
```bash
# In macOS terminal:
sudo nvram boot-args="vm-hw-model=MacPro7,1"
sudo reboot
```

**For older versions:**
- Configure proper SMBIOS in OpenCore config.plist
- Use GenSMBIOS for unique serial numbers

### 3. GPU Passthrough (For Better Performance)

**Requirements:**
- Dedicated GPU (not your host's primary GPU)
- IOMMU enabled in BIOS
- IOMMU groups separated

**Quick Check:**
```bash
# On Proxmox host:
dmesg | grep -i iommu
# Should show: IOMMU enabled
```

**Steps:**
1. Enable IOMMU in GRUB
2. Verify IOMMU groups
3. Bind GPU to VFIO drivers
4. Add PCI device to VM
5. Disable "Above 4G Decoding" in BIOS if VM won't boot

[Detailed GPU Passthrough Guide](https://pve.proxmox.com/wiki/PCI_Passthrough)

### 4. Performance Tuning

See [Performance Optimization](#-performance-optimization) section below

---

## Troubleshooting

### ❌ Sequoia Recovery Download Fails

**Symptoms:**
- "Failed to download recovery" error
- Download times out or hangs
- Checksum validation fails

**Causes:**
1. Network connectivity issues to Apple CDN (swcdn.apple.com)
2. Insufficient disk space (<3GB)
3. BCM94360 WiFi hardware conflict
4. Firewall blocking Apple servers

**Automatic Fixes Applied:**
- Script now retries download 3 times with exponential backoff (10s, 20s, 40s)
- Pre-flight checks validate disk space, CDN connectivity, kernel compatibility
- Progress indicators for large downloads (1450M)

**Manual Troubleshooting:**

**Check 1: Network Connectivity**
```bash
# Test Apple CDN
ping -c3 swcdn.apple.com
curl -I https://swcdn.apple.com

# Check firewall
iptables -L | grep DROP
```

**Check 2: Disk Space**
```bash
# Verify 3GB+ available in /tmp
df -h /tmp
```

**Check 3: BCM94360 Kernel Compatibility (Sequoia only)**
```bash
# Check current kernel
uname -r

# If you have BCM94360 WiFi and kernel != 5.15.53-1-pve:
apt install pve-kernel-5.15.53-1-pve
proxmox-boot-tool kernel pin 5.15.53-1-pve
reboot
```

**Check 4: Review Logs**
```bash
# View detailed download logs
ls -lh /var/log/proxmox-osx-generator/crt-recovery-sequoia.log
less /var/log/proxmox-osx-generator/crt-recovery-sequoia.log
```

**Manual Download (Last Resort):**
If automated retry fails, download manually on another machine and transfer:
```bash
# On Mac/Linux with internet:
git clone https://github.com/acidanthera/OpenCorePkg
cd OpenCorePkg/Utilities/macrecovery
python3 macrecovery.py -b Mac-7BA5B2D9E42DDD94 -m 00000000000000000 -os latest download

# Transfer BaseSystem.dmg and BaseSystem.chunklist to Proxmox
# Place in /mnt/APPLE/ directory during setup
```

---

### ❌ VM Won't Boot Past Apple Logo (Proxmox 9.x)

**Symptom:** macOS boot hangs at Apple logo

**Cause:** ACPI PCI hotplug not disabled (QEMU 6.1+ issue)

**Fix:**
```bash
# Option 1: Use script's built-in fix
# From OSX-PROXMOX menu: Select option 101

# Option 2: Manual fix
# Edit /etc/pve/qemu-server/[YOUR-VM-ID].conf
# Find the 'args:' line and ensure it contains:
args: ... -global ICH9-LPC.acpi-pci-hotplug-with-bridge-support=off
```

**Restart VM after applying fix**

---

### ❌ OpenCore Shows "No bootable device"

**Symptoms:**
- OpenCore menu is empty
- Only shows "UEFI Shell" and "Reset NVRAM"

**Causes:**
1. Recovery ISO not properly created
2. Media parameter issue (Proxmox 9.x)
3. ISO file corrupted

**Fixes:**

**Check 1: Verify VM Config**
```bash
# On Proxmox host:
cat /etc/pve/qemu-server/[VM-ID].conf | grep ide
```

**Expected output:**
```
ide0: storage:iso/opencore-osx-proxmox-vm.iso,media=disk,cache=unsafe,size=96M
ide2: storage:iso/recovery-sequoia.iso,media=disk,cache=unsafe,size=1450M
```

**Key point:** Must say `media=disk`, NOT `media=cdrom`

**Fix if wrong:**
```bash
# Stop VM first
qm stop [VM-ID]

# Edit config
nano /etc/pve/qemu-server/[VM-ID].conf

# Change all 'media=cdrom' to 'media=disk'
# Save and exit (Ctrl+X, Y, Enter)

# Start VM
qm start [VM-ID]
```

---

### ❌ High Sierra / Mojave: "Recovery Server Could Not Be Contacted"

**Symptom:** Installation fails with server connection error

**Cause:** Apple's HTTPS recovery servers deprecated for old macOS

**Fix:**
1. When error appears, **don't close it**
2. Open **Installer Log**: `Window` → `Installer Log`
3. Search for "Failed to load catalog"
4. Copy the URL from the log (e.g., `https://...sucatalog`)
5. Close error, return to macOS Utilities
6. Open **Terminal** (`Utilities` → `Terminal`)
7. Run:
   ```bash
   # Change https:// to http:// in the URL you copied
   nvram IASUCatalogURL="http://swscan.apple.com/content/catalogs/others/index-10.13-10.12-10.11-10.10-10.9-mountainlion-lion-snowleopard-leopard.merged-1.sucatalog"
   ```
8. Quit Terminal, retry installation

[Detailed Guide](https://mrmacintosh.com/how-to-fix-the-recovery-server-could-not-be-contacted-error-high-sierra-recovery-is-still-online-but-broken/)

---

### ❌ GPU Passthrough: Black Screen on External Display

**Symptoms:**
- Internal Proxmox console shows Apple logo
- External monitor connected to passed-through GPU is black

**Cause:** "Above 4G Decoding" enabled in BIOS

**Fix:**
1. Reboot host into BIOS/UEFI
2. Find "Above 4G Decoding" (usually in Advanced → PCI settings)
3. **Disable** it
4. Save and reboot

**Alternative causes:**
- Primary GPU not set correctly in BIOS (must NOT be the passed-through GPU)
- IOMMU groups not properly separated

**IOMMU Group Check:**
```bash
# On Proxmox host:
find /sys/kernel/iommu_groups/ -type l | sort -V
```

**If GPU shares group with other devices, add to GRUB:**
```bash
# Edit /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet pcie_acs_override=downstream,multifunction"

# Then:
update-grub && reboot
```

---

### ❌ TSC-Related Crashes (Multi-Core Issues)

**Symptom:** macOS crashes or reboots when using >1 CPU core

**Cause:** Unstable TSC (timestamp counter)

**Diagnosis:**
```bash
dmesg | grep tsc
```

**Bad output:**
```
tsc: Marking TSC unstable
```

**Fixes (in order of preference):**

**1. BIOS Settings (Best):**
- Disable ErP mode
- Disable all C-states (C1E, C3, C6)
- Disable SpeedStep (Intel) / Cool'n'Quiet (AMD)
- Set PCIe speed to Gen 3 (not Auto)
- Power off completely (not reboot!) and restart

**2. GRUB Override (Risky):**
```bash
nano /etc/default/grub
# Add to GRUB_CMDLINE_LINUX_DEFAULT:
clocksource=tsc tsc=reliable

update-grub
reboot
```

**⚠️ Warning:** May cause host instability

**3. Use Single Core:**
- If nothing works, allocate only 1 CPU core to VM
- Slower but stable

---

### ❌ Installation Fails: "Not Enough Space"

**Symptom:** macOS installer says insufficient disk space

**Cause:** Disk too small or not formatted correctly

**Fix:**
1. Delete VM
2. Recreate with larger disk (minimum 64GB, recommended 128GB+)
3. During setup, use Disk Utility → View → Show All Devices
4. Erase **top-level** disk, not just partition
5. Format: APFS, Scheme: GUID Partition Map

---

## FAQ

### Q: Can I run macOS on AMD CPUs?

**A:** Yes! Select "AMD" platform during VM creation. The script applies:
- Specialized CPU arguments for AMD processors
- Spoofed Intel vendor ID (required for macOS)
- Adjusted Penryn/Cascadelake CPU models

**Performance notes:**
- Slightly slower than Intel due to emulation
- Some Adobe apps may have issues
- Gaming performance reduced

---

### Q: Will this work on Proxmox 9.x?

**A:** ✅ **YES!** This is the main improvement of this fork.

**What's different:**
- Original scripts use `media=disk` directly → **FAILS on Proxmox 9.x**
- This fork uses `media=cdrom` → auto-converts to `media=disk` → **WORKS**

**Why manual configs fail:**
- QEMU 10.x (in Proxmox 9) rejects `media=disk` for ISOs during VM creation
- Script bypasses validation by editing config post-creation

---

### Q: Can I upgrade Proxmox 8.x → 9.x with existing macOS VMs?

**A:** Yes, with caveats:

**Before upgrading:**
```bash
# Backup VM config
cp /etc/pve/qemu-server/[VM-ID].conf ~/macos-vm-backup.conf

# Shutdown VM
qm shutdown [VM-ID]
```

**After upgrading to Proxmox 9:**
```bash
# Verify media parameters are correct
cat /etc/pve/qemu-server/[VM-ID].conf | grep media

# Should see media=disk, if not:
nano /etc/pve/qemu-server/[VM-ID].conf
# Change media=cdrom to media=disk
```

**Start VM** - should boot normally

---

### Q: How do I enable iCloud/iMessage?

**For macOS Sequoia:**
```bash
# In macOS Terminal:
sudo nvram boot-args="vm-hw-model=MacPro7,1"
sudo reboot
```

**For older versions:**
1. Generate unique SMBIOS with GenSMBIOS
2. Edit OpenCore config.plist
3. Update PlatformInfo → Generic:
   - SystemProductName
   - SystemSerialNumber
   - SystemUUID
   - MLB (board serial)
4. Reboot

**⚠️ Important:** Use a serial number that's valid but NOT in use

---

### Q: Can I run this in the cloud (AWS/Azure/etc)?

**A:** Limited support:

**✅ Works on:**
- Vultr (bare metal instances) - [Tutorial](https://youtu.be/8QsMyL-PNrM)
- OVH (bare metal)
- Hetzner (dedicated servers)

**❌ Doesn't work on:**
- AWS EC2 (no nested virtualization + Intel VT-x)
- Azure VMs (same limitations)
- DigitalOcean droplets
- Most VPS providers

**Requirements:**
- Bare metal or dedicated server
- VT-x/AMD-V passthrough
- KVM support

---

### Q: What's the performance like?

**CPU Performance:** 80-95% of bare metal
**Disk I/O:** 70-90% (depends on storage type)
**GPU (passthrough):** 95-98% of bare metal
**GPU (emulated):** 20-30%, unusable for graphics work

**Best performance:**
- NVMe SSD storage
- Dedicated GPU passthrough
- TSC-stable CPU
- 8+ GB RAM
- 4+ CPU cores

---

### Q: Is this legal?

**A:** Complicated:

**Apple's EULA:** macOS license only allows installation on "Apple-branded hardware"

**However:**
- This is for **development, education, and testing purposes**
- Many developers use Hackintosh/VM setups for iOS/macOS dev
- No different than running macOS in VMware/VirtualBox

**⚠️ Disclaimer:**
- Not legal advice
- Use at your own risk
- Not for commercial production use
- Don't pirate macOS (use genuine Apple recovery)

---

## ⚡ Performance Optimization

### 1. Storage Optimization

**Use NVMe/SSD storage:**
```bash
# Check current storage type
pvesm status

# For best performance:
# - Create LVM-thin on NVMe/SSD
# - Avoid network storage (NFS/CIFS) for macOS system disk
```

**Enable discard (TRIM):**
VM config already includes: `discard=on`

**Disable cache for main disk:**
VM config already includes: `cache=none`

**Use unsafe cache for ISOs:**
VM config already includes: `cache=unsafe` for boot ISOs

---

### 2. CPU Optimization

**Pin CPU cores (reduces latency):**
```bash
# Edit /etc/pve/qemu-server/[VM-ID].conf
# Add:
affinity: 0,1,2,3  # Pin to cores 0-3

# Or use taskset:
# Not recommended, use affinity instead
```

**Enable CPU host passthrough:**
Already configured: `-cpu host` for Intel

**Disable CPU mitigations (risky but faster):**
```bash
# On Proxmox host in /etc/default/grub:
GRUB_CMDLINE_LINUX_DEFAULT="quiet mitigations=off"

# Then:
update-grub && reboot
```

⚠️ **Warning:** Reduces security, only for isolated environments

---

### 3. Network Optimization

**Use VirtIO (fastest):**
Script already uses `vmxnet3` (VMware paravirtualized NIC)

**Alternative (requires driver):**
```bash
# Change in VM config:
net0: virtio=XX:XX:XX:XX:XX:XX,bridge=vmbr0

# Download VirtIO drivers for macOS:
# https://github.com/pmj/virtio-net-osx
```

**Enable multi-queue:**
```bash
# In VM config:
net0: vmxnet3=XX:XX:XX:XX:XX:XX,bridge=vmbr0,queues=4
```

---

### 4. GPU Passthrough

**Best performance boost** - see [Post-Installation](#-post-installation) section

**Quick setup:**
1. Enable IOMMU
2. Bind GPU to VFIO
3. Add to VM
4. Configure in macOS

**Performance gain:** 3-5x graphics performance vs emulated GPU

---

### 5. Memory Optimization

**Disable ballooning:**
Already configured: `balloon=0`

**Use hugepages (advanced):**
```bash
# On Proxmox host:
echo "vm.nr_hugepages=2048" >> /etc/sysctl.conf
sysctl -p

# In VM config:
hugepages: 1024  # 2GB pages for 8GB VM
```

**Benefits:** Reduced memory latency, slight performance gain

---

## Technical Details (Proxmox 9.x)

### Why Manual Configs Break on Proxmox 9.x

**Timeline of Changes:**

| Proxmox Version | QEMU Version | Media Parameter Behavior |
|----------------|--------------|--------------------------|
| **7.x** | 6.2 | ISOs as disks: works without `media=` param |
| **8.0-8.3** | 8.0 | Same as 7.x |
| **8.4** | 8.0 | ⚠️ Requires explicit `media=disk` parameter |
| **9.x** | 10.0 | ❌ Rejects `media=disk` for ISOs in `qm create` |

**The Problem:**
```bash
# This worked on Proxmox 8.4:
qm create 100 --ide0 local:iso/opencore.iso,media=disk,cache=unsafe

# This FAILS on Proxmox 9.x with QEMU 10:
# Error: "media=disk not allowed for ISO images"
```

**The Solution (What This Script Does):**
```bash
# Step 1: Create with media=cdrom (passes validation)
qm create 100 --ide0 local:iso/opencore.iso,media=cdrom,cache=unsafe

# Step 2: Edit config file post-creation (bypasses validation)
sed -i 's/media=cdrom/media=disk/' /etc/pve/qemu-server/100.conf

# Result: Works on Proxmox 9.x!
```

**Why it works:**
- Proxmox validates parameters during `qm create` command
- Proxmox does NOT re-validate when reading existing config files
- Post-creation edits bypass the validation layer

---

### ACPI Hotplug Issue (QEMU 6.1+)

**Background:**
- QEMU 6.1+ changed default behavior for q35 machine type
- Machine version 6.1+ enables ACPI PCI hotplug on bridges by default
- macOS doesn't expect this and hangs during boot

**Symptom:** VM boots to Apple logo, progress bar freezes

**Fix Applied by Script:**
```bash
# In VM args parameter:
-global ICH9-LPC.acpi-pci-hotplug-with-bridge-support=off
```

**Auto-detection:**
```bash
# Script checks QEMU version:
qemu_version=$(qemu-system-x86_64 --version | awk '{print $4}')

# If >= 6.1, adds the fix automatically
if version_compare "$qemu_version" "6.1"; then
    args="$args -global ICH9-LPC.acpi-pci-hotplug-with-bridge-support=off"
fi
```

---

### Machine Version Baseline

**Proxmox 8.x:** Default q35 machine < version 6.0
**Proxmox 9.x:** Default q35 machine >= version 6.0

**Impact:**
- Different device enumeration
- Different ACPI tables
- Different PCI topology

**Script approach:**
- Uses default q35 (no version pinning)
- Applies ACPI hotplug fix for compatibility
- Works across all Proxmox versions

---

## Disclaimer

**🚨 FOR DEVELOPMENT, STUDENT, AND TESTING PURPOSES ONLY**

- Not responsible for any issues, damage, or data loss
- Always backup your system before changes
- Not for commercial production use
- Use genuine Apple recovery (not pirated macOS)
- Respect Apple's intellectual property

---

## Demonstration

📽️ [Watch on YouTube](https://youtu.be/dil6iRWiun0) *(Portuguese with auto-translate captions)*

---

## Credits

- **OpenCore/Acidanthera Team** - Open-source bootloader
- **Corpnewt** - Tools (ProperTree, GenSMBIOS, etc.)
- **Gabriel Luchina** - Original OSX-PROXMOX script
- **Apple** - macOS
- **Proxmox** - Virtualization platform

---

## License

This project is licensed under the terms specified in the original repository. See LICENSE file for details.

**Important:** This is a fork with enhanced Proxmox 9.x support. All credits for the original work go to Gabriel Luchina.

---

<div align="center">

**⭐ Star this repo if it helped you!**

Made with ❤️ for the Hackintosh community

</div>
