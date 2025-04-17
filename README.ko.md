<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">터미널에서 실행되는 가벼운 코딩 에이전트</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | [Español](./README.es.md) | [Português](./README.pt.md) | [Русский](./README.ru.md) | [Français](./README.fr.md) | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | **한국어** | [繁體中文](./README.zh-TW.md) | [Bahasa Indonesia](./README.id.md) | [Italiano](./README.it.md)

![Codex demo GIF 사용: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>목차</strong></summary>

- [실험적 기술 면책 조항](#실험적-기술-면책-조항)
- [빠른 시작](#빠른-시작)
- [왜 Codex인가?](#왜-codex인가)
- [보안 모델 및 권한](#보안-모델-및-권한)
  - [플랫폼 샌드박싱 세부 정보](#플랫폼-샌드박싱-세부-정보)
- [시스템 요구 사항](#시스템-요구-사항)
- [CLI 참조](#cli-참조)
- [메모리 및 프로젝트 문서](#메모리-및-프로젝트-문서)
- [비대화형 / CI 모드](#비대화형--ci-모드)
- [레시피](#레시피)
- [설치](#설치)
- [설정](#설정)
- [FAQ](#faq)
- [자금 지원 기회](#자금-지원-기회)
- [기여](#기여)
  - [개발 워크플로우](#개발-워크플로우)
  - [고영향 코드 변경 작성](#고영향-코드-변경-작성)
  - [풀 리퀘스트 열기](#풀-리퀘스트-열기)
  - [검토 프로세스](#검토-프로세스)
  - [커뮤니티 가치](#커뮤니티-가치)
  - [도움 받기](#도움-받기)
  - [기여자 라이선스 계약 (CLA)](#기여자-라이선스-계약-cla)
    - [빠른 수정](#빠른-수정)
  - [Codex 릴리스](#codex-릴리스)
- [보안 및 책임 있는 AI](#보안-및-책임-있는-ai)
- [라이선스](#라이선스)

</details>

---

## 실험적 기술 면책 조항

Codex CLI는 활발히 개발 중인 실험적 프로젝트입니다. 아직 안정적이지 않으며, 버그, 불완전한 기능, 또는 중대한 변경이 발생할 수 있습니다. 우리는 커뮤니티와 함께 공개적으로 개발하고 있으며 다음을 환영합니다:

- 버그 보고
- 기능 요청
- 풀 리퀘스트
- 긍정적인 피드백

이슈를 제출하거나 PR을 제출하여 개선에 도움을 주세요 (기여 방법은 아래 섹션을 참조하세요)!

## 빠른 시작

글로벌 설치:

```shell
npm install -g @openai/codex
```

다음으로, OpenAI API 키를 환경 변수로 설정하세요:

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **참고:** 이 명령은 현재 터미널 세션에만 키를 설정합니다. 영구적으로 설정하려면 셸 설정 파일(예: `~/.zshrc`)에 `export` 줄을 추가하세요.

대화형으로 실행:

```shell
codex
```

또는 입력 프롬프트와 함께 실행 (선택적으로 `Full Auto` 모드 사용):

```shell
codex "이 코드베이스를 설명해줘"
```

```shell
codex --approval-mode full-auto "가장 멋진 할 일 목록 앱을 만들어줘"
```

이제 끝입니다 – Codex는 파일을 생성하고 샌드박스 내에서 실행하며, 누락된 의존성을 설치하고 실시간 결과를 보여줍니다. 변경 사항을 승인하면 작업 디렉토리에 커밋됩니다.

---

## 왜 Codex인가?

Codex CLI는 이미 **터미널에서 작업하는** 개발자를 위해 설계되었으며, ChatGPT 수준의 추론 능력과 **실제로 코드를 실행하고, 파일을 조작하며, 반복 작업을 수행**하는 기능을 결합하고자 합니다 – 모두 버전 관리 하에 있습니다. 간단히 말해, 이는 저장소를 이해하고 실행하는 _대화 주도 개발_ 도구입니다.

- **설정 제로** — OpenAI API 키만 가져오면 바로 작동합니다!
- **완전 자동 승인, 안전하고 보안 유지** — 네트워크 비활성화 및 디렉토리 샌드박싱으로 구현
- **다중 모달** — 스크린샷이나 다이어그램을 전달하여 기능을 구현 ✨

그리고 이는 **완전 오픈 소스**로, 개발 과정을 확인하고 기여할 수 있습니다!

---

## 보안 모델 및 권한

Codex는 `--approval-mode` 플래그(또는 대화형 온보딩 프롬프트)를 통해 에이전트가 받는 _자율성 수준_과 자동 승인 정책을 결정할 수 있게 합니다:

| 모드                      | 에이전트가 묻지 않고 할 수 있는 작업            | 여전히 승인이 필요한 작업                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **제안** <br>(기본값) | • 저장소의 모든 파일 읽기                     | • **모든** 파일 쓰기/패치 <br>• **모든** 셸/Bash 명령 |
| **자동 편집**             | • 파일 읽기 **및** 패치 쓰기 적용      | • **모든** 셸/Bash 명령                                   |
| **완전 자동**             | • 파일 읽기/쓰기 <br>• 셸 명령 실행 | —                                                               |

**완전 자동** 모드에서는 모든 명령이 **네트워크 비활성화** 상태로 실행되며, 현재 작업 디렉토리(및 임시 파일)에 국한되어 심층 방어를 제공합니다. 디렉토리가 Git에 의해 _추적되지 않는_ 경우, **자동 편집** 또는 **완전 자동** 모드에서 시작할 때 Codex는 경고/확인을 표시하므로 항상 안전망이 있습니다.

곧 출시 예정: 추가 안전 조치에 확신이 생기면 특정 명령에 대해 네트워크를 활성화한 자동 실행을 허용할 수 있습니다.

### 플랫폼 샌드박싱 세부 정보

Codex가 사용하는 강화 메커니즘은 OS에 따라 다릅니다:

- **macOS 12+** – 명령은 **Apple Seatbelt** (`sandbox-exec`)로 래핑됩니다.

  - 쓰기 가능한 루트 디렉토리(`$PWD`, `$TMPDIR`, `~/.codex` 등)를 제외한 모든 것은 읽기 전용 감옥에 배치됩니다.
  - 기본적으로 아웃바운드 네트워크는 _완전히 차단_됩니다 – 자식 프로세스가 `curl`로 어딘가에 접근하려 해도 실패합니다.

- **Linux** – 샌드박싱을 위해 Docker를 사용하는 것이 좋습니다. Codex는 **최소 컨테이너 이미지** 내에서 자신을 실행하고 저장소를 동일한 경로에 _읽기/쓰기_로 마운트합니다. 사용자 지정 `iptables`/`ipset` 방화벽 스크립트는 OpenAI API를 제외한 모든 아웃바운드 트래픽을 차단합니다. 이를 통해 호스트에서 루트 권한 없이도 결정적이고 재현 가능한 실행이 가능합니다. 자세한 내용은 [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh)에서 확인할 수 있습니다.

두 접근 방식 모두 일상적인 사용에서 _투명_합니다 – 저장소 루트에서 `codex`를 실행하고 평소처럼 단계를 승인/거부할 수 있습니다.

---

## 시스템 요구 사항

| 요구 사항                 | 세부 정보                                                         |
| --------------------------- | --------------------------------------------------------------- |
| 운영 체제           | macOS 12+, Ubuntu 20.04+/Debian 10+, 또는 Windows 11 **WSL2를 통해** |
| Node.js                     | **22 이상** (LTS 권장)                               |
| Git (선택 사항, 권장) | 2.23+ 내장 PR 헬퍼용                                   |
| RAM                         | 최소 4GB (8GB 권장)                                 |

> 절대 `sudo npm install -g`를 실행하지 마세요; 대신 npm 권한을 수정하세요.

---

## CLI 참조

| 명령                              | 목적                             | 예제                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | 대화형 REPL                    | `codex`                              |
| `codex "…"`                          | 대화형 REPL용 초기 프롬프트 | `codex "린트 오류 수정"`            |
| `codex -q "…"`                       | 비대화형 "조용한 모드"        | `codex -q --json "utils.ts 설명"` |
| `codex completion <bash\|zsh\|fish>` | 셸 완성 스크립트 출력       | `codex completion bash`              |

주요 플래그: `--model/-m`, `--approval-mode/-a`, `--quiet/-q`.

---

## 메모리 및 프로젝트 문서

Codex는 다음 순서로 Markdown 지침을 병합합니다:

1. `~/.codex/instructions.md` – 개인 글로벌 가이드
2. 저장소 루트의 `codex.md` – 공유 프로젝트 노트
3. 현재 작업 디렉토리의 `codex.md` – 서브패키지 세부 사항

`--no-project-doc` 또는 `CODEX_DISABLE_PROJECT_DOC=1`로 비활성화하세요.

---

## 비대화형 / CI 모드

파이프라인에서 헤드리스로 Codex를 실행하세요. GitHub Action 단계 예제:

```yaml
- name: Codex를 통해 변경 로그 업데이트
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "다음 릴리스용 CHANGELOG 업데이트"
```

`CODEX_QUIET_MODE=1`을 설정하여 대화형 UI 노이즈를 제거하세요.

---

## 레시피

아래는 복사하여 붙여넣을 수 있는 몇 가지 간단한 예제입니다. 따옴표 안의 텍스트를 자신의 작업으로 교체하세요. 더 많은 팁과 사용 패턴은 [프롬프트 가이드](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md)를 참조하세요.

| ✨  | 입력하는 내용                                                                   | 발생하는 일                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Dashboard 컴포넌트를 React Hooks로 리팩터링"`                       | Codex는 클래스 컴포넌트를 재작성하고, `npm test`를 실행하며 차이점을 보여줍니다.   |
| 2   | `codex "사용자 테이블 추가를 위한 SQL 마이그레이션 생성"`                      | ORM을 추론하고, 마이그레이션 파일을 생성하며 샌드박스 DB에서 실행합니다. |
| 3   | `codex "utils/date.ts용 유닛 테스트 작성"`                                    | 테스트를 생성하고 실행하며, 통과할 때까지 반복합니다.              |
| 4   | `codex "git mv로 *.jpeg → *.jpg 일괄 이름 변경"`                                | 파일을 안전하게 이름 변경하고 가져오기/사용을 업데이트합니다.                           |
| 5   | `codex "이 정규식의 역할 설명: ^(?=.*[A-Z]).{8,}$"`                      | 단계별로 인간이 이해할 수 있는 설명을 출력합니다.                                  |
| 6   | `codex "이 저장소를 신중히 검토하고, 영향력 있는 3개의 잘 정의된 PR 제안"` | 현재 코드베이스에서 영향력 있는 PR을 제안합니다.                            |
| 7   | `codex "취약점을 찾아 보안 검토 보고서 작성"`          | 보안 버그를 찾아 설명합니다.                                          |

---

## 설치

<details open>
<summary><strong>npm에서 설치 (권장)</strong></summary>

```bash
npm install -g @openai/codex
# 또는
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>소스에서 빌드</strong></summary>

```bash
# 저장소를 복제하고 CLI 패키지로 이동
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# 의존성 설치 및 빌드
npm install
npm run build

# 사용법 및 옵션 확인
node ./dist/cli.js --help

# 로컬에서 빌드된 CLI 직접 실행
node ./dist/cli.js

# 편의를 위해 명령을 글로벌로 링크
npm link
```

</details>

---

## 설정

Codex는 **`~/.codex/`**에서 설정 파일을 찾습니다.

```yaml
# ~/.codex/config.yaml
model: o4-mini # 기본 모델
fullAutoErrorMode: ask-user # 또는 ignore-and-continue
```

사용자 지정 지침도 정의할 수 있습니다:

```yaml
# ~/.codex/instructions.md
- 항상 이모지로 응답
- 내가 명시적으로 언급할 때만 git 명령 사용
```

---

## FAQ

<details>
<summary>OpenAI는 2021년에 Codex라는 모델을 출시했는데, 이것과 관련이 있나요?</summary>

2021년 OpenAI는 자연어 프롬프트에서 코드를 생성하도록 설계된 AI 시스템 Codex를 출시했습니다. 이 원래 Codex 모델은 2023년 3월에 폐기되었으며 CLI 도구와는 별개입니다.

</details>

<details>
<summary>Codex가 내 저장소를 건드리지 않도록 어떻게 하나요?</summary>

Codex는 항상 **샌드박스에서 먼저 실행**됩니다. 제안된 명령이나 파일 변경이 의심스러워 보이면 프롬프트에서 **n**을 답변하면 작업 트리에 아무 변화도 일어나지 않습니다.

</details>

<details>
<summary>Windows에서 작동하나요?</summary>

직접적으로는 작동하지 않습니다. [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install)가 필요합니다 – Codex는 macOS와 Linux에서 Node ≥ 22로 테스트되었습니다.

</details>

<details>
<summary>어떤 모델이 지원되나요?</summary>

[Responses API](https://platform.openai.com/docs/api-reference/responses)에서 사용 가능한 모든 모델을 지원합니다. 기본값은 `o4-mini`이지만, `--model gpt-4o`를 전달하거나 설정 파일에서 `model: gpt-4o`를 설정하여 재정의할 수 있습니다.

</details>

---

## 자금 지원 기회

우리는 Codex CLI 및 기타 OpenAI 모델을 사용하는 오픈 소스 프로젝트를 지원하는 **100만 달러 이니셔티브**를 시작하게 되어 기쁩니다.

- 보조금은 **25,000달러** API 크레딧 단위로 수여됩니다.
- 신청은 **상시 검토**됩니다.

**관심 있으신가요? [여기에서 신청하세요](https://openai.com/form/codex-open-source-fund/).**

---

## 기여

이 프로젝트는 활발히 개발 중이며 코드가 상당히 많이 변경될 가능성이 있습니다. 완료되면 이 메시지를 업데이트할 예정입니다!

더 넓게 보면, 첫 번째 풀 리퀘스트를 여는 초보자든 숙련된 유지 관리자든 모든 기여를 환영합니다. 동시에 우리는 신뢰성과 장기적인 유지 관리 가능성을 중요하게 여기므로 코드 병합의 기준은 의도적으로 **높게** 설정되어 있습니다. 아래 가이드라인은 "고품질"이 실제로 무엇을 의미하는지 설명하며, 전체 프로세스를 투명하고 친근하게 만들어야 합니다.

### 개발 워크플로우

- `main`에서 _토픽 브랜치_를 생성하세요 – 예: `feat/interactive-prompt`.
- 변경 사항을 집중적으로 유지하세요. 관련 없는 여러 수정은 별도의 PR로 열어야 합니다.
- 개발 중에는 `npm run test:watch`를 사용하여 빠른 피드백을 받으세요.
- 우리는 유닛 테스트에 **Vitest**, 스타일에 **ESLint** + **Prettier**, 타입 체크에 **TypeScript**를 사용합니다.
- 푸시 전에 전체 테스트/타입/린트 스위트를 실행하세요:

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- 아직 기여자 라이선스 계약(CLA)을 **서명하지 않았다면**, PR에 다음 정확한 텍스트를 포함한 댓글을 추가하세요:

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  CLA-Assistant 봇은 모든 작성자가 서명하면 PR 상태를 녹색으로 표시합니다.

```bash
# 감시 모드 (변경 시 테스트 재실행)
npm run test:watch

# 파일 생성 없이 타입 체크
npm run typecheck

# 린트 및 prettier 문제 자동 수정
npm run lint:fix
npm run format:fix
```

### 고영향 코드 변경 작성

1. **이슈로 시작하세요.** 새 이슈를 열거나 기존 논의에 댓글을 달아 코드 작성 전에 해결책에 대해 합의하세요.
2. **테스트 추가 또는 업데이트.** 모든 새 기능 또는 버그 수정은 변경 전에는 실패하고 변경 후에는 통과하는 테스트 커버리지를 포함해야 합니다. 100% 커버리지는 필수 아니지만, 의미 있는 단언을 목표로 하세요.
3. **동작 문서화.** 변경 사항이 사용자에게 보이는 동작에 영향을 미친다면, README, 인라인 도움말(`codex --help`), 또는 관련 예제 프로젝트를 업데이트하세요.
4. **커밋을 원자적으로 유지.** 각 커밋은 컴파일 가능해야 하며 테스트가 통과해야 합니다. 이는 검토와 잠재적 롤백을 쉽게 만듭니다.

### 풀 리퀘스트 열기

- PR 템플릿을 작성하거나(또는 유사한 정보 포함) – **무엇? 왜? 어떻게?**
- 로컬에서 **모든** 체크를 실행하세요(`npm test && npm run lint && npm run typecheck`). 로컬에서 잡을 수 있었던 CI 실패는 프로세스를 느리게 합니다.
- 브랜치가 `main`과 최신 상태이며 병합 충돌이 해결되었는지 확인하세요.
- PR이 병합 가능한 상태라고 믿을 때만 **검토 준비**로 표시하세요.

### 검토 프로세스

1. 한 명의 유지 관리자가 주요 검토자로 지정됩니다.
2. 변경 요청이 있을 수 있습니다 – 이를 개인적으로 받아들이지 마세요. 우리는 작업을 소중히 여기지만, 일관성과 장기적인 유지 관리 가능성도 중요하게 생각합니다.
3. PR이 기준을 충족한다는 합의가 이루어지면, 유지 관리자가 스쿼시 병합을 수행합니다.

### 커뮤니티 가치

- **친절하고 포용적.** 다른 사람을 존중하며 대하세요; 우리는 [Contributor Covenant](https://www.contributor-covenant.org/)를 따릅니다.
- **선한 의도 가정.** 서면 커뮤니케이션은 어렵습니다 – 관대한 태도를 취하세요.
- **가르치고 배우기.** 혼란스러운 점을 발견하면 이슈 또는 PR을 열어 개선하세요.

### 도움 받기

프로젝트 설정에 문제가 있거나, 아이디어에 대한 피드백을 원하거나, 그냥 _안녕_이라고 말하고 싶다면 – 토론을 열거나 관련 이슈에 참여하세요. 우리는 기꺼이 도와드립니다.

함께 Codex CLI를 멋진 도구로 만들 수 있습니다. **해피 해킹!** :rocket:

### 기여자 라이선스 계약 (CLA)

모든 기여자는 **CLA를 수락해야** 합니다. 프로세스는 간단합니다:

1. 풀 리퀘스트를 여세요.
2. 다음 댓글을 붙여넣으세요(또는 이전에 서명했다면 `recheck`로 답변):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. CLA-Assistant 봇이 저장소에 서명을 기록하고 상태 체크를 통과로 표시합니다.

특별한 Git 명령, 이메일 첨부 파일, 또는 커밋 푸터는 필요 없습니다.

#### 빠른 수정

| 시나리오          | 명령                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| 마지막 커밋 수정 | `git commit --amend -s --no-edit && git push -f`                                          |
| GitHub UI 전용    | PR에서 커밋 메시지를 편집 → 추가<br>`Signed-off-by: Your Name <email@example.com>` |

**DCO 체크**는 PR의 모든 커밋이 푸터를 포함할 때까지 병합을 차단합니다(스쿼시 병합 시 하나만 해당).

### Codex 릴리스

CLI의 새 버전을 게시하려면 `codex-cli/package.json`에 정의된 릴리스 스크립트를 실행하세요:

1. `codex-cli` 디렉토리를 여세요
2. `git checkout -b bump-version` 같은 브랜치에 있는지 확인하세요
3. 버전 및 `CLI_VERSION`을 현재 날짜/시간으로 업데이트: `npm run release:version`
4. 버전 업데이트 커밋(DCO 서명 포함):
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. README 복사, 빌드, npm에 게시: `npm run release`
6. 브랜치로 푸시: `git push origin HEAD`

---

## 보안 및 책임 있는 AI

취약점을 발견했거나 모델 출력에 대한 우려가 있나요? **security@openai.com**으로 이메일을 보내주시면 신속히 답변드리겠습니다.

---

## 라이선스

이 저장소는 [Apache-2.0 라이선스](LICENSE)에 따라 라이선스가 부여됩니다.