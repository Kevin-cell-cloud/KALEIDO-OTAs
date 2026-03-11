*[Read this in English](#-macos--cp2102-driver-installation-guide) | [中文说明](#-macos--cp2102-驱动安装指南)*

# 🍎 macOS — CP2102 驱动安装指南

适用于 **macOS 12 (Monterey)** 及以上版本

---

## 💡 您可能不需要安装驱动

从 **macOS 10.13 (High Sierra)** 开始，系统已内置 CP2102 驱动。大多数情况下，连接设备后即可自动识别。

**快速检测**：连接设备后，打开终端运行：

```bash
ls /dev/cu.usbserial-*
```

如果能看到类似 `/dev/cu.usbserial-0001` 的输出，说明驱动正常，**无需额外安装**。

---

## 📥 步骤一：获取驱动程序（仅在未自动识别时）

如果系统未自动识别设备，请下载官方驱动：

- 从 [Silicon Labs 官网](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) 下载 `CP210x macOS VCP Driver`

---

## ⚙️ 步骤二：安装驱动

1. 打开下载的 `.dmg` 文件
2. 双击 `Silicon Labs VCP Driver.pkg`
3. 按安装向导提示操作

<!-- <p align="center">
  <img src="images/mac_installer.png" width="500" alt="macOS 安装向导">
  <br><em>macOS 驱动安装向导</em>
</p> -->

> **⚠️ 安全提示**：安装过程中系统可能提示「系统扩展已被阻止」，请按以下步骤处理。

---

## 🔒 步骤三：允许系统扩展

### macOS 13 (Ventura) 及以上

1. 打开「系统设置」→「隐私与安全性」
2. 滚动到底部，找到被阻止的扩展提示
3. 点击「允许」按钮
4. 输入管理员密码确认
5. **重启电脑**

### macOS 12 (Monterey)

1. 打开「系统偏好设置」→「安全性与隐私」→「通用」
2. 点击底部的「允许」按钮
3. **重启电脑**

<!-- <p align="center">
  <img src="images/mac_security.png" width="500" alt="允许系统扩展">
  <br><em>在安全性设置中允许系统扩展</em>
</p> -->

---

## ✅ 步骤四：验证安装

1. 通过 USB 线连接 Kaleido 设备
2. 打开终端，运行以下命令：

```bash
# 查看串口设备
ls /dev/cu.usbserial-*

# 或查看更详细的信息
system_profiler SPUSBDataType | grep -A 5 "CP210"
```

1. 确认出现类似以下输出：

```
/dev/cu.usbserial-0001
```

在 Kaleido Toolbox 中选择该端口即可。

---

## ❓ 故障排除

### 终端中没有出现 usbserial 设备

1. 确认 USB 线是**数据线**（非纯充电线）
2. 尝试更换 USB 端口或 USB-C 转接器
3. 在终端检查 USB 设备是否被识别：

```bash
system_profiler SPUSBDataType
```

如果能看到 `CP2102` 相关条目但没有串口设备，说明驱动未正确加载，请重新安装驱动并重启。

### Apple Silicon (M1/M2/M3/M4) 特别说明

Apple Silicon Mac 通常能自动识别 CP2102，如果遇到问题：

1. 确认下载的是最新版本的驱动（支持 Apple Silicon 原生）
2. 安装后**必须重启电脑**（不是注销，是完全重启）
3. 如果仍有问题，尝试在「恢复模式」中降低安全策略：
   - 关机 → 长按电源键进入恢复模式
   - 菜单栏 →「实用工具」→「启动安全性实用工具」
   - 选择「降低安全性」→ 勾选「允许来自已识别开发者的内核扩展」

### 升级 macOS 后驱动失效

macOS 大版本更新后可能需要重新安装驱动：

1. 从官网下载最新版驱动
2. 重新安装并允许系统扩展
3. 重启电脑

---

# 🍎 macOS — CP2102 Driver Installation Guide

For **macOS 12 (Monterey)** and later

---

## 💡 You Might Not Need to Install Drivers

Starting from **macOS 10.13 (High Sierra)**, the system has a built-in CP2102 driver. In most cases, it will be automatically recognized when you connect the device.

**Quick Check**: After connecting the device, open Terminal and run:

```bash
ls /dev/cu.usbserial-*
```

If you see an output similar to `/dev/cu.usbserial-0001`, your driver is working properly and **no additional installation is required**.

---

## 📥 Step 1: Get Driver (Only if not automatically recognized)

If the system doesn't automatically recognize the device, download the official driver:

- Download `CP210x macOS VCP Driver` from the [Silicon Labs Website](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

---

## ⚙️ Step 2: Install Driver

1. Open the downloaded `.dmg` file.
2. Double-click `Silicon Labs VCP Driver.pkg`.
3. Follow the installation wizard prompts.

<!-- <p align="center">
  <img src="images/mac_installer.png" width="500" alt="macOS Installer Wizard">
  <br><em>macOS Driver Installation Wizard</em>
</p> -->

> **⚠️ Security Warning**: During installation, the system might prompt "System Extension Blocked". Please follow the steps below to handle this.

---

## 🔒 Step 3: Allow System Extensions

### macOS 13 (Ventura) and later

1. Go to "System Settings" → "Privacy & Security".
2. Scroll to the bottom and find the blocked extension prompt.
3. Click the "Allow" button.
4. Enter administrator password to confirm.
5. **Restart your Mac**.

### macOS 12 (Monterey)

1. Go to "System Preferences" → "Security & Privacy" → "General".
2. Click the "Allow" button at the bottom.
3. **Restart your Mac**.

<!-- <p align="center">
  <img src="images/mac_security.png" width="500" alt="Allow System Extensions">
  <br><em>Allow system extensions in security settings</em>
</p> -->

---

## ✅ Step 4: Verify Installation

1. Connect the Kaleido device using a USB cable.
2. Open Terminal and run the following commands:

```bash
# Check for serial devices
ls /dev/cu.usbserial-*

# Or check for more detailed info
system_profiler SPUSBDataType | grep -A 5 "CP210"
```

1. Confirm that a similar output to the following appears:

```
/dev/cu.usbserial-0001
```

Simply select this port in Kaleido Toolbox.

---

## ❓ Troubleshooting

### No usbserial device appears in Terminal

1. Ensure your USB cable is a **data cable** (not a charge-only cable).
2. Try changing to a different USB port or USB-C adapter.
3. Check if the USB device is recognized in Terminal:

```bash
system_profiler SPUSBDataType
```

If you see a `CP2102` related entry but no serial port `/dev/cu.usbserial-*`, the driver is not correctly loaded. Please reinstall the driver and restart.

### Special Notes for Apple Silicon (M1/M2/M3/M4)

Apple Silicon Macs usually automatically recognize CP2102. If you encounter issues:

1. Make sure you downloaded the latest version of the driver (which natively supports Apple Silicon).
2. You **must restart your Mac** after installation (not just log out, a full restart).
3. If it still doesn't work, try lowering the security policy in "Recovery Mode":
   - Shut down your Mac → Press and hold the power button to boot into Recovery.
   - Menu bar → "Utilities" → "Startup Security Utility".
   - Choose "Reduced Security" → Check "Allow user management of kernel extensions from identified developers".

### Driver stops working after macOS upgrade

After major macOS updates, you may need to reinstall the driver:

1. Download the latest driver from the official website.
2. Reinstall it and allow system extensions.
3. Restart your Mac.
