# Desk Mate Issue 全量清单

> 自动生成备忘 · PRD v2: docs/PRD.md · 分支 product/desk-mate
> 正式跟踪以 GitHub Issues 为准。

| # | 状态 | Milestone | 标题 |
|---|------|-----------|------|
| 1 | OPEN | v0.1-connect | [Epic] 设备接入与屏幕协议（Desk Mate 平台） |
| 2 | OPEN | v0.2-tutor | [Epic] 英语口语 Tutor |
| 3 | OPEN | v0.3-vocab | [Epic] 背单词 SRS |
| 4 | OPEN | v0.4-home | [Epic] 家居语音中控 |
| 5 | OPEN | v0.5-loop | [Epic] 听力训练 |
| 6 | OPEN | v0.1-connect | [feat] 设备鉴权注册与断线重连 |
| 7 | OPEN | v0.1-connect | [feat] 墨水屏卡片协议 v1 |
| 8 | OPEN | v0.1-connect | [feat] 会话模式路由 tutor/vocab/home/chat |
| 9 | OPEN | v0.1-connect | [chore] ASR/LLM/TTS 供应商可配置 |
| 10 | OPEN | v0.1-connect | [docs] Desk Mate 协议与模式文档 |
| 11 | OPEN | v0.2-tutor | [feat] Tutor：PTT 半双工对讲与识别上屏 |
| 12 | OPEN | v0.2-tutor | [feat] Tutor：三套场景角色与纠错 JSON |
| 13 | OPEN | v0.2-tutor | [feat] Tutor：每日 5 分钟模式 |
| 14 | OPEN | v0.3-vocab | [feat] Vocab：Deck 与卡片导入 API |
| 15 | OPEN | v0.3-vocab | [feat] Vocab：SRS 调度与三种抽查 |
| 16 | OPEN | v0.3-vocab | [feat] Vocab：due 词数与卡片上屏 |
| 17 | OPEN | v0.4-home | [feat] Home：Provider 抽象与 MQTT 最小实现 |
| 18 | OPEN | v0.4-home | [feat] Home：Home Assistant REST Provider |
| 19 | OPEN | v0.4-home | [feat] Home：中文开/关灯意图规则解析 |
| 20 | OPEN | v0.4-home | [feat] Home：执行结果 OK/FAIL 上屏与播报 |
| 21 | OPEN | v0.4-home | [feat] Home：灯状态订阅刷新屏 |
| 22 | OPEN | v0.5-loop | [feat] 听力：播句 + ABC 选择题 |
| 23 | OPEN | v0.5-loop | [feat] 听力：语速 0.8× 与音频缓存重播 |
| 24 | OPEN | v0.5-loop | [feat] LAN 场景快捷 API（手机/电脑） |
| 25 | OPEN | v0.5-loop | [feat] Tutor 常用命令词表 |
| 26 | OPEN | v0.1-connect | [chore] Session 结构体与数据库表设计 |
| 27 | OPEN | v0.1-connect | [docs] HIL 真机验收清单（发版门禁） |
| 28 | OPEN | v0.1-connect | [chore] ASR/LLM/TTS/Mock 供应商与 CI 零成本联调 |
| 29 | OPEN | v0.1-connect | [chore] desk/ 子包骨架与上游侵入边界 |
| 30 | OPEN | v0.1-connect | [feat] 错误码目录与 desk_events 埋点规范 |
| 31 | OPEN | v0.1-connect | [feat] idle 待机屏（时间 · due · 灯状态 · 模式） |
| 32 | OPEN | v0.4-home | [feat] Home 别名与场景 YAML 配置 |
| 33 | OPEN | v0.4-home | [feat] Home：alias 未命中时的澄清追问 |
| 34 | OPEN | v0.3-vocab | [feat] Vocab：示例词库 seed + 会话摘要报告 |
| 35 | OPEN | v0.2-tutor | [feat] Tutor：角色 prompt 文件化（可自定义） |
| 36 | OPEN | v0.1-connect | [test] Golden 夹具：Tutor/Vocab/Home 纯逻辑 |
| 37 | OPEN | v0.5-loop | [feat] Listen：内置题库 seed JSON |
| 38 | OPEN | v0.4-home | [chore] Desk Mate docker-compose 一键启动 |
| 39 | OPEN | v0.5-loop | [feat] Tutor 延迟埋点（可配置观测） |
| 40 | OPEN | v0.1-connect | [feat] 设备 caps 协商与无屏/无麦降级 |
| 41 | OPEN | v0.4-home | [chore] 安全基线：密钥、token、设备 secret |

## 固件仓

| # | 标题 |
|---|------|
| 1 | 确认 ePaper-1.54 V2 板级可编译可出音频 |
| 2 | 消费 screen.card 协议 |
| 3 | PTT 键位与模式环 |
| 4 | DESK-MATE 固件集成说明 |

## 建议开工顺序（v0.1）

1. #29 desk 包骨架
2. #26 数据模型
3. #28 mock 供应商
4. #7 屏幕协议文档
5. #6 设备鉴权/重连
6. #8 模式路由
7. #30 错误码与埋点
8. #36 golden 测试
9. #27 HIL 清单
10. #10/#31 协议文档与 idle 屏

固件并行：#1 → #2 → #3
