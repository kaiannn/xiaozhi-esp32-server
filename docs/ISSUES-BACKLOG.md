# Issue 草稿清单（Desk Mate）

> 复制标题/正文到 GitHub Issues 即可。标签建议：`type:feature` / `type:task` / `prio:P0|P1|P2`。  
> 先建 **Milestone**：`v0.1-connect` `v0.2-tutor` `v0.3-vocab` `v0.4-home` `v0.5-loop`。

---

## Epic / 主题（先建，便于关联）

### [Epic] 设备接入与屏幕协议
- 里程碑：v0.1-connect
- 优先级：P0
- 描述：定义并实现 Server↔ePaper 设备的最小控制面：上行文本/事件，下行卡片指令。不依赖具体家居后端。

### [Epic] 英语口语 Tutor
- 里程碑：v0.2-tutor
- 优先级：P0
- 描述：PTT 对讲、角色 prompt、单行纠错、5 分钟模式。

### [Epic] 背单词 SRS
- 里程碑：v0.3-vocab
- 优先级：P0
- 描述：词库导入、间隔重复、语音/按键抽查、due 推送。

### [Epic] 家居语音中控
- 里程碑：v0.4-home
- 优先级：P1
- 描述：意图层 + Provider 抽象 + 结果回显；后端可换。

### [Epic] 听力训练
- 里程碑：v0.5-loop
- 优先级：P1
- 描述：播句、选择题、语速、可选复述。

---

## 可直接开的 Issue

### [feat] Server 与设备最小握手协议
- Epic：设备接入与屏幕协议
- 里程碑：v0.1-connect / 优先级：P0
- 用户故事：作为开发者，我希望设备注册后 Server 能下发一行状态，以便确认链路可用。
- 验收标准：
  - [ ] 设备以 token/设备号连接（沿用上游鉴权或等价）
  - [ ] Server 可推送 `screen.text`（标题+一行正文）
  - [ ] 设备 30s 心跳，断线重连指数退避
  - [ ] 集成测试：无设备时用 mock client 通过

### [feat] 墨水屏卡片协议 v1（行级文本 + 刷新标记）
- Epic：设备接入与屏幕协议
- 里程碑：v0.1-connect / 优先级：P0
- 用户故事：作为 Tutor/Home，我希望 Server 用同一套卡片格式刷屏，以便固件不必为每个功能定制 UI。
- 验收标准：
  - [ ] 卡片字段：`title` / `lines[]` / `footer` / `refresh: full|partial`
  - [ ] 文档写入 `docs/screen-protocol.md`
  - [ ] 固件 fork 或 prototype 有示例消费端（可另开固件 issue）

### [feat] 会话模式路由 tutor / vocab / home / chat
- Epic：Epic 总览见 PRD M2–M5
- 里程碑：v0.1-connect / 优先级：P0
- 验收标准：
  - [ ] 语音或配置可切换模式
  - [ ] 模式切换后 system prompt + 工具集切换
  - [ ] 屏幕角标或 footer 显示当前模式

### [feat] 英语 Tutor：PTT 轮次 + 识别结果上屏
- Epic：英语口语 Tutor
- 里程碑：v0.2-tutor / 优先级：P0
- 验收标准：
  - [ ] 用户话音结束后 STT 完整句先刷屏
  - [ ] LLM 回复 TTS 播放
  - [ ] 全程延迟（本机/局域网）目标 &lt; 2s 至开始播报
  - [ ] 支持打断：新的 PTT 停止当前 TTS

### [feat] 英语 Tutor：三套场景角色 prompt
- Epic：英语口语 Tutor
- 里程碑：v0.2-tutor / 优先级：P0
- 验收标准：
  - [ ] `daily` / `cafe` / `interview_b1` 可选
  - [ ] 每轮结束输出 JSON：`score_fluency`, `score_accuracy`, `issue`, `better_sentence`（表达级）
  - [ ] `better_sentence` 与 `issue` 读出并上屏

### [feat] 英语 Tutor：每日 5 分钟模式
- Epic：英语口语 Tutor
- 里程碑：v0.2-tutor / 优先级：P1
- 验收标准：
  - [ ] 开始后倒计时；结束简报：轮次、平均分、最长 issues
  - [ ] 简报可读可屏显

### [feat] 词库 CSV/JSON 导入 API
- Epic：背单词 SRS
- 里程碑：v0.3-vocab / 优先级：P0
- 验收标准：
  - [ ] `POST /api/vocab/decks` 创建 deck
  - [ ] 导入字段：word, phonetic?, meaning, example?
  - [ ] 重复词冲突策略可配（skip/overwrite）

### [feat] SRS 调度与抽查会话
- Epic：背单词 SRS
- 里程碑：v0.3-vocab / 优先级：P0
- 验收标准：
  - [ ] 每日 due 队列
  - [ ] 三种抽查：中译英 / 英译中 / 听例句
  - [ ] 答对/答错更新 next_review
  - [ ] 算法文档链接（FSRS 或 SM-2 变体）

### [feat] 将 due 词数推送到设备屏
- Epic：背单词 SRS
- 里程碑：v0.3-vocab / 优先级：P1
- 验收标准：
  - [ ] 待机或整点推送 `Vocab due: N`
  - [ ] 与 tutor 简报可共用卡片协议

### [feat] 家居 Provider 抽象 + MQTT 遥控最小实现
- Epic：家居语音中控
- 里程碑：v0.4-home / 优先级：P1
- 验收标准：
  - [ ] 接口 `turn_on(device) / turn_off(device) / scene(name)`
  - [ ] 至少一个实现：向指定 MQTT topic 发 JSON
  - [ ] 单元测试 mock broker

### [feat] 家居 Provider：Home Assistant REST
- Epic：家居语音中控
- 里程碑：v0.4-home / 优先级：P1
- 验收标准：
  - [ ] `POST /api/services/light/turn_on|turn_off`
  - [ ] token 从环境变量读取
  - [ ] 映射表：中文房间/灯名 → entity_id（可配置 YAML/表）

### [feat] 家居意图：开/关 + 灯 + 房间 的规则解析
- Epic：家居语音中控
- 里程碑：v0.4-home / 优先级：P1
- 验收标准：
  - [ ] 正则/词表覆盖：开、关、打开、关闭 + 书房/客厅… 
  - [ ] 失败时返回结构化 error 供上屏
  - [ ] 置信度低时可转 LLM 兜底（可开关）

### [feat] 家居结果回显（OK/FAIL）上屏
- Epic：家居语音中控
- 里程碑：v0.4-home / 优先级：P1
- 验收标准：
  - [ ] 屏显原句 + 结果
  - [ ] TTS 播报「已打开书房灯」/「失败：找不到设备」

### [feat] 设备状态订阅（灯状态变更刷新屏）
- Epic：家居语音中控
- 里程碑：v0.4-home / 优先级：P1
- 验收标准：
  - [ ] MQTT retain 或 HA 状态变化 → 推送卡片
  - [ ] 带时间戳「Updated HH:MM」避免过期误导

### [feat] 听力：播句 + ABC 选择题
- Epic：听力训练
- 里程碑：v0.5-loop / 优先级：P1
- 验收标准：
  - [ ] 预置题库或 LLM 生成 1 题
  - [ ] 播 1–2 遍；按键/语音选答案
  - [ ] 对错反馈 + 可重播

### [feat] 听力：语速 0.8× 与重播
- Epic：听力训练
- 里程碑：v0.5-loop / 优先级：P1
- 验收标准：
  - [ ] 播放速度可设
  - [ ] 重播不重新生成（缓存音频）

### [feat] LAN 场景快捷 API（手机/电脑）
- Epic：家居语音中控
- 里程碑：v0.5-loop / 优先级：P2
- 验收标准：
  - [ ] `POST /api/scenes/{name}` 带共享密钥
  - [ ] 文档给出快捷指令示例

### [chore] 模型供应商可配置化（ASR/LLM/TTS）
- 里程碑：v0.1-connect / 优先级：P0
- 验收标准：
  - [ ] 环境变量或 yaml 切换供应商
  - [ ] 至少一条默认开发配置可跑通

### [docs] Screen protocol & mode router 说明
- 里程碑：v0.1-connect / 优先级：P0
- 验收标准：
  - [ ] `docs/screen-protocol.md`
  - [ ] `docs/modes.md`
  - [ ] PRD 链接正确

### [chore] 保持 main 可合并上游的分支策略说明
- 里程碑：v0.1-connect / 优先级：P1
- 验收标准：
  - [ ] `docs/CONTRIBUTING-UPSTREAM.md` 说明如何定期 merge `xinnan-tech/xiaozhi-esp32-server`

---

## 固件仓 `kaiannn/xiaozhi-esp32` 建议 Issue（可另开）

### [feat] 确认 ESP32-S3-ePaper-1.54 V2 板级目录可用
- 优先级：P0
- 验收：官方变体或自定义 board 能编译；音频/屏引脚与 V2 原理图一致

### [feat] 消费 Server 卡片协议（全刷/局刷）
- 优先级：P0
- 依赖：Server `screen-protocol.md`
- 验收：Tutor 两行提示 + Home OK/FAIL 均可显示

### [feat] PTT 按键语义：BOOT=说话，长按切换模式
- 优先级：P1

### [bug/chore] 若上游板支持不完整：补 ePaper 初始化与字体
- 优先级：P0（视编译结果）

---

## 建议创建顺序

1. Milestones  
2. Labels（`type:feature` 等）  
3. 5 个 Epic  
4. v0.1 全部 `feat/docs/chore`（握手、协议、模式、供应商、文档）  
5. v0.2 Tutor 三条  
6. 其余按里程碑加 backlog 标签  

## 不要先提进池子的

- 音素级发音评分  
- 端侧大模型  
- 米家完整协议栈（无明确设备清单前）  
- OTA 商城皮肤  
