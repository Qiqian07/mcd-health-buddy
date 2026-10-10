# MCP 接入与调用说明

本文档说明本项目（麦麦健康搭子）实际使用的麦当劳 MCP Server、调用的 Tool、调用流程，以及由此产生的业务价值。

## 一、使用的 MCP Server

- **服务端点**：`https://mcp.mcd.cn`
- **传输协议**：Streamable HTTP
- **认证方式**：请求头 `Authorization: Bearer ${MCD_MCP_TOKEN}`（见仓库 `mcp-config.example.json`，仅使用环境变量占位符）
- **Token 申请官网**：https://open.mcd.cn/mcp （登录 → 控制台 → 激活）
- **官方文档仓库**：[M-China/mcd-mcp-server](https://github.com/M-China/mcd-mcp-server)
- **限流**：600 次/分钟/Token，超出返回 429

> 说明：`mcp.mcd.cn` 是程序调用的 API 端点，浏览器直接访问会返回系统错误，属正常现象；申请 Token 请访问 `open.mcd.cn/mcp`。

本项目全部数据与交易能力均来自上述官方 MCP Server，未使用任何非官方接口或爬虫手段。

## 二、实际调用的 Tool 清单

### 1. 营养与菜单（健康搭配的核心）

| Tool | 用途 |
| --- | --- |
| `list-nutrition-foods` | 获取餐品营养成分（能量 kJ/kcal、蛋白质、脂肪、碳水、钠、钙），返回为 toon 紧凑格式，Skill 内显式解析 |
| `query-meals` | 查询当前门店可售餐品列表（分类、餐品编码、标签） |
| `query-meal-detail` | 查询餐品详情与套餐组成、可替换项 |

### 2. 门店与地址（可售性校验）

| Tool | 用途 |
| --- | --- |
| `query-nearby-stores` | 查询用户地址附近门店 |
| `delivery-query-addresses` / `delivery-create-address` | 查询/新增配送地址 |
| `delivery-query-stores` | 外送场景查询可配送门店 |

### 3. 优惠与积分（省钱报告）

| Tool | 用途 |
| --- | --- |
| `query-my-account` | 查询积分账户（可用/累计/冻结/即将过期） |
| `query-my-coupons` / `available-coupons` / `auto-bind-coupons` | 卡包券查询、可领券列表、一键领券 |
| `query-store-coupons` | 查询当前门店可用券 |
| `calculate-price` | 计算商品金额、配送费、优惠金额与应付总价 |
| `mall-points-products` / `mall-product-detail` / `mall-create-order` | 积分商城商品查询与兑换下单 |
| `query-lottery-info` / `draw-lottery` / `query-my-prizes` | 积分抽奖活动查询、抽奖、中奖记录 |

### 4. 下单与活动

| Tool | 用途 |
| --- | --- |
| `create-order` / `cancel-order` | 创建订单（返回支付链接）/ 取消订单 |
| `query-order` / `order-list` | 订单详情 / 历史订单 |
| `campaign-calendar` | 当月营销活动日历 |
| `now-time-info` | 服务器当前时间 |
| `query-party-city` / `query-party-store` / `query-partystore-date` / `query-partystore-session` / `party-order-create` | 主题活动（派对/品鉴会）逐级查询与预约下单 |

## 三、调用流程

```
用户："我在减脂，帮我搭一顿 500 大卡以内的麦当劳"
   │
   ├─1─ list-nutrition-foods ──────► 按 ≤500 kcal、蛋白 ≥20g、钠 <1500mg、饮品 0 卡
   │                                 约束，配出 2~3 套组合方案
   │
   ├─2─ query-nearby-stores / delivery-query-stores
   │    └─ query-meals ────────────► 校验方案餐品在门店可售，缺货自动替换相近餐品
   │
   ├─3─ query-my-account
   │    ├─ query-my-coupons / query-store-coupons
   │    ├─ mall-points-products（积分商城比价）
   │    └─ calculate-price ───────► 产出「原价 → 券后价 → 积分组合价」省钱报告
   │
   ├─4─ 用户确认 ─► create-order ──► 返回支付链接（强制确认，禁止静默下单）
   │
   └─5─ campaign-calendar ─────────► 提示"哪天吃更划算"
```

扩展流程：

- **积分抽奖**：`query-lottery-info` → 用户确认 → `draw-lottery` → `query-my-prizes`，抽中餐品券回流并入方案当作 0 元加餐重新核算
- **主题活动预约**：`query-party-city` → `query-party-store` → `query-partystore-date` → `query-partystore-session` → `party-order-create`

## 四、业务价值

1. **健康价值**：把官方营养数据转化为可执行的热量/蛋白质/钠约束，让用户在快餐场景下也能做出有依据的选择，而不是凭印象估算。
2. **省钱价值**：券、积分商城兑换、积分抽奖三路比价，输出可量化的省钱金额；优惠决策受营养约束（例如某券对应餐品会突破热量预算时不予采用），避免"为了省钱牺牲健康目标"。
3. **体验价值**：从选品、可售性校验、比价到下单全流程自然语言完成，无需在 App 中反复切换页面比对。
4. **安全价值**：所有涉及资金与积分的操作均强制展示摘要并经用户明确确认后执行；配置仅使用环境变量占位符，不落库真实凭证。
5. **可扩展性**：纯 Skill（提示词工程 + MCP 工具编排），零自定义代码，官方新增工具后可直接在 Skill 中编排复用。

## 五、常见问题

- **401**：Token 无效或过期 → 到 https://open.mcd.cn/mcp 控制台重新获取
- **429**：超过 600 次/分钟限流 → 降低调用频率后重试
- **calculate-price / create-order 报缺少参数**：商品列表字段名为 `items`，且需包含 `storeCode`、`beCode`、`addressId`（外送）等顶层字段
- **营养数据解析异常**：`list-nutrition-foods` 返回 toon 紧凑格式（首行字段名、后续行数据），需按首行解析，不可按标准 JSON 数组处理
