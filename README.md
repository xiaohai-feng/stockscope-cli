# stockscope-cli

A股数据分析 CLI 工具，覆盖估值、行情、基本面、资金等维度，输出 PNG 图表和结构化 JSON，可被 AI agent（如 OpenClaw、Claude Code）直接调用。

## 安装

```bash
npm install -g stockscope-cli
```

> Windows 用户如安装失败，通常不是 CLI 本身有问题，而是 `canvas` 或 `better-sqlite3` 这类原生模块没有拿到预编译包，转而尝试本地编译失败。可按下面的“安装失败排查”处理。

### 安装失败排查

安装失败通常不是 CLI 本身有问题，而是原生模块没有拿到预编译二进制，转而尝试本地编译失败。  
最常见的模块是：

- `canvas`
- `better-sqlite3`
- `node-gyp`

常见触发原因：

- 当前 Node 版本没有对应的预编译二进制包
- 网络或代理导致预编译包下载失败
- 本机缺少编译环境或系统库

建议按下面顺序处理。

#### 1. 先确认 Node 版本

推荐使用 Node.js 22+。

```bash
node -v
npm -v
```

如果原生模块安装失败，建议优先切换到稳定的 LTS 版本后重新安装。

#### 2. 按平台补齐依赖

##### Windows

请安装：

- [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
- Python 3

安装 `Visual Studio Build Tools` 时，至少勾选：

- `Desktop development with C++`
- `MSVC v143` 或更新版本的 C++ 编译工具
- `Windows 10 SDK` 或 `Windows 11 SDK`

安装完成后，重新打开一个新的终端窗口。

##### macOS

请先安装 Xcode Command Line Tools：

```bash
xcode-select --install
```

如果后续仍因 `canvas` 编译失败，可先安装 Homebrew，再补常见依赖：

```bash
brew install pkg-config cairo pango libpng jpeg giflib librsvg
```

##### Linux（Ubuntu / Debian）

先安装构建工具和 `canvas` 常见依赖：

```bash
sudo apt update
sudo apt install -y build-essential python3 pkg-config libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev
```

如果你使用的是其他发行版，需要安装等价的：

- C/C++ 构建工具链
- Python 3
- `pkg-config`
- `cairo`
- `pango`
- `jpeg`
- `gif`
- `librsvg`

#### 3. 重新执行安装

```bash
npm install -g stockscope-cli
```

#### 4. 如果仍失败，再重建原生模块

```bash
npm rebuild canvas better-sqlite3
```

如果你是全局安装后重建失败，也可以先进入一个临时目录做本地安装排查：

```bash
mkdir stockscope-install-test
cd stockscope-install-test
npm init -y
npm install stockscope-cli
```

然后再执行：

```bash
npm rebuild canvas better-sqlite3
```

#### 5. 如果报 `node-gyp` 或 Python 相关错误

先确认 Python 可用：

```bash
python --version
```

如果系统里有多个 Python，可显式指定：

```bash
npm config set python python
```

或者指定完整路径：

```bash
npm config set python "C:\\Path\\To\\Python\\python.exe"
```

#### 6. 如果报预编译包下载失败

这通常是网络、代理或镜像问题。可以尝试：

- 更换网络环境
- 关闭有问题的代理
- 恢复官方 npm registry

```bash
npm config set registry https://registry.npmjs.org/
```

然后重新执行：

```bash
npm install -g stockscope-cli
```

#### 7. 分平台常见额外问题

##### Windows

- 最常见是没有 `Visual Studio Build Tools`
- 也可能是 Python 未安装或未加入 PATH

##### macOS

- 最常见是没有安装 `xcode-select --install`
- Apple Silicon 机器上，如果 Node 版本较新，也更容易因为没有预编译包而回退源码编译

##### Linux

- 最常见是缺少 `build-essential`
- 其次是缺少 `cairo/pango` 等图形库开发包
- Alpine Linux 这类环境通常比 Ubuntu/Debian 更容易遇到原生模块安装问题

#### 8. 让 AI 协助排查

如果你不确定是哪一步出错，可以把完整报错日志交给 AI 助手。  
AI 通常可以帮助你判断：

- 是 `canvas` 失败还是 `better-sqlite3` 失败
- 是缺少 Build Tools、Windows SDK，还是 Python
- 下一步应该执行哪条命令

#### 9. 仍无法安装

如果你只是想使用 CLI，而不是自己折腾本地编译环境，后续更推荐直接使用预编译二进制发行版；这样可以绕开 Windows 上原生模块安装链路。

## 使用

```bash
# PE 估值通道（默认）
stock-cli band 600519.SH

# PB 估值通道，5年数据
stock-cli band 600519.SH --type pb --years 5

# 指定图片输出目录
stock-cli band 600519.SH --output ./charts/

# 日K线图（前复权），近 90 天
stock-cli kline 600519.SH --range 90d

# K线图输出到指定目录
stock-cli kline 600519.SH --range 90d --output ./charts/

# 获取最新价格和涨跌幅
stock-cli quote 600519.SH

# 查看单只股票某个报告期的全部财务指标
stock-cli financial detail 600519.SH --period 20241231

# 获取财务字段字典
stock-cli financial fields

# 按最多 3 个财务条件筛选股票
stock-cli financial screen --period 20241231 --cond "roe>10" --cond "grossprofit_margin>=30"

# 按报告类型筛选
stock-cli financial screen --period 20241231 --cond "report_type=年报"

# 周K线图，近 2 个月
stock-cli kline 600519.SH --period w --range 2m

# 强制刷新缓存
stock-cli kline 600519.SH --refresh
```

### 注意事项

- 如果通过 OpenClaw、Claude Code 等 AI agent 使用本工具，生成 K 线图、估值图等图片时，建议将 `--output` 指定到 agent 的 workspace 目录下，避免图片生成后 agent 没有权限读取。

### band 参数说明

| 参数 | 简写 | 默认值 | 说明 |
|------|------|--------|------|
| `--type` | `-t` | `pe` | 估值类型：pe / pb / ps / pcf |
| `--years` | `-y` | `10` | 数据年限：3 / 5 / 10 |
| `--output` | `-o` | 系统临时目录 | 图片输出目录 |

### kline 参数说明

| 参数 | 简写 | 默认值 | 说明 |
|------|------|--------|------|
| `--period` | `-p` | `d` | K线周期：d=日 / w=周 / m=月 |
| `--range` | - | `1y` | 显示区间，如 `90d` / `2m` / `1y` |
| `--output` | `-o` | 系统临时目录 | 图片输出目录 |
| `--refresh` | `-r` | false | 强制刷新本地缓存 |

### quote 参数说明

无额外参数。

### financial fields 参数说明

无额外参数。

`financial fields` 用于给人或 AI agent 获取财务字段的中文含义、单位、类型和筛选能力说明。

### financial detail 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--period` | 无 | 报告期，固定 `YYYYMMDD` |

字段名保持原始 snake_case；如需字段中文含义，请先调用 `financial fields`。

### financial screen 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--period` | 无 | 报告期，固定 `YYYYMMDD` |
| `--cond` | 无 | 筛选条件，如 `roe>10`，最多 3 个 |
| `--page` | `1` | 页码 |
| `--page-size` | `50` | 每页条数，最大 `200` |

`--cond` 规则：

- 数值字段与 `report_year` 支持 `> >= < <= =`
- `report_type` 仅支持 `=`，例如 `--cond "report_type=年报"`


### band 输出格式

```json
{
  "success": true,
  "imagePath": "/tmp/600519_SH_pe_band.png",
  "stockCode": "600519.SH",
  "type": "pe",
  "currentPrice": 1498.00,
  "currentBand": "24.7x ~ 34.7x",
  "bands": [14.66, 24.66, 34.66, 44.66, 54.66]
}
```

### kline 输出格式

```json
{
  "success": true,
  "imagePath": "/tmp/600519_SH_d_kline.png",
  "stockCode": "600519.SH",
  "period": "d",
  "range": "90d",
  "latestDate": "2026-03-17",
  "latestClose": 1485.0,
  "latestPctChg": 1.70,
  "count": 250
}
```

### quote 输出格式

```json
{
  "success": true,
  "stockCode": "600519.SH",
  "name": "贵州茅台",
  "current": 1459.44,
  "pctChg": 0.65,
  "chg": 9.44,
  "preClose": 1450,
  "provider": "eastmoney",
  "fetchedAt": "2026-04-01T13:49:03.300Z"
}
```

### financial detail 输出格式

```json
{
  "success": true,
  "tsCode": "600519.SH",
  "period": "20241231",
  "data": {
    "ts_code": "600519.SH",
    "ann_date": "20250328",
    "end_date": "20241231",
    "roe": 34.21,
    "grossprofit_margin": 91.23
  }
}
```

### financial fields 输出格式

```json
{
  "success": true,
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
  ],
  "count": 1
}
```

### financial screen 输出格式

```json
{
  "success": true,
  "period": "20241231",
  "conditions": ["roe>10", "grossprofit_margin>=30"],
  "list": [
    {
      "ts_code": "600519.SH",
      "values": {
        "grossprofit_margin": 91.23,
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

## 交流

祝大家投资顺利！

欢迎交流使用问题。

![微信二维码](https://raw.githubusercontent.com/xiaohai-feng/stockscope-cli/main/docs/assets/wechat-qrcode.jpg)
