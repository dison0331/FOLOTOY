# 交接文件 · FOLOTOY AI Passport 应用开发

> 生成时间：2026-11-03
> 阶段：需求规格设计（spec-requirement-agent 已完成调研，**阻塞于应用需求未明确**）
> 下一步：等待用户补充"要做什么应用"，再生成 spec.md 并推进

---

## 1. 任务原始请求

基于完整项目 https://gitee.com/FoloToy/ai-passport 开发【应用（用户一会单独补充）】。

要求：
- 本地已有项目则复用
- 先读 AGENTS.md 和 docs/README.md，按需查阅相关文档、硬件接口及示例
- 自行准备环境并安装项目要求的 Skill
- 保留已有修改，新建分支开发
- 复用 BSP，重新设计应用界面，不照搬 demo
- 中文界面检查字体覆盖
- 关键需求不明确时询问，其余自主完成
- 运行项目验证，交付可从 0x0 刷写的合并固件
- 说明测试结果与未验证项

---

## 2. 项目概况

| 项 | 值 |
| --- | --- |
| 项目名 | Passport OS（FoloToy AI Passport 社区固件） |
| 上游仓库 | https://gitee.com/FoloToy/ai-passport |
| 本地路径 | `Z:\FOLOTOY\passport-os\passport-os-main\` |
| git 仓库根 | `Z:\FOLOTOY`（passport-os-main 是子目录，非独立仓库） |
| 当前分支 | `main`（working tree clean） |
| 平台 | ESP32-C3 · RISC-V 单核 · 8MB Flash · 无 PSRAM |
| 框架 | ESP-IDF 5.5.3 + LVGL |
| 显示 | ST7789 240×320 SPI 屏 |
| 输入 | 三键（UP/DOWN/OK，ADC 分压） |
| 构建环境 | WSL（`/opt/esp-idf`），Windows 侧无工具链 |
| 刷写 | Windows COM3（esptool 或 usbipd 绑定 WSL） |

---

## 3. 目录结构（关键路径）

```
Z:\FOLOTOY\passport-os\passport-os-main\
├── CMakeLists.txt              # 顶层构建
├── build.sh                    # 构建脚本（WSL）：bash build.sh release
├── partitions.csv              # 8MB 分区表
├── sdkconfig.defaults          # 默认配置
├── README.md / README.zh_CN.md # 项目说明
├── DESIGN.md / PRODUCT.md      # 设计与产品文档
├── main/
│   ├── main.cc                 # 入口
│   ├── CMakeLists.txt
│   ├── idf_component.yml
│   ├── apps/                   # 各应用实现
│   │   ├── app_base.h          # 应用接口定义（枚举 + passport_app_t）
│   │   ├── home/               # 首页（时钟/天气/名片）
│   │   ├── radio/              # 收音机
│   │   ├── voice/              # AI 随身语音
│   │   ├── xiaozhi/            # 小智 AI
│   │   ├── muse/               # Muse
│   │   ├── game/               # 你在狗叫什么
│   │   ├── sound_jump/         # 声音跳一跳
│   │   ├── stopwatch/          # 秒表
│   │   └── pomodoro/           # 番茄钟
│   ├── launcher/               # 应用抽屉/注册表
│   │   ├── app_registry.cc/.h  # 应用注册表 + 省电堆需求表
│   │   ├── launcher.cc/.h      # 启动器（图标/主题色/轮播）
│   │   ├── timer_resume.h
│   │   ├── wifi_setup.h
│   │   └── wake_gate.h
│   ├── ui/                     # 共享 UI 构件
│   │   ├── ui_pixel.h/.c       # 240×320 共享构件（面板/标签/页脚/吉祥物）
│   │   ├── ui_palette.h        # 语义颜色角色（UI_TEXT/UI_ACCENT…）
│   │   ├── display_text.h
│   │   ├── pixel_scene.h
│   │   └── ui_capture.c/.h     # 截屏
│   ├── fonts/                  # 字体
│   │   ├── app_fonts.h         # 字体注册
│   │   ├── buddy_font_16.c     # 思源黑体 16px（主字体）
│   │   ├── ui_font_20.c        # 20px
│   │   ├── home_font_12.c
│   │   ├── radio_font.c/.h
│   │   ├── radio_font_title.c
│   │   ├── voice_font_20.c
│   │   └── gen_buddy_font.sh   # 字体生成脚本
│   ├── audio_workspace.h
│   ├── boot_chime.c/.h         # 开机音
│   ├── boot_splash.c/.h        # 开机画面
│   └── ui_sfx.c/.h             # UI 音效
├── components/
│   ├── bsp/                    # 板级支持包
│   │   └── include/
│   │       ├── bsp_display.h   # 显示（含 LVGL 锁）
│   │       ├── bsp_button.h    # 按键
│   │       ├── bsp_audio.h     # 音频
│   │       ├── bsp_audio_workspace.h
│   │       ├── bsp_battery.h   # 电池
│   │       ├── bsp_i2c.h       # I2C
│   │       ├── bsp_memory.h    # 存储
│   │       └── bsp_pins.h      # 引脚
│   ├── esp-wifi-connect/
│   ├── noise_core/             # 噪声处理（被 muse 依赖）
│   └── passport_muse/          # Muse 组件
├── docs/                       # 文档
│   ├── README.en.md
│   ├── app-review-2026-09-12.md
│   ├── firmware-review-2026-09-15.md / .zh_CN.md
│   ├── home-page-plan.md
│   ├── open-source-audit-20261025.md
│   ├── open-source-guide-draft-20261025.md
│   ├── release-rc56.md
│   ├── travel-instrument-ui.md
│   ├── verification-rc56.json
│   ├── voice-udp-channel.md
│   └── xiaozhi-app.md
├── tests/                      # 测试
│   ├── check_launcher_apps.py
│   └── xiaozhi_mocks/
├── tools/                      # 工具（含 verify_firmware.py）
├── third_party/
├── bootloader_components/
└── dependencies.lock
```

---

## 4. 应用架构

### 4.1 应用接口（`main/apps/app_base.h`）

```c
typedef enum {
    APP_ID_LAUNCHER = 0,
    APP_ID_RADIO,
    APP_ID_VOICE,
    APP_ID_XIAOZHI,
    APP_ID_WIFI,
    APP_ID_SETTINGS,
    APP_ID_HOME,
    APP_ID_GAME,
    APP_ID_TETRIS,
    APP_ID_STOPWATCH,
    APP_ID_SOUND_JUMP,
    APP_ID_MUSE,
    APP_ID_POMODORO,
    APP_ID_COUNT
} passport_app_id_t;

typedef enum {
    BTN_EVT_CLICK = 0,
    BTN_EVT_PRESS,
    BTN_EVT_RELEASE,
    BTN_EVT_LONG,
    BTN_EVT_LONG_UP,
} app_btn_event_t;

typedef struct {
    passport_app_id_t id;
    const char *name;           // 中文显示名称
    const char *en_name;        // 英文标签
    const char *desc;           // 简述
    const char *tag;            // 协议/网络特性标签
    uint32_t theme_color;       // 主题色 (RGB888)
    void (*init)(void);         // 启动时初始化
    void (*start)(void);        // 进入前台
    void (*stop)(void);         // 退出前台（清理）
    void (*on_key)(uint8_t btn, app_btn_event_t event, uint16_t mv);
    bool (*prepare_stop)(void); // 协作式停止请求
} passport_app_t;
```

### 4.2 应用注册（`main/launcher/app_registry.cc`）

- 外部声明：`g_radio_app`、`g_voice_app`、`g_xiaozhi_app`、`g_wifi_app`、`g_settings_app`、`g_home_app`、`g_game_app`、`g_tetris_app`、`g_stopwatch_app`、`g_sound_jump_app`、`g_muse_app`、`g_pomodoro_app`
- Launcher 轮播数组 `s_apps[]`：radio、voice、xiaozhi、muse、game、sound_jump、tetris、stopwatch、pomodoro、settings
- 首页 `APP_ID_HOME` 不在轮播数组，通过 `app_registry_get_by_id()` 单独寻址
- `APP_ID_WIFI` 为隐藏系统设置，不显示为卡片

### 4.3 新增应用步骤

1. 在 `app_base.h` 枚举中新增 `APP_ID_XXX`（在 `APP_ID_COUNT` 之前）
2. 创建 `main/apps/xxx/` 目录，实现 `passport_app_t g_xxx_app`
3. 在 `app_registry.cc` 添加 `extern` 声明 + 加入 `s_apps[]` 数组
4. 在 `launcher.cc` 添加图标绘制（`draw_app_icon`）和主题色
5. 在 `main/CMakeLists.txt` 注册源文件
6. 若有省电堆需求，更新 `app_registry.cc` 省电堆需求表
7. 核对字体字形覆盖（中文文案）

---

## 5. BSP 接口（可复用）

| 接口 | 头文件 | 用途 |
| --- | --- | --- |
| 显示 | `bsp_display.h` | LVGL 初始化、锁、刷屏 |
| 按键 | `bsp_button.h` | 三键事件（UP/DOWN/OK） |
| 音频 | `bsp_audio.h` | 麦克风、扬声器、编解码 |
| 音频工作区 | `bsp_audio_workspace.h` | 音频缓冲管理 |
| 电池 | `bsp_battery.h` | 电量读取 |
| I2C | `bsp_i2c.h` | I2C 总线 |
| 存储 | `bsp_memory.h` | NVS、SPIFFS |
| 引脚 | `bsp_pins.h` | GPIO 定义 |

---

## 6. UI 框架

### 6.1 共享构件（`main/ui/ui_pixel.h`）

- `ui_pixel_background(scr)` — 终端背景
- `ui_pixel_screen_create(title)` — 创建屏幕
- `ui_pixel_panel_create(parent,x,y,w,h,color)` — 面板
- `ui_pixel_label(parent,text,font,color)` — 标签
- `ui_pixel_footer(parent,action,navigation,font)` — 双行页脚
- `ui_pixel_title_plate(parent,y,text,font)` — 标题牌
- `ui_pixel_hint_pill(parent,text,...)` — 提示药丸
- `ui_pixel_mascot_create/jump` — 吉祥物
- `ui_pixel_focus_feedback(obj)` — 焦点反馈
- `ui_pixel_set_selected(panel,selected,enabled)` — 选中态

### 6.2 语义颜色（`ui_palette.h`）

使用语义角色：`UI_TEXT`、`UI_ACCENT`、`UI_BG`、`UI_PANEL` 等，不硬编码颜色。

### 6.3 字体

| 字体 | 文件 | 用途 |
| --- | --- | --- |
| `buddy_font_16` | `buddy_font_16.c` | 思源黑体 16px，主字体 |
| `ui_font_20` | `ui_font_20.c` | 20px 标题 |
| `home_font_12` | `home_font_12.c` | 首页 12px |
| `radio_font` | `radio_font.c` | 收音机 |
| `voice_font_20` | `voice_font_20.c` | 语音 |

**⚠️ 中文文案必须核对字形覆盖**：字体是思源黑体子集，新增中文字符需用 `gen_buddy_font.sh` 重新生成，否则显示为方块。

---

## 7. 构建与刷写

### 7.1 构建（WSL）

```bash
cd /mnt/z/FOLOTOY/passport-os/passport-os-main
bash build.sh release
```

- 产物：`build-public/FoloToy-AI-Passport-full.bin`（合并固件，从 0x0 刷写）
- 校验：`tools/verify_firmware.py build-public --public`
- 其他模式：`bash build.sh`（本地版）、`bash build.sh clean`（清缓存）、`bash build.sh diag`（内存诊断）

### 7.2 分区表（8MB Flash）

| 分区 | 类型 | 偏移 | 大小 |
| --- | --- | --- | --- |
| nvs | data | 0x9000 | 0x6000 |
| phy_init | data | 0xf000 | 0x1000 |
| factory | app | 0x10000 | 0x400000 |
| cardid | data | 0x410000 | 0x4000 |
| storage | spiffs | 0x420000 | 0x200000 |

### 7.3 刷写（Windows）

```bash
esptool.py --port COM3 --baud 460800 write_flash 0x0 build-public/FoloToy-AI-Passport-full.bin
```

---

## 8. 现有应用清单

| ID | 应用 | 目录 | 联网 | 音频 |
| --- | --- | --- | --- | --- |
| HOME | 首页（时钟/天气/名片） | apps/home | 天气需 | 否 |
| RADIO | 收音机 | apps/radio | 是 | 是 |
| VOICE | AI 随身语音 | apps/voice | 是（UDP） | 是（ADPCM） |
| XIAOZHI | 小智 AI | apps/xiaozhi | 是（WS） | 是（Opus） |
| MUSE | Muse | apps/muse | 是 | 是 |
| GAME | 你在狗叫什么 | apps/game | 否 | 是（声控） |
| SOUND_JUMP | 声音跳一跳 | apps/sound_jump | 否 | 是 |
| TETRIS | 俄罗斯方块 | （枚举有，目录无） | 否 | 否 |
| STOPWATCH | 秒表 | apps/stopwatch | 否 | 否 |
| POMODORO | 番茄钟 | apps/pomodoro | 否 | 否 |

**注意**：`APP_ID_TETRIS` 在枚举和注册表中存在，但 `main/apps/` 下无 `tetris/` 目录。需确认 tetris 实现位置或是否为遗留。

---

## 9. 当前状态与进度

### 9.1 已完成

1. **项目复用确认**：本地 `passport-os/passport-os-main/` 即目标项目，无需重新克隆
2. **文档调研**：AGENTS.md 与 docs/README.md 不存在；已读取 README.md、README.zh_CN.md、DESIGN.md、PRODUCT.md、docs/ 下各文档
3. **Skill 检查**：`.codeartsdoer/skills/` 与 `agents/` 为空，项目未要求额外自定义 Skill
4. **平台/框架/接口调研**：已完成（见上文）
5. **应用架构调研**：已完成（app_base.h、注册表、launcher、UI 构件、字体）
6. **构建方式确认**：`build.sh release` → `FoloToy-AI-Passport-full.bin`

### 9.2 阻塞项（⚠️ 唯一）

**用户未明确"要做什么应用"**。用户原话"应用（一会单独补充）"。

需用户补充：
1. **应用功能描述**（一句话）
2. **新增应用 还是 改造现有应用？**
3. **是否联网 / 是否用音频？**
4. **交互特殊要求？**（无则按三键+LVGL 自主设计）

### 9.3 待办（解除阻塞后）

1. 生成 spec.md（需求规格）
2. 呈现 spec.md 摘要，询问用户是否修改
3. 生成 design.md（实现方案）
4. 生成 tasks.md（编码任务规划）
5. 新建开发分支
6. 实现应用代码
7. 核对中文字体覆盖
8. WSL 构建验证
9. 交付合并固件 + 测试结果 + 未验证项说明

---

## 10. 关键约束与注意事项

### 10.1 开发约束

- **保留已有修改**：当前 working tree clean，但需在新分支开发
- **新建分支**：不直接在 main 上开发
- **复用 BSP**：使用 `components/bsp/` 接口，不直接操作硬件
- **不照搬 demo**：重新设计应用界面
- **中文界面**：必须核对字体字形覆盖，用 `gen_buddy_font.sh` 生成
- **语义颜色**：使用 `ui_palette.h` 角色，不硬编码
- **共享 UI 构件**：优先用 `ui_pixel_*`，保持视觉一致

### 10.2 构建约束

- **必须在 WSL 中构建**：Windows 侧无 ESP-IDF 工具链
- **ESP-IDF 路径**：`/opt/esp-idf`（可用 `IDF_DIR` 环境变量覆盖）
- **刷写走 Windows COM3**

### 10.3 之前会话的教训（⚠️ 重要）

**编辑操作必须验证结果**：
- 之前会话声称完成的"6 个应用清理"（删除 voice/xiaozhi/muse/game/sound_jump/tetris）**实际上未生效**
- 当前 apps 目录仍含 game/muse/voice/xiaozhi/sound_jump，枚举仍含所有 APP_ID
- **规则**：每次 write/edit 后必须用 read 验证；关键修改用 grep/glob 验证残留引用

---

## 11. 给后续 AI 的建议

1. **先确认应用需求**：在用户补充"做什么应用"前，不要开始编码
2. **读取本文件**：本文件包含全部上下文，无需重新调研
3. **验证一切**：编辑后必验证，构建后必检查产物
4. **字体覆盖**：任何中文文案都要核对 `buddy_font_16` 字形覆盖
5. **分支策略**：从 main 新建分支，保留已有修改
6. **构建验证**：在 WSL 中 `bash build.sh release`，确认 `FoloToy-AI-Passport-full.bin` 生成且校验通过

---

## 12. 相关文档索引

| 文档 | 路径 | 用途 |
| --- | --- | --- |
| 项目说明 | `README.md` / `README.zh_CN.md` | 十款应用介绍、安装、改版 |
| 设计 | `DESIGN.md` | 设计理念 |
| 产品 | `PRODUCT.md` | 产品定义 |
| 应用评审 | `docs/app-review-2026-09-12.md` | 应用增删决策 |
| 固件评审 | `docs/firmware-review-2026-09-15.md` | 固件评审 |
| 首页规划 | `docs/home-page-plan.md` | 首页设计 |
| UI 指南 | `docs/travel-instrument-ui.md` | UI 设计规范 |
| 开源审计 | `docs/open-source-audit-20261025.md` | 开源合规 |
| 发布说明 | `docs/release-rc56.md` | RC56 发布 |
| 语音通道 | `docs/voice-udp-channel.md` | Voice UDP 协议 |
| 小智应用 | `docs/xiaozhi-app.md` | 小智 AI 设计 |
