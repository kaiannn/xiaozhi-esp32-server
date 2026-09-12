# Desk Mate 产品需求文档 v2（详细版）

| 字段 | 值 |
|------|-----|
| 产品代号 | Desk Mate |
| 设备 | Waveshare ESP32-S3-ePaper-1.54 **V2**（ESP32-S3-PICO-1-N8R8，8MB Flash / 8MB PSRAM） |
| 主仓库 | `kaiannn/xiaozhi-esp32-server`（本仓 fork 自 xinnan-tech） |
| 设备仓 | `kaiannn/xiaozhi-esp32` |
| 文档分支 | `product/desk-mate` |
| 状态 | 需求基线 v2 · 可拆任务实现 |
| Owner | kaiannn |

---

## 0. 修订

| 版本 | 日期 | 说明 |
|------|------|------|
| v1 | 2026-09-12 | 初版模块列表 |
| v2 | 2026-09-12 | 补全旅程、数据模型、API、非功能、边界与验收矩阵 |

---

## 1. 产品定义

### 1.1 一句话

**Desk Mate = 墨水屏桌面语音终端**：按一下练英语，看一眼背单词，说一句控家居；业务大脑私有部署在本 Server。

### 1.2 成功标准（产品级，非工程）

| 指标 | 目标（上线后 4 周） |
|------|---------------------|
| 口语 | 每周 ≥3 次、每次 ≥5 分钟完整 session |
| 背词 | 每日 due 清空率 ≥60% |
| 家居 | 常用 3 个场景语音成功率 ≥95%（无人工修 alias） |
| 可用性 | 插电桌面场景下，Server 崩溃不影响设备安全（可回待机） |
| 工程 | v0.1 mock 路径 CI 绿；v0.2 起有 golden 对话夹具 |

### 1.3 用户与环境

| 项 | 假设（可改，改则回写本表） |
|----|---------------------------|
| 用户 | 单用户自用（kaiannn），非多租户 SaaS |
| 供电 | 优先插电；电池模式为增强项 |
| 网络 | 局域网 Wi-Fi；Server 在同一 LAN 或可路由主机 |
| 外设 | 板载麦/喇叭；BOOT / PWR；可选外接编码器 |
| 屏 | 200×200 单色 ePaper；全刷慢，局刷可接受残影 |
| 语言 | 界面中英混排；Tutor 输出英文为主 |

### 1.4 非目标（P0 阶段明确排除）

1. 多用户账号体系 / 订阅计费  
2. 公网无鉴权暴露  
3. 端侧 LLM / 端侧音素发音评分  
4. 全双工 AEC、连续自动多轮（无 PTT）  
5. 完整家庭自动化规则引擎（规则在 HA/外设侧）  
6. OTA 应用商店、皮肤商城  
7. 米家私有协议完整栈（无明确设备清单前）  
8. 摄像头视觉、表情动画（板无彩屏）

---

## 2. 系统架构

### 2.1 逻辑视图

```text
┌───────────── Device (ESP32-S3 ePaper) ─────────────┐
│ 音频 I2S · 按键 · ePaper · Wi-Fi/BLE 配网           │
│ 模式状态机: idle|listening|thinking|speaking|busy   │
└───────────────────────┬────────────────────────────┘
                        │ WS 或 MQTT+UDP（优先沿用上游）
                        ▼
┌────────────── Desk Mate Server (本仓) ─────────────┐
│ 会话层 Session/ModeRouter                           │
│ 能力层 ASR · LLM · TTS · 可插拔供应商               │
│ 业务层 Tutor · Vocab · Home · Listen                │
│ 平台层 Screen cards · Device registry · Events DB  │
└───────────┬───────────────────────┬────────────────┘
            │                       │
            ▼                       ▼
     外部模型 API            Home Provider
     (STT/LLM/TTS)        (MQTT | HA REST | mock)
```

### 2.2 部署形态

| 形态 | 说明 | 阶段 |
|------|------|------|
| D1 开发机 | Server 跑 Mac/PC，设备 USB/Wi-Fi | v0.1 |
| D2 家用常驻 | Server 跑 NAS/小主机 systemd/docker | v0.4+ |
| D3 官方云对照 | 设备可切 xiaozhi.me（对照/回退） | 可选 |

### 2.3 固件 / Server 责任边界

| 职责 | 设备 | Server |
|------|------|--------|
| 音频采集/播放/PA | ✓ | |
| VAD 可选本地 | 可选 | 必备逻辑 |
| STT/LLM/TTS | | ✓ |
| 纠错 JSON、SRS、意图 | | ✓ |
| 卡片渲染字号/截断 | 可选协助 | ✓ 语义 |
| 全刷/局刷策略 | ✓ | 给 refresh 标记 |
| 密钥 | 仅 device 凭据 | 模型 key / HA token |
| 配网 | ✓ | 可推送 Wi-Fi 提示 |

---

## 3. 模式与全局交互

### 3.1 模式矩阵

| mode | 入口 | 主交互 | 屏主内容 | 可退出 |
|------|------|--------|----------|--------|
| `chat` | 默认/长按 | 自由聊天 | 当前轮摘要 | 任意 |
| `tutor` | 语音/配置/键 | PTT 英语陪练 | 角色 + you + tip | `exit tutor` |
| `vocab` | 语音/键 | 抽查背词 | 词/义/进度 | `next session` |
| `home` | 语音/长按 | 家居指令 | 原句 + OK/FAIL | 自动回 idle |
| `listen` | 语音 | 听力题 | 题干选项 | 做完 |

### 3.2 模式切换规则

1. 任意模式下用户说/按「模式词」→ 切换；进行中 TTS 打断。  
2. Home 执行失败不离开 home 模式；连续 3 次失败 footer 提示改 alias。  
3. Tutor session 结束（5 分钟到点）→ 回 `idle` 待机屏（含 due 词数）。  
4. 设备重启后从 Server 拉 `device.mode` 或本地 NVS 缓存。  

### 3.3 待机屏（idle）

```text
Desk Mate          09:41
Vocab due 12
Home: 书房灯 ON
Mode: idle · v0.3.0
```

---

## 4. 用户旅程（端到端）

### J1 口语陪练（Tutor）

| 步 | 用户 | 系统 | 屏 | 可失败点 |
|----|------|------|-----|----------|
| 1 | 切到 tutor | 加载角色 prompt | `TUTOR · cafe` | 无角色配置 |
| 2 | 按住 BOOT 说 | listening → 收音 | `listening` | 过短语音 |
| 3 | 松开 | STT | `YOU: …` 先刷 | STT 超时 |
| 4 | 等待 | LLM + 纠错 JSON | `thinking` | JSON 解析失败→降级 |
| 5 | 听回复 | TTS | `TIP: better…` | TTS 失败→纯文本播报或静默 |
| 6 | 再说/结束 | 循环或 summary | Score 行 | |

### J2 背词（Vocab）

| 步 | 行为 |
|----|------|
| idle 显示 due N → 说「背单词」/键进入 vocab |
| 抽 1 张：显示中文，问英文 → ASR 匹配 |
| good → 间隔↑，下一题；again → 明日/更短 |
| 会话 10 词或用户说停 → 报「今天复习 X，对 Y」 |
| 状态立即写 DB，不靠设备 |

### J3 家居（Home）

| 步 | 行为 |
|----|------|
| 说「关书房灯」 |
| 规则解析 → target=`书房灯` action=off |
| Provider 调用（HA/MQTT） |
| 屏：`> 关书房灯` / `OK 120ms` 或 `FAIL timeout` |
| 可选 TTS 确认 |

### J4 听力（Listen）

| 步 | 行为 |
|----|------|
| 「听力练习」→ 出 1 题（题库或生成） |
| 播句 2 遍（可 0.8×） |
| A/B/C 或说答案 → 判分 → 显示正确句 |

---

## 5. 功能需求（详细）

### M1 平台 / 接入

#### F1.1 设备注册与会话

- 沿用上游鉴权模型，扩展：`device_name`、`fw_version`、`caps`（has_epaper, has_ptt）。  
- 会话 `desk_sessions` 与上游 chat session 关联或独立。  
- 心跳 30s；3 次失败重连 1–60s 退避。  

#### F1.2 传输

- 优先上游 WebSocket 文档化路径；若上游默认 MQTT，Desk 扩展消息放在 MQTT 属性主题，避免平行协议分叉。  
- 音频仍走上游 Opus 管道。  

#### F1.3 模式路由

- 输入：`mode.set` 上行 或 Server 侧意图。  
- 每个 mode 绑定：`system_prompt`、`tools`、`on_tts`、`on_key`。  

#### F1.4 屏幕卡片

见 §6.1 协议。Server 是唯一语义源。  

#### F1.5 供应商配置

```yaml
asr: { provider: openai_whisper, model: whisper-1 }
llm: { provider: openai_compat, base_url: ..., model: qwen... }
tts: { provider: edge, voice: en-US-AriaNeural }
```

密钥仅环境变量。  

### M2 Tutor

#### F2.1 PTT 半双工

- 按键映射：BOOT=PTT；配置可改。  
- 最短语音 300ms；最长 30s。  
- 空结果：不进 LLM。  

#### F2.2 角色包

| id | 情境 | 输出侧重 |
|----|------|----------|
| daily | 朋友 | 自然、B1 |
| cafe | 点单 | 礼貌、数量 |
| interview_b1 | 面试 | 过去时、完整句 |
| custom | 配置文件自定义 | 用户填 |

#### F2.3 纠错契约（LLM 必须可 JSON）

```json
{
  "score_fluency": 78,
  "score_accuracy": 70,
  "issue": "used 'get' instead of 'have'",
  "better_sentence": "Could I have a large latte, please?",
  "keywords": ["have", "large"]
}
```

- 解析失败：整轮降级为 `issue=null`，仍要 TTS 对话。  

#### F2.4 五分钟模式

- `tutor.length_minutes` 默认 5。  
- 结束简报写入 `desk_events`，上屏 1 张 summary card。  

#### F2.5 命令词表（后续）

`repeat`, `slower`, `easier`, `exit tutor`, `score`…  

### M3 Vocab

#### F3.1 词库

- Deck：`id, name, lang, tags`  
- Card：`id, deck_id, word, phonetic, meaning, example, example_zh`  
- 导入：CSV/JSON；`on_conflict`  

示例 CSV：

```csv
word,phonetic,meaning,example,example_zh
latte,/ˈlɑːte/,拿铁,I'd like a latte.,我想要一杯拿铁。
```

#### F3.2 SRS

- 默认轻量 SM-2：`again|hard|good|easy` → 间隔秒数表可配。  
- 状态机：`new → learning → review`；`again → learning`。  
- 排序：`due_at ASC`，同 due 随机。  

#### F3.3 抽查

| 类型 | 提示 | 自动判分 |
|------|------|----------|
| en_from_zh | 显示中文，听用户英文 | ASR 与 word 编辑距离/相似度 |
| zh_from_en | 显示英文，用户中文或自评 | 中文 ASR 或按键自评 |
| listen_blank | 播例句挖空 | 选词/说词 |

- 相似度阈值可配，默认 `0.85` 归一化。  
- 语音失败时按键自评兜底（A 过 / B 难 / C 忘了）。  

#### F3.4 推送

- 定时（默认每小时）或模式空闲时推 `due N` 卡片。  

### M4 Home

#### F4.1 Provider

见 issue #17/#18。  

#### F4.2 别名表 `home_alias.yaml`

```yaml
aliases:
  书房灯: { provider: ha, entity: light.study }
  客厅主灯: { provider: ha, entity: light.living_main }
scenes:
  阅读: [书房灯 ON, 台灯 ON]
  睡眠: [all_lights OFF]
```

#### F4.3 意图规则（第一版）

正则 + 最长匹配；失败可选 LLM JSON：  
`{"action":"turn_on","target":"书房灯"}`  

#### F4.4 结果

`Result.ok` → 屏 OK + 可选 TTS；`FAIL` → 短码：`timeout|not_found|auth|denied`。  

#### F4.5 状态订阅

- HA 状态 / MQTT retain → 节流 30s 内最多刷 1 次 idle/home 卡片。  

### M5 Listen

- 题库：`docs/listen_samples.json` 内置 ≥20 句。  
- 生成模式：LLM 出 1 道三选一（P2 可做）。  

---

## 6. 契约与数据

### 6.1 Screen card v1

```json
{
  "type": "screen.card",
  "refresh": "full|partial",
  "card": {
    "title": "string",
    "lines": [{"role": "you|tip|word|mean|result|meta", "text": "string"}],
    "footer": "string",
    "progress": 0.0
  }
}
```

约束：

- `title ≤ 24 字符（半角当量）`  
- `lines` 最多 3 行；每行建议 ≤ 40 半角当量；Server 截断加 `…`  
- UTF-8；避免控制字符  

事件上行：`input.key`, `mode.current`, `screen.ack`  

### 6.2 数据库（Desk 扩展）

| 表 | 关键列 |
|----|--------|
| desk_devices | device_id PK, secret_hash, name, mode, caps_json, last_seen |
| desk_sessions | id, device_id, mode, started_at, ended_at, meta_json |
| desk_events | id, session_id, kind, payload_json, ts |
| desk_decks | id, name, lang, tags_json |
| desk_cards | id, deck_id, word, phonetic, meaning, example, example_zh, state, due_at, ease, reps |
| desk_reviews | id, card_id, session_id, grade, score, ts |

原则：不改坏上游表；迁移可重复执行。  

### 6.3 内部 API（Server）

| 方法 | 路径 | 用途 |
|------|------|------|
| POST | /api/desk/vocab/decks | 建 deck |
| POST | /api/desk/vocab/decks/{id}/import | 导入 |
| GET | /api/desk/vocab/decks/{id}/due?limit=10 | 取 due |
| POST | /api/desk/vocab/cards/{id}/grade | 打分 |
| POST | /api/desk/home/invoke | 意图/显式调用 |
| GET | /api/desk/devices/{id}/screen | 调试：最后卡片 |
| GET | /api/desk/health | 健康检查 |

鉴权：与上游管理端一致或 Bearer 内网 token。  

### 6.4 错误码（屏显短码）

| code | 含义 |
|------|------|
| ok | 成功 |
| timeout | 上游/设备超时 |
| stt_empty | 空识别 |
| llm_json | 纠错 JSON 失败 |
| tts_fail | 合成失败 |
| not_found | 设备/实体不存在 |
| alias_miss | 未命中家居别名 |
| provider_err | Provider 内部错误 |

---

## 7. 非功能需求

| 类别 | 要求 |
|------|------|
| 延迟 | 局域网 Tutor：松开 PTT → 开始 TTS **&lt; 2s**（可配模型，指标先记录） |
| 家居 | 解析+调用 **&lt 2.5s** 至屏幕 OK/FAIL |
| 可靠 | 单 Provider 失败不影响其他 mode；进程崩溃可重启恢复（DB 落盘） |
| 安全 | LLM/HA 密钥不进 git；设备 secret hash 存储；LAN 默认防火墙 |
| 资源 | Server 常驻 &lt 512MB RSS 目标；词库 5k 卡以内 sqlite 足够 |
| 测试 | 业务纯逻辑单测；Provider mock；golden JSON 夹具 |
| 可观测 | `desk_events` + 结构化 log：session_id, mode, kind, latency_ms |
| 兼容 | 定期 merge 上游；业务放在 `desk/` 子包尽量少侵入 |

---

## 8. 界面内容规范（Server 负责的文案）

| 场景 | title | lines 示例 | footer |
|------|-------|------------|--------|
| Tutor 听 | TUTOR·cafe | you: I'd like… | listening |
| Tutor 纠 | TUTOR·cafe | tip: Could I have… | Score 78 |
| Vocab | VOCAB·CET4 | word: latte / mean: 拿铁 | due 8 · 3/10 |
| Home OK | HOME | > 关书房灯 / OK | 120ms |
| Home FAIL | HOME | > 关书房灯 / FAIL timeout | alias_miss? |
| Listen | LISTEN | Q: … / A. … | 0.8x · 2 plays |

---

## 9. 验收矩阵（发版门禁）

| 门禁 | v0.1 | v0.2 | v0.3 | v0.4 | v0.5 |
|------|------|------|------|------|------|
| CI unit | ✓ | ✓ | ✓ | ✓ | ✓ |
| mock client 握手+卡片 | ✓ | ✓ | ✓ | ✓ | ✓ |
| 真机 ePaper 显示卡片 | ✓ | ✓ | ✓ | ✓ | ✓ |
| Tutor 5 轮真语音 | | ✓ | | | |
| Vocab 10 词 session | | | ✓ | | |
| HA/MQTT 开关灯真机 | | | | ✓ | |
| 听力 1 题完整 | | | | | ✓ |
| 文档与 PRD 链接有效 | ✓ | ✓ | ✓ | ✓ | ✓ |

HIL 清单见 `docs/hil-checklist.md`（独立 issue 产出）。  

---

## 10. 开放问题（需拍板）

1. 传输：继续 WebSocket 还是 MQTT+UDP 为主？  
2. 默认 LLM 供应商与是否允许云？  
3. 家居默认 Provider：HA 还是先 MQTT 仿真？  
4. 单词是否要 TTS 例句音频缓存到 TF 卡？  
5. 是否保留一键切回官方 xiaozhi.me？  

---

## 11. 与 Issue 的映射

| 模块 | Issues |
|------|--------|
| Epic 平台 | #1, #6–#10, #26 + 新增平台细化 |
| Tutor | #2, #11–#13, #25 |
| Vocab | #3, #14–#16 |
| Home | #4, #17–#21, #24 |
| Listen | #5, #22–#23 |
| 固件 | 另见 `kaiannn/xiaozhi-esp32` |

---

## 12. 参考

- 小智上游固件：https://github.com/78/xiaozhi-esp32  
- 上游 Server：https://github.com/xinnan-tech/xiaozhi-esp32-server  
- 板卡：https://docs.waveshare.net/ESP32-S3-ePaper-1.54  
- 原型：`/Users/kai/Developer/smart-hub-prototype`  
