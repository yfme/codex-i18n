<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">轻量级终端编码助手</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | **简体中文** | [Español](./README.es.md) | [Português](./README.pt.md) | [Русский](./README.ru.md) | [Français](./README.fr.md) | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [繁體中文](./README.zh-TW.md) | [Bahasa Indonesia](./README.id.md) | [Italiano](./README.it.md)

![Codex 演示 GIF：使用 codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>目录</strong></summary>

- [实验性技术声明](#实验性技术声明)
- [快速开始](#快速开始)
- [为什么选择 Codex？](#为什么选择-codex)
- [安全模型与权限](#安全模型与权限)
  - [平台沙箱详情](#平台沙箱详情)
- [系统要求](#系统要求)
- [CLI 参考](#cli-参考)
- [记忆与项目文档](#记忆与项目文档)
- [非交互式/CI 模式](#非交互式ci-模式)
- [使用示例](#使用示例)
- [安装](#安装)
- [配置](#配置)
- [常见问题](#常见问题)
- [资助机会](#资助机会)
- [贡献指南](#贡献指南)
  - [开发流程](#开发流程)
  - [编写高影响力代码变更](#编写高影响力代码变更)
  - [提交 Pull Request](#提交-pull-request)
  - [审查流程](#审查流程)
  - [社区价值观](#社区价值观)
  - [获取帮助](#获取帮助)
  - [贡献者许可协议 (CLA)](#贡献者许可协议-cla)
    - [快速修复](#快速修复)
  - [发布 `codex`](#发布-codex)
- [安全与负责任 AI](#安全与负责任-ai)
- [许可证](#许可证)

</details>

---

## 实验性技术声明

Codex CLI 是一个正在积极开发的实验性项目。它尚未稳定，可能包含错误、不完整的功能，或经历重大变更。我们正在与社区一起公开构建它，欢迎：

- 错误报告
- 功能请求
- Pull Request
- 积极反馈

通过提交问题或 PR 来帮助我们改进（请参阅下面的贡献部分）！

## 快速开始

全局安装：

```shell
npm install -g @openai/codex
```

接下来，设置您的 OpenAI API 密钥作为环境变量：

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **注意：** 此命令仅在当前终端会话中设置密钥。要使其永久生效，请将 `export` 行添加到您的 shell 配置文件（例如 `~/.zshrc`）。

以交互方式运行：

```shell
codex
```

或者，使用提示作为输入运行（可选择在 `Full Auto` 模式下）：

```shell
codex "explain this codebase to me"
```

```shell
codex --approval-mode full-auto "create the fanciest todo-list app"
```

就是这样 - Codex 将创建文件，在沙箱中运行，安装任何缺失的依赖项，并显示实时结果。批准更改后，它们将被提交到您的工作目录。

---

## 为什么选择 Codex？

Codex CLI 专为已经**生活在终端中**的开发者打造，他们希望获得 ChatGPT 级别的推理能力**加上**实际运行代码、操作文件和迭代的能力 - 所有这些都在版本控制下进行。简而言之，这是一个能够理解和执行您的代码库的_聊天驱动开发_工具。

- **零配置** - 只需提供 OpenAI API 密钥即可使用！
- **全自动批准，同时保持安全** - 通过禁用网络和目录沙箱实现
- **多模态** - 可以传入截图或图表来实现功能 ✨

而且它是**完全开源的**，您可以查看并参与其开发！

---

## 安全模型与权限

Codex 让您通过 `--approval-mode` 标志（或交互式引导提示）决定_代理获得多少自主权_和自动批准策略：

| 模式                      | 代理无需询问即可执行的操作            | 仍需要批准的操作                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **建议** <br>(默认) | • 读取仓库中的任何文件                     | • **所有**文件写入/补丁 <br>• **所有** shell/Bash 命令 |
| **自动编辑**             | • 读取**和**应用补丁写入文件      | • **所有** shell/Bash 命令                                   |
| **全自动**             | • 读取/写入文件 <br>• 执行 shell 命令 | –                                                               |

在**全自动**模式下，每个命令都在**禁用网络**的情况下运行，并限制在当前工作目录（加上临时文件）中，以实现深度防御。如果目录_未_被 Git 跟踪，Codex 还会在**自动编辑**或**全自动**模式下显示警告/确认，这样您始终有一个安全网。

即将推出：一旦我们对额外的安全措施有信心，您将能够将特定命令列入白名单以启用网络自动执行。

### 平台沙箱详情

Codex 使用的加固机制取决于您的操作系统：

- **macOS 12+** - 命令使用 **Apple Seatbelt** (`sandbox-exec`) 包装。

  - 除了少量可写根目录（`$PWD`、`$TMPDIR`、`~/.codex` 等）外，所有内容都放在只读监狱中。
  - 默认情况下，出站网络被_完全阻止_ - 即使子进程尝试 `curl` 到某处也会失败。

- **Linux** - 我们建议使用 Docker 进行沙箱化，Codex 在**最小容器镜像**中启动自己，并以相同的路径挂载您的仓库_读写_。自定义 `iptables`/`ipset` 防火墙脚本拒绝除 OpenAI API 之外的所有出口流量。这样您就可以获得确定性的、可重现的运行，而无需主机上的 root 权限。您可以在 [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh) 中阅读更多内容

这两种方法对日常使用都是_透明的_ - 您仍然从仓库根目录运行 `codex` 并按常规批准/拒绝步骤。

---

## 系统要求

| 要求                 | 详情                                                         |
| --------------------------- | --------------------------------------------------------------- |
| 操作系统           | macOS 12+、Ubuntu 20.04+/Debian 10+，或通过 WSL2 的 Windows 11 |
| Node.js                     | **22 或更新版本**（推荐 LTS）                               |
| Git（可选，推荐） | 2.23+ 用于内置 PR 助手                                   |
| 内存                         | 最低 4GB（推荐 8GB）                                 |

> 永远不要运行 `sudo npm install -g`；请修复 npm 权限。

---

## CLI 参考

| 命令                              | 用途                             | 示例                              |
| -------------------------------- | -------------------------------- | -------------------------------- |
| `codex`                          | 交互式 REPL                    | `codex`                          |
| `codex "..."`                    | 交互式 REPL 的初始提示 | `codex "fix lint errors"`        |
| `codex -q "..."`                 | 非交互式"静默模式"        | `codex -q --json "explain utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | 打印 shell 补全脚本       | `codex completion bash`          |

关键标志：`--model/-m`、`--approval-mode/-a` 和 `--quiet/-q`。

---

## 记忆与项目文档

Codex 按以下顺序合并 Markdown 指令：

1. `~/.codex/instructions.md` - 个人全局指导
2. 仓库根目录的 `codex.md` - 共享项目说明
3. 当前工作目录的 `codex.md` - 子包特定说明

使用 `--no-project-doc` 或 `CODEX_DISABLE_PROJECT_DOC=1` 禁用。

---

## 非交互式/CI 模式

在流水线中无头运行 Codex。GitHub Action 步骤示例：

```yaml
- name: 通过 Codex 更新变更日志
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "update CHANGELOG for next release"
```

设置 `CODEX_QUIET_MODE=1` 以静默交互式 UI 噪音。

---

## 使用示例

以下是几个可以复制粘贴的小示例。将引号中的文本替换为您自己的任务。有关更多提示和使用模式，请参阅[提示指南](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md)。

| ✨  | 您输入的内容                                                                   | 发生的情况                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "将 Dashboard 组件重构为 React Hooks"`                       | Codex 重写类组件，运行 `npm test`，并显示差异。   |
| 2   | `codex "为添加用户表生成 SQL 迁移"`                      | 推断您的 ORM，创建迁移文件，并在沙箱化数据库中运行它们。 |
| 3   | `codex "为 utils/date.ts 编写单元测试"`                                    | 生成测试，执行它们，并迭代直到通过。              |
| 4   | `codex "使用 git mv 批量重命名 *.jpeg → *.jpg"`                                | 安全地重命名文件并更新导入/使用。                           |
| 5   | `codex "解释这个正则表达式的含义：^(?=.*[A-Z]).{8,}$"`                      | 输出逐步的人类解释。                                  |
| 6   | `codex "仔细审查此仓库，并提出 3 个高影响力且范围明确的 PR"` | 在当前代码库中建议有影响力的 PR。                            |
| 7   | `codex "查找漏洞并创建安全审查报告"`          | 查找并解释安全漏洞。                                          |

---

## 安装

<details open>
<summary><strong>从 npm 安装（推荐）</strong></summary>

```bash
npm install -g @openai/codex
# 或
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>从源码构建</strong></summary>

```bash
# 克隆仓库并导航到 CLI 包
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# 安装依赖并构建
npm install
npm run build

# 获取使用说明和选项
node ./dist/cli.js --help

# 直接运行本地构建的 CLI
node ./dist/cli.js

# 或全局链接命令以方便使用
npm link
```

</details>

---

## 配置

Codex 在 **`~/.codex/`** 中查找配置文件。

```yaml
# ~/.codex/config.yaml
model: o4-mini # 默认模型
fullAutoErrorMode: ask-user # 或 ignore-and-continue
```

您还可以定义自定义指令：

```yaml
# ~/.codex/instructions.md
- 始终使用表情符号回复
- 仅在我明确提到时才使用 git 命令
```

---

## 常见问题

<details>
<summary>OpenAI 在 2021 年发布了一个名为 Codex 的模型 - 这与此相关吗？</summary>

2021 年，OpenAI 发布了 Codex，这是一个设计用于从自然语言提示生成代码的 AI 系统。该原始 Codex 模型已于 2023 年 3 月弃用，与 CLI 工具是分开的。

</details>

<details>
<summary>如何阻止 Codex 修改我的仓库？</summary>

Codex 始终在**沙箱中首先运行**。如果建议的命令或文件更改看起来可疑，您只需在提示时回答 **n**，工作树就不会发生任何变化。

</details>

<details>
<summary>它能在 Windows 上运行吗？</summary>

不能直接运行。它需要 [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) - Codex 已在 macOS 和 Linux 上使用 Node ≥ 22 进行了测试。

</details>

<details>
<summary>支持哪些模型？</summary>

任何可通过 [Responses API](https://platform.openai.com/docs/api-reference/responses) 获得的模型。默认为 `o4-mini`，但可以通过传递 `--model gpt-4o` 或在配置文件中设置 `model: gpt-4o` 来覆盖。

</details>

---

## 资助机会

我们很高兴推出一个**100 万美元计划**，支持使用 Codex CLI 和其他 OpenAI 模型的开源项目。

- 资助以**25,000 美元** API 信用额度为单位发放。
- 申请**滚动审核**。

**感兴趣？[在此申请](https://openai.com/form/codex-open-source-fund/)。**

---

## 贡献指南

这个项目正在积极开发中，代码可能会发生重大变化。一旦完成，我们将更新此消息！

更广泛地说，我们欢迎贡献 - 无论您是第一次提交 Pull Request 还是经验丰富的维护者。同时，我们关心可靠性和长期可维护性，因此合并代码的门槛**很高**。以下指南详细说明了"高质量"在实践中意味着什么，应该使整个过程透明且友好。

### 开发流程

- 从 `main` 创建_主题分支_ - 例如 `feat/interactive-prompt`。
- 保持更改集中。多个不相关的修复应作为单独的 PR 提交。
- 在开发过程中使用 `npm run test:watch` 获得超快速反馈。
- 我们使用 **Vitest** 进行单元测试，**ESLint** + **Prettier** 进行样式检查，**TypeScript** 进行类型检查。
- 在推送之前，运行完整的测试/类型/检查套件：

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- 如果您**尚未**签署贡献者许可协议 (CLA)，请添加包含以下确切文本的 PR 评论：

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  CLA 助手机器人将在所有作者签署后将 PR 状态变为绿色。

```bash
# 监视模式（更改时重新运行测试）
npm run test:watch

# 类型检查而不生成文件
npm run typecheck

# 自动修复 lint + prettier 问题
npm run lint:fix
npm run format:fix
```

### 编写高影响力代码变更

1. **从问题开始。** 打开新问题或评论现有讨论，以便我们在编写代码之前就解决方案达成一致。
2. **添加或更新测试。** 每个新功能或错误修复都应附带测试覆盖，在您的更改之前失败，之后通过。不需要 100% 的覆盖率，但目标是进行有意义的断言。
3. **记录行为。** 如果您的更改影响用户界面行为，请更新 README、内联帮助（`codex --help`）或相关示例项目。
4. **保持提交原子性。** 每个提交都应该编译，测试应该通过。这使得审查和潜在回滚更容易。

### 提交 Pull Request

- 填写 PR 模板（或包含类似信息）- **是什么？为什么？怎么做？**
- 在本地运行**所有**检查（`npm test && npm run lint && npm run typecheck`）。可以在本地捕获的 CI 失败会减慢流程。
- 确保您的分支与 `main` 保持最新，并且您已解决合并冲突。
- 只有当您认为 PR 处于可合并状态时，才将其标记为**准备审查**。

### 审查流程

1. 将指定一名维护者作为主要审查者。
2. 我们可能会要求更改 - 请不要对此感到不快。我们重视这项工作，我们只是也重视一致性和长期可维护性。
3. 当达成共识认为 PR 达到标准时，维护者将进行压缩合并。

### 社区价值观

- **友善和包容。** 尊重他人；我们遵循[贡献者公约](https://www.contributor-covenant.org/。
- **假设善意。** 书面交流很困难 - 倾向于慷慨。
- **教学与学习。** 如果您发现令人困惑的内容，请打开问题或 PR 进行改进。

### 获取帮助

如果您在设置项目时遇到问题，想要获得想法反馈，或者只是想打个招呼 - 请打开讨论或跳转到相关问题。我们很乐意提供帮助。

让我们一起使 Codex CLI 成为一个令人难以置信的工具。**祝您编码愉快！** :rocket:

### 贡献者许可协议 (CLA)

所有贡献者**必须**接受 CLA。这个过程很简单：

1. 打开您的 Pull Request。
2. 粘贴以下评论（如果您之前已签署，则回复 `recheck`）：

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. CLA 助手机器人会在仓库中记录您的签名，并将状态检查标记为通过。

不需要特殊的 Git 命令、电子邮件附件或提交页脚。

#### 快速修复

| 场景          | 命令                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| 修改最后一次提交 | `git commit --amend -s --no-edit && git push -f`                                          |
| 仅 GitHub UI    | 在 PR 中编辑提交消息 → 添加<br>`Signed-off-by: Your Name <email@example.com>` |

**DCO 检查**会阻止合并，直到 PR 中的每个提交都带有页脚（使用压缩时只有一个）。

### 发布 `codex`

要发布 CLI 的新版本，请运行 `codex-cli/package.json` 中定义的发布脚本：

1. 打开 `codex-cli` 目录
2. 确保您在类似 `git checkout -b bump-version` 的分支上
3. 将版本和 `CLI_VERSION` 提升到当前日期时间：`npm run release:version`
4. 提交版本提升（带有 DCO 签名）：
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. 复制 README，构建并发布到 npm：`npm run release`
6. 推送到分支：`git push origin HEAD`

---

## 安全与负责任 AI

您是否发现了漏洞或对模型输出有疑虑？请发送电子邮件至 **security@openai.com**，我们将及时回复。

---

## 许可证

本仓库根据 [Apache-2.0 许可证](LICENSE) 授权。
