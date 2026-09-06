# TY MUSIC 音乐盒

基于 FoloToy ai-passport（ESP32-C3）硬件定制的**网络音乐播放器**固件。设备连接 2.4GHz WiFi 后，从音乐的公开歌单拉取曲目列表，通过 Meting API 解析音频流，本机软解 MP3 播放，屏幕以复古电台风 UI 展示。

## 功能

- **播放**：MP3 软解（Helix）流式播放、暂停/继续、上下曲切换、播放进度与曲号显示
- **播放页 UI**：TY MUSIC 顶栏（NTP 网络时钟、WiFi 信号格、真实电量）、20px 大字歌名（超长自动滚动）、曲目格子卡（当前曲红色高亮）
- **配网**：设备热点 `TYMusic-XXXX` → 手机浏览器打开 `192.168.4.1`，支持**一键搜索周边 WiFi** 点选填入， captive DNS 劫持（连上热点自动弹出页面）
- **歌单浏览**：曲目列表，选中行超长歌名滚动
- **设置**：音量 / 屏幕亮度调节（NVS 掉电保存）、恢复出厂（清配置重启）
## 硬件

| 项目 | 规格 |
|---|---|
| MCU | ESP32-C3，8MB Flash |
| 屏幕 | 240×320 彩色 TFT |
| 音频 | ES8311 codec + 内置扬声器 / 麦克风 |
| 电量计 | CW2017（I2C），顶栏显示真实电量，<20% 变红 |
| 无线 | 2.4GHz WiFi（**不支持 5GHz**） |
| 按键 | 上 / 下 / OK 三键 + 独立电源键 |
| 供电 | USB Type-C 充电，500mAh 锂电池 |

## 快速上手

### 1. 烧录固件

## 1.方式一
使用ai-passport官方提供的刷机工具烧录：https://ai-passport.folotoy.cn/tools/web-flasher/

## 2.方式二
打开 `烧录固件` 文件夹，设备用 USB 连电脑（COM 口按实际修改）：

```powershell
# 先清空 Flash（整包烧录会清掉 WiFi 配置，必须重新配网）
esptool.py --chip esp32c3 --port COM3 --baud 115200 erase_flash

# 烧录整包固件（二选一，不要混烧）
esptool.py --chip esp32c3 --port COM3 --baud 115200 write_flash 0x0 FoloToy-AI-Passport-full-0x0.bin
```

也可用 [flash_download_tool](https://www.espressif.com/zh-hans/support/download/other-tools) 图形界面，地址按 `烧录说明.txt`。

### 2. 配网

1. 设备开机自动进入配网模式，屏幕显示提示
2. 手机连接热点 **TYMusic-XXXX**（无密码）
3. 浏览器打开 `http://192.168.4.1`（部分手机会自动弹出）
4. 点「搜索附近 WiFi」选择家里 **2.4GHz** 网络，输入密码
5. 选择音源平台，填入**歌单 ID**（Meting API 留空用默认）
6. 点「保存并重启」，约 20 秒后设备自动连网开播

**歌单 ID 获取**：音乐平台网页版打开任意歌单，地址栏 `音乐平台/playlist?id=______` 中的数字即为歌单 ID。

### 3. 按键操作

| 屏幕 | 上键 | 下键 | OK 单击 | OK 长按 |
|---|---|---|---|---|
| 播放页 | 上一曲 | 下一曲 | 暂停 / 继续 | 打开菜单 |
| 菜单 | 上移 | 下移 | 进入 | 返回播放页 |
| 曲目列表 | 上移 | 下移 | 播放选中 | 返回菜单 |
| 音量/亮度 | +5 | -5 | 保存返回 | 返回菜单 |
| 重置确认 | — | — | 确认清空重启 | 返回菜单 |
| 错误页 | — | — | 重试 | 重新配网 |

## 从源码构建

- 环境：ESP-IDF v5.5（Python 3.10 env）
- **工程路径必须为纯 ASCII**（中文路径会触发 ccache "Illegal byte sequence"），本项目构建副本位于 `C:\passport_build\ai-passport-main`
- 20px 标题字库 `main/cjk20.c` 由 `C:\passport_build\fontgen`（lv_font_conv + SimHei）生成，一级常用字子集，生僻字回退 16px 字库

```powershell
idf.py --no-ccache build
idf.py merge_bin   # 生成整包 FoloToy-AI-Passport-full-0x0.bin
```

## 目录结构

```
passport-/
├── ai-passport-main/     # 固件源码（ESP-IDF 工程）
│   ├── main/             # 应用：UI / 播放器 / 配网 / 歌单 / 状态机
│   └── components/bsp/   # 板级：屏幕 / 音频 / 按键 / I2C / 电量计
├── ui_sim/index.html     # 屏幕效果 HTML 模拟器（浏览器直接打开）
└── 烧录固件/              # 编译产物与烧录说明
```

## 注意事项

- ESP32-C3 仅支持 **2.4GHz WiFi**，5GHz 热点搜不到属正常
- 整包固件从 0x0 烧录会清空 NVS → **每次烧录后需重新配网**
- 音源解析依赖 Meting API，需设备能正常访问外网
- 固件体积接近分区上限（约 2.99MB / 3MB），新增大功能前先 `idf.py size` 核对

---
© 2026 [tangyuan](https://www.mancs.cn/)
