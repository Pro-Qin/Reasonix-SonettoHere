<p align="center">
  <img src="docs/logo.svg" alt="Reasonix" width="640"/>
</p>

<p align="center">
  <a href="./README.md">English</a>
  &nbsp;·&nbsp;
  <strong>简体中文</strong>
  &nbsp;·&nbsp;
  <a href="./docs/GUIDE.zh-CN.md">指南</a>
  &nbsp;·&nbsp;
  <a href="./docs/SPEC.md">规格</a>
</p>

<p align="center">
  <a href="./LICENSE"><img src="https://img.shields.io/npm/l/reasonix.svg?style=flat-square&color=8b949e&labelColor=161b22" alt="license"/></a>
  <a href="https://github.com/Pro-Qin/Reasonix-SonettoHere/stargazers"><img src="https://img.shields.io/github/stars/Pro-Qin/Reasonix-SonettoHere?style=flat-square&color=dbab09&labelColor=161b22&logo=github&logoColor=white" alt="GitHub stars"/></a>
</p>

<br/>

<h3 align="center">面向终端的 DeepSeek 原生 AI coding agent —— Reasonix 内核，SonettoHere 生态。</h3>
<p align="center">由配置与插件驱动的极薄 harness——单一静态 Go 二进制，围绕 DeepSeek 的前缀缓存调优，长会话也能把 token 成本压低。</p>

<br/>

# 🎆 SonettoHere 集成

**Reasonix-SonettoHere** 将 [SonettoHere](https://github.com/Miso2233/SonettoHere) 的领域工具生态融入到 [Reasonix](https://github.com/esengine/DeepSeek-Reasonix) 的 Go 原生架构中。

### 内置工具一览

| 类别 | 工具 | 所需 API Key |
|------|------|-------------|
| 🌤️ **天气与日历** | `get_current_weather`、`holiday_calendar` | `UAPIS_API_KEY` |
| 🗺️ **高德地图** | `geocode_address`、`regeocode`、`nearby_search`、`fuzzy_search_poi`、`get_transit_route`、`get_cycling_route` | `AMAP_API_KEY` |
| ✅ **Todoist 任务** | `todoist_add`、`todoist_list_tasks`、`todoist_complete_task`、`todoist_delete_task`、`todoist_list_projects` | `TODOIST_API_TOKEN` |
| 🎮 **娱乐** | `tarot_reading`、`answer_book` | `UAPIS_API_KEY` |
| ⏰ **系统** | `current_time` | — |
| ❤️ **健康检查** | `health_check`、`GET /health` | — |

> API Key 仅在用到对应工具时需要，不影响基础对话。可通过 `.env` 文件或系统环境变量配置。

<br/>

## 特性

- **配置驱动**：provider、agent、启用的工具、插件全部在 `reasonix.toml` 中声明，
  内核无硬编码模型。
- **多模型 · 可组合**：DeepSeek（flash/pro）与 MiMo 作为预设内置；任何 OpenAI 兼容
  端点都只是一条配置。可选让两个模型协同（执行器 + 规划器），各自独立、缓存稳定的 session。
- **插件驱动**：外部工具以子进程形式运行，通过 stdio JSON-RPC 通信（MCP 兼容）；
  内置工具在编译期自注册。
- **领域工具生态**：额外 15+ 个内置工具，涵盖天气、地图、任务管理、娱乐和健康检查——
  全部 Go 原生实现，无需 Python 运行环境。

<br/>

## 安装

```sh
npm i -g reasonix                  # 任意系统;自动拉取对应平台的原生二进制
brew install esengine/reasonix/reasonix   # macOS
```

预编译归档(`darwin|linux|windows × amd64|arm64`)和 `SHA256SUMS` 见每个
[GitHub release](https://github.com/Pro-Qin/Reasonix-SonettoHere/releases)。

### 从源码构建

```sh
make build      # -> bin/reasonix(.exe)
make cross      # -> dist/（darwin|linux|windows × amd64|arm64）
```

## 快速开始

```sh
reasonix setup                      # 配置向导 → ./reasonix.toml
export DEEPSEEK_API_KEY=sk-...      # 也可以让 setup 保存到凭据存储
reasonix                            # 然后在会话里运行 /init 生成 AGENTS.md（项目记忆）
reasonix run "把 main.go 里的 TODO 实现掉"
reasonix run --model mimo-pro "给这个函数补单元测试"
echo "解释这段代码" | reasonix run
```

## 配置

一个最小的 `reasonix.toml`——一个 provider 加一个默认模型——就够跑起来:

```toml
default_model = "deepseek-flash"

[[providers]]
name        = "deepseek-flash"
kind        = "openai"
base_url    = "https://api.deepseek.com"
model       = "deepseek-v4-flash"
api_key_env = "DEEPSEEK_API_KEY"
```

优先级为 **flag > `./reasonix.toml` > 用户配置文件 > 内置默认值**；从
**Reasonix v1.8.1** 开始，用户配置位于 macOS/Linux 的 `~/.reasonix/config.toml`，
Windows 为 `%AppData%\reasonix\config.toml`。迁移细节见
**[配置路径](./docs/CONFIG_PATHS.zh-CN.md)**。
密钥经环境变量通过 `api_key_env` 注入，绝不写入配置文件；新密钥默认优先保存到系统凭据存储，
不可用时才 fallback 到 Reasonix 管理的凭据文件。项目 `.env` 只作为兼容覆盖读取，
Reasonix 不会把新密钥写入项目 `.env`。权限、沙盒、插件(MCP)、
斜杠命令、`@` 引用与双模型设置,全部在 **[指南](./docs/GUIDE.zh-CN.md)** 里。

## 文档

- **[指南](./docs/GUIDE.zh-CN.md)** —— 配置、权限与沙盒、插件(MCP)、斜杠命令、
  `@` 引用、双模型协同。
- **[规格](./docs/SPEC.md)** —— 工程契约:架构、registry、数据类型与路线图。
- **[Checkpoints 与 rewind](./docs/CHECKPOINTS.md)** —— 基于快照的编辑安全网
  (Esc-Esc / `/rewind`)。

<br/>

---

<p align="center">
  <sub>MIT —— 见 <a href="./LICENSE">LICENSE</a></sub>
  <br/>
  <sub>上游: <a href="https://github.com/esengine/DeepSeek-Reasonix">esengine/DeepSeek-Reasonix</a> · SonettoHere: <a href="https://github.com/Miso2233/SonettoHere">Miso2233/SonettoHere</a></sub>
</p>
