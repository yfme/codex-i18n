<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">在終端機中運行的輕量級程式設計代理</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | [Español](./README.es.md) | [Português](./README.pt.md) | [Русский](./README.ru.md) | [Français](./README.fr.md) | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | **繁體中文** | [Bahasa Indonesia](./README.id.md) | [Italiano](./README.it.md)

![Codex demo GIF 使用: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>目錄</strong></summary>

- [實驗性技術免責聲明](#實驗性技術免責聲明)
- [快速入門](#快速入門)
- [為什麼選擇 Codex？](#為什麼選擇-codex)
- [安全模型與權限](#安全模型與權限)
  - [平台沙箱化詳情](#平台沙箱化詳情)
- [系統需求](#系統需求)
- [CLI 參考](#cli-參考)
- [記憶與項目文件](#記憶與項目文件)
- [非互動 / CI 模式](#非互動--ci-模式)
- [範例](#範例)
- [安裝](#安裝)
- [設定](#設定)
- [常見問題](#常見問題)
- [資金機會](#資金機會)
- [貢獻](#貢獻)
  - [開發工作流程](#開發工作流程)
  - [撰寫高影響力程式碼變更](#撰寫高影響力程式碼變更)
  - [開啟拉取請求](#開啟拉取請求)
  - [審查流程](#審查流程)
  - [社群價值觀](#社群價值觀)
  - [尋求幫助](#尋求幫助)
  - [貢獻者許可協議 (CLA)](#貢獻者許可協議-cla)
    - [快速修復](#快速修復)
  - [發佈 `codex`](#發佈-codex)
- [安全與負責任 AI](#安全與負責任-ai)
- [許可證](#許可證)

</details>

---

## 實驗性技術免責聲明

Codex CLI 是一個正在積極開發中的實驗性專案。它尚未穩定，可能包含錯誤、不完整的功能，或會發生重大變更。我們正在與社群一起公開開發，並歡迎：

- 錯誤報告
- 功能請求
- 拉取請求
- 正面回饋

請透過提交問題或拉取請求來幫助我們改進（請參閱下面的貢獻說明）！

## 快速入門

全域安裝：

```shell
npm install -g @openai/codex
```

接下來，設定您的 OpenAI API 金鑰為環境變數：

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **注意：** 此命令僅在當前的終端機工作階段中設定金鑰。要使其永久生效，請將 `export` 行添加到您的 shell 設定檔中（例如 `~/.zshrc`）。

以互動方式執行：

```shell
codex
```

或者，使用提示作為輸入執行（可選使用 `Full Auto` 模式）：

```shell
codex "explain this codebase to me"
```

```shell
codex --approval-mode full-auto "create the fanciest todo-list app"
```

就是這樣 – Codex 將在沙箱中建立檔案、執行它、安裝任何缺少的依賴項，並向您顯示即時結果。核准變更後，它們將被提交到您的工作目錄。

---

## 為什麼選擇 Codex？

Codex CLI 專為那些已經**習慣使用終端機**的開發者設計，他們希望擁有 ChatGPT 級別的推理能力**加上**實際執行程式碼、操作檔案和迭代的能力 – 所有這些都在版本控制之下。簡而言之，這是一個能夠理解並執行您的程式碼庫的_對話驅動開發_工具。

- **零設定** — 只需提供您的 OpenAI API 金鑰，它就能立即運作！
- **完全自動核准，同時保持安全** — 透過網路禁用和目錄沙箱化實現
- **多模態** — 可以傳入螢幕截圖或圖表來實現功能 ✨

而且它是**完全開源**的，所以您可以查看並參與它的發展！

---

## 安全模型與權限

Codex 讓您可以透過 `--approval-mode` 標誌（或互動式入門提示）決定代理程式獲得的_自主權程度_和自動核准策略：

| 模式                      | 代理程式可以不經詢問執行的操作            | 仍需要核准的操作                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **建議** <br>(預設) | • 讀取儲存庫中的任何檔案                     | • **所有**檔案寫入/修補 <br>• **所有** shell/Bash 命令 |
| **自動編輯**             | • 讀取**並**套用修補寫入檔案      | • **所有** shell/Bash 命令                                   |
| **完全自動**             | • 讀取/寫入檔案 <br>• 執行 shell 命令 | –                                                               |

在**完全自動**模式下，每個命令都在**網路禁用**的情況下執行，並限制在當前工作目錄（加上暫存檔案）中，以實現深度防禦。如果目錄_未_被 Git 追蹤，Codex 會在**自動編輯**或**完全自動**模式下顯示警告/確認，因此您始終有安全網。

即將推出：一旦我們對額外的安全措施有信心，您將能夠為特定命令啟用網路自動執行。

### 平台沙箱化詳情

Codex 使用的強化機制取決於您的作業系統：

- **macOS 12+** – 命令使用 **Apple Seatbelt** (`sandbox-exec`) 包裝。

  - 除了少數可寫入的根目錄（`$PWD`、`$TMPDIR`、`~/.codex` 等）外，所有內容都放在唯讀的監獄中。
  - 預設情況下，出站網路被_完全阻止_ – 即使子進程嘗試 `curl` 到某處也會失敗。

- **Linux** – 我們建議使用 Docker 進行沙箱化，Codex 在**最小容器映像**中啟動自己，並以相同的路徑掛載您的儲存庫_讀寫_。自定義的 `iptables`/`ipset` 防火牆腳本拒絕除 OpenAI API 之外的所有出口。這使您能夠在不需主機 root 權限的情況下獲得確定性、可重現的執行。您可以在 [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh) 中閱讀更多資訊。

這兩種方法對日常使用都是_透明的_ – 您仍然可以從儲存庫根目錄執行 `codex` 並像往常一樣核准/拒絕步驟。

---

## 系統需求

| 需求                 | 詳情                                                         |
| --------------------------- | --------------------------------------------------------------- |
| 作業系統           | macOS 12+, Ubuntu 20.04+/Debian 10+, 或 Windows 11 **透過 WSL2** |
| Node.js                     | **22 或更新版本** (建議使用 LTS 版本)                               |
| Git (可選，建議使用) | 2.23+ 用於內建的 PR 輔助工具                                   |
| 記憶體                         | 4GB 最低（建議 8GB）                                 |

> 切勿執行 `sudo npm install -g`；請改進 npm 權限問題。

---

## CLI 參考

| 命令                              | 用途                             | 範例                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | 互動式 REPL                    | `codex`                              |
| `codex "…"`                          | 互動式 REPL 的初始提示 | `codex "fix lint errors"`            |
| `codex -q "…"`                       | 非互動式 "靜默模式"        | `codex -q --json "explain utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | 列印 shell 自動完成腳本       | `codex completion bash`              |

關鍵標誌：`--model/-m`、`--approval-mode/-a` 和 `--quiet/-q`。

---

## 記憶與項目文件

Codex 按照以下順序合併 Markdown 指令：

1. `~/.codex/instructions.md` – 個人全域指導
2. 儲存庫根目錄中的 `codex.md` – 共享項目筆記
3. 當前工作目錄中的 `codex.md` – 子套件具體說明

使用 `--no-project-doc` 或 `CODEX_DISABLE_PROJECT_DOC=1` 停用。

---

## 非互動 / CI 模式

在管道中無頭運行 Codex。範例 GitHub Action 步驟：

```yaml
- name: 透過 Codex 更新變更日誌
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "update CHANGELOG for next release"
```

設定 `CODEX_QUIET_MODE=1` 以消除互動式 UI 噪音。

---

## 範例

以下是一些可複製粘貼的簡短範例。將引號中的文字替換為您自己的任務。請參閱[提示指南](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md)以獲取更多提示和使用模式。

| ✨  | 您輸入的內容                                                                   | 會發生什麼                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refactor the Dashboard component to React Hooks"`                       | Codex 重寫類別組件，運行 `npm test`，並顯示差異。   |
| 2   | `codex "Generate SQL migrations for adding a users table"`                      | 推斷您的 ORM，創建遷移檔案，並在沙箱化資料庫中運行。 |
| 3   | `codex "Write unit tests for utils/date.ts"`                                    | 生成測試，執行它們，並迭代直到通過。              |
| 4   | `codex "Bulk‑rename *.jpeg → *.jpg with git mv"`                                | 安全地重命名檔案並更新導入/使用。                           |
| 5   | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"`                      | 輸出逐步的人類解釋。                                  |
| 6   | `codex "Carefully review this repo, and propose 3 high impact well-scoped PRs"` | 在當前程式碼庫中建議有影響力的 PR。                            |
| 7   | `codex "Look for vulnerabilities and create a security review report"`          | 發現並解釋安全漏洞。                                          |

---

## 安裝

<details open>
<summary><strong>從 npm 安裝（推薦）</strong></summary>

```bash
npm install -g @openai/codex
# 或
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>從源碼構建</strong></summary>

```bash
# 克隆儲存庫並導航到 CLI 套件
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# 安裝依賴並構建
npm install
npm run build

# 獲取使用方法和選項
node ./dist/cli.js --help

# 直接運行本地構建的 CLI
node ./dist/cli.js

# 或全域鏈接命令以方便使用
npm link
```

</details>

---

## 設定

Codex 會在 **`~/.codex/`** 中尋找設定檔。

```yaml
# ~/.codex/config.yaml
model: o4-mini # 預設模型
fullAutoErrorMode: ask-user # 或 ignore-and-continue
```

您也可以定義自訂指令：

```yaml
# ~/.codex/instructions.md
- 始終以表情符號回應
- 僅在我明確提及時使用 git 命令
```

---

## 常見問題

<details>
<summary>OpenAI 在 2021 年發布了一個名為 Codex 的模型 - 這與其有關嗎？</summary>

在 2021 年，OpenAI 發布了 Codex，一個旨在從自然語言提示生成程式碼的 AI 系統。該原始 Codex 模型已於 2023 年 3 月停用，與此 CLI 工具無關。

</details>

<details>
<summary>如何阻止 Codex 觸碰我的儲存庫？</summary>

Codex 總是先在**沙箱**中運行。如果提議的命令或檔案變更看起來可疑，您只需在提示時回答 **n**，您的作業樹就不會發生任何變化。

</details>

<details>
<summary>它在 Windows 上能運行嗎？</summary>

不能直接運行。需要使用 [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex 已在 macOS 和 Linux 上測試，Node 版本 ≥ 22。

</details>

<details>
<summary>支持哪些模型？</summary>

支援任何與 [Responses API](https://platform.openai.com/docs/api-reference/responses) 兼容的模型。預設為 `o4-mini`，但可通過 `--model gpt-4o` 或在設定檔中設定 `model: gpt-4o` 來覆蓋。

</details>

---

## 資金機會

我們很高興啟動一項**100 萬美元的計劃**，支持使用 Codex CLI 和其他 OpenAI 模型的開源項目。

- 補助金以 **25,000 美元** API 信用額度為單位發放。
- 申請將**滾動審查**。

**有興趣？[在此申請](https://openai.com/form/codex-open-source-fund/)。**

---

## 貢獻

此項目正處於積極開發中，程式碼可能會發生重大變化。我們將在完成後更新此訊息！

更廣泛地說，我們歡迎貢獻 – 無論您是首次開啟拉取請求還是經驗豐富的維護者。同時，我們重視可靠性和長期可維護性，因此合併程式碼的標準故意**較高**。以下指南闡明了"高品質"的實際含義，並應使整個過程透明且友好。

### 開發工作流程

- 從 `main` 創建一個_主題分支_ – 例如 `feat/interactive-prompt`。
- 保持您的變更專注。多個不相關的修復應開啟為獨立的 PR。
- 在開發過程中使用 `npm run test:watch` 以獲得超快速回饋。
- 我們使用 **Vitest** 進行單元測試，**ESLint** + **Prettier** 進行樣式檢查，以及 **TypeScript** 進行類型檢查。
- 在推送前，運行完整的測試/類型/格式檢查套件：

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- 如果您**尚未**簽署貢獻者許可協議 (CLA)，請在 PR 中添加包含以下確切文字的評論：

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  CLA-Assistant 機器人將在所有作者簽署後將 PR 狀態標記為綠色。

```bash
# 監控模式（測試在更改時重新運行）
npm run test:watch

# 不生成檔案的類型檢查
npm run typecheck

# 自動修復 lint + prettier 問題
npm run lint:fix
npm run format:fix
```

### 撰寫高影響力程式碼變更

1. **從問題開始。** 開啟新問題或在現有討論中評論，以便在編寫程式碼前達成解決方案共識。
2. **添加或更新測試。** 每個新功能或錯誤修復都應附帶測試覆蓋，這些測試在您的變更前失敗且在之後通過。100% 覆蓋率不是必需的，但需有意義的斷言。
3. **記錄行為。** 如果您的變更影響面向使用者的行為，請更新 README、內聯幫助 (`codex --help`) 或相關範例項目。
4. **保持提交原子化。** 每個提交都應可編譯且測試通過。這使得審查和可能的回滾更容易。

### 開啟拉取請求

- 填寫 PR 模板（或包含類似資訊） – **什麼？為什麼？如何？**
- 在本地運行**所有**檢查 (`npm test && npm run lint && npm run typecheck`)。本地可捕獲的 CI 失敗會減慢流程。
- 確保您的分支與 `main` 保持最新，並已解決合併衝突。
- 僅在您認為 PR 處於可合併狀態時，將其標記為**準備審查**。

### 審查流程

1. 將指派一名維護者作為主要審查者。
2. 我們可能會要求進行更改 – 請勿將此視為針對個人。我們重視您的工作，但也重視一致性和長期可維護性。
3. 當達成共識認為 PR 達到標準時，維護者將進行壓縮合併。

### 社群價值觀

- **友善且包容。** 以尊重對待他人；我們遵循 [Contributor Covenant](https://www.contributor-covenant.org/)。
- **假設善意。** 書面溝通很難 – 請傾向於寬容。
- **教學與學習。** 如果發現令人困惑的內容，請開啟問題或 PR 進行改進。

### 尋求幫助

如果您在設定項目時遇到問題、想獲得對某個想法的回饋，或只是想說聲_你好_ – 請開啟討論或加入相關問題。我們樂於提供幫助。

讓我們一起使 Codex CLI 成為一個出色的工具。**快樂駭客！** :rocket:

### 貢獻者許可協議 (CLA)

所有貢獻者**必須**接受 CLA。流程很簡單：

1. 開啟您的拉取請求。
2. 貼上以下評論（或如果您之前已簽署，則回覆 `recheck`）：

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. CLA-Assistant 機器人將在儲存庫中記錄您的簽名，並將狀態檢查標記為通過。

無需特殊的 Git 命令、電子郵件附件或提交頁尾。

#### 快速修復

| 情境          | 命令                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| 修改最後提交 | `git commit --amend -s --no-edit && git push -f`                                          |
| 僅 GitHub UI    | 在 PR 中編輯提交訊息 → 添加<br>`Signed-off-by: Your Name <email@example.com>` |

**DCO 檢查**會阻止合併，直到 PR 中的每個提交都帶有頁尾（使用壓縮合併時僅為一個）。

### 發佈 `codex`

要發布 CLI 的新版本，請運行 `codex-cli/package.json` 中定義的發布腳本：

1. 開啟 `codex-cli` 目錄
2. 確保您在一個分支上，例如 `git checkout -b bump-version`
3. 將版本和 `CLI_VERSION` 升級到當前日期時間：`npm run release:version`
4. 提交版本升級（帶有 DCO 簽署）：
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. 複製 README，構建並發布到 npm：`npm run release`
6. 推送到分支：`git push origin HEAD`

---

## 安全與負責任 AI

您是否發現了漏洞或對模型輸出有疑慮？請發送電子郵件至 **security@openai.com**，我們將迅速回應。

---

## 許可證

此儲存庫採用 [Apache-2.0 許可證](LICENSE) 授權。