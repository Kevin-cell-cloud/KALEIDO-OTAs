*[Read this in English](#-windows--cp2102-driver-installation-guide) | [中文说明](#-windows--cp2102-驱动安装指南)*

# 🪟 Windows — CP2102 驱动安装指南

适用于 **Windows 10 / Windows 11**（64-bit）

---

## 📥 步骤一：获取驱动程序

驱动获取方式（任选其一）：

1. 从本仓库的 **[Release 页面](../../../releases/latest)** 下载 `CP210x_Windows_Driver.zip`
2. 从 [Silicon Labs 官网](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) 下载 `CP210x Windows Drivers`

---

## ⚙️ 步骤二：安装驱动

### 方法 A：使用安装程序（推荐）

1. 解压下载的 `CP210x_Windows_Driver.zip`
2. 找到 `CP210xVCPInstaller_x64.exe`（64位系统）
3. **右键** → 选择「以管理员身份运行」
4. 按提示完成安装

<!-- 将安装截图放在 images/ 目录下 -->
<!-- <p align="center">
  <img src="images/win_installer.png" width="500" alt="运行安装程序">
  <br><em>运行安装程序</em>
</p> -->

### 方法 B：手动安装（inf 方式）

1. 解压驱动压缩包
2. 找到 `silabser.inf` 文件
3. **右键** → 选择「安装」
4. 等待系统提示安装成功

---

## ✅ 步骤三：验证安装

1. 通过 USB 线连接 Kaleido 设备
2. 打开「设备管理器」（`Win + X` → 设备管理器）
3. 展开「端口 (COM 和 LPT)」分类
4. 确认出现：**`Silicon Labs CP210x USB to UART Bridge (COMx)`**

<!-- <p align="center">
  <img src="images/win_device_manager.png" width="500" alt="设备管理器中显示 CP2102">
  <br><em>设备管理器中确认 CP2102 驱动加载成功</em>
</p> -->

记住此处显示的 **COM 端口号**（如 `COM3`），在 Kaleido Toolbox 中选择对应端口即可。

---

## ❓ 故障排除

### 设备管理器中显示黄色感叹号

1. 右键该设备 → 选择「更新驱动程序」
2. 选择「浏览我的电脑以查找驱动程序」
3. 指向解压后的驱动文件夹
4. 点击「下一步」等待安装完成

### 设备管理器中完全不显示

- 确认使用的是**数据线**（非纯充电线）
- 尝试更换 USB 端口
- 重启电脑后重试

### 端口号频繁变化

1. 右键该 COM 端口 → 选择「属性」
2. 切换到「端口设置」→ 点击「高级」
3. 手动指定一个固定的 COM 端口号

---

# 🪟 Windows — CP2102 Driver Installation Guide

For **Windows 10 / Windows 11** (64-bit)

---

## 📥 Step 1: Get the Driver

Methods to get the driver (Choose one):

1. Download `CP210x_Windows_Driver.zip` from this repository's **[Release page](../../../releases/latest)**
2. Download `CP210x Windows Drivers` from the [Silicon Labs Website](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

---

## ⚙️ Step 2: Install the Driver

### Method A: Use Installer (Recommended)

1. Extract the downloaded `CP210x_Windows_Driver.zip`.
2. Find `CP210xVCPInstaller_x64.exe` (for 64-bit systems).
3. **Right-click** → Select "Run as administrator".
4. Follow the prompts to complete the installation.

<!-- Put installation screenshots in images/ -->
<!-- <p align="center">
  <img src="images/win_installer.png" width="500" alt="Run Installer">
  <br><em>Run Installer</em>
</p> -->

### Method B: Manual Installation (inf method)

1. Extract the driver zip file.
2. Find the `silabser.inf` file.
3. **Right-click** → Select "Install".
4. Wait for the system to prompt successful installation.

---

## ✅ Step 3: Verify Installation

1. Connect the Kaleido device using a USB cable.
2. Open "Device Manager" (`Win + X` → Device Manager).
3. Expand the "Ports (COM & LPT)" category.
4. Confirm the appearance of: **`Silicon Labs CP210x USB to UART Bridge (COMx)`**

<!-- <p align="center">
  <img src="images/win_device_manager.png" width="500" alt="CP2102 in Device Manager">
  <br><em>Confirm CP2102 driver loaded successfully in Device Manager</em>
</p> -->

Remember the **COM port number** displayed here (e.g., `COM3`). Simply select the corresponding port in the Kaleido Toolbox.

---

## ❓ Troubleshooting

### Yellow exclamation mark in Device Manager

1. Right-click the device → Select "Update driver".
2. Select "Browse my computer for drivers".
3. Point to the extracted driver folder.
4. Click "Next" and wait for the installation to complete.

### Device completely missing from Device Manager

- Ensure you are using a **data cable** (not a charge-only cable).
- Try changing to a different USB port.
- Restart your computer and try again.

### Port number changes frequently

1. Right-click the COM port → Select "Properties".
2. Switch to the "Port Settings" tab → Click "Advanced".
3. Manually specify a fixed COM Port Number.
