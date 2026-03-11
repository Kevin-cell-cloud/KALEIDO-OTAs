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
