# VM Prerequisites Guide (Windows)

## 1. Enable in BIOS / UEFI
**Where to find:** Restart your PC and press `F2`, `F10`, `F12`, or `Del` (depending on your motherboard brand) to enter BIOS settings. Look under the **Advanced**, **CPU Configuration**, or **Security** tab.

* **Enable:**
  * **VT-x** / **AMD-V**
  * **Virtualization Technology**
  * **Secure Virtual Machine (SVM)**

---

## 2. Disable in Windows Features
**Where to find:** Press `Win + R`, type `optionalfeatures` and hit **Enter** (or search "Turn Windows features on or off" in the Start Menu).

* **Disable (Uncheck):**
  * **Hyper-V**
  * **Windows Hypervisor Platform**
  * **Virtual Machine Platform**
  * **Windows Subsystem for Linux (WSL)**

---

## 3. Disable Background Apps
**Where to find:** System Tray or Start Menu.

* **Disable / Close:**
  * **Docker Desktop**