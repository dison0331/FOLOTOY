# 交接文件 · FOLOTOY AI Passport 应用开发

> 生成时间：2026-11-03（已根据 IDEA.md 更新）
> 阶段：需求规格设计（应用需求已明确，即将调用 spec-requirement-agent）
> 文档目录：`Z:\FOLOTOY\wen\`

---

## 1. 开发目标

基于 https://gitee.com/FoloToy/ai-passport 开发多款应用固件，复用本地 `passport-os/passport-os-main/` 作为实现参考。

### 1.1 应用清单（7 款）

| 应用 | 功能 | 使用条件 |
| --- | --- | --- |
| 首页 | 时钟、个人资料、头像与名片、天气和空气质量 | 天气需联网 |
| 收音机 | 选择城市和电台，听广播，设置睡眠计时 | Wi-Fi + 网络电台 |
| AI 随身语音 | 设备作为电脑语音输入工具，联动快捷键 | Windows 配套程序 |
| Muse | 语音发送给 Muse，屏幕阅读回复 | 手机配对 + 账号 + 网络 |
| 俄罗斯方块 | 堆叠、消行 | 离线 |
| 秒表 | 计时、分段、暂停与继续 | 离线 |
| 番茄钟 | 专注、休息、到时提醒 | 离线 |

**与 passport-os-main 现有应用对比**：少了"小智 AI""你在狗叫什么""声音跳一跳"3 款。

### 1.2 系统层功能要求

1. 预置 4 个主题
2. 音量、亮度调节
3. 自动熄屏、短按唤醒
4. Wi-Fi 配网
5. 电池显示与功耗管理

### 1.3 交付要求

- 复用 BSP，重新设计应用界面，不照搬 demo
- 中文界面检查字体覆盖
- 保留已有修改，新建分支开发
- 运行项目验证，交付可从 0x0 刷写的合并固件
- 说明测试结果与未验证项

---

## 2. 项目概况

| 项 | 值 |
| --- | --- |
| 官方库 | https://github.com/folotoy/ai-passport（本地 `Z:\FOLOTOY\ai-passport`） |
| 复用示例 | https://github.com/manchunx7-bit/passport-os（本地 `Z:\FOLOTOY\passport-os\passport-os-main`） |
| git 仓库根 | `Z:\FOLOTOY` |
| 当前分支 | `main`（working tree clean） |
| 平台 | ESP32-C3 · 8MB Flash · 无 PSRAM |
| 框架 | ESP-IDF 5.5.3 + LVGL |
| 显示 | ST7789 240×320 SPI 屏 |
| 输入 | 三键（UP/DOWN/OK，ADC 分压） |
| 构建环境 | WSL（`/opt/esp-idf`） |
| 刷写 | Windows COM3，从 0x0 刷写合并固件 |

---

## 3. 关键路径

```
Z:\FOLOTOY\passport-os\passport-os-main\
├── build.sh                    # bash build.sh release → build-public/FoloToy-AI-Passport-full.bin
├── partitions.csv              # 8MB 分区表
├── sdkconfig.defaults
├── main/
│   ├── apps/                   # 应用实现（app_base.h 定义接口）
│   │   ├── home/ radio/ voice/ muse/ game/ sound_jump/ stopwatch/ pomodoro/ xiaozhi/
│   ├── launcher/               # app_registry.cc/.h（注册表）、launcher.cc/.h（启动器）
│   ├── ui/                     # ui_pixel.h（共享构件）、ui_palette.h（语义颜色）
│   ├── fonts/                  # buddy_font_16.c（思源黑体子集）、gen_buddy_font.sh
│   └── main.cc
├── components/
│   ├── bsp/include/            # bsp_display/button/audio/battery/i2c/memory/pins.h
│   ├── esp-wifi-connect/ noise_core/ passport_muse/
├── docs/                       # 各设计/评审文档
└── tools/verify_firmware.py
```

---

## 4. 应用架构要点

- **接口**：`passport_app_t`（id/name/init/start/stop/on_key/prepare_stop），定义于 `main/apps/app_base.h`
- **注册**：`main/launcher/app_registry.cc` 中 `s_apps[]` 数组 + `extern` 声明
- **图标/主题色**：`main/launcher/launcher.cc` 的 `draw_app_icon()`
- **UI 构件**：`main/ui/ui_pixel.h`（panel/label/footer/title_plate/mascot…）
- **颜色**：`main/ui/ui_palette.h` 语义角色（UI_TEXT/UI_ACCENT…）
- **字体**：`buddy_font_16`（思源黑体子集），新增中文需用 `gen_buddy_font.sh` 重新生成

### 新增/调整应用步骤

1. `app_base.h` 枚举调整 `APP_ID_*`
2. `main/apps/xxx/` 实现应用
3. `app_registry.cc` 注册 + 省电堆需求表
4. `launcher.cc` 图标 + 主题色
5. `main/CMakeLists.txt` 注册源文件
6. 字体字形覆盖核对

---

## 5. 构建与刷写

```bash
# WSL 中构建
cd /mnt/z/FOLOTOY/passport-os/passport-os-main
bash build.sh release
# 产物：build-public/FoloToy-AI-Passport-full.bin（合并固件，从 0x0 刷写）

# Windows 刷写
esptool.py --port COM3 --baud 460800 write_flash 0x0 build-public/FoloToy-AI-Passport-full.bin
```

分区表（8MB）：nvs 0x9000/0x6000 · phy_init 0xf000/0x1000 · factory 0x10000/0x400000 · cardid 0x410000/0x4000 · storage 0x420000/0x200000

---

## 6. 当前状态

### 已完成
- 项目复用确认、文档调研、平台/框架/接口调研、应用架构调研、构建方式确认
- IDEA.md 应用需求已明确（7 款应用 + 5 项系统层功能）

### 待办
1. 调用 spec-requirement-agent 生成 spec.md
2. 呈现 spec.md 摘要，询问用户是否修改
3. 生成 design.md（实现方案）
4. 生成 tasks.md（编码任务规划）
5. 新建开发分支
6. 实现应用代码 + 系统层功能
7. 核对中文字体覆盖
8. WSL 构建验证
9. 交付合并固件 + 测试结果 + 未验证项说明

---

## 7. 关键约束

- **保留已有修改**，新建分支开发
- **复用 BSP**，不直接操作硬件
- **不照搬 demo**，重新设计界面
- **中文界面**必须核对字体字形覆盖
- **语义颜色**，不硬编码
- **构建必须在 WSL**（Windows 无工具链）
- **每步可溯源**，关键操作记录到交接文件
- **编辑后必须验证**（read/grep 确认生效）

---

## 8. 文档索引

| 文档 | 路径 |
| --- | --- |
| 任务需求 | `Z:\FOLOTOY\IDEA.md` |
| 项目说明 | `passport-os-main/README.md` |
| 设计 | `passport-os-main/DESIGN.md` |
| UI 规范 | `passport-os-main/docs/travel-instrument-ui.md` |
| 应用评审 | `passport-os-main/docs/app-review-2026-09-12.md` |
| 交接文件 | `Z:\FOLOTOY\wen\HANDOVER.md` |
| 进度文件 | `Z:\FOLOTOY\wen\PROGRESS.md` |
