# 麦当劳官方 MCP Server 接入说明

本 Skill 的全部数据与交易能力均来自麦当劳中国官方 MCP Server。使用前请按以下步骤完成接入。

## 一、申请 MCP Token

1. 访问麦当劳 MCP Server **开放平台官网**：[https://open.mcd.cn/mcp](https://open.mcd.cn/mcp)
2. 点击右上角【登录】，使用手机号验证码登录
3. 登录后点击右上角【控制台】
4. 点击【激活】按钮申请 MCP Token
5. 同意服务协议后，Token 申请成功，一键复制并妥善保管（**勿泄露给他人**）

> ⚠️ **两个域名别搞混**
>
> | 域名 | 是什么 | 怎么用 |
> | --- | --- | --- |
> | `open.mcd.cn/mcp` | 开放平台官网（网页） | 人访问：登录、申请/查看 Token |
> | `mcp.mcd.cn` | MCP 服务端点（API） | 程序调用：填进 MCP Client 配置 |
>
> 用浏览器直接打开 `mcp.mcd.cn` 会返回 `{"code":"0001","msg":"系统错误"}`，这是正常的——它不是给人浏览的页面。

## 二、接入配置

- **接入地址（服务端点）**：`https://mcp.mcd.cn`
- **传输协议**：Streamable HTTP（请确保你的 MCP Client 支持该协议）
- **认证方式**：请求头携带 `Authorization: Bearer YOUR_MCP_TOKEN`
- **版本要求**：MCP Server 仅支持 MCP Version 2025-06-18 及之前的版本
- **限流**：每个 Token 每分钟最多 600 次请求，超过返回 429

### WorkBuddy / 通用 JSON 配置

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

将 `YOUR_MCP_TOKEN` 替换为实际 Token 后，粘贴到 MCP Client 的 MCP Server 配置中并启用。

### 其他平台

Cherry Studio、Cursor、TRAE、Kiro、VSCode 等平台的图形化接入步骤，请参考官方仓库教程：
[https://github.com/M-China/mcd-mcp-server](https://github.com/M-China/mcd-mcp-server)

## 三、本 Skill 使用的工具清单

### 营养与菜单

| 工具 | 用途 |
| --- | --- |
| `list-nutrition-foods` | 获取常见餐品营养数据（能量 kJ/kcal、蛋白质、脂肪、碳水、钠、钙），为 toon 紧凑格式 |
| `query-meals` | 查询当前门店可售餐品列表（分类、餐品编码、标签） |
| `query-meal-detail` | 查询餐品详情（套餐组成、默认选择） |

### 门店与地址

| 工具 | 用途 |
| --- | --- |
| `query-nearby-stores` | 查询用户地址附近门店 |
| `delivery-query-addresses` | 查询用户配送地址列表 |
| `delivery-create-address` | 新增配送地址 |
| `delivery-query-stores` | 外送场景查询可配送门店 |

### 优惠与积分

| 工具 | 用途 |
| --- | --- |
| `query-my-account` | 查询积分账户（可用/累计/冻结/即将过期积分） |
| `query-my-coupons` | 查询卡包可用优惠券 |
| `available-coupons` | 查询可领取的麦麦省券列表 |
| `auto-bind-coupons` | 一键领取所有可用优惠券 |
| `query-store-coupons` | 查询当前门店可用券 |
| `calculate-price` | 计算商品金额、配送费、优惠金额及应付总价（可含券） |
| `mall-points-products` / `mall-product-detail` | 积分商城商品列表/详情 |
| `mall-create-order` | 积分兑换虚拟/实物商品下单 |
| `mall-order-list` / `mall-order-detail` | 麦麦商城订单列表/详情 |
| `query-lottery-info` | 查看积分抽奖活动信息（奖品、消耗规则） |
| `draw-lottery` | 执行一次积分抽奖 |
| `query-my-prizes` | 查看我的中奖记录 |

### 下单与活动

| 工具 | 用途 |
| --- | --- |
| `create-order` | 创建订单，返回订单详情与支付链接 |
| `cancel-order` | 取消点餐订单 |
| `query-order` | 查询订单状态与内容 |
| `order-list` | 查询近期到店/外送历史订单 |
| `campaign-calendar` | 查询当月营销活动日历 |
| `now-time-info` | 获取服务器当前时间 |

### 主题活动（麦当劳派对 / 品鉴会预约，v1.0.7+）

| 工具 | 用途 |
| --- | --- |
| `query-party-city` | 查询主题活动可参与城市列表 |
| `query-party-store` | 查询指定城市下可参与门店 |
| `query-partystore-date` | 查询指定门店可预约日期 |
| `query-partystore-session` | 查询指定日期可预约场次 |
| `party-order-create` | 创建主题活动订单 |

> 工具清单对应官方 MCP Server v1.0.9（2026-09-10）。官方仍在持续新增工具，以官方仓库 README 为准。

## 四、常见问题

### calculate-price / create-order 报"缺少参数"

这两个工具需传完整 JSON 参数：商品列表字段名为 `items`（不是 `productList`/`products`），且需包含 `storeCode`、`beCode`、`addressId`（外送下单时）等顶层字段。

### 返回 401

Token 无效、过期或未提供。请检查 `Authorization` 请求头，或到 https://open.mcd.cn/mcp 控制台重新获取 Token。

### 返回 429

触发限流（超过 600 次/分钟）。请降低请求频率，稍后重试。

### 营养数据解析异常

`list-nutrition-foods` 为优化 Token 消耗采用 toon 紧凑格式返回（首行为字段名，后续行为数据），请按首行字段名解析，不要按标准 JSON 数组处理。

---

*以上信息整理自麦当劳中国官方仓库 [M-China/mcd-mcp-server](https://github.com/M-China/mcd-mcp-server)，具体功能以 MCP 实际开放能力为准。*
