<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">Agente de programação leve que roda no seu terminal</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | [Español](./README.es.md) | **Português** | [Русский](./README.ru.md) | [Français](./README.fr.md) | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [繁體中文](./README.zh-TW.md) | [Bahasa Indonesia](./README.id.md) | [Italiano](./README.it.md)

![Codex demo GIF usando: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>Índice</strong></summary>

- [Aviso de Tecnologia Experimental](#aviso-de-tecnologia-experimental)
- [Início Rápido](#início-rápido)
- [Por que Codex?](#por-que-codex)
- [Modelo de Segurança e Permissões](#modelo-de-segurança-e-permissões)
  - [Detalhes de sandbox da plataforma](#detalhes-de-sandbox-da-plataforma)
- [Requisitos do Sistema](#requisitos-do-sistema)
- [Referência CLI](#referência-cli)
- [Memória e Documentos do Projeto](#memória-e-documentos-do-projeto)
- [Modo Não Interativo / CI](#modo-não-interativo--ci)
- [Receitas](#receitas)
- [Instalação](#instalação)
- [Configuração](#configuração)
- [Perguntas Frequentes](#perguntas-frequentes)
- [Oportunidade de Financiamento](#oportunidade-de-financiamento)
- [Contribuindo](#contribuindo)
  - [Fluxo de trabalho de desenvolvimento](#fluxo-de-trabalho-de-desenvolvimento)
  - [Escrevendo mudanças de código de alto impacto](#escrevendo-mudanças-de-código-de-alto-impacto)
  - [Abrindo um pull request](#abrindo-um-pull-request)
  - [Processo de revisão](#processo-de-revisão)
  - [Valores da comunidade](#valores-da-comunidade)
  - [Obtendo ajuda](#obtendo-ajuda)
  - [Acordo de Licença de Contribuidor (CLA)](#acordo-de-licença-de-contribuidor-cla)
    - [Correções rápidas](#correções-rápidas)
  - [Lançando `codex`](#lançando-codex)
- [Segurança e IA Responsável](#segurança-e-ia-responsável)
- [Licença](#licença)

</details>

---

## Aviso de Tecnologia Experimental

Codex CLI é um projeto experimental em desenvolvimento ativo. Ainda não é estável, pode conter bugs, funcionalidades incompletas ou sofrer mudanças drásticas. Estamos construindo-o abertamente com a comunidade e recebemos:

- Relatórios de bugs
- Solicitações de funcionalidades
- Pull requests
- Boas vibrações

Ajude-nos a melhorar enviando relatórios de problemas ou submetendo PRs (veja a seção abaixo sobre como contribuir)!

## Início Rápido

Instale globalmente:

```shell
npm install -g @openai/codex
```

Em seguida, defina sua chave de API da OpenAI como uma variável de ambiente:

```shell
export OPENAI_API_KEY="sua-chave-api-aqui"
```

> **Nota:** Este comando define a chave apenas para a sessão atual do terminal. Para torná-la permanente, adicione a linha `export` ao arquivo de configuração do seu shell (por exemplo, `~/.zshrc`).

Execute interativamente:

```shell
codex
```

Ou execute com um prompt como entrada (e opcionalmente no modo `Full Auto`):

```shell
codex "explique esta base de código para mim"
```

```shell
codex --approval-mode full-auto "crie o aplicativo de lista de tarefas mais sofisticado"
```

É isso – Codex criará um arquivo, executará dentro de uma sandbox, instalará quaisquer dependências faltantes e mostrará o resultado ao vivo. Aprove as mudanças e elas serão commitadas no seu diretório de trabalho.

---

## Por que Codex?

Codex CLI é projetado para desenvolvedores que já **vivem no terminal** e desejam raciocínio no nível do ChatGPT **mais** o poder de executar código, manipular arquivos e iterar – tudo sob controle de versão. Em resumo, é _desenvolvimento guiado por chat_ que entende e executa seu repositório.

- **Configuração zero** — traga sua chave de API da OpenAI e ele simplesmente funciona!
- **Aprovação automática completa, mas segura** ao rodar com rede desativada e sandbox de diretórios
- **Multimodal** — passe capturas de tela ou diagramas para implementar funcionalidades ✨

E é **totalmente open-source**, então você pode ver e contribuir para seu desenvolvimento!

---

## Modelo de Segurança e Permissões

Codex permite que você decida _quanta autonomia_ o agente recebe e a política de aprovação automática por meio do sinalizador `--approval-mode` (ou do prompt interativo de integração):

| Modo                      | O que o agente pode fazer sem perguntar            | Ainda requer aprovação                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **Sugerir** <br>(padrão) | • Ler qualquer arquivo no repositório                     | • **Todas** as escritas/patches de arquivos <br>• **Todos** os comandos shell/Bash |
| **Edição Automática**             | • Ler **e** aplicar patches de escrita em arquivos      | • **Todos** os comandos shell/Bash                                   |
| **Automático Completo**             | • Ler/escrever arquivos <br>• Executar comandos shell | –                                                               |

No modo **Automático Completo**, cada comando é executado com **rede desativada** e confinado ao diretório de trabalho atual (mais arquivos temporários) para defesa em profundidade. Codex também mostrará um aviso/confirmação se você iniciar no modo **edição automática** ou **automático completo** enquanto o diretório _não_ for rastreado pelo Git, garantindo sempre uma rede de segurança.

Em breve: você poderá liberar comandos específicos para execução automática com a rede ativada, assim que estivermos confiantes em salvaguardas adicionais.

### Detalhes de sandbox da plataforma

O mecanismo de reforço usado pelo Codex depende do seu sistema operacional:

- **macOS 12+** – os comandos são encapsulados com **Apple Seatbelt** (`sandbox-exec`).

  - Tudo é colocado em uma jaula somente leitura, exceto por um pequeno conjunto de raízes graváveis (`$PWD`, `$TMPDIR`, `~/.codex`, etc.).
  - A rede de saída é _totalmente bloqueada_ por padrão – mesmo que um processo filho tente fazer `curl` para algum lugar, ele falhará.

- **Linux** – recomendamos usar Docker para sandbox, onde Codex se inicia dentro de uma **imagem de contêiner mínima** e monta seu repositório com leitura/escrita no mesmo caminho. Um script de firewall personalizado `iptables`/`ipset` bloqueia todo o tráfego de saída, exceto a API da OpenAI. Isso proporciona execuções determinísticas e reproduzíveis sem necessidade de root no host. Saiba mais em [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh).

Ambas as abordagens são _transparentes_ para o uso diário – você ainda executa `codex` a partir da raiz do repositório e aprova/rejeita os passos como de costume.

---

## Requisitos do Sistema

| Requisito                 | Detalhes                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Sistemas operacionais           | macOS 12+, Ubuntu 20.04+/Debian 10+, ou Windows 11 **via WSL2** |
| Node.js                     | **22 ou mais recente** (LTS recomendado)                               |
| Git (opcional, recomendado) | 2.23+ para ajudantes de PR integrados                                   |
| RAM                         | Mínimo de 4 GB (8 GB recomendado)                                 |

> Nunca execute `sudo npm install -g`; corrija as permissões do npm em vez disso.

---

## Referência CLI

| Comando                              | Finalidade                             | Exemplo                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | REPL interativo                    | `codex`                              |
| `codex "…"`                          | Prompt inicial para REPL interativo | `codex "corrigir erros de lint"`            |
| `codex -q "…"`                       | Modo não interativo "silencioso"        | `codex -q --json "explicar utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | Imprimir script de autocompletar do shell       | `codex completion bash`              |

Sinalizadores principais: `--model/-m`, `--approval-mode/-a` e `--quiet/-q`.

---

## Memória e Documentos do Projeto

Codex mescla instruções em Markdown nesta ordem:

1. `~/.codex/instructions.md` – orientação global pessoal
2. `codex.md` na raiz do repositório – notas compartilhadas do projeto
3. `codex.md` no diretório atual – especificidades de subpacotes

Desative com `--no-project-doc` ou `CODEX_DISABLE_PROJECT_DOC=1`.

---

## Modo Não Interativo / CI

Execute Codex sem interface em pipelines. Exemplo de passo de GitHub Action:

```yaml
- name: Atualizar changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "atualizar CHANGELOG para o próximo lançamento"
```

Defina `CODEX_QUIET_MODE=1` para silenciar o ruído da interface interativa.

---

## Receitas

Abaixo estão alguns exemplos curtos que você pode copiar e colar. Substitua o texto entre aspas pela sua própria tarefa. Veja o [guia de prompts](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md) para mais dicas e padrões de uso.

| ✨  | O que você digita                                                                   | O que acontece                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refatorar o componente Dashboard para React Hooks"`                       | Codex reescreve o componente de classe, executa `npm test` e mostra o diff.   |
| 2   | `codex "Gerar migrações SQL para adicionar uma tabela de usuários"`                      | Infere seu ORM, cria arquivos de migração e os executa em um banco de dados em sandbox. |
| 3   | `codex "Escrever testes unitários para utils/date.ts"`                                    | Gera testes, os executa e itera até que passem.              |
| 4   | `codex "Renomear em massa *.jpeg → *.jpg com git mv"`                                | Renomeia arquivos com segurança e atualiza importações/usos.                           |
| 5   | `codex "Explicar o que esse regex faz: ^(?=.*[A-Z]).{8,}$"`                      | Produz uma explicação passo a passo para humanos.                                  |
| 6   | `codex "Revisar cuidadosamente este repositório e propor 3 PRs bem definidos de alto impacto"` | Sugere PRs impactantes na base de código atual.                            |
| 7   | `codex "Procurar vulnerabilidades e criar um relatório de revisão de segurança"`          | Encontra e explica bugs de segurança.                                          |

---

## Instalação

<details open>
<summary><strong>Do npm (Recomendado)</strong></summary>

```bash
npm install -g @openai/codex
# ou
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>Construir a partir do código-fonte</strong></summary>

```bash
# Clonar o repositório e navegar para o pacote CLI
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# Instalar dependências e construir
npm install
npm run build

# Obter o uso e as opções
node ./dist/cli.js --help

# Executar o CLI construído localmente diretamente
node ./dist/cli.js

# Ou vincular o comando globalmente para conveniência
npm link
```

</details>

---

## Configuração

Codex procura arquivos de configuração em **`~/.codex/`**.

```yaml
# ~/.codex/config.yaml
model: o4-mini # Modelo padrão
fullAutoErrorMode: ask-user # ou ignore-and-continue
```

Você também pode definir instruções personalizadas:

```yaml
# ~/.codex/instructions.md
- Sempre responder com emojis
- Usar comandos git apenas se eu mencionar explicitamente
```

---

## Perguntas Frequentes

<details>
<summary>A OpenAI lançou um modelo chamado Codex em 2021 - isso está relacionado?</summary>

Em 2021, a OpenAI lançou o Codex, um sistema de IA projetado para gerar código a partir de prompts em linguagem natural. Esse modelo Codex original foi descontinuado em março de 2023 e é separado da ferramenta CLI.

</details>

<details>
<summary>Como faço para impedir que o Codex toque no meu repositório?</summary>

Codex sempre roda primeiro em uma **sandbox**. Se um comando ou mudança de arquivo proposto parecer suspeito, você pode simplesmente responder **n** quando solicitado, e nada acontecerá com sua árvore de trabalho.

</details>

<details>
<summary>Funciona no Windows?</summary>

Não diretamente. Requer [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex foi testado em macOS e Linux com Node ≥ 22.

</details>

<details>
<summary>Quais modelos são suportados?</summary>

Qualquer modelo disponível com a [Responses API](https://platform.openai.com/docs/api-reference/responses). O padrão é `o4-mini`, mas passe `--model gpt-4o` ou defina `model: gpt-4o` no seu arquivo de configuração para substituir.

</details>

---

## Oportunidade de Financiamento

Estamos empolgados em lançar uma **iniciativa de $1 milhão** para apoiar projetos open-source que usam o Codex CLI e outros modelos da OpenAI.

- As bolsas são concedidas em incrementos de **$25.000** em créditos de API.
- As candidaturas são revisadas **de forma contínua**.

**Interessado? [Inscreva-se aqui](https://openai.com/form/codex-open-source-fund/).**

---

## Contribuindo

Este projeto está em desenvolvimento ativo, e o código provavelmente mudará significativamente. Atualizaremos esta mensagem quando isso estiver concluído!

De forma mais ampla, recebemos contribuições – seja você abrindo seu primeiro pull request ou sendo um mantenedor experiente. Ao mesmo tempo, valorizamos confiabilidade e manutenção a longo prazo, então o padrão para mesclar código é intencionalmente **alto**. As diretrizes abaixo explicam o que "alta qualidade" significa na prática e devem tornar o processo transparente e amigável.

### Fluxo de trabalho de desenvolvimento

- Crie uma _branch de tópico_ a partir de `main` – por exemplo, `feat/interactive-prompt`.
- Mantenha suas mudanças focadas. Várias correções não relacionadas devem ser abertas como PRs separados.
- Use `npm run test:watch` durante o desenvolvimento para feedback super rápido.
- Usamos **Vitest** para testes unitários, **ESLint** + **Prettier** para estilo e **TypeScript** para verificação de tipos.
- Antes de enviar, execute a suíte completa de testes/tipos/lint:

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- Se você **não** assinou o Acordo de Licença de Contribuidor (CLA), adicione um comentário no PR contendo o texto exato:

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  O bot CLA-Assistant marcará o status do PR como verde assim que todos os autores assinarem.

```bash
# Modo de observação (testes reexecutados ao mudar)
npm run test:watch

# Verificação de tipos sem emitir arquivos
npm run typecheck

# Corrigir automaticamente problemas de lint e prettier
npm run lint:fix
npm run format:fix
```

### Escrevendo mudanças de código de alto impacto

1. **Comece com um issue.** Abra um novo ou comente em uma discussão existente para que possamos concordar com a solução antes de escrever o código.
2. **Adicione ou atualize testes.** Cada nova funcionalidade ou correção de bug deve vir com cobertura de teste que falha antes da sua mudança e passa depois. 100% de cobertura não é necessário, mas busque asserções significativas.
3. **Documente o comportamento.** Se sua mudança afetar o comportamento voltado ao usuário, atualize o README, a ajuda inline (`codex --help`) ou projetos de exemplo relevantes.
4. **Mantenha commits atômicos.** Cada commit deve compilar e os testes devem passar. Isso facilita revisões e possíveis reversões.

### Abrindo um pull request

- Preencha o modelo de PR (ou inclua informações semelhantes) – **O quê? Por quê? Como?**
- Execute **todas** as verificações localmente (`npm test && npm run lint && npm run typecheck`). Falhas de CI que poderiam ser detectadas localmente atrasam o processo.
- Certifique-se de que sua branch está atualizada com `main` e que os conflitos de mesclagem foram resolvidos.
- Marque o PR como **Pronto para revisão** apenas quando acreditar que está em um estado mesclável.

### Processo de revisão

1. Um mantenedor será designado como revisor primário.
2. Podemos pedir alterações – por favor, não leve isso para o lado pessoal. Valorizamos o trabalho, mas também valorizamos consistência e manutenção a longo prazo.
3. Quando houver consenso de que o PR atende ao padrão, um mantenedor fará o squash-and-merge.

### Valores da comunidade

- **Seja gentil e inclusivo.** Trate os outros com respeito; seguimos o [Contributor Covenant](https://www.contributor-covenant.org/).
- **Presuma boa intenção.** Comunicação escrita é difícil – erre pelo lado da generosidade.
- **Ensine e aprenda.** Se você encontrar algo confuso, abra um issue ou PR com melhorias.

### Obtendo ajuda

Se você encontrar problemas ao configurar o projeto, quiser feedback sobre uma ideia ou apenas quiser dizer _oi_ – por favor, abra uma Discussão ou entre no issue relevante. Estamos felizes em ajudar.

Juntos, podemos tornar o Codex CLI uma ferramenta incrível. **Feliz hacking!** :rocket:

### Acordo de Licença de Contribuidor (CLA)

Todos os contribuidores **devem** aceitar o CLA. O processo é leve:

1. Abra seu pull request.
2. Cole o seguinte comentário (ou responda `recheck` se já assinou antes):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. O bot CLA-Assistant registra sua assinatura no repositório e marca a verificação de status como aprovada.

Nenhum comando Git especial, anexos de e-mail ou rodapés de commit são necessários.

#### Correções rápidas

| Cenário          | Comando                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Alterar último commit | `git commit --amend -s --no-edit && git push -f`                                          |
| Apenas GitHub UI    | Edite a mensagem do commit no PR → adicione<br>`Signed-off-by: Seu Nome <email@example.com>` |

A verificação **DCO** bloqueia mesclagens até que cada commit no PR tenha o rodapé (com squash, é apenas um).

### Lançando `codex`

Para publicar uma nova versão do CLI, execute os scripts de lançamento definidos em `codex-cli/package.json`:

1. Abra o diretório `codex-cli`
2. Certifique-se de estar em uma branch como `git checkout -b bump-version`
3. Atualize a versão e `CLI_VERSION` para a data e hora atuais: `npm run release:version`
4. Faça o commit da atualização de versão (com assinatura DCO):
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. Copie o README, construa e publique no npm: `npm run release`
6. Envie para a branch: `git push origin HEAD`

---

## Segurança e IA Responsável

Descobriu uma vulnerabilidade ou tem preocupações com a saída do modelo? Por favor, envie um e-mail para **security@openai.com** e responderemos prontamente.

---

## Licença

Este repositório é licenciado sob a [Licença Apache-2.0](LICENSE).