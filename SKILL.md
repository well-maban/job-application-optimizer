---
name: job‑application‑optimizer
description: 面向国内求职者的岗位匹配分析、简历定向改写、打招呼话术生成、公司背调、薪资参考与投递跟进。当用户发来 BOSS直聘/前程无忧/猎聘等岗位 JD 文字或截图，询问"这个岗位怎么样/这家公司/匹配吗/值不值得投/帮我写打招呼/打招呼没回复/简历怎么改/投哪个方向/转行还有机会吗/投递优先级/面试怎么答/这家公司靠谱吗/薪资多少合理/帮我排个序"时使用。支持多赛道并行（工程/技术/文创/销售），支持批量 JD 加权评分排序，支持投递台账 CLI 工具。不执行自动登录、自动批量投递、反爬采集等违规操作。
---
# 招聘岗位匹配与简历话术定制（国内版）
## 能力边界
**做什么：**
- 解析用户提供的 JD 文本/截图，输出匹配度评估、匹配点、短板应对、投递优先级
- 基于候选人固定档案，按赛道与平台（BOSS / 51job）定向改写简历要点与打招呼话术
- 批量 JD 一次性加权评分排序（5–10 个岗位）
- 公司风险背调 checklist 与薪资合理性判断
- 用 `scripts/application_tracker.py` 维护投递台账与跟进提醒

**不做什么（明确拒绝或降级处理）：**
- 不写自动登录、自动批量投递、绕过验证码/风控的爬虫或脚本；BOSS/51job 页面采集默认由用户粘贴文字或截图完成
- 不编造候选人不存在的学历、证书、工作年限、项目数据；不确定的事实标注"待用户确认"
- 不承诺面试结果、不提供虚假包装经历；短板只做"可迁移能力 + 弥补方案"表达
- 浏览器仅在用户主动提供已登录页面、且 browser‑use 可用时辅助只读提取可见 JD；不代替用户点"沟通/投递"按钮

## 标准工作流
### 第 1 步：读取候选人档案
先读 `references/candidate‑profile.template.md`，获取教育、工作、软件、项目、赛道定位、硬约束与历史反馈。后续所有输出不得与该档案冲突；档案未覆盖的新事实，先向用户确认，必要时更新档案文件。
> 首次使用：复制 `references/candidate‑profile.template.md` 为 `candidate‑profile.md` 并填写真实信息；复制 `assets/resume‑master.template.md` 为 `resume‑master.md` 并填写简历事实底稿。

### 第 2 步：获取 JD
输入形式按优先级处理：
1. 用户直接粘贴 JD 文字 → 直接进入分析
2. 用户上传岗位截图 → 用 Read 读取图片，逐块提取：岗位名、薪资、地点、学历/年限要求、岗位职责、任职要求、福利标签、公司名、发布方
3. 用户只给岗位链接或说"打开这个岗位" → 提示用户粘贴 JD 文字/截图最稳；若环境有 browser‑use 且用户已登录，可辅助只读提取可见内容，禁止代投
4. 用户只说"再帮我看个销售岗"没有 JD → 不虚构岗位；先给赛道选择建议并请其提供 JD

### 第 3 步：匹配评估
读 `references/jd‑matching.md`，按其框架输出：
- 硬性门槛核对（学历、年限、行业经验、地点通勤、硬性技能）
- 匹配点（技能 + 经历迁移，逐条对应用户档案事实）
- 短板与应对话术（短板类型、为什么不致命、怎么在面试/沟通中化解）
- 公司与岗位风险提示（加班标签、大小周、岗位是否像储备岗、猎头性质、通勤）
- 投递优先级结论：`首选投递 / 主推投递 / 备选练手 / 建议跳过`

**批量 JD 模式**：用户一次性发 5–10 个 JD 要求排序时，切换到 `jd‑matching.md` 第七节的加权评分模型，输出 S/A/B/C/D 分级与今日投递建议。

**薪资合理性判断**：涉及薪资讨论时，读 `references/salary‑benchmarks.md`，判断 JD 薪资是否在合理区间，给出谈薪建议。

**公司背调**：用户问"这家公司靠谱吗"或 JD 出现可疑信号时，读 `references/company‑due‑diligence.md`，输出风险评级与面试必问清单。

### 第 4 步：定向产出（按用户目标选择模块，不要一次堆砌全部）
- **改简历**：读 `references/resume‑playbooks.md`，选赛道，再按平台规则产出（BOSS：标题+个人简介；51job：自我评价+技能栏+完整经历）
- **打招呼话术**：读 `references/outreach‑messages.md`，按赛道与平台产出 2–3 个长度档位
- **跟进话术**：投递后 3 天/7 天没回复时，读 outreach‑messages.md 的跟进话术部分
- **面试问答**：读 `references/interview‑qa.md`，针对该 JD 预判 3–5 个追问并给应答骨架
- **投递策略**：综合多条 JD 时按赛道汇总优先级、每日投递量建议、跟进节奏

### 第 5 步：记录台账（用户确认投递动作后执行）
使用 `scripts/application_tracker.py`：
```bash
python3 scripts/application_tracker.py add --company "公司名" --job "岗位名" --track A --status applied
python3 scripts/application_tracker.py list
python3 scripts/application_tracker.py followup --days 3
python3 scripts/application_tracker.py remind
python3 scripts/application_tracker.py stats
## References 导航
| 文件 | 何时读取 |
|---|---|
| `references/candidate‑profile.template.md` | 每次触发必读：候选人档案与赛道定位（首次使用复制为 candidate‑profile.md 填写个人信息） |
| `references/jd‑matching.md` | 解析JD、打分、优先级、批量排序 |
| `references/resume‑playbooks.md` | 改写简历、BOSS/51job简介文案 |
| `references/outreach‑messages.md` | 打招呼、跟进话术 |
| `references/interview‑qa.md` | 面试问题和回答框架 |
| `references/company‑due‑diligence.md` | 公司背调、风险检查 |
| `references/salary‑benchmarks.md` | 薪资参考、谈薪建议 |

## Assets
- `assets/resume‑master.template.md`：简历母版，存放你的原始经历，所有简历输出都从这里提取，禁止编造信息
