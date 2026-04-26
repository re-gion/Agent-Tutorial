# Codex 从 0 到 1 使用教程

> 基于 8 篇公众号文章、笔者旧版自用手册和一组真实 Codex 配置/排障记录整理，并按 OpenAI 官方文档在 2026-04-26 做了校正。本文尽量写成一份能直接照着用的教程，而不是功能清单。

## 这篇教程适合谁

这篇教程写给三类人：

- 没怎么用过 Codex，但想把它当成长期工作工具的人。
- 会一点代码，想让 Codex 帮自己改项目、查问题、写文档、做数据分析的人。
- 已经用过 ChatGPT、Cursor、Claude Code，但还没把 Codex 的项目、线程、工作树、技能和自动化串起来的人。

读完后，你应该能做到：

- 正确安装、登录并创建 Codex 项目。
- 分清 Project、Thread、Local、Worktree 的关系。
- 写出能让 Codex 真正干活的任务型 Prompt。
- 给项目写一份可用的 `AGENTS.md`。
- 知道 `config.toml`、Skills、MCP、Subagents、Automations、Plugins 分别解决什么问题。
- 建立一套适合个人或团队长期使用的 Codex 工作流。

## 先说结论：Codex 不只是“会写代码的聊天框”

如果把 Codex 当成另一个 ChatGPT，用法通常会变成这样：

> 帮我写个页面。  
> 帮我改个 Bug。  
> 帮我优化一下代码。

这当然也能用，但很快会遇到几个问题：上下文乱、改动不可控、同样的规则要反复讲、出了错不知道怎么验证。

Codex 更接近一个“项目工作台”。它围绕项目目录、线程、Git、终端、权限、技能、外部工具和自动化任务工作。你不是只和它聊天，而是把一个具体任务交给它，让它在项目里读文件、改文件、跑命令、检查结果，然后把可审查的成果交回来。

一个实用的心智模型是：

- ChatGPT 更像问答助手。
- Codex 更像能进入你项目目录工作的执行型 Agent。
- 用好 Codex 的关键，不是把 Prompt 写得很玄，而是把项目边界、任务目标、约束和验收标准讲清楚。

## 第 1 步：确认你能用 Codex

截至 2026-04-26，OpenAI 官方 Help Center 说明：Codex 可随 ChatGPT Plus、Pro、Business、Edu、Enterprise 计划使用。不同地区、账号和客户端可能有灰度差异，具体以你账号里能看到的入口为准。

常见入口有几种：

- Codex App：适合希望使用图形界面的人。
- Codex CLI：适合习惯终端的人。
- IDE Extension：适合在 VS Code、Cursor、Windsurf 等编辑器里直接使用的人。
- Codex Web / Cloud：适合把任务交给云端环境执行的人。

如果你是第一次上手，建议先用 Codex App。原因很简单：它能看到项目、线程、改动差异、终端、权限和自动化任务，对新手更直观。

### 安装与登录

1. 打开 Codex 官方入口：<https://chatgpt.com/codex>
2. 下载适合你系统的客户端。
3. 安装后用 ChatGPT 账号登录。
4. 登录后先不要急着让它写代码，先把下面几件事看明白。

如果你用 CLI，可以查看官方当前安装方式。官方 changelog 在 2026-04-24 给出的 CLI 示例是：

```bash
npm install -g @openai/codex@0.125.0
```

版本会变化，不要死记这一行。真正安装时以官方文档或客户端提示为准。

### 补充学习材料

笔者旧版自用手册里推荐过一个 B 站入门视频，可以作为图形界面上手参考：

- 《16分钟，Codex从入门到毕业，上线作品集网站》  
  <https://www.bilibili.com/video/BV1dQXrBVECR>

这个视频的价值不在于每个界面细节都永远不变，而在于它演示了一个完整工作流：用 Codex 做一个作品集网站、部署到 Vercel、顺带理解 Worktree、Skills 和自动化任务。看视频时建议重点学“任务怎么拆、怎么让 Codex 自己配置、怎么验证结果”，不要死记按钮位置。

### App 里建议先调的几个设置

第一次用 Codex App，可以先检查这几个设置。

| 设置 | 建议 | 原因 |
| --- | --- | --- |
| 长文本发送 | 开启“需要 Ctrl + Enter 发送” | 写复杂 Prompt 时，普通 Enter 用来换行，避免没写完就误发送 |
| Follow-up behavior | 日常用 Queue，复杂任务中可用 Steer | Queue 更稳；Steer 适合任务执行中途纠偏 |
| Speed / Service tier | 默认 Standard | Fast 会更快，但可能消耗更多额度，赶时间再开 |
| Code review 显示方式 | 大审查用 Detached | 长篇 Review 不会把当前任务线程淹没 |
| Pop-out Window | 按需设置快捷键 | 做前端或长任务时，可以把当前线程悬浮在旁边看 |

Follow-up behavior 需要单独解释一下。Codex 正在工作时，你又发一条消息：

- Queue：先排队，等当前任务结束再处理。适合多数情况。
- Steer：尝试立刻影响当前任务。适合发现方向错了，需要马上纠偏。

如果你经常写多段 Prompt，建议开启“Ctrl + Enter 发送”。这个设置比想象中重要，因为复杂任务最怕误按 Enter 让 Codex 提前开工。

## 第 2 步：把项目目录选对

Codex 里的 Project，本质上就是一个工作目录。这个目录选错了，后面所有事情都会跟着跑偏。

建议这样选：

| 场景 | 建议 |
| --- | --- |
| 单项目仓库 | 直接选择仓库根目录 |
| monorepo 里有多个 app/package | 只选择当前真正要工作的 app 或 package |
| 只是写文章、整理资料、处理脚本 | 选择最小的资料目录或脚本目录 |
| 不确定项目怎么跑 | 先让 Codex 只读分析，不要马上改代码 |

这里容易踩坑：很多人一上来把整个大仓库丢给 Codex，然后抱怨它乱看、乱改。实际上，是工作区边界给得太大了。

一个好习惯是：一个 Project 对应一个清晰的工作范围。项目越聚焦，Codex 越不容易跑偏。

## 第 3 步：理解 Project、Thread、Local、Worktree

这是 Codex 的基础概念，必须先搞清楚。

### Project：工作目录

Project 是你添加进 Codex 的文件夹。它决定 Codex 能看到哪些文件、能在哪个目录里跑命令、能对哪些内容做修改。

### Thread：一条任务线

Thread 是一次具体任务的对话和执行过程。一个 Project 下面可以有很多 Thread。

推荐规则：

- 一个 Thread 只做一件具体的事。
- 一个复杂项目可以有多个 Thread。
- 不要把写页面、改 Bug、整理数据、写报告全塞进一个 Thread。

这样做的好处是上下文不会互相污染，之后也更容易回看和继续。

### Local：直接改当前项目

Local 模式就是在你当前项目目录里工作。Codex 改的就是你正在看的这份文件。

适合：

- 写文档。
- 小范围代码修改。
- 你已经知道修改范围，风险不高。
- 你希望改动立刻出现在当前目录。

### Worktree：在 Git 副本里改

Worktree 是 Git 创建出来的独立工作副本。Codex 在这个副本里改文件，不会直接碰你当前项目目录。

可以把它理解成：

- Local 是改原件。
- Worktree 是先复制一份试改版，再在试改版里改。

适合：

- 让 Codex 尝试一个不确定的大方案。
- 同一个项目里并行跑多个任务。
- 担心 Codex 把当前工作区弄乱。
- 自动化任务需要隔离改动。

使用 Worktree 前要注意：项目必须是 Git 仓库，并且通常需要至少有一次提交。刚初始化 Git 但没有提交时，Worktree 可能无法正常创建。

### 新手怎么选

刚开始不用复杂：

- 问问题、写文章、小改动：用 Local。
- 大改、试方案、并行任务：用 Worktree。
- 自动化任务：优先用 Worktree，避免影响你正在编辑的文件。

## 第 4 步：Windows 用户先决定 PowerShell 还是 WSL2

Windows 上使用 Codex 时，最重要的不是“哪个更高级”，而是不要混乱。

如果你的项目本来就在 Windows 原生环境里跑得很好，比如 Node、Python、小工具脚本，直接用 PowerShell 就可以。

如果项目明显依赖 Linux 工具链，比如 bash、make、Docker、Linux 路径、shell 脚本，那就尽早使用 WSL2。

不要今天在 PowerShell 里装依赖，明天又去 WSL2 里跑同一个项目。这样最容易出现路径、权限、依赖、Node/Python 版本全部混在一起的问题。

建议：

- Windows 原生项目：PowerShell。
- Linux 生态项目：WSL2。
- 电脑用户名或项目路径含中文且反复遇到权限问题：尽量把项目和 `CODEX_HOME` 放在纯英文路径。

## 第 5 步：第一次真正让 Codex 干活

不要一上来就给大任务。先从一个低风险任务开始。

比如在一个代码项目里，你可以这样问：

```text
先不要修改代码。请阅读当前项目结构，告诉我：
1. 这个项目大概是做什么的
2. 主要入口文件在哪里
3. 常用的启动、测试、构建命令可能是什么
4. 如果我要改一个页面或接口，应该先看哪些目录
```

这个任务的价值是让 Codex 先熟悉项目，也让你确认它有没有看对目录。

如果是文档或资料目录，可以这样问：

```text
先不要修改文件。请阅读当前目录结构，告诉我：
1. 这里有哪些主要资料
2. 哪些文件像是草稿，哪些像是成稿
3. 如果我要写一篇教程，建议把新文件放在哪里
4. 有哪些命名或格式习惯需要遵守
```

确认它理解正确后，再让它改。

## 第 6 步：把 Prompt 写成任务单

Codex 不是不能理解自然语言，而是模糊任务会放大风险。好的 Prompt 不需要花哨，但要像任务单。

一个实用模板：

```text
目标：
在 [范围] 内完成 [具体任务]。

上下文：
- 重点阅读 [文件/目录]
- 不要改 [文件/目录]

约束：
- 不新增依赖
- 保持现有代码风格
- 不改变现有 API

完成标准：
- [某个行为成立]
- [某个命令通过]
- [某个输出符合要求]

执行要求：
先阅读相关文件并说明方案，再动手修改。修改后运行最小必要验证，并汇报改了什么、验证了什么、还有什么风险。
```

示例：

```text
在 `src/features/login` 范围内修复登录页表单提交失败后按钮不恢复的问题。

要求：
1. 不要改 API 结构
2. 不要新增依赖
3. 先阅读相关组件、hook 和提交逻辑，再动手
4. 修改后运行 `npm test` 和 `npm run lint`
5. 最终告诉我根因、改了哪些文件、是否还有风险
```

如果你只是想让它先分析，不想它立刻动手，加一句：

```text
先不要改代码。先定位根因，给我一个最小修改方案，我确认后再动手。
```

## 第 7 步：学会看 diff、审查和验证

Codex 改完文件后，你不能只看它的总结。至少要看三件事：

1. 改了哪些文件。
2. diff 里具体删了什么、加了什么。
3. 它有没有真的跑验证。

Codex App 的 diff 面板适合看“它改了什么”。Review 功能更适合看“这些改动有没有潜在问题”。

简单区分：

- diff：看改动内容。
- Review：查风险、Bug、遗漏测试和行为回归。
- 终端验证：确认代码真的能跑、测试真的通过。

每次修改后，至少让 Codex 做一个最小验证。不要把“看起来对”当成“已经完成”。

常见验证命令包括：

```bash
npm test
npm run lint
npm run build
pnpm test
pytest
mvn test
go test ./...
cargo test
```

具体用哪一个，要看你的项目。

## 第 8 步：写一份项目级 AGENTS.md

`AGENTS.md` 是给 Codex 的项目说明书。它会在 Codex 开始工作前自动读入，让 Codex 知道这个项目怎么协作、怎么测试、哪些地方不能乱碰。

OpenAI 官方说明里，`AGENTS.md` 支持全局和项目级：

- 全局：通常在 `~/.codex/AGENTS.md`，适合写你的个人偏好。
- 项目级：放在项目根目录，适合写当前项目规则。
- 子目录级：大型项目里可以在子目录放更具体的规则。

越靠近当前工作目录的说明，越应该具体。

### 一个通用 AGENTS.md 模板

可以在项目根目录创建：

```md
# AGENTS.md

## 项目概况

- 这是一个 [项目类型] 项目。
- 主要技术栈：[语言/框架/数据库/构建工具]。
- 主要入口：[入口文件或目录]。

## 常用命令

- 安装依赖：`...`
- 本地启动：`...`
- 测试：`...`
- Lint：`...`
- 构建：`...`

## 代码约定

- 优先遵循现有目录结构和命名方式。
- 不要做与当前任务无关的大规模重构。
- 不要修改生成文件、构建产物或生产配置，除非任务明确要求。

## 安全与配置

- 不要提交密钥、token、`.env` 真实值。
- 需要新增依赖时，先说明原因。
- 涉及数据库、权限、支付、生产部署的改动，需要单独说明风险。

## 完成标准

- 修改后运行与任务相关的最小验证。
- 汇报改了什么、验证了什么、还有什么没验证。
```

写 `AGENTS.md` 时，不要塞常识。比如“Java 语句要加分号”没必要写。要写 Codex 不可能凭常识知道的东西，比如“不要改 `application-prod.yml`”“本项目只用 pnpm，不用 npm”。

### 一个实测全局 AGENTS.md 为什么好用

笔者长期使用的全局 `AGENTS.md` 已经形成了一套很适合协作的规则。它的核心不是“命令 Codex 必须怎样说话”，而是把验收标准、验证意识、工作区安全和 Windows 联网特殊情况都写清楚。

可以提炼成 6 条：

1. 默认用自然、清晰的中文汇报。
2. 先交付真实完成和验证过的结果，不把计划当成果。
3. 小问题能顺手修就顺手修，修完重新验证。
4. 代码改动要克制，尊重已有工作区，不乱重构、不乱回滚。
5. 能运行就运行，能测试就测试，不能验证就明确说风险。
6. Windows 上遇到联网失败，先区分宿主机、WSL2、Codex 沙箱三层，不把沙箱网络失败误判为项目代码问题。

如果你想把这套规则迁移到另一台电脑，可以先用下面这个精简版：

```md
# Global AGENTS.md

## 沟通与汇报

默认用通俗、清晰、自然的中文汇报。
最终回复优先说明：做了什么、结果是否可用、做过哪些验证、还有什么风险。
不要把计划、推测或理论可行说成已完成。

## 执行原则

先判断完成标准，再推进到可交付结果。
小错误、明显遗漏和简单验证缺口，优先自行修复并重新验证。
只有缺少关键权限、资源、业务决策或继续执行会明显增加误改风险时，才请求用户介入。

## 代码改动

做必要且克制的修改。
遵循现有命名、结构、格式和工程约定。
不做与当前目标弱相关的大规模重构。
不覆盖、回滚或破坏用户已有改动。

## 验证要求

能运行就运行，能测试就测试，能 lint/typecheck/build 就执行对应检查。
只汇报实际做过的验证。
没有验证的内容必须说明原因和剩余风险。

## Windows 联网规则

这台 Windows 机器上，Codex 沙箱终端可能无法联网，即使浏览器或宿主 PowerShell 可以联网。
涉及 npm、pip、git、curl、Supabase、外部 API、依赖下载时：
- 先用最小命令确认当前执行环境是否真的联网。
- 区分宿主 Windows、WSL2、Codex 沙箱三层。
- 不要把沙箱 DNS 或代理失败直接判断为项目代码问题。
- 必要时让联网命令在沙箱外执行，结束后再回到普通沙箱做本地检查。
```

注意：全局 `AGENTS.md` 不要写得过满。真正适合放进去的是稳定偏好和稳定环境事实；项目特有命令、目录结构、测试方式，应该放到项目根目录的 `AGENTS.md`。

## 第 9 步：理解 config.toml

`AGENTS.md` 主要告诉 Codex“项目是什么、规矩是什么”。`config.toml` 主要告诉 Codex“应该怎么工作”。

常见配置项包括：

- 默认模型。
- 推理力度。
- 沙箱权限。
- 审批策略。
- Web 搜索模式。
- MCP servers。
- Subagent 并发设置。
- 环境变量传递规则。

OpenAI 官方说明：用户级配置通常在 `~/.codex/config.toml`，项目级配置可以放在项目根目录的 `.codex/config.toml`。项目级配置只有在项目被信任时才会加载。

### 推荐新手理解三个权限档位

| 模式 | 含义 | 适合场景 |
| --- | --- | --- |
| `read-only` | 只能读，不能改 | 只想分析、审查、问问题 |
| `workspace-write` | 能读写工作区内文件 | 日常开发推荐 |
| `danger-full-access` | 几乎不限制 | 只在你明确信任任务和环境时使用 |

新手不要一开始就长期 Full Access。Full Access 不是“更高级”，只是限制更少，风险也更大。

更稳的策略：

- 默认用 `workspace-write`。
- 需要越权操作时单次审批。
- 自动化任务尽量不要全局开 Full Access。
- 需要固定放行某些命令时，用规则精确控制，而不是直接放开全部权限。

### 一个保守的 config.toml 参考

```toml
model = "gpt-5.5"
model_reasoning_effort = "medium"

approval_policy = "on-request"
sandbox_mode = "workspace-write"

web_search = "cached"

[shell_environment_policy]
inherit = "all"
exclude = ["*SECRET*", "*TOKEN*", "*API_KEY*", "*PASSWORD*"]

[agents]
max_threads = 6
max_depth = 1
```

注意：模型名变化很快。2026-04-23 官方 changelog 写明 GPT-5.5 已在 Codex 中可用，并在可见时推荐用于多数 Codex 任务。如果你的客户端还看不到 GPT-5.5，按官方建议继续使用可用的最新模型，例如 GPT-5.4。不要为了追模型名而破坏本地配置。

### 一份实测 config.toml 参考

笔者当前全局 `config.toml` 的关键设置大致是：

```toml
model = "gpt-5.5"
review_model = "gpt-5.5"
model_reasoning_effort = "high"
personality = "pragmatic"

sandbox_mode = "danger-full-access"
approval_policy = "never"
network_access = "enabled"

model_auto_compact_token_limit = 240000
stream_idle_timeout_ms = 600000
stream_max_retries = 2

[features]
multi_agent = true
memories = true

[shell_environment_policy]
inherit = "core"

[sandbox_workspace_write]
network_access = true

[mcp_servers.exa]
url = "https://mcp.exa.ai/mcp"

[mcp_servers.context7]
url = "https://mcp.context7.com/mcp"

[windows]
sandbox = "elevated"
```

这套配置体现了几个取舍：

- 模型默认用 `gpt-5.5`，Review 也用同一档模型，适合复杂代码和长文档任务。
- `model_reasoning_effort = "high"` 比 medium 更稳，但会更慢、更耗额度。
- `personality = "pragmatic"` 和你的全局 `AGENTS.md` 风格一致，偏实用汇报。
- `multi_agent = true`、`memories = true` 说明你已经在使用多代理和记忆能力。
- Exa、Context7 已经作为 MCP 接入，适合做资料搜索和查最新开发文档。
- `windows.sandbox = "elevated"` 表示 Windows 原生沙箱走 elevated 模式。

这里最需要谨慎的是：

```toml
sandbox_mode = "danger-full-access"
approval_policy = "never"
```

这相当于让 Codex 在本机少问确认、放开权限。对你这种已经积累了全局规则、知道如何验证和回滚的人，可以提升效率；但不建议新手照抄。新手更稳的默认值仍然是：

```toml
sandbox_mode = "workspace-write"
approval_policy = "on-request"
```

如果要用 Full Access，至少满足三个条件：

1. 只在你信任的项目目录里使用。
2. `AGENTS.md` 已写清楚不要破坏工作区、不要乱动密钥、不要做高风险操作。
3. 你会看 diff、会跑验证、知道怎么用 Git 找回改动。

### 自定义 API / 本地模型怎么配

如果你不用 ChatGPT 登录，而是想接 API 或第三方 OpenAI 兼容服务，可以在 `config.toml` 里配置 provider。笔者旧版手册里有两类常见写法。

OpenAI 兼容接口示例：

```toml
model_provider = "OpenAI"
model = "gpt-5.4"
review_model = "gpt-5.4"
model_reasoning_effort = "high"

[model_providers.OpenAI]
name = "OpenAI"
base_url = "https://你的服务商地址"
wire_api = "responses"
requires_openai_auth = true
```

`auth.json` 里只放占位说明，不要把真实 key 写进教程或聊天：

```json
{
  "OPENAI_API_KEY": "YOUR_API_KEY"
}
```

Ollama / DeepSeek 这类 provider 示例：

```toml
[model_providers.ollama]
name = "Ollama"
base_url = "http://localhost:11434/v1"
wire_api = "chat"

[model_providers.deepseek]
name = "DeepSeek"
base_url = "https://api.deepseek.com/v1"
env_key = "DEEPSEEK_API_KEY"
wire_api = "chat"
```

切换使用：

```toml
model = "qwen2.5-coder"
model_provider = "ollama"
```

这里容易踩坑：第三方服务是否支持 `responses`、`chat`、工具调用、图片、MCP、长上下文，各家不一样。配置能连上，不代表所有 Codex 能力都完整可用。遇到异常时，先换回官方 provider 做对照。

## 第 10 步：用 Local Environment 固化常用命令

Local Environment 主要解决两个问题：

1. 新 Worktree 创建后，自动执行依赖安装、构建等 setup script。
2. 把常用命令做成 Codex App 顶部按钮，比如启动、测试、构建。

例如一个 Node 项目，setup script 可以是：

```bash
npm install
npm run build
```

常用 action 可以是：

```bash
npm run dev
```

```bash
npm test
```

```bash
npm run lint
```

这不是第一天必须学的功能。建议你先手动跑通项目，再把稳定命令固化成 Local Environment。

## 第 11 步：Skills，把重复工作做成技能

Skill 是给 Codex 的专项操作手册。它解决的是“同一类任务不要每次都重新解释”。

一句话区分：

- `AGENTS.md`：项目长期规矩。
- Prompt：一次性任务。
- Skill：一类可复用任务的操作手册。

一个 Skill 是一个文件夹，核心文件是 `SKILL.md`：

```text
my-skill/
└── SKILL.md
```

复杂一点可以加：

```text
my-skill/
├── SKILL.md
├── scripts/
├── references/
└── assets/
```

`SKILL.md` 最小格式：

```md
---
name: java-review
description: 当用户要求审查 Java 后端代码、做 Code Review、检查 Spring Boot + MyBatis + Redis 项目风险时使用。不适用于前端或 Python 项目。
---

# Java 后端代码审查

按安全、事务、SQL、缓存一致性、异常处理和测试覆盖进行审查。
每个问题必须给出文件路径、具体位置、风险说明和修复建议。
跳过纯风格类建议，只关注可能导致 Bug、安全问题或维护风险的内容。
```

调用方式：

```text
$java-review 审查当前分支相对于 main 的改动。
```

Skill 的关键在 `description`。它要说清楚什么时候该用、什么时候不该用。写得太泛，Codex 会乱触发；写得太窄，又可能该触发时触发不了。

### Skill 放在哪里

官方当前文档说明，常见位置是：

- 个人级：`$HOME/.agents/skills`
- 项目级：项目里的 `.agents/skills`
- 系统内置：Codex 自带

项目级 Skill 可以提交到仓库，团队共享。个人级 Skill 适合自己长期使用。

### 什么时候该做成 Skill

如果你发现自己第三次写类似 Prompt，就应该考虑做成 Skill。

适合做 Skill 的任务：

- 固定风格改稿。
- 代码审查清单。
- 单元测试生成规范。
- PR 描述模板。
- 数据分析流程。
- 日志排查流程。
- PPT 或文档生成流程。

## 第 12 步：MCP，让 Codex 连接外部工具

MCP 的全称是 Model Context Protocol。你可以先不用关心协议细节，只要知道它解决一个问题：

> Codex 需要的信息不在当前项目目录里时，怎么让它自己去拿？

比如：

- 查最新框架文档。
- 读取 Figma 设计稿。
- 操作浏览器。
- 查看 Sentry 日志。
- 管理 GitHub Issue 和 PR。
- 连接内部知识库。

没有 MCP 时，你只能复制粘贴。接上 MCP 后，Codex 可以通过工具去读。

### 两类常见 MCP

| 类型 | 说明 | 例子 |
| --- | --- | --- |
| STDIO | 在你电脑上启动一个本地进程 | Context7、Playwright |
| Streamable HTTP | 连接远程服务地址 | Figma、远程文档服务 |

### Context7 示例

Context7 常用于查较新的开发文档。官方 MCP 文档给过类似示例：

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

也可以在 `config.toml` 里写：

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

用法：

```text
帮我查一下 Next.js 当前版本里 server actions 的推荐写法，并结合本项目代码给一个最小修改方案。
```

### Figma 示例

如果你做前端，Figma MCP 很有价值。接入后，Codex 可以读取设计稿结构、截图和元数据，再按项目已有组件实现。

典型 Prompt：

```text
读取这个 Figma 页面：[链接]
请复用本项目现有组件和设计 token，实现登录页。
完成后用浏览器检查桌面和移动端布局。
```

### Playwright / Browser 示例

前端项目里，最有价值的是让 Codex 自己打开页面验证。

```text
用浏览器打开 http://localhost:3000/login，截图检查页面是否正常。
然后点击登录按钮，确认表单校验和跳转是否符合预期。
```

不要为了“高级”一口气接十个 MCP。先接一两个真正能减少复制粘贴的工具。

## 第 13 步：Subagents，把复杂任务拆给多个 Agent

Subagent 适合并行、互相独立或需要不同视角的问题。

不要把它理解成“更强的 Codex”。它更像把一个任务拆成几个角色：

- 一个只读代码结构。
- 一个专门看安全。
- 一个专门看测试。
- 一个专门复现浏览器问题。
- 一个负责最后做最小修复。

适合用 Subagents 的场景：

- 大型代码审查。
- 多角度 Bug 排查。
- 多模块影响面分析。
- 批量文件审计。
- 需要并行探索多个方案。

不适合：

- 单文件小修改。
- 强顺序依赖任务。
- 上下文必须高度共享的问题。
- 你还没想清楚任务边界时。

### 一句 Prompt 体验 Subagents

```text
请审查当前分支相对于 main 的改动。每个维度 spawn 一个 agent：
1. 安全问题
2. 代码质量
3. 可能的 Bug
4. 测试缺口
5. 可维护性风险

等所有 agent 完成后，汇总成一份按严重程度排序的报告。
```

如果你要更长期使用，可以定义自定义 Agent。个人级 Agent 放在 `~/.codex/agents/`，项目级 Agent 放在 `.codex/agents/`。

一个只读代码探索 Agent 示例：

```toml
name = "code_explorer"
description = "只读代码探索者，用于梳理改动范围、调用链和影响面。"
model = "gpt-5.4-mini"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"

developer_instructions = """
保持探索模式，不要修改任何代码。
重点输出：
1. 改动涉及哪些文件和模块
2. 关键调用链是什么
3. 可能影响哪些测试或接口
4. 哪些地方需要主线程进一步确认
引用具体文件路径，不要泛泛而谈。
"""
```

再配一个代码审查 Agent：

```toml
name = "java_reviewer"
description = "专注 Java 后端代码审查，关注安全、事务、SQL、缓存一致性和异常处理。"
model = "gpt-5.5"
model_reasoning_effort = "high"
sandbox_mode = "read-only"

developer_instructions = """
像资深 Java 后端工程师一样审查代码。
只报告可能导致 Bug、安全问题、性能问题或维护风险的内容。
每个发现必须包含文件位置、问题说明、影响和修复建议。
跳过纯风格问题。
"""
```

启动：

```text
审查当前分支相对于 main 的改动。
让 code_explorer 先梳理影响面，让 java_reviewer 查风险。
两个 agent 并行执行，最后合并为一份报告。
```

## 第 14 步：Automations，让 Codex 定时干活

Automations 是 Codex 的定时任务。你设定时间表和 Prompt，到点后 Codex 自动运行，并把结果放到 App 里供你查看。

适合：

- 每天扫描最近提交。
- 每周整理依赖风险。
- 自动生成变更摘要。
- 定期检查日志或 CI。
- 周期性维护某个文档或数据报表。

不适合：

- 还没手动跑通过的任务。
- 需要频繁确认的任务。
- 高风险生产操作。
- 需要全电脑无限制权限的任务。

官方建议很明确：先手动跑通，再设定时。

### 自动生成每日代码摘要

```text
查看最近 24 小时 origin/main 上的提交。
生成一份中文简报：
1. 按功能模块分组，不按提交者分组
2. 每个模块说明改了什么、为什么重要
3. 跳过纯格式化和纯注释提交
4. 如果有关联 PR，附上 PR 链接
5. 最后用一句话总结今天最重要的变化
```

### 自动扫描最近提交中的明显 Bug

```text
查看最近 24 小时内我提交到 origin/main 的代码改动。
重点检查：
1. 空指针风险
2. 吞异常
3. SQL 注入风险
4. 事务注解失效
5. 缺失必要测试

如果没有发现问题，简短说明没有发现高风险问题。
如果发现问题，列出文件路径、风险说明和修复建议。
不要直接 push。
```

安全建议：

- Git 仓库里，优先让 Automation 跑在 Worktree。
- 自动化任务不要默认 Full Access。
- 第一周要人工检查输出质量。
- 如果任务会创建很多 Worktree，定期清理不需要的运行记录。

## 第 15 步：Plugins，把能力打包给团队

Skill 是单个工作流。Plugin 是可安装的能力包。

简单区分：

- Skill：先解决你自己某一类重复任务。
- Plugin：当多个 Skill、MCP 配置或 App 集成稳定后，打包给别人安装。

适合做 Plugin 的场景：

- 团队共享代码审查、单测、发布说明等 Skills。
- 多个项目都需要同一套 MCP 配置。
- 新同事加入时，希望一键安装团队 Codex 能力包。

不适合：

- 还在频繁修改的 Skill。
- 只有你自己用的个人配置。
- 没有稳定使用场景的“功能收藏夹”。

一个典型 Plugin 结构：

```text
java-backend-kit/
├── .codex-plugin/
│   └── plugin.json
├── skills/
│   ├── java-review/
│   │   └── SKILL.md
│   └── java-unit-test/
│       └── SKILL.md
└── .mcp.json
```

对于个人用户，先别急着做 Plugin。先把 1 到 3 个高频 Skill 用顺，再考虑打包。

## 第 16 步：12 个官方案例能给你的启发

OpenAI 官方 Codex use cases 里列了很多场景。结合公众号文章的整理，最值得新手理解的是：Codex 不只适合写业务代码。

你可以把它用在这些事情上：

| 场景 | 关键做法 |
| --- | --- |
| PR 自动审查 | 让 Codex 查回归、缺测试和风险改动 |
| 截图转前端页面 | 提供截图、组件规范、响应式要求，再用浏览器验证 |
| 数据分析 | 先定义问题，再让 Codex 清洗、建模、出报告 |
| ChatGPT 应用 | 先规划工具接口，再搭 MCP Server 和 Widget |
| iOS/macOS 开发 | 用命令行构建和验证，减少 GUI 依赖 |
| 浏览器游戏 | 先写 `PLAN.md`，再迭代实现和测试 |
| PPT/文档生成 | 保持内容可编辑，生成后渲染检查 |
| 难题迭代优化 | 建立评估指标，让 Codex 一轮轮改进 |
| Slack/GitHub 任务 | 从协作工具里直接派活 |
| Figma 转代码 | 读取设计结构，复用现有设计系统 |
| 读懂大代码库 | 让 Codex 解释请求流转、模块职责和风险点 |
| API 迁移 | 先盘点现状，再按官方文档做最小迁移 |

这背后的共同点是：先定义工作流，再让 Codex 执行。不要只丢一句“帮我搞一下”。

## 第 17 步：第一周推荐练习路线

如果你是新手，不建议一天内把所有高级功能都开起来。按这个顺序来更稳。

### 第一天：跑通基础

目标：

- 安装并登录 Codex。
- 创建一个 Project。
- 新建一个 Thread。
- 让 Codex 只读分析项目结构。

练习 Prompt：

```text
先不要修改文件。请阅读当前项目，告诉我它的目录结构、启动方式、测试方式，以及最应该先读的 5 个文件。
```

### 第二天：完成一个小修改

目标：

- 让 Codex 做一个低风险修改。
- 看 diff。
- 运行最小验证。

练习 Prompt：

```text
在不改变功能的前提下，帮我改进 README 的快速开始部分。
要求：保留现有结构，只补充缺失步骤。修改后检查 Markdown 格式是否正常。
```

### 第三天：写 AGENTS.md

目标：

- 把项目规则固化。
- 新开线程验证 Codex 是否读到规则。

练习 Prompt：

```text
请根据当前项目生成一份简洁的 AGENTS.md。
重点写：项目结构、常用命令、不要修改的文件、完成标准。
不要写空泛口号。
```

### 第四天：尝试 Worktree

目标：

- 在 Worktree 里做一次试验性修改。
- 确认 Local 文件没有被直接改动。

练习 Prompt：

```text
在 Worktree 中尝试重构这个小模块。
先说明方案，再修改。完成后告诉我和 Local 工作区的关系，以及如何合并回来。
```

### 第五天：做一个 Skill

目标：

- 把重复任务做成 Skill。
- 显式用 `$skill-name` 调用一次。

练习 Prompt：

```text
$skill-creator
我想创建一个用于中文技术教程润色的 Skill，要求能检查结构、术语、可操作性和 AI 味。
```

### 第六天：接一个 MCP

目标：

- 只接一个真正有用的 MCP。
- 用它减少复制粘贴。

优先级：

- 查文档：Context7。
- 前端验证：Playwright / Browser。
- 设计稿：Figma。
- 代码协作：GitHub。

### 第七天：做一次完整任务闭环

目标：

- 让 Codex 规划、修改、验证、复盘。

练习 Prompt：

```text
请完成一个小功能改动。
流程要求：
1. 先读相关文件并给出计划
2. 我确认后再实现
3. 实现后运行最小验证
4. 使用 /review 或等价方式检查改动风险
5. 最终汇报改了什么、验证了什么、还有什么风险
```

## 常见坑与处理方式

### 1. Codex 改得太散

原因通常是项目范围太大或 Prompt 太模糊。

处理：

- 缩小 Project 或明确目录范围。
- 在 Prompt 里写“只修改这些文件/目录”。
- 在 `AGENTS.md` 里写“不要做无关重构”。

### 2. Codex 总是问权限

原因通常是沙箱和审批策略较严格。

处理：

- 日常使用可考虑 `workspace-write + on-request`。
- 不要因为烦就直接长期 Full Access。
- 对固定命令用规则精确放行。

### 3. Worktree 创建失败

常见原因：

- 项目不是 Git 仓库。
- Git 刚初始化但还没有提交。
- 当前分支或工作区状态不适合派生。

处理：

- 先确认 `git status`。
- 至少做一次初始提交。
- 再创建 Worktree。

### 4. 它说验证了，但其实没跑

不要只看总结。检查终端输出或让它明确列出命令和结果。

可以要求：

```text
最终汇报时只列出你实际运行过的验证命令和结果。没有运行的不要写成已验证。
```

### 5. Windows 下 npm、pip、git、curl 联网失败

这不一定是项目问题。Codex 的沙箱终端、Windows PowerShell、WSL2、浏览器可能走的是不同网络路径。

笔者在 Windows 环境里多次验证过类似现象：宿主 Windows PowerShell 能访问 npm registry，但 Codex 沙箱里可能 DNS 失败、npm install 失败，或者访问不到 `127.0.0.1:7897` 本地代理。GitHub 上也有同类问题反馈，例如 `openai/codex#14221` 提到 Codex-spawned PowerShell 中 `curl` DNS 失败但普通终端正常，`openai/codex#15447` 讨论了 Windows 系统代理未传递到 WSL2 Codex 进程，`openai/codex#17137` 则记录了 Windows 沙箱和 localhost 代理相关的不稳定行为。所以排查时不要只说“网络问题”，要分层。

处理顺序：

1. 先区分是宿主机不能联网，还是 Codex 沙箱不能联网。
2. 需要联网安装依赖时，不要在沙箱里反复失败。
3. 使用最小探测命令，比如 `npm ping`、`curl -I`、`git ls-remote`。
4. 确认网络通了，再安装依赖或跑构建。

建议按这三层排查：

| 层级 | 要确认什么 | 常用探测 |
| --- | --- | --- |
| 宿主 Windows | 普通 PowerShell 是否能联网 | `npm ping`、`curl.exe -I https://registry.npmjs.org` |
| Codex 沙箱 | 当前 Codex 执行环境是否被禁网或 DNS 失败 | 让 Codex 在同一线程里跑 `npm ping`、`nslookup registry.npmjs.org` |
| WSL2 | 是否误用了 Windows npm shim，或 Linux nvm 没加载 | `which node npm npx`、`node -p "process.execPath"` |

如果你让 Codex 代执行联网命令，可以直接在 Prompt 里写清楚：

```text
这个任务需要联网安装依赖。请先做最小网络验证：
1. 在当前 Codex 环境里运行 npm ping 或 curl -I
2. 如果失败，不要反复重试
3. 请改用沙箱外的 Windows PowerShell 执行联网命令
4. 联网命令成功后，再回到普通沙箱做本地测试
5. 不要把沙箱网络失败判断为项目代码问题
```

如果本地代理是 `127.0.0.1:7897`，可以检查：

```powershell
git config --show-origin --get http.proxy
git config --show-origin --get https.proxy
netsh winhttp show proxy
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" /v ProxyServer
```

如果出现这些现象，要按“环境问题”处理，而不是继续改项目代码：

- `getaddrinfo EAI_FAIL registry.npmjs.org`
- `getaddrinfo() thread failed to start`
- `WinError 10106`
- `connect UNKNOWN 127.0.0.1:7897 - Local`
- 浏览器能联网，但 Codex 里的 `npm`、`curl`、`git` 不能联网

一个实用原则：联网命令失败时，先证明“当前执行环境真的能联网”，再继续安装依赖、构建或测试。

### 6. Codex 远程自动压缩断连

常见报错类似：

```text
Error running remote compact task: stream disconnected before completion
```

这个问题不要只归因于一个原因。笔者此前的现场证据显示，它可能同时涉及三层：

1. OpenAI / Codex 远程 compact 服务本身的短时波动。
2. 本机代理、DNS 或 `chatgpt.com` 连接异常。
3. 当前线程太长，压缩时上下文体积过大，流式传输更容易断。

先做快速止血：

- 让 Codex 生成当前任务的交接提示词。
- 新开一个线程继续，不要一直在同一个超长线程里硬重试。
- 大任务拆成多个 Thread，不要把审计、修复、部署、写文档全塞在一条线里。
- 重要阶段手动写 checkpoint，比如“当前已完成、未完成、下一步、关键文件、验证结果”。

笔者当前 `config.toml` 使用了这一组实测经验配置：

```toml
model_auto_compact_token_limit = 240000
stream_idle_timeout_ms = 600000
stream_max_retries = 2
```

它的思路是：不要等上下文接近极限才压缩，提前触发压缩；同时给流式传输更长等待时间，并允许有限重试。这里要注意两点：

- 这些属于实测经验配置，不要当成所有版本都稳定支持的公开通用方案。
- 如果某个 Codex 版本不识别其中某个字段，应该删掉或按官方最新配置调整。

如果你想判断是不是本机网络问题，可以用最小命令检查：

```powershell
nslookup chatgpt.com
curl.exe -I https://chatgpt.com/backend-api/codex/responses/compact
```

如果这里也失败，再查代理：

```powershell
git config --show-origin --get http.proxy
git config --show-origin --get https.proxy
netsh winhttp show proxy
```

处理建议：

| 情况 | 更合适的做法 |
| --- | --- |
| OpenAI status 有 Codex 事故 | 保存当前进度，稍后重试或新开线程 |
| `chatgpt.com` DNS / curl 失败 | 先修本机代理、DNS、VPN，再继续 |
| 只有超长线程失败 | 降低单线程长度，提前交接，新线程继续 |
| 每次都在压缩时断 | 降低 compact 阈值，减少 xhigh 超长任务堆积 |

最简单、最可靠的办法仍然是：遇到连续 compact 失败时，先生成交接提示词，新开会话继续。

### 7. WSL2 下 npm 路径选错

如果你在 WSL2 里用 Codex，另一个常见问题是误选 Windows 侧 npm。例如路径变成：

```text
/mnt/d/app/nodejs/npm
```

然后报：

```text
WSL 1 is not supported. Please upgrade to WSL 2 or above.
Could not determine Node.js install directory
```

这通常不是“机器没有 Node”，而是 WSL2 里选错了 Windows npm shim。先检查：

```bash
which node npm npx
node -p "process.execPath"
npm config get cache
```

笔者此前的稳定处理思路是：WSL2 走 Linux 自己的 nvm Node，例如：

```bash
/home/user/.nvm/versions/node/v22.22.2/bin/node
/home/user/.nvm/versions/node/v22.22.2/bin/npm
/home/user/.nvm/versions/node/v22.22.2/bin/npx
```

临时修正可以先在当前 shell 里显式放前面：

```bash
export PATH="/home/user/.nvm/versions/node/v22.22.2/bin:$PATH"
```

如果要长期修，优先做 WSL2 内部的持久化修复，不要影响 Windows PowerShell 原本的 `D:\app\nodejs`。修完后重新跑：

```bash
which node npm npx
node -v
npm -v
npm ping
```

### 8. Skill 没触发

常见原因：

- `description` 写得太模糊。
- Skill 放错目录。
- 新增 Skill 后客户端还没刷新。

处理：

- 先用 `$skill-name` 显式调用。
- 检查 `SKILL.md` 的 frontmatter。
- 重启 Codex。

### 9. 自动化任务输出不稳定

原因通常是还没手动跑顺。

处理：

- 先在普通线程里手动跑 2 到 3 次。
- 把成功 Prompt 固化成 Skill。
- 再把 Skill 放进 Automation。
- 第一周人工检查输出。

## 一套推荐的日常工作流

个人使用可以按这套流程：

1. 新任务先开新 Thread。
2. 复杂任务先 Plan，不要马上改。
3. 小改用 Local，大改或试验用 Worktree。
4. Prompt 写清目标、范围、约束、完成标准。
5. 改完看 diff。
6. 跑最小验证。
7. 需要时用 Review 再查一遍风险。
8. 同类任务重复 3 次，做成 Skill。
9. 稳定重复任务，再做 Automation。

团队使用可以再加三件事：

1. 仓库根目录维护 `AGENTS.md`。
2. 团队共享 Skill 放进 `.agents/skills`。
3. 稳定能力打包成 Plugin，降低新成员配置成本。

## 最小可用模板包

### 任务 Prompt 模板

```text
请完成以下任务：

目标：
[写清楚要完成什么]

范围：
[限定目录、文件或模块]

约束：
1. 不做无关重构
2. 不新增依赖，除非先说明原因
3. 遵循现有代码风格

完成标准：
1. [行为或产物要求]
2. [需要通过的检查]

执行方式：
先阅读相关文件并说明计划，再修改。
修改后运行最小必要验证。
最终说明：改了什么、验证了什么、还有什么风险。
```

### 只读分析模板

```text
先不要修改任何文件。
请阅读当前项目，回答：
1. 项目主要做什么
2. 核心目录和入口文件是什么
3. 常用启动、测试、构建命令是什么
4. 如果我要改 [具体功能]，应该先看哪些文件
5. 有哪些明显风险或不确定点
```

### Bug 排查模板

```text
请排查这个问题：[描述问题]

要求：
1. 先复现或定位触发路径
2. 找到最可能的根因
3. 不要一上来大改
4. 给出最小修复方案
5. 修改后运行能证明问题解决的最小验证

最终输出：
- 根因
- 修改文件
- 验证命令和结果
- 仍然没验证的地方
```

### 代码审查模板

```text
请审查当前分支相对于 main 的改动。

重点关注：
1. 行为回归
2. 安全风险
3. 缺失测试
4. 错误处理
5. 边界条件

要求：
- 只报告真实风险，不要输出泛泛建议
- 每个问题给出文件路径和具体说明
- 按严重程度排序
- 如果没有发现问题，明确说明剩余风险
```

## 来源与校正说明

本文参考并重组了以下材料，没有复刻原文结构或大段原文：

1. 《Codex App Windows 版入门教程：手把手教你从不会用到真正用顺》  
   <https://mp.weixin.qq.com/s/1nuf9VNSF1HxHhfISuB37g>
2. 《Codex App Windows 版高级教程（一）：worktree教你从用顺到真正高手》  
   <https://mp.weixin.qq.com/s/s0iiw8qE7-EZMKjufW4XNg>
3. 《Codex App Windows 版高级教程（二）：Subagent 手把手教你从用顺到真正高手》  
   <https://mp.weixin.qq.com/s/hKhWF-g_Q6JB4Zgsspuv6Q>
4. 《Codex App windows版高级教程（三）：AGENTS.md + config.toml，把 Codex 调成你想要的样子》  
   <https://mp.weixin.qq.com/s/ZCBwX-UAuIbRWbYuAQ5szw>
5. 《Codex App Windows 版高级教程（四）：Skills + MCP，教 Codex 学本事、连外脑》  
   <https://mp.weixin.qq.com/s/CSnTGSYzl_41UBB-ykHkBg>
6. 《Codex App Windows 版高级教程（五）：Automations + Plugins，让 Codex 自己干活、把能力打包分享》  
   <https://mp.weixin.qq.com/s/1cwoVZWpov54sQT9shRhpA?scene=1&click_id=1>
7. 《全网最详细的Codex入门教程，手把手教你玩转Vibe Coding。》  
   <https://mp.weixin.qq.com/s/SJco8KqYUJT3DjMdrMu_Iw>
8. 《OpenAI 发布 12 个 Codex 官方案例库，手把手教到你会为止》  
   <https://mp.weixin.qq.com/s/iRtNOAwH0RXSHIRaGsK0tA>
9. 《Codex使用手册（上）》：`D:\a311\科普与综合\Codex使用手册（上）.md`
10. 笔者全局指令配置：`C:\Users\user\.codex\AGENTS.md`
11. 笔者全局 Codex 配置：`C:\Users\user\.codex\config.toml`
12. 笔者命令审批规则：`C:\Users\user\.codex\rules\default.rules`
13. Windows / WSL2 / compact 断连相关的历史排障记录
14. GitHub Issue：Codex-spawned PowerShell 中 `curl` DNS 失败但普通终端正常  
    <https://github.com/openai/codex/issues/14221>
15. GitHub Issue：Windows 系统代理没有传递到 WSL2 Codex 进程  
    <https://github.com/openai/codex/issues/15447>
16. GitHub Issue：Windows 沙箱与 localhost 代理不稳定  
    <https://github.com/openai/codex/issues/17137>
17. GitHub Issue：Windows Codex App 沙箱中 npm/DNS 失败的复现报告  
    <https://github.com/openai/codex/issues/18675>
18. GitHub Issue 搜索：Codex remote compact 断连相关问题  
    <https://github.com/openai/codex/issues?q=%22stream+disconnected+before+completion%22+compact>

同时按以下 OpenAI 官方资料做了当前性校正：

- Codex 总览：<https://developers.openai.com/codex/>
- Codex App 功能：<https://developers.openai.com/codex/app/features>
- Codex 最佳实践：<https://developers.openai.com/codex/learn/best-practices>
- AGENTS.md 官方指南：<https://developers.openai.com/codex/guides/agents-md/>
- config.toml 配置基础：<https://developers.openai.com/codex/local-config>
- Skills 官方说明：<https://developers.openai.com/codex/skills/>
- MCP 官方说明：<https://developers.openai.com/codex/mcp>
- Subagents 官方说明：<https://developers.openai.com/codex/subagents>
- Automations 官方说明：<https://developers.openai.com/codex/app/automations>
- Sandbox 官方说明：<https://developers.openai.com/codex/concepts/sandboxing/>
- Codex Help Center：<https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan>
- Codex Changelog：<https://help.openai.com/en/articles/11428266-codex-changelog>

时间敏感内容以 2026-04-26 能读取到的官方资料为准。模型名称、客户端入口、Plugins 灰度状态、Windows 版体验和具体 UI 文案都可能继续变化，实际操作时应以读者当前客户端显示为准。
