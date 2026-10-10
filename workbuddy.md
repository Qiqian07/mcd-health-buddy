# 开发对话记录 — 麦麦健康搭子

> 本文档为大赛要求提交的 `workbuddy.md`——使用腾讯 WorkBuddy 开发的对话上下文（文件名由官方规定，不可改动），记录了从需求分析到作品成型的完整开发过程，用于核验 WorkBuddy 专项奖励资格。开发时间：2026 年 10 月 10 日。

---

## 第 1 轮：需求输入

**用户**：

> （提供了大赛宣传图 × 4：活动主视觉、「可以开发什么？」、「如何参与比赛？」、「丰厚奖励等你来拿」）请帮我完成图片内容。

**WorkBuddy 分析**：

识别出任务为参加"麦当劳程序员创意开发大赛"，并从图片中提取关键要求：

1. 查看活动规则：登录 GitHub 搜索 McDonald's China（M-China），查看赛事详情和规则
2. 开发作品：基于麦当劳 MCP 能力完成创意 Skill 开发；使用 WorkBuddy 开发可参与专项奖励，需保存完整开发对话（MD 文档格式）
3. 开源发布：将项目发布至个人 GitHub 公开仓库，按要求上传项目介绍、参赛声明、MCP 接入说明等文件
4. 提交报名：进入麦当劳 GitHub 官方账号活动页面，使用指定 Issue 模板填写：项目名称 + 项目地址 + 项目简介

## 第 2 轮：调研 MCP 实际开放能力

**WorkBuddy**：检索了大赛官方信息与麦当劳中国官方 MCP 仓库（M-China/mcd-mcp-server），确认：

- 官方 MCP 服务端点：`https://mcp.mcd.cn`（程序调用）；开放平台官网：`https://open.mcd.cn/mcp`（申请 Token 的网页，浏览器打开 mcp.mcd.cn 会返回系统错误属正常）。Streamable HTTP 协议，`Authorization: Bearer <Token>` 认证，限流 600 次/分钟/Token
- 实际开放 20+ 工具，覆盖：
  - 点餐链路：`query-meals`、`query-meal-detail`、`calculate-price`、`create-order`、`query-order`
  - 营养数据：`list-nutrition-foods`（toon 紧凑格式返回能量 kJ/kcal、蛋白质、脂肪、碳水、钠、钙）
  - 优惠与积分：`query-my-coupons`、`available-coupons`、`auto-bind-coupons`、`query-store-coupons`、`query-my-account`、`mall-points-products`、`mall-create-order` 等
  - 门店/地址：`query-nearby-stores`、`delivery-query-addresses`、`delivery-create-address`、`delivery-query-stores`
  - 活动：`campaign-calendar`、`now-time-info`
- 关键坑位：`calculate-price`/`create-order` 必须传 `items` 字段并包含 `storeCode`、`beCode`、`addressId` 等顶层字段

**WorkBuddy 向用户确认作品方向**，提供 4 个候选（官方示例有点餐/省钱/活动推荐助手，但评委易审美疲劳，建议差异化）：

1. **麦麦健康搭子（推荐）**：营养数据 + 健康搭配 + 优惠联动，角度独特
2. 麦麦省钱助手：实用但同类多
3. 麦麦点餐管家：体验流畅但偏基础整合
4. 麦麦活动日历精灵：轻量有趣

**用户选择**：麦麦健康搭子（推荐项）。

## 第 3 轮：Skill 设计

**WorkBuddy 设计决策**：

- 核心逻辑："先讲营养，再算优惠"——健康价值与省钱价值双主线，形成差异化记忆点
- 工作流 6 步：明确健康目标 → 营养搭配（2~3 套备选）→ 门店可售校验（缺货自动替换）→ 优惠叠加比价 → 用户确认后下单 → 活动日历联动
- 安全设计：`create-order`/`mall-create-order` 为资金操作，Skill 中强制"先展示订单摘要、用户明确确认后才执行"，禁止静默下单
- 数据真实性：所有营养与价格数字必须来自 MCP 实时数据，显式处理 toon 紧凑格式解析，禁止编造
- 健康搭配规则：热量对齐目标留 10% 余量、蛋白质 ≥ 20g 优先、钠 < 1500mg/餐、饮品优先 0 卡
- 错误处理：401（Token 失效引导重新获取）、429（限流退避）、缺参数（items/storeCode/beCode 修正）、单点咨询轻量直答

## 第 4 轮：产出参赛交付物

**WorkBuddy 生成了完整参赛作品包**（目录 `mcd-health-buddy/`）：

| 文件 | 作用 | 对应大赛要求 |
| --- | --- | --- |
| `SKILL.md` | Skill 本体（WorkBuddy Skill 规范，含触发条件与完整工作流） | 开发作品 |
| `README.md` | 项目介绍（功能表、使用示例、快速开始、技术说明） | 项目介绍 |
| `docs/MCP接入说明.md` | Token 申请流程、接入 JSON 配置、工具清单、FAQ | MCP 接入说明 |
| `docs/参赛声明.md` | 原创性、官方接口合规、授权与评判依据声明 | 参赛声明 |
| `submissions/issue-报名内容.md` | 按官方模板填好的：项目名称 + 项目地址 + 项目简介 | 提交报名 |
| `开发对话记录.md` | 本文档 | WorkBuddy 专项奖励要求 |

## 第 4.5 轮：功能扩展（用户确认后追加）

用户确认"可以"后，将官方 v1.0.7~v1.0.9 新增的两组能力做进 Skill，形成两个可选扩展模块：

- **扩展模块 A：积分抽奖（先抽奖，再点单）**
  `query-lottery-info` 查活动与消耗规则 → 用户明确确认后才 `draw-lottery` → `query-my-prizes` 查奖品。
  关键设计：抽中的餐品券**回流到第 2 步并入健康搭配方案**，当作 0 元加餐重新核算总热量与总价，在省钱报告中体现"抽奖所得抵扣"。抽奖消耗积分，强制确认、单次执行不循环。

- **扩展模块 B：主题活动预约（麦当劳派对 / 品鉴会）**
  `query-party-city` → `query-party-store` → `query-partystore-date` → `query-partystore-session` → `party-order-create`，每级都让用户选择，下单前确认。
  差异化设计：派对餐偏甜食高热量，下单前主动用营养数据算出活动热量参考值，给出"当天其他两餐控制在 XXX kcal 以内"的预算提示。

- 同时在下单环节补充 `cancel-order`（取消需确认）与 `order-list`（历史订单可作为"上次热量超标，这次换方案"的参考）。

## 第 5 轮：交付收尾（WorkBuddy 已完成）

- 生成 `LICENSE`（MIT，附商标声明）与 `.gitignore`
- 初始化 Git 仓库并完成首次提交（28c9e6f，署名 Qiqian07；共 3 次提交，均已修正作者为 Qiqian07）
- 添加远程 `origin → https://github.com/Qiqian07/mcd-health-buddy.git`
- 将 `SKILL.md` 安装到 WorkBuddy 用户级技能目录 `~/.workbuddy/skills/mcd-health-buddy/`，可直接实测
- 生成 `~/.workbuddy/mcp.json` 麦当劳 MCP 配置骨架，待填入个人 Token
- 在 README、参赛声明、Issue 报名内容中填入真实 GitHub 用户名 Qiqian07

## 第 6 轮：对照官方规则修正文件名（关键）

核对官方活动仓库 `M-China/mcd-developer-innovation-challenge` 的 README 与 `activityGuidelines.md` 后，发现官方对文件名有强制要求（**文件名不可改动**），此前的中文命名会导致报名失败，遂全部修正：

| 官方规定文件名 | 原命名 | 处理 |
| --- | --- | --- |
| `README.md`（项目介绍、安装方法、使用示例、目标用户） | `README.md` | 保留，补充"目标用户"章节 |
| `CONTEST_DECLARATION.md`（**文件名与内容均不可改动，须用官方原文件**） | `docs/参赛声明.md` | 改用官方原文新建于根目录，删除自撰版本 |
| `MCP_INTEGRATION.md`（使用的 Server、Tool、调用流程、业务价值） | `docs/MCP接入说明.md` | 重写为根目录 `MCP_INTEGRATION.md`，补充调用流程图与业务价值 |
| `mcp-config.example.json`（脱敏 MCP 配置示例，**只允许环境变量占位符**） | 无 | 新增，使用 `${MCD_MCP_TOKEN}` |
| `workbuddy.md`（WorkBuddy 对话上下文，专项奖励核验用） | `开发对话记录.md` | 更名 |

同时按官方要求重写 Issue 报名内容：正文严格使用 `【参赛申请】` 格式，控制在 1000 字内且**不含图片**（官方明确：携带图片会导致报名失败）。

## 第 7 轮：剩余待办（需参赛者本人操作）

1. 到 https://open.mcd.cn/mcp 申请 MCP Token（已完成，实测通过）
2. 在 GitHub 同步上述文件变更（新增 3 个根目录文件、删除 2 个旧命名文件、更新 README）
3. 大赛期间（2026.10.9 10:30 — 10.25 23:59）在活动仓库 https://github.com/M-China/mcd-developer-innovation-challenge 提交报名 Issue，内容见 `submissions/issue-报名内容.md`
4. （冲榜建议）邀请朋友 Star——排名以公开 Star 数为依据，Star 为 0 不进入排行榜；定榜时间 2026.10.26 00:00

---

*开发环境：腾讯 WorkBuddy · 2026-10-10*
