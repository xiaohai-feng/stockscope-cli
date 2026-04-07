---
name: stockscope-cli
description: 当 agent 需要调用本地 `stock-cli`、选择正确的 CLI 子命令并解析 JSON 输出时使用。适用于实时价格、K 线图、估值通道图、财务字段字典、财务详情与财务筛选。
---

# Stockscope CLI

这是给 OpenClaw、Claude Code 等 AI agent 使用的技能说明，不是面向普通终端用户的使用文档。

## 适用场景

当任务需要调用本地 `stock-cli` 获取 A 股估值、K 线、财务或最新价格数据时，使用这个技能。

适用范围：

- 最新价格与涨跌幅
- K 线图
- 估值通道图
- 财务字段字典
- 单股单报告期财务详情
- 财务条件筛选

## 输出约定

CLI 的 stdout 始终输出 JSON。

- 成功：`{"success": true, ...}`
- 失败：`{"success": false, "error": "..."}`

如果需要给上层 agent 继续处理，优先直接解析 stdout JSON，不要依赖 stderr 文本。

## 命令映射

按任务类型选择命令：

如需查看命令参数或可选项，直接执行：

```powershell
stock-cli --help
stock-cli <命令> --help
```

- 最新价格与涨跌幅

```powershell
stock-cli quote <股票代码>
# 示例：stock-cli quote 600519.SH
```

- K 线图

```powershell
stock-cli kline <股票代码> [--period d|w|m] [--range 90d|2m|1y]
# 示例：stock-cli kline 600519.SH --period d --range 90d
```

- K 线图输出到指定目录

```powershell
stock-cli kline <股票代码> [--period d|w|m] [--range 90d|2m|1y] --output <目录>
# 示例：stock-cli kline 600519.SH --period d --range 90d --output ./workspace/charts
```

- 估值通道图

```powershell
stock-cli band <股票代码> [--type pe|pb|ps|pcf] [--years 3|5|10]
# 示例：stock-cli band 600519.SH --type pe --years 10
```

- 估值通道图输出到指定目录

```powershell
stock-cli band <股票代码> [--type pe|pb|ps|pcf] [--years 3|5|10] --output <目录>
# 示例：stock-cli band 600519.SH --type pe --years 10 --output ./workspace/charts
```

- 财务字段字典

```powershell
stock-cli financial fields
```

- 单只股票、单个报告期的完整财务详情

```powershell
stock-cli financial detail <股票代码> --period <YYYYMMDD>
# 示例：stock-cli financial detail 600519.SH --period 20241231
```

- 财务条件筛选

```powershell
stock-cli financial screen --period <YYYYMMDD> --cond "<字段><运算符><值>"
# 示例：stock-cli financial screen --period 20241231 --cond "roe>10"
```

## 使用规则

### 图片输出

- 如果任务需要读取 CLI 生成的图片，优先显式传入 `--output`。
- 对 OpenClaw 这类只能读取 agent workspace 文件的环境，图片输出目录必须放在当前 workspace 下。
- 不要把图片输出到系统临时目录、用户桌面或 workspace 之外的目录，否则上层 agent 可能无法读取。

### 财务相关

- 如果字段含义很重要，先执行 `financial fields`。
- `financial detail` 返回单只股票、单个报告期的完整原始 snake_case 字段。
- `financial screen` 只返回 `ts_code` 和命中的字段值。
- 报告期必须是 `YYYYMMDD`。
- `financial screen` 最多支持 3 个 `--cond`。
- 运算符支持：`>`、`>=`、`<`、`<=`、`=`
- `report_type` 只支持 `=`

## 常用示例

```powershell
stock-cli financial fields
stock-cli financial detail 600519.SH --period 20241231
stock-cli financial screen --period 20241231 --cond "roe>10" --cond "grossprofit_margin>=30"
stock-cli financial screen --period 20241231 --cond "report_type=年报"
stock-cli quote 600519.SH
stock-cli band 600519.SH --type pe --years 10 --output ./workspace/charts
stock-cli kline 600519.SH --period d --range 90d --output ./workspace/charts
```

## 典型输出

`financial fields`

```json
{
  "success": true,
  "count": 85,
  "list": [
    {
      "field": "roe",
      "label": "净资产收益率",
      "description": "净利润与平均净资产的比率",
      "dataType": "number",
      "unit": "%",
      "filterable": true,
      "operators": [">", ">=", "<", "<=", "="]
    }
  ]
}
```

`financial detail`

```json
{
  "success": true,
  "tsCode": "600519.SH",
  "period": "20241231",
  "data": {
    "ts_code": "600519.SH",
    "report_type": "年报",
    "roe": 8.22
  }
}
```

`financial screen`

```json
{
  "success": true,
  "period": "20241231",
  "conditions": ["roe>10"],
  "list": [
    {
      "ts_code": "600519.SH",
      "values": {
        "roe": 34.21
      }
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 50,
    "total": 128,
    "totalPages": 3
  }
}
```
