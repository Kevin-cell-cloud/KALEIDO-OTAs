# 🔌 驱动程序安装指南

Kaleido Toolbox 通过 **CP2102** (Silicon Labs) USB 转串口芯片与设备通信。  
首次连接设备时，需要为您的操作系统安装对应的驱动程序。

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
