
# SnapDo iOS 平台适配方案

## 平台迁移总览

| 维度 | Android/OriginOS（原方案） | iOS 18（适配方案） |
|------|--------------------------|---------------------|
| 操作系统 | OriginOS 6 (Android 15) | iOS 18 |
| AI 引擎 | 蓝心大模型（端侧3B） | Apple Intelligence + Core ML |
| 语音助手 | 小V / 蓝心小V | Siri + App Intents |
| 意图框架 | vivo 意图框架 2.0 (MCP 协议) | App Intents + Shortcuts |
| 通知管理 | 系统通知流监听 | UNNotificationServiceExtension + Notification Center |
| 悬浮交互 | 原子岛 / 快日程球 | Dynamic Island + Live Activities + Widget |
| 系统日历 | Android Calendar Provider | EventKit (EKEventStore) |
| 系统提醒 | Android AlarmManager | EventKit (EKReminder) + UserNotifications |
| OCR 引擎 | 端侧 OCR + PaddleOCR-VL | Vision.framework (VNRecognizeTextRequest) + PaddleOCR-VL Cloud |
| 语音识别 | 端侧 ASR | Speech.framework (SFSpeechRecognizer) |
| 文档解析 | PaddleOCR-VL | PaddleOCR-VL (Cloud/MCP) + VisionKit |
| UI 框架 | Jetpack Compose / 原子组件 | SwiftUI + UIKit (混合) |
| 地图导航 | 高德/百度地图 | MapKit + Apple Maps |
| 跨应用动作 | 意图框架 MCP Adapter | URL Schemes + Universal Links + App Intents |
| IDE | Android Studio | Xcode 16 |
| 语言 | Kotlin/Java | Swift 6 |

---

# SnapDo for iOS

## 1. 产品定位、解决的痛点

**SnapDo** 是多模态意图执行助手，将截图、拍照、语音、PDF、通知流等任意形式的信息，一键转化为可执行的日历、提醒、清单、导航和跨应用动作。

### iOS 平台特有优势

| iOS 特性 | SnapDo 利用方式 |
|---------|---------------|
| 🏝️ **Dynamic Island** | 显示即将到来的事件倒计时、导航提示、出发提醒 |
| 🧩 **Live Activities** | 锁屏实时显示当前进行中的任务、导航路线 |
| 📲 **Widget（锁屏/主屏/待机）** | 晨间简报卡片、今日日程一览、快速录入入口 |
| 🧠 **Apple Intelligence** | Writing Tools 辅助备注润色、Image Playground 生成事件封面 |
| 🔗 **App Intents + Shortcuts** | 将识别结果直接映射为系统级动作，支持 Siri 语音触发 |
| 📋 **Universal Clipboard** | Mac 端复制文本 → iPhone 端 SnapDo 自动弹出识别卡片 |
| 🛡️ **端侧处理优先** | 所有 NLP/NER/意图分类均在 Apple Neural Engine 上执行，数据不出设备 |

### 解决痛点

| 痛点 | SnapDo for iOS 解法 |
|------|-------------------|
| 🖼️ 聊天截图/照片里的时间地点，要手动抄录到日历 | **截图/拍照即识别** → Vision 框架提取文本 → NaturalLanguage 抽取实体 → 一键创建日程 |
| 🎤 口头说「周六下午提醒我…」但说完就忘 | **Siri + 端侧语音识别** → Speech.framework 转录 → App Intents 结构化 |
| 📄 PDF、邮件、合同里的截止日期散落各处 | **拖入/分享 PDF** → PaddleOCR-VL 云解析 → 批量提取并导入 EventKit |
| 🔔 通知栏爆炸，不知道哪些该马上处理 | **Notification Service Extension** → 通知聚类 + 优先级分级 → Dynamic Island 三态建议 |
| 📋 批量日程创建困难 | **PaddleOCR-VL 批量解析** → 逐条预览 → 一键写入 EKEventStore |
| 🗺️ 有地址的事件不会自动导航 | **MapKit 联动** → 预约时间前自动弹出 Live Activity 导航卡片 |
| 🔗 涉及第三方 App 的动作要手动跳转 | **URL Schemes + App Intents** → 直接唤起对应 App 执行（订票、发消息、扫码等） |

---

## 2. 产品名称

**SnapDo · 瞬达** (for iOS)

---

## 3. 数据录入方式

| 方式 | 触发场景 | iOS 核心技术 |
|---------|-----------------|-----------------------|
| 📸 截图 | 聊天记录、网页 | Vision (VNRecognizeTextRequest) + NaturalLanguage 实体抽取 |
| 📷 拍照 | 海报、名片、票据 | AVFoundation 拍摄 + Vision 结构化提取 |
| 🎤 语音 | 边走边说、会议 | Speech.framework (SFSpeechRecognizer) + App Intents 意图理解 |
| 📄 PDF | 合同、课表、行程单 | PaddleOCR-VL (MCP Cloud) + PDFKit 预览 |
| 🔔 通知 | 系统通知流 | UNNotificationServiceExtension + Create ML 聚类模型 |
| ✍️ 手动 | 速记、补充修正 | SwiftUI TextEditor + Markdown 快捷语法 |
| 📋 共享菜单 | 从任意 App 分享内容 | Share Extension (UIActivityViewController) |
| 🖥️ Universal Clipboard | Mac→iPhone 跨设备 | UIPasteboard 检测 → 自动弹出识别卡片 |

### 截图识别示例流程

```
用户在微信里收到：「下周三下午3点，在望京SOHO T2 15层开会，带方案初稿」
→ 截图
→ SnapDo 通过 Share Extension 或 Widget 快捷入口接收图片
→ Vision.framework 自动识别文字区域
→ NaturalLanguage (NLTokenizer + NLTagger) 抽取实体
→ 输出结构化卡片：

📅 事件：开会
📍 地点：望京SOHO T2 15层
🕒 时间：2026年5月13日（周三）15:00
📎 备注：带方案初稿
🚗 导航：提前30分钟提醒 + 自动激活 Apple Maps 导航
```

### PDF 批量解析示例流程

```
用户在微信/邮件中收到课程表 PDF
→ 点击「分享」→ 选择 SnapDo
→ SnapDo 调用 PaddleOCR-VL MCP 服务解析文档内容
→ 批量预览 16 周 × 每周 5 节课 = 80 条日程
→ 用户勾选/去勾选后「一键全部导入系统日历（EventKit）」
```

---

## 4. AI 智能整理安排方式

### 4.1 实体抽取 + 意图分类

```
输入 → [Vision OCR] → [NaturalLanguage NER] → [App Intents 意图分类器] → [结构化输出]
          ├─ 文本识别         ├─ 时间表达式 (NSDataDetector)    ├─ 创建日程 (EKEvent)
          ├─ 图片预处理       ├─ 地点/地址 (CLGeocoder)         ├─ 设置提醒 (EKReminder)
          │                   ├─ 人物/联系人 (CNContactStore)    ├─ 创建清单 (Reminders App)
          │                   ├─ 数字/金额                       ├─ 导航任务 (MapKit/Maps URL)
          │                   └─ 动作/动词                       └─ 快捷指令 (Shortcuts)
```

### 4.2 冲突检测与智能排期

```
已有日程：周三 14:00–16:00 产品评审会 (EKEventStore 查询)
新识别：  周三 15:00 望京开会
→ ⚠️ 时间冲突！
→ Apple Intelligence 建议：
   ① 将新事件设为 16:30（评审会后）
   ② 保持 15:00，将评审会提前至 13:30
   ③ 标记为「待确认」并通过 iMessage 通知冲突双方
```

### 4.3 优先级自动计算（三维评分模型）

```swift
// 运行在 Apple Neural Engine 上的 Core ML 模型
struct PriorityCalculator {
    let urgencyWeight = 0.5    // 紧急度：是否含「急」「尽快」「deadline」「ASAP」
    let importanceWeight = 0.3 // 重要性：是否涉及「老板」「客户」「付款」「合同」
    let timeWeight = 0.2       // 时间紧迫度：距截止时间的小时数倒数归一化

    // 得分 ≥ 4.0 → 🔴 现在办 (Dynamic Island 红色脉冲)
    // 得分 2.0–3.9 → 🟡 稍后办 (Dynamic Island 黄色常驻)
    // 得分 < 2.0  → ⚪ 可忽略 (静默归档)
}
```

### 4.4 通知聚类处理

```
iOS 通知流（通过 UNNotificationServiceExtension 获取）
    │
    ▼
┌─────────────────────────────────┐
│  通知聚类引擎（Core ML 端侧模型）  │
│                                 │
│  按三个维度聚类：                 │
│  ├─ 来源维度（Bundle ID）        │
│  ├─ 语义维度（NLTagger）         │
│  └─ 时间维度（时效性评分）        │
└──────────┬──────────────────────┘
           ▼
┌─────────────────────────────────┐
│  每日三次批次整理（通过 BGTaskScheduler 注册）：│
│  🌅 晨间简报 (8:00)             │
│  🌤 午后整理 (13:00)             │
│  🌙 晚间回顾 (21:00)            │
└──────────┬──────────────────────┘
           ▼
┌───────────────────────────────────────────┐
│           智能行动建议卡片                   │
│          （通过 Widget + Dynamic Island 呈现）│
│                                           │
│  🔴 现在办 (3条)                           │
│  · 支付宝：房租到期，今日24:00前缴纳          │
│  · 老板微信：方案今天下班前提交               │
│  · 航班提醒：明天CA1234已值机，选座           │
│                                           │
│  🟡 稍后办 (7条)                           │
│  · 快递已签收，记得取                        │
│  · 订阅App续费提醒，3天后到期                 │
│  · 朋友圈有人@你                            │
│                                           │
│  ⚪ 已忽略 (15条)                          │
│  · 各类App营销推送（自动归档到通知摘要）       │
│                                           │
│  [一键全部处理] [逐条确认]                   │
└───────────────────────────────────────────┘
```

### 4.5 跨应用动作引擎（App Intents 体系）

利用 iOS 18 **App Intents** + **URL Schemes** 体系，将识别结果直接映射为系统级动作：

| 识别内容 | iOS 跨应用动作 |
|----------|---------------|
| 「订机票/火车票/酒店」 | 唤起携程/飞猪 App（URL Scheme）或通过 Siri App Intents 委派 |
| 「给XX发消息」 | Messages App Intent：预填联系人 + MessageComposeSheet |
| 「扫码付款」 | 唤起支付宝/微信 URL Scheme 跳转扫码页 |
| 「XX地点见」 | MapKit 创建导航 + EKEvent 写入日历 |
| 「买XX」 | 唤起淘宝/京东 URL Scheme 搜索 |
| 「记一笔」 | 写入 Apple Notes（Notes App Intent / EventKit） |
| 「提醒我XX时做XX」 | EKReminder + UNUserNotificationCenter |
| 「发邮件给XX」 | MFMailComposeViewController / Mail URL Scheme |

```
SnapDo ≡ App Intents Dispatcher
        │
        ├──→ EKEventStore（创建/查询/修改日历事件）
        ├──→ EKReminderStore（创建提醒事项）
        ├──→ MapKit + MKMapItem（路径规划 + 出发提醒）
        ├──→ Notes.framework（创建笔记）
        ├──→ Messages.framework（发送消息）
        ├──→ URL Schemes（唤起第三方 App）
        │     ├── alipay:// → 支付宝
        │     ├── weixin:// → 微信
        │     ├── taobao:// → 淘宝
        │     └── ctrip:// → 携程
        └──→ Shortcuts.framework（复杂任务编排）
```

---

## 5. 可视化展示、智能提醒方式

### 5.1 三种视图

```
┌──────────────────────────────────────┐
│  📱 主界面三视图切换 (SwiftUI TabView) │
│  [时间线]   [看板]   [地图]           │
├──────────────────────────────────────┤
│                                      │
│  📅 时间线视图（默认）                 │
│  ┌──────────────────────────────┐    │
│  │ 今天 5月6日 周三               │    │
│  │ ┌──────────────────────────┐ │    │
│  │ │ 🟡 09:00 每日站会         │ │    │
│  │ │ 📍 3楼会议室              │ │    │
│  │ │ 📎 同步进度               │ │    │
│  │ └──────────────────────────┘ │    │
│  │ ┌──────────────────────────┐ │    │
│  │ │ 🔴 15:00 客户提案会议     │ │    │
│  │ │ 📍 望京SOHO T2           │ │    │
│  │ │ 🚗 14:20出发 \| 导航      │ │    │
│  │ │ 📎 带方案初稿 + 报价单     │ │    │
│  │ └──────────────────────────┘ │    │
│  │           ···                │    │
│  └──────────────────────────────┘    │
│                                      │
│  📋 看板视图                          │
│  [待开始] [进行中] [已完成]             │
│  支持 SwiftUI Drag & Drop 改变状态     │
│                                      │
│  🗺️ 地图视图                          │
│  MapKit 打点显示所有带地址的事件         │
│  支持 MKDirections 路线串联规划        │
└──────────────────────────────────────┘
```

### 5.2 智能提醒机制

| 提醒类型 | 触发条件 | iOS 呈现方式 |
|---------|---------|-------------|
| ⏰ 时间提醒 | 事件前 N 分钟（可自定义）| UNNotification + 横幅 + 触觉反馈 |
| 📍 地点提醒 | CLCircularRegion 进入/离开地理围栏 | 锁屏 Live Activity 卡片 |
| 🚗 出发提醒 | MKDirections 基于实时路况倒推出发时间 | **Dynamic Island** 弹窗 |
| 📊 晨间简报 | BGAppRefreshTask 每天 8:00 刷新 | Widget 聚合卡片推送 |
| 🔔 遗漏提醒 | 过期未完成事项查询 | 晚间回顾 Live Activity 高亮 |
| 👥 社交提醒 | CloudKit 同步共享事件变动 | 双方同步 UNNotification |

---

## 6. AI 对话（Siri + App Intents）

### 6.1 对话触发场景

```
用户：「嘿 Siri，帮我安排下周行程」
  ↓
Siri (App Intent: "PlanMyWeek"):
  「我在你的截图记录里找到3条待安排的事项，
    同时日历上空闲时段如下：
  
    ✅ 周一 14:00-15:30 → 牙科复诊（来自截图）
    ✅ 周三 10:00-11:00 → 团队周会（已有）
    ✅ 周五 16:00-17:00 → 接机（来自语音备忘录）
  
    要把这3条安排上吗？」

用户：「安排上」

Siri：「已创建3条日程。另外，周五接机需要导航吗？」
```

### 6.2 对话设计原则

1. **不闲聊，不废话** — 对话仅用于确认歧义、解决冲突
2. **渐进式确认** — 高置信度直接执行，低置信度才通过 Siri 询问
3. **批量处理优先** — 一次 Siri 对话处理尽可能多的事项
4. **可回退** — 每条操作通过 `UndoManager` 支持撤销（保留 30 秒撤销窗口）
5. **Siri 快捷短语** — 预置常用 Siri Shortcuts：`"安排我的下周"`、`"今天有什么"`、`"帮我导航去下一个会议"`

---

## 7. 大致 UI 设计（iOS Human Interface Guidelines）

### 7.1 设计语言

遵循 iOS 18 **Human Interface Guidelines**，融合：
- **Dynamic Island** — 实时状态指示与快捷操作
- **Materials（毛玻璃材质）** — SwiftUI `.material(.regular)` / `.ultraThinMaterial`
- **SF Symbols 6** — 统一图标体系，支持动画
- **弹性动效** — SwiftUI `.spring()` 符合物理直觉
- **Adaptive Layout** — Size Class 自适应 iPhone / iPad

### 7.2 关键界面线稿

```
┌─────────────────────────────────────┐
│  Dynamic Island ┌──────┐           │
│  🔴 3条待处理    │ 快录  │           │
│                 └──────┘           │
├─────────────────────────────────────┤
│  ◀ Snapshot          🔔 3条待处理  │
│                                     │
│  ┌─────────────────────────────┐    │
│  │  🌅 晨间简报 · 5月6日周三     │    │
│  │              (SwiftUI Card) │    │
│  │  🔴 今天必须完成             │    │
│  │  · 15:00 客户提案 (3h后)     │    │
│  │  · 房租缴纳截止 (今日)        │    │
│  │                             │    │
│  │  🟡 有空就处理               │    │
│  │  · 快递已到前台              │    │
│  │  · 回复小王微信              │    │
│  │                             │    │
│  │  ⚪ 已自动归档 12条          │    │
│  │                             │    │
│  │  [逐条确认] [一键处理]        │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─────────────────────────────┐    │
│  │  快速录入 (HStack)           │    │
│  │  [📸截图] [📷拍照] [🎤语音] │    │
│  │  [📄文档] [🔔通知] [✍️速记] │    │
│  └─────────────────────────────┘    │
│                                     │
│  ┌─ 今天 ─────────────────────┐    │
│  │ ● 09:00 每日站会           │     │
│  │ ● 15:00 客户提案 🚗        │    │
│  │ ○ 19:00 健身房             │    │
│  └────────────────────────────┘     │
│                                     │
│  [时间线]  [看板]  [地图]  [设置]    │
│      (SwiftUI TabView)              │
└─────────────────────────────────────┘
```

### 7.3 关键交互组件

**「Dynamic Island 快捷球」替代「快日程球」：**
- 长按 Dynamic Island → 弹出快捷菜单（语音、截图、速记）
- 从其他 App 拖拽图片/文本到 Dynamic Island → 识别录入（Drag & Drop API）
- 正在识别时 Dynamic Island 显示旋转进度动画

**「Live Activities」替代「原子岛」：**
- 即将到来的事件在锁屏 Live Activity 显示倒计时
- 导航、出发提醒通过 Live Activity 实时更新
- 点击 Live Activity 直接进入对应事件详情

**「卡片堆叠」交互（SwiftUI `.stacked` 布局）：**
- 同类事件自动堆叠
- 上滑展开查看全部
- 左滑删除/完成（`.swipeActions`）
- 右滑推迟

**「Widget 家族」：**
- **锁屏 Widget**（1×1 圆形）：今日待办数量
- **主屏 Widget**（2×2 / 4×2）：晨间简报卡片
- **待机 Widget**（StandBy）：全天时间线

---

## 8. 用户交互模式

```
┌──────────────────────────────────────────────────────┐
│                  iOS 用户交互模式全景                  │
├──────────────────────────────────────────────────────┤
│                                                      │
│  📱 主动录入模式                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │ 用户在任意App看到需要安排的信息                  │    │
│  │   ↓                                          │    │
│  │ 方式A：截图 → 通知中心「SnapDo 识别」快捷操作    │    │
│  │ 方式B：Share Sheet → 选择 SnapDo               │    │
│  │ 方式C：Widget 一键「快录」                      │    │
│  │ 方式D：Siri「嘿 Siri，用 SnapDo 安排……」        │    │
│  │ 方式E：从相册/文件拖入 SnapDo 卡片               │    │
│  │   ↓                                          │    │
│  │ 弹出结果卡片 → 用户确认/修改 → 落地执行           │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  🤖 被动感知模式                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │ UNNotificationServiceExtension 接收通知     │    │
│  │ Core ML 端侧模型聚类（隐私安全）                │    │
│  │   ↓                                          │    │
│  │ BGAppRefreshTask 每日三个批次整理             │    │
│  │   ↓                                          │    │
│  │ Widget / Dynamic Island 推送聚合卡片          │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  💬 Siri 对话修正模式                                 │
│  ┌──────────────────────────────────────────────┐    │
│  │ 「Siri，把明天下午的会议改到后天」               │    │
│  │   ↓                                          │    │
│  │ App Intents 自然语言修改 EKEvent               │    │
│  │ 支持：改时间、改地点、加备注、取消、重复设置       │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  🏃 主动执行模式                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │ 事件时间临近 → 自动触发关联动作                  │    │
│  │                                              │    │
│  │ 例：14:20 → Live Activity 弹出导航卡片         │    │
│  │     14:25 → UNNotification 检测用户未出发      │    │
│  │     14:50 → iMessage 自动发送「路上稍晚5分钟」   │    │
│  │     15:00 → Focus Filter 进入会议模式          │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 9. 技术架构概览（iOS 适配）

```
┌─────────────────────────────────────────────┐
│           应用层 (SnapDo.app)                │
│  ┌─────────┐ ┌─────────┐ ┌───────────────┐  │
│  │SwiftUI  │ │ Widget  │ │ Siri Intents  │  │
│  │录入/展示 │ │Extension│ │ Extension     │  │
│  └────┬────┘ └────┬────┘ └───────┬───────┘  │
├───────┼───────────┼──────────────┼─────────-┤
│       │    调度层 (Orchestrator  Actor)     │
│  ┌────┴───────────┴──────────────┴───────┐  │
│  │  意图解析 + 冲突检测 + 优先级引擎        │  │
│  │  (Swift Actor Model, async/await)     │  │
│  └────┬───────────┬──────────────┬───────┘  │
├───────┼───────────┼──────────────┼─────────-┤
│       │     AI 能力层（Apple Intelligence    │
│       │      + Core ML + Cloud Services）    │
│  ┌────┴────┐┌─────┴─────┐┌──────┴──────┐    │
│  │Vision   ││NLP        ││Core ML      │    │
│  │OCR/NER  ││意图分类    ││通知聚类模型  │    │
│  │(端侧ANE)││(端侧ANE)  ││(端侧ANE)    │    │
│  └────┬────┘└─────┬─────┘└──────┬──────┘    │
│       │           │              │          │
│  ┌────┴───────────┴──────────────┴───────┐  │
│  │    Cloud Services (按需)               │  │
│  │    ├─ PaddleOCR-VL MCP (PDF复杂解析)   │  │
│  │    └─ Apple Private Cloud Compute      │  │
│  └──────────────────────────────────────┘  │
├───────┼───────────┼──────────────┼─────────-┤
│       │   App Intents 分发层               │
│  ┌────┴───────────┴──────────────┴───────┐  │
│  │EKEventStore│EKReminder │MapKit       │  │
│  │Notes       │Messages   │URL Schemes  │  │
│  │Shortcuts.framework (复杂编排)          │  │
│  └──────────────────────────────────-───┘  │
├─────────────────────────────────────────────┤
│           iOS 18 系统服务层                  │
│  EventKit │ UserNotifications │ CoreLocation │
│  MapKit   │ CloudKit          │ WidgetKit    │
│  Speech   │ Vision            │ NaturalLang  │
│  AppIntents│ ActivityKit      │ BackgroundTasks│
└─────────────────────────────────────────────┘
```

---

## 10. 核心依赖与工具调用清单

### 10.1 iOS 系统框架依赖

| 框架 | 用途 | 调用位置 |
|------|------|---------|
| **Vision.framework** | 端侧 OCR 文本识别（VNRecognizeTextRequest） | 截图/拍照识别模块 |
| **NaturalLanguage.framework** | NER 实体抽取、意图分类（NLTagger, NLTokenizer） | 意图解析引擎 |
| **Speech.framework** | 端侧语音转文字（SFSpeechRecognizer） | 语音录入模块 |
| **Core ML + ANE** | 端侧优先级评分模型、通知聚类模型 | 优先级/聚类引擎 |
| **EventKit** | 系统日历读写（EKEventStore）、提醒事项（EKReminder） | 日程/提醒落地 |
| **MapKit** | 地理编码（CLGeocoder）、路径规划（MKDirections） | 导航联动 |
| **App Intents** | Siri 快捷指令、跨应用意图分发 | Siri 对话/跨应用动作 |
| **ActivityKit** | Live Activities（锁屏/动态岛实时卡片） | 提醒/导航展示 |
| **WidgetKit** | 锁屏/主屏/待机 Widget | 晨间简报/快捷入口 |
| **BackgroundTasks** | BGAppRefreshTask 后台批次整理 | 通知聚类调度 |
| **UserNotifications** | 本地通知推送、Notification Service Extension | 提醒触达 |
| **CoreLocation** | 地理围栏（CLCircularRegion） | 地点提醒 |
| **CloudKit** | iCloud 多设备同步、共享事件协作 | 数据同步 |
| **Share Extension** | 从任意 App 接收截图/PDF/文本 | 数据录入 |
| **PDFKit** | PDF 预览、页面渲染 | PDF 文档处理 |
| **UIKit Drag & Drop** | 从其他 App 拖拽图片/文件到 SnapDo | 快速录入 |

### 10.2 Cloud / MCP 服务依赖

| 服务 | 用途 | 调用方式 |
|------|------|---------|
| **PaddleOCR-VL MCP** | 复杂 PDF/文档批量解析（表格/公式/图表） | MCP `invoke` → `paddleocrVlPaddleocrVl` |
| **Apple Private Cloud Compute** | 复杂语义推理（Apple Intelligence 溢出） | 系统自动路由 |

### 10.3 PaddleOCR-VL MCP 工具调用示例

```javascript
// 在 SnapDo App 中，当用户分享一份 PDF 时：

// Step 1: 调用 PaddleOCR-VL 解析 PDF
const result = await mcp.callTool("paddleocrVlPaddleocrVl", {
  input_data: "https://example.com/user-uploaded/schedule.pdf",
  file_type: "pdf",
  output_mode: "detailed",
  return_images: false
});

// Step 2: 将解析结果中的每条日程结构化
// 使用 NaturalLanguage 框架对 markdown 内容进行实体抽取
// 逐条创建 EKEvent 写入系统日历
```

### 10.4 URL Schemes（第三方 App 唤起）

| App | URL Scheme | 用途 |
|-----|-----------|------|
| 微信 | `weixin://` | 发消息、扫码 |
| 支付宝 | `alipay://` | 扫码付款 |
| 淘宝 | `taobao://` | 搜索商品 |
| 京东 | `openapp.jdmobile://` | 搜索商品 |
| 携程 | `ctrip://` | 订票/酒店 |
| Apple Maps | `maps://` | 导航 |
| Apple Notes | `mobilenotes://` | 创建笔记 |

---

## 11. 安全与隐私（iOS 特色）

```
┌─────────────────────────────────────────────┐
│              iOS 隐私安全保障                  │
├─────────────────────────────────────────────┤
│                                             │
│  🔒 端侧优先                                  │
│  · Vision OCR / NLP / Core ML 全部在 ANE 执行 │
│  · 截图/语音数据从不离开设备                    │
│                                             │
│  🔐 权限最小化                                │
│  · NSCameraUsageDescription（拍照）           │
│  · NSMicrophoneUsageDescription（语音）       │
│  · NSLocationWhenInUseUsageDescription（导航） │
│  · NSCalendarsUsageDescription（日历读写）     │
│  · NSRemindersUsageDescription（提醒事项）     │
│  · NSSpeechRecognitionUsageDescription（语音） │
│                                             │
│  ☁️ Cloud 部分                                │
│  · PaddleOCR-VL 调用仅在用户主动触发 PDF 解析时  │
│  · Apple Private Cloud Compute 端到端加密     │
│  · 无用户数据持久化存储于云端                    │
│                                             │
│  🧩 App Tracking Transparency                │
│  · 完全不追踪用户，无广告，无需 ATT 弹窗        │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 12. 开发技术栈总结

| 层 | 技术选型 |
|----|---------|
| UI 框架 | SwiftUI 5 + UIKit (混合，复杂手势用 UIKit) |
| 架构模式 | MVVM + Actor Model (Swift Concurrency) |
| 数据持久化 | SwiftData + CloudKit（iCloud 同步） |
| 依赖注入 | Swift Package Manager (SPM) |
| 测试 | XCTest + XCUITest |
| CI/CD | Xcode Cloud |
| 最低支持 | iOS 18.0（充分利用最新系统能力） |
| 语言 | Swift 6（严格并发检查） |