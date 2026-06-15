# AGENTS.md

## 项目概览

MediaCrawler 是一个基于 Python、Playwright/CDP 的多平台公开内容采集工具，入口文件是 `main.py`。支持小红书、抖音、快手、B 站、微博、贴吧、知乎等平台。

本项目仅供学习和研究使用。执行任何采集任务前，必须遵守目标平台条款、robots.txt、频率限制和项目 `LICENSE` 声明；不得用于商业用途、大规模抓取、绕过平台安全策略或任何非法用途。

## 启动条件判断

### 必需条件

- Python 版本：`>=3.11`，仓库 `.python-version` 指定为 `3.11`。
- 包管理工具：推荐使用 `uv`，项目已有 `pyproject.toml` 和 `uv.lock`。
- Node.js：`>=16.0.0`。运行抖音、知乎相关能力和部分 JS 执行逻辑时需要；文档站也依赖 Node。
- 浏览器：安装 Chrome 或 Edge。默认配置启用 CDP 模式并连接已有浏览器。
- 网络访问：需要能访问目标平台和 PyPI/镜像源以安装依赖。
- 登录方式：默认 `qrcode`，实际采集通常需要扫码或提供 Cookie/手机号登录。

### 默认启动前准备

```powershell
uv --version
node --version
uv sync
```

默认 `config/base_config.py` 中：

- `ENABLE_CDP_MODE = True`
- `CDP_CONNECT_EXISTING = True`
- `CDP_DEBUG_PORT = 9222`
- `SAVE_DATA_OPTION = "jsonl"`

因此默认不要求 MySQL、PostgreSQL、MongoDB 或 Redis。数据会以文件形式保存到 `data/` 目录。

### Chrome/CDP 要求

默认模式会连接用户已有浏览器。启动采集前需要在 Chrome 地址栏打开：

```text
chrome://inspect/#remote-debugging
```

勾选 `Allow remote debugging for this browser instance`，并确认页面显示类似：

```text
Server running at: 127.0.0.1:9222
```

如果不使用 CDP 模式，需要把 `config/base_config.py` 中 `ENABLE_CDP_MODE` 改为 `False`，并安装 Playwright 浏览器驱动：

```powershell
uv run playwright install
```

### 可选条件

- SQLite：使用 `--save_data_option sqlite` 时需要先执行 `uv run main.py --init_db sqlite`。
- MySQL：使用 `--save_data_option db` 时需要 MySQL 服务、数据库和 `.env`/环境变量配置。
- PostgreSQL：使用 `--save_data_option postgres` 时需要 PostgreSQL 服务、数据库和 `.env`/环境变量配置。
- MongoDB：使用 `--save_data_option mongodb` 时需要 MongoDB 服务和连接配置。
- Redis：只有启用 Redis 缓存或相关测试时需要。
- 代理服务：只有 `ENABLE_IP_PROXY = True` 时需要配置快代理、豌豆 HTTP、极速 HTTP 或静态代理。
- 词云：只有 `ENABLE_GET_WORDCLOUD = True` 且保存为 `json/jsonl` 时会使用字体、停用词和词云相关依赖。

## 常用命令

查看命令参数：

```powershell
uv run main.py --help
```

关键词搜索：

```powershell
uv run main.py --platform xhs --lt qrcode --type search
```

执行 rumor_crawler 生成的谣言题材跨平台搜索任务：

```powershell
.\.venv\Scripts\python.exe .\run_topic_tasks.py --tasks-file ..\rumor_crawler\data\topic_search_tasks_all.json --platforms dy,ks,wb,bili,xhs --max-notes-count 3 --limit 20
```

先预览不实际采集：

```powershell
.\.venv\Scripts\python.exe .\run_topic_tasks.py --tasks-file ..\rumor_crawler\data\topic_search_tasks_all.json --platforms dy,ks --limit 5 --dry-run
```

`run_topic_tasks.py` 默认把数据写入相邻项目 `..\rumor_crawler\data\mediacrawler`，并使用 `..\rumor_crawler\state\topic_crawler_checkpoint.json` 断点续跑。平台目录沿用 MediaCrawler 平台代码：`dy/jsonl`、`ks/jsonl`、`wb/jsonl`、`bili/jsonl`、`xhs/jsonl`。

详情采集：

```powershell
uv run main.py --platform xhs --lt qrcode --type detail
```

启动 WebUI：

```powershell
uv run uvicorn api.main:app --port 8080 --reload
```

或：

```powershell
uv run python -m api.main
```

如果 `uv` 不在 PATH，但仓库已有 `.venv`，直接使用本地虚拟环境启动已有接口服务：

```powershell
.\.venv\Scripts\python.exe -m uvicorn api.main:app --host 127.0.0.1 --port 8080
```

用户要求“启动项目”时，优先按上述已有接口服务入口前台启动并给出访问地址；不要为了后台常驻反复尝试 `Start-Process`、`cmd start`、`pythonw`、`Start-Job` 等托管方式。只有用户明确要求后台运行或守护进程时，才处理后台启动方案。

运行测试：

```powershell
uv run pytest
```

文档站：

```powershell
npm install
npm run docs:dev
npm run docs:build
```

## 配置文件

- 主配置：`config/base_config.py`
- 各平台配置：`config/xhs_config.py`、`config/dy_config.py`、`config/ks_config.py`、`config/bilibili_config.py`、`config/weibo_config.py`、`config/tieba_config.py`、`config/zhihu_config.py`
- 数据库配置：`config/db_config.py`
- 环境变量示例：`.env.example`
- 命令行参数：`cmd_arg/arg.py`

修改 `.env`、数据库连接、代理密钥、登录 Cookie、环境变量相关代码前，需要先确认不会泄露敏感信息。

## 数据保存

支持 `csv`、`json`、`jsonl`、`excel`、`sqlite`、`db`、`mongodb`、`postgres`。默认是 `jsonl`。

数据库初始化命令：

```powershell
uv run main.py --init_db sqlite
uv run main.py --init_db mysql
uv run main.py --init_db postgres
```

MySQL/PostgreSQL 需要提前创建数据库并配置 `.env` 或系统环境变量。SQLite 默认保存到 `database/sqlite_tables.db`。

## 开发约定

- 默认使用中文交流、中文说明和中文注释；代码标识符保持英文。
- 优先遵循现有代码结构和配置风格，不做无关重构。
- 修改启动、存储、登录、代理、数据库、平台规则等核心逻辑前，先确认影响范围。
- 不提交密钥、Cookie、Token、手机号、验证码或真实账号信息。
- 不执行会修改远程仓库或重写历史的 Git 命令，除非用户明确要求。
- 不删除或重命名文件，除非用户明确要求。
- 不运行高风险系统命令，例如 `rm -rf`、`git reset --hard`、`kill -9`，除非用户明确确认。

## 排查提示

- Git 报 `dubious ownership` 时，不要自动修改全局 Git 配置；如确需使用 Git，再让用户确认是否执行 `git config --global --add safe.directory D:/productProgram/MediaCrawler`。
- README 在 PowerShell 中乱码时，使用 `Get-Content -Encoding UTF8 README.md`。
- 默认 CDP 连接失败时，先检查 Chrome 是否启用远程调试、端口是否为 `9222`、浏览器是否弹出确认框。
- 如果目标平台一直登录失败，优先降低并发、关闭 headless、使用真实浏览器登录态，并检查是否触发平台风控。
