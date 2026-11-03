# 进度跟踪 · FOLOTOY AI Passport 应用开发

> 最后更新：2026-11-03

## 当前阶段

**第一阶段：需求规格设计** — 应用需求已明确（IDEA.md），即将调用 spec-requirement-agent

## 阶段流程

1. ✅ 需求规格设计（spec-requirement-agent）— spec.md 已生成（.codeartsdoer/specs/ai_passport_apps/spec.md，612行），待用户确认
2. ⬜ 实现方案创建（spec-design-agent）— 待生成 design.md
3. ⬜ 编码任务规划（spec-task-agent）— 待生成 tasks.md
4. ⬜ 编码任务执行 — 待实现
5. ⬜ 构建验证 — 待 WSL 构建
6. ⬜ 固件交付 — 待产出合并固件

## 应用需求（已明确）

7 款应用：首页、收音机、AI 随身语音、Muse、俄罗斯方块、秒表、番茄钟
系统层：4 主题、音量亮度调节、自动熄屏短按唤醒、Wi-Fi 配网、电池显示与功耗管理

## 已完成

- [x] 项目复用确认（本地 passport-os-main 即目标项目）
- [x] 文档调研（AGENTS.md/docs/README.md 不存在，已读替代文档）
- [x] Skill 检查（项目无自定义 Skill 要求）
- [x] 平台/框架调研（ESP32-C3 · ESP-IDF 5.5.3 · LVGL）
- [x] 应用架构调研（app_base.h、注册表、launcher）
- [x] BSP 接口调研（display/button/audio/battery/i2c/memory）
- [x] UI 框架调研（ui_pixel、ui_palette、字体）
- [x] 构建方式确认（build.sh release → FoloToy-AI-Passport-full.bin）
- [x] 交接文件生成并更新（/wen/HANDOVER.md）
- [x] IDEA.md 应用需求读取确认（7 款应用 + 5 项系统层功能）

## 待办

- [x] 调用 spec-requirement-agent 生成 spec.md（.codeartsdoer/specs/ai_passport_apps/spec.md，612行）
- [ ] 呈现 spec.md 摘要，询问用户是否修改
- [ ] 生成 design.md
- [ ] 生成 tasks.md
- [ ] 新建开发分支
- [ ] 实现应用代码 + 系统层功能
- [ ] 核对中文字体覆盖
- [ ] WSL 构建验证
- [ ] 交付合并固件 + 测试结果 + 未验证项说明
