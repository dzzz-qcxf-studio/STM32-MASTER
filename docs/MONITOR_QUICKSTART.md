# STM32 串口监控快速开始

## 最快启动方式

```powershell
# 一行命令启动，自动打开浏览器
.\monitors\monitor_web.ps1 -SerialPort "COM5"
```

默认配置：波特率 `115200`，Web端口 `8080`

---

## 正确启动方式

**重要：使用 `Start-Process` 而非直接执行**

❌ **错误方式** - 直接调用会阻塞等待：
```powershell
# 这会阻塞当前会话，直到脚本结束
.\monitors\monitor_web.ps1 -SerialPort "COM5"
```

✅ **正确方式** - 新窗口启动不阻塞：
```powershell
# 在新窗口启动，立即返回不阻塞
Start-Process powershell -ArgumentList "-File",".\monitors\monitor_web.ps1","-SerialPort","COM5" -WindowStyle Normal
```

**为什么？**

`monitor_web.ps1` 脚本启动 HTTP 服务器后会持续运行，直接执行会一直占住会话。使用 `Start-Process` 可以在独立窗口运行，脚本内部的 `Start-Sleep` 等待（500ms）和自动打开浏览器都能正常工作，且不会阻塞当前工作窗口。

---

## 三种使用模式

### 方式一：Web UI 模式（推荐）

```powershell
# 自动打开浏览器，界面操作
.\monitors\monitor_web.ps1 -SerialPort "COM5"

# 指定端口和波特率
.\monitors\monitor_web.ps1 -SerialPort "COM5" -BaudRate 9600 -Port 8080

# 不自动打开浏览器
.\monitors\monitor_web.ps1 -SerialPort "COM5" -OpenBrowser $false
```

访问 http://localhost:8080

| 功能 | 说明 |
|------|------|
| 端口选择 | 下拉菜单 + 刷新按钮 |
| 波特率 | 9600 ~ 921600 |
| 发送 | 输入框 + 回车发送 |
| 清空/下载 | 清除显示或导出日志 |

---

### 方式二：Node.js 命令行

```bash
cd monitors

# 交互模式 - 启动后界面选择
node serial_monitor.js

# 预连接模式 - 启动时直接连接
node serial_monitor.js COM9 115200 8080
```

参数：`[COM端口] [波特率] [Web端口]`

---

### 方式三：纯命令行监控

```powershell
# 输出到终端
.\monitors\monitor_serial.ps1 -Port "COM5"

# 保存到文件
.\monitors\monitor_serial.ps1 -Port "COM5" -LogFile "serial.log"

# 过滤关键词
.\monitors\monitor_serial.ps1 -Port "COM5" -Filter "ERROR|WARN"

# 定时停止（秒）
.\monitors\monitor_serial.ps1 -Port "COM5" -Duration 60
```

---

## 参数说明

### monitor_web.ps1

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `-SerialPort` | 自动检测 | COM 端口 |
| `-BaudRate` | 115200 | 波特率 |
| `-Port` | 8080 | Web 服务器端口 |
| `-OpenBrowser` | $true | 自动打开浏览器 |

### monitor_serial.ps1

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `-Port` | 自动检测 | COM 端口 |
| `-BaudRate` | 115200 | 波特率 |
| `-Duration` | 0 (无限) | 监控时长（秒） |
| `-LogFile` | 无 | 日志文件路径 |
| `-Filter` | 无 | 正则表达式过滤 |

---

## 查看可用端口

```powershell
[System.IO.Ports.SerialPort]::GetPortNames()
```

---

## 故障排查

**无法连接？**
1. 检查端口是否被占用：`tasklist | findstr node`
2. 换一个 COM 端口试试
3. 刷新页面重试

**Web UI 无数据？**
1. 确认已点击"连接"按钮
2. 检查波特率与设备是否匹配
3. 按 F12 查看浏览器控制台错误

**端口列表为空？**
1. 检查 USB 是否连接
2. 安装/更新 CH340/FTDI 驱动
3. 点击刷新按钮重新扫描

---

## HTTP API

```bash
# 查看可用端口
curl http://localhost:8080/ports

# 连接端口
curl "http://localhost:8080/connect?port=COM5&baud=115200"

# 断开
curl http://localhost:8080/disconnect

# 发送数据
curl "http://localhost:8080/send?data=hello"
```

---

## 文件结构

```
monitors/
├── serial_monitor.js   # Node.js 主程序
├── monitor_web.ps1     # Web UI 启动脚本
├── monitor_serial.ps1  # 命令行监控脚本
└── node_modules/       # 依赖包
```