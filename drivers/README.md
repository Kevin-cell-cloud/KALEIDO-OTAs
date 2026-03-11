*[Read this in English](#-driver-installation-guide) | [中文说明](#-驱动程序安装指南)*

# 🔌 驱动程序安装指南

Kaleido Toolbox 通过 **CP2102** (Silicon Labs) USB 转串口芯片与设备通信。  
首次连接设备时，需要为您的操作系统安装对应的驱动程序。

> **💡 提示：** 如果你已经使用过 Artisan 并且能够操作你的设备，证明你的电脑已经能够识别设备端口，则**无需**再配置驱动！
>
> **💡 蓝牙无线连接（可选）：** 如果你使用了 Intel CPU 的 Mac 电脑或 Windows 电脑，并且不想使用 USB 有线连接 (CP2102 串口)，你可以使用电脑蓝牙链接设备。设备通常的识别名称为 `Roaster_SRSv6`。在系统蓝牙菜单里完成配对后，你可以在工具上找到它的端口（如果链接后显示了两个 COM 端口，只有一个可用，请依次尝试）。

---

## 📋 芯片信息

| 项目 | 说明 |
|------|------|
| 芯片型号 | Silicon Labs CP2102 / CP2102N |
| 用途 | USB 转 UART 串口通信 |
| 识别方式 | 设备管理器中显示为 `Silicon Labs CP210x USB to UART Bridge` |

---

## 📥 驱动下载

您可以从以下来源获取驱动程序：

| 平台 | 下载方式 |
|------|---------|
| Windows | [Release 页面下载](../../releases/latest) 或 [Silicon Labs 官网](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) |
| macOS | macOS 10.13+ 通常**自带驱动**，无需安装；如不识别请 [下载官方驱动](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) |

---

## 📖 安装指南

请根据您的操作系统选择对应的安装指南：

### 🪟 [Windows 安装指南](Windows/README.md)

适用于 Windows 10 / 11

### 🍎 [macOS 安装指南](macOS/README.md)

适用于 macOS 12 (Monterey) 及以上

---

## ❓ 常见问题

### 如何确认驱动是否安装成功？

**Windows**：打开「设备管理器」→「端口 (COM 和 LPT)」→ 应显示 `Silicon Labs CP210x USB to UART Bridge (COMx)`

**macOS**：打开「终端」运行以下命令：

```bash
ls /dev/tty.usbserial-*
# 或
ls /dev/cu.usbserial-*
```

如果显示类似 `/dev/tty.usbserial-0001` 的设备，说明驱动正常。

### 安装驱动后仍无法识别？

1. 拔出 USB 线，等待 5 秒后重新插入
2. 尝试更换 USB 端口（优先使用主板直连端口，避免集线器）
3. 尝试更换 USB 数据线（确认是数据线而非纯充电线）
4. 重启电脑后重试

### macOS 提示"系统扩展已被阻止"？

前往「系统设置」→「隐私与安全性」→ 底部点击「允许」→ 重启电脑

---

# 🔌 Driver Installation Guide

Kaleido Toolbox uses the **CP2102** (Silicon Labs) USB-to-UART bridge controller to communicate with your device.  
When connecting the device for the first time, you need to install the corresponding driver for your operating system.

> **💡 Note:** If you have used Artisan before and are able to operate your device, it means your computer can already recognize the device port, and you **do not** need to configure the driver again!
>
> **💡 Bluetooth Connection (Optional):** If you are using an Intel-based Mac or a Windows PC and prefer a wireless connection over the USB wire (CP2102 serial), you can connect to the device via Bluetooth. The device is usually named `Roaster_SRSv6`. Pair it in your system's Bluetooth menu, and you'll find its port in the tool (if two COM ports appear after connecting, only one is active—try them sequentially).

---

## 📋 Chip Information

| Item | Description |
|------|-------------|
| Chip Model | Silicon Labs CP2102 / CP2102N |
| Function | USB to UART Serial Communication |
| Recognition | Shown as `Silicon Labs CP210x USB to UART Bridge` in Device Manager |

---

## 📥 Download Driver

You can get the driver from the following sources:

| Platform | Download Method |
|----------|-----------------|
| Windows | [Download from Release](../../releases/latest) or [Silicon Labs Website](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) |
| macOS | macOS 10.13+ usually has **built-in drivers**, no installation required; if not recognized, [Download Official Driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) |

---

## 📖 Installation Guide

Please choose the installation guide corresponding to your operating system:

### 🪟 [Windows Installation Guide](Windows/README.md)

For Windows 10 / 11

### 🍎 [macOS Installation Guide](macOS/README.md)

For macOS 12 (Monterey) and above

---

## ❓ FAQ

### How to verify if the driver is installed successfully?

**Windows**: Open "Device Manager" → "Ports (COM & LPT)" → It should display `Silicon Labs CP210x USB to UART Bridge (COMx)`.

**macOS**: Open "Terminal" and run the following command:

```bash
ls /dev/tty.usbserial-*
# or
ls /dev/cu.usbserial-*
```

If a device like `/dev/tty.usbserial-0001` is displayed, the driver is working properly.

### Driver installed but device still not recognized?

1. Unplug the USB cable, wait for 5 seconds, and plug it back in.
2. Try a different USB port (prefer straight connection to motherboard, avoid USB hubs).
3. Try a different USB cable (make sure it's a data cable, not a charge-only cable).
4. Restart your computer and try again.

### macOS shows "System Extension Blocked"?

Go to "System Settings" → "Privacy & Security" → click "Allow" at the bottom → Restart your computer.
