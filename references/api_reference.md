# 次方量化API参考文档

本文档详细说明次方量化数据接口的使用方法。

## 基础信息

### 认证
- **请求头**: `x-api-key: YOUR_API_KEY`
- **获取API Key**: 访问 https://www.cifangquant.com 注册并获取API Key
- **权限**: 仅限VIP会员使用

### 响应格式
所有接口返回统一的JSON格式：

```json
{
  "code": 0,           // 业务码，0表示成功
  "message": "ok",     // 人类可读提示
  "data": {}           // 业务数据，失败时为null
}
```

### 错误码
- `0`: 成功
- `401`: API Key无效或未提供
- `400`: 参数错误
- `404`: 资源未找到
- `429`: 请求频率超限

## 基金行情接口

### 1. 基金列表接口
获取基金清单，支持关键词过滤。

**端点**: `GET /api/fund/list`

**参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `key_word` | string | 否 | 基金代码或名称关键词 |

**响应示例**:
```json
[
  {
    "fund_code": "510300",
    "fund_name": "华泰柏瑞沪深300ETF",
    "fund_type": "指数型-股票",
    "fund_market": "SH",
    "establish_date": "2012-05-04"
  }
]
```

**字段说明**:
- `fund_code`: 基金代码
- `fund_name`: 基金名称
- `fund_type`: 基金类型
- `fund_market`: 上市市场（SH-上海，SZ-深圳）
- `establish_date`: 成立日期

### 2. 历史行情接口
批量查询基金历史K线数据，支持复权选项。

**端点**: `GET /api/fund/hist_em`

**参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `symbol` | string | 是 | 基金代码，多个以英文逗号分隔，最多50个 |
| `start_date` | date | 是 | 开始日期，格式YYYY-MM-DD |
| `end_date` | date | 是 | 结束日期，格式YYYY-MM-DD |
| `adjust` | string | 否 | 复权类型：`none`(不复权)/`qfq`(前复权)/`hfq`(后复权)，默认`none` |

**响应格式**:
```json
{
  "510300": [
    ["2024-01-02", 3.45, 3.48, 3.50, 3.44, 1.23, 1000000],
    ["2024-01-03", 3.48, 3.50, 3.52, 3.47, 0.57, 1200000]
  ]
}
```

**数据字段说明**（每个数组元素）:
| 索引 | 类型 | 字段 | 说明 |
|------|------|------|------|
| 0 | string | 日期 | 交易日期，YYYY-MM-DD格式 |
| 1 | number | 开盘价 | 当日开盘价 |
| 2 | number | 收盘价 | 当日收盘价 |
| 3 | number | 最高价 | 当日最高价 |
| 4 | number | 最低价 | 当日最低价 |
| 5 | number | 涨跌幅 | 百分比数值，如1.23表示1.23% |
| 6 | number | 成交量 | 当日成交量 |

## 策略接口（可选）

### 3. 策略列表接口
获取自动化交易策略列表。

**端点**: `GET /api/trading/list`

**参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `account` | string | 否 | 按资金账号筛选 |

### 4. 策略信号接口
获取指定策略的当日交易信号。

**端点**: `GET /api/trading/get_strategy_signal`

**参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `strategy_id` | integer | 是 | 自动化交易ID |

## 使用示例

### 示例1：获取基金列表
```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://www.cifangquant.com/api/fund/list?key_word=沪深300"
```

### 示例2：获取历史数据
```bash
curl -H "x-api-key: YOUR_API_KEY" \
  "https://www.cifangquant.com/api/fund/hist_em?symbol=510300,510500&start_date=2024-01-01&end_date=2024-12-31&adjust=qfq"
```

### 示例3：Python代码示例
```python
import requests

api_key = "YOUR_API_KEY"
headers = {"x-api-key": api_key}

# 获取基金列表
response = requests.get(
    "https://www.cifangquant.com/api/fund/list",
    headers=headers,
    params={"key_word": "510300"}
)

# 获取历史数据
response = requests.get(
    "https://www.cifangquant.com/api/fund/hist_em",
    headers=headers,
    params={
        "symbol": "510300,510500",
        "start_date": "2024-01-01",
        "end_date": "2024-12-31",
        "adjust": "qfq"
    }
)
```

## 注意事项

### 1. 数据限制
- 历史行情接口最多支持50个基金代码
- 需要有效的API Key（VIP会员）
- 数据更新频率：交易日收盘后

### 2. 日期处理
- 日期格式必须为YYYY-MM-DD
- 开始日期不能晚于结束日期
- 建议合理设置日期范围，避免数据量过大

### 3. 错误处理
- 401错误：检查API Key是否正确
- 400错误：检查参数格式
- 429错误：降低请求频率

### 4. 复权说明
- `none`: 不复权，原始价格
- `qfq`: 前复权，以当前价格为基准向前调整
- `hfq`: 后复权，以历史价格为基准向后调整

## 常见问题

### Q1: 如何获取API Key？
A: 访问 https://www.cifangquant.com 注册账号，在个人中心获取API Key。

### Q2: 支持实时行情吗？
A: 当前API主要提供历史行情数据。实时行情可能需要其他数据源。

### Q3: 数据来源是什么？
A: 数据来源于东方财富等公开数据源，经过清洗和整理。

### Q4: 请求频率有限制吗？
A: 文档未明确说明，但建议合理控制请求频率，避免被封禁。

### Q5: 支持哪些类型的基金？
A: 主要支持A股场内交易基金，包括ETF、LOF等。

## 更新日志

### V1.0 (2024-01-01)
- 初始版本发布
- 支持基金列表查询
- 支持历史行情数据获取
- 支持复权选项

---

**重要提示**: 使用本API需要遵守次方量化的服务条款，仅限个人或内部使用，不得用于商业用途或大规模数据爬取。