<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">Agente di programmazione leggero che funziona nel tuo terminale</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | [Español](./README.es.md) | [Português](./README.pt.md) | [Русский](./README.ru.md) | [Français](./README.fr.md) | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [繁體中文](./README.zh-TW.md) | [Bahasa Indonesia](./README.id.md) | **Italiano**

![Codex demo GIF usando: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>Indice</strong></summary>

- [Dichiarazione di esclusione di responsabilità per tecnologia sperimentale](#dichiarazione-di-esclusione-di-responsabilità-per-tecnologia-sperimentale)
- [Avvio rapido](#avvio-rapido)
- [Perché Codex?](#perché-codex)
- [Modello di sicurezza e permessi](#modello-di-sicurezza-e-permessi)
  - [Dettagli sulla sandbox della piattaforma](#dettagli-sulla-sandbox-della-piattaforma)
- [Requisiti di sistema](#requisiti-di-sistema)
- [Riferimento CLI](#riferimento-cli)
- [Memoria e documenti di progetto](#memoria-e-documenti-di-progetto)
- [Modalità non interattiva / CI](#modalità-non-interattiva--ci)
- [Ricette](#ricette)
- [Installazione](#installazione)
- [Configurazione](#configurazione)
- [FAQ](#faq)
- [Opportunità di finanziamento](#opportunità-di-finanziamento)
- [Contribuire](#contribuire)
  - [Flusso di lavoro di sviluppo](#flusso-di-lavoro-di-sviluppo)
  - [Scrivere modifiche al codice ad alto impatto](#scrivere-modifiche-al-codice-ad-alto-impatto)
  - [Aprire una pull request](#aprire-una-pull-request)
  - [Processo di revisione](#processo-di-revisione)
  - [Valori della comunità](#valori-della-comunità)
  - [Ottenere aiuto](#ottenere-aiuto)
  - [Accordo di licenza per i contributori (CLA)](#accordo-di-licenza-per-i-contributori-cla)
    - [Correzioni rapide](#correzioni-rapide)
  - [Rilascio di `codex`](#rilascio-di-codex)
- [Sicurezza e IA responsabile](#sicurezza-e-ia-responsabile)
- [Licenza](#licenza)

</details>

---

## Dichiarazione di esclusione di responsabilità per tecnologia sperimentale

Codex CLI è un progetto sperimentale in fase di sviluppo attivo. Non è ancora stabile, potrebbe contenere bug, funzionalità incomplete o subire modifiche sostanziali. Lo stiamo costruendo apertamente con la comunità e accogliamo con piacere:

- Segnalazioni di bug
- Richieste di funzionalità
- Pull request
- Feedback positivi

Aiutaci a migliorare segnalando problemi o inviando PR (vedi la sezione qui sotto su come contribuire)!

## Avvio rapido

Installa globalmente:

```shell
npm install -g @openai/codex
```

Successivamente, imposta la tua chiave API OpenAI come variabile d'ambiente:

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **Nota:** Questo comando imposta la chiave solo per la sessione corrente del terminale. Per renderla permanente, aggiungi la riga `export` al file di configurazione della tua shell (es. `~/.zshrc`).

Esegui in modalità interattiva:

```shell
codex
```

Oppure, esegui con un prompt come input (e opzionalmente in modalità `Full Auto`):

```shell
codex "spiega questo codebase"
```

```shell
codex --approval-mode full-auto "crea l'app di lista delle cose da fare più sofisticata"
```

Questo è tutto – Codex genererà un file, lo eseguirà in una sandbox, installerà eventuali dipendenze mancanti e ti mostrerà il risultato in tempo reale. Approva le modifiche e verranno salvate nella tua directory di lavoro.

---

## Perché Codex?

Codex CLI è progettato per gli sviluppatori che già **vivono nel terminale** e desiderano un ragionamento a livello di ChatGPT **più** la capacità di eseguire effettivamente codice, manipolare file e iterare – tutto sotto controllo di versione. In breve, è uno _sviluppo guidato dalla chat_ che comprende ed esegue il tuo repository.

- **Configurazione zero** — porta la tua chiave API OpenAI e funziona subito!
- **Approvazione completamente automatica, ma sicura** grazie all'esecuzione con rete disabilitata e directory in sandbox
- **Multimodale** — passa screenshot o diagrammi per implementare funzionalità ✨

Ed è **completamente open-source**, quindi puoi vedere e contribuire al suo sviluppo!

---

## Modello di sicurezza e permessi

Codex ti permette di decidere _quanto autonomia_ concedere all'agente e la politica di approvazione automatica tramite il flag `--approval-mode` (o il prompt interattivo di onboarding):

| Modalità                      | Cosa può fare l'agente senza chiedere            | Richiede ancora approvazione                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **Suggerisci** <br>(predefinita) | • Leggere qualsiasi file nel repository                     | • **Tutte** le scritture/patch dei file <br>• **Tutti** i comandi shell/Bash |
| **Modifica automatica**             | • Leggere **e** applicare patch di scrittura ai file      | • **Tutti** i comandi shell/Bash                                   |
| **Completamente automatica**             | • Leggere/scrivere file <br>• Eseguire comandi shell | —                                                               |

In modalità **Completamente automatica**, ogni comando viene eseguito con **rete disabilitata** e confinato nella directory di lavoro corrente (più file temporanei) per una difesa in profondità. Codex mostrerà anche un avviso/conferma se avvii in modalità **modifica automatica** o **completamente automatica** mentre la directory _non_ è tracciata da Git, garantendoti sempre una rete di sicurezza.

Prossimamente: potrai autorizzare comandi specifici per l'esecuzione automatica con rete abilitata, una volta che saremo sicuri di ulteriori misure di sicurezza.

### Dettagli sulla sandbox della piattaforma

Il meccanismo di rafforzamento utilizzato da Codex dipende dal tuo sistema operativo:

- **macOS 12+** – i comandi sono avvolti con **Apple Seatbelt** (`sandbox-exec`).

  - Tutto è collocato in una prigione di sola lettura, tranne un piccolo insieme di radici scrivibili (`$PWD`, `$TMPDIR`, `~/.codex`, ecc.).
  - La rete in uscita è _completamente bloccata_ per impostazione predefinita – anche se un processo figlio tenta di eseguire `curl` verso un indirizzo, fallirà.

- **Linux** – consigliamo di usare Docker per la sandbox, dove Codex si avvia all'interno di un'**immagine di container minimale** e monta il tuo repository in _lettura/scrittura_ nello stesso percorso. Uno script firewall personalizzato `iptables`/`ipset` blocca tutto il traffico in uscita tranne l'API OpenAI. Questo ti offre esecuzioni deterministiche e riproducibili senza bisogno di privilegi di root sull'host. Puoi leggere di più in [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh).

Entrambi gli approcci sono _trasparenti_ per l'uso quotidiano – esegui comunque `codex` dalla radice del tuo repository e approvi/rifiuti i passaggi come al solito.

---

## Requisiti di sistema

| Requisito                 | Dettagli                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Sistemi operativi           | macOS 12+, Ubuntu 20.04+/Debian 10+, o Windows 11 **tramite WSL2** |
| Node.js                     | **22 o superiore** (LTS consigliato)                               |
| Git (opzionale, consigliato) | 2.23+ per gli helper PR integrati                                   |
| RAM                         | Minimo 4 GB (8 GB consigliati)                                 |

> Non eseguire mai `sudo npm install -g`; correggi invece i permessi di npm.

---

## Riferimento CLI

| Comando                              | Scopo                             | Esempio                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | REPL interattivo                    | `codex`                              |
| `codex "…"`                          | Prompt iniziale per REPL interattivo | `codex "correggi errori di lint"`            |
| `codex -q "…"`                       | Modalità non interattiva "silenziosa"        | `codex -q --json "spiega utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | Stampa script di completamento shell       | `codex completion bash`              |

Flag principali: `--model/-m`, `--approval-mode/-a`, e `--quiet/-q`.

---

## Memoria e documenti di progetto

Codex unisce le istruzioni Markdown in questo ordine:

1. `~/.codex/instructions.md` – guida globale personale
2. `codex.md` nella radice del repository – note di progetto condivise
3. `codex.md` nella directory di lavoro corrente – specifiche dei sottopacchetti

Disabilita con `--no-project-doc` o `CODEX_DISABLE_PROJECT_DOC=1`.

---

## Modalità non interattiva / CI

Esegui Codex senza interfaccia nelle pipeline. Esempio di passo GitHub Action:

```yaml
- name: Aggiorna changelog tramite Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "aggiorna CHANGELOG per il prossimo rilascio"
```

Imposta `CODEX_QUIET_MODE=1` per silenziare il rumore dell'interfaccia interattiva.

---

## Ricette

Di seguito alcuni esempi brevi che puoi copiare e incollare. Sostituisci il testo tra virgolette con il tuo compito. Consulta la [guida ai prompt](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md) per ulteriori suggerimenti e modelli d'uso.

| ✨  | Cosa digiti                                                                   | Cosa succede                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Rifattorizza il componente Dashboard in React Hooks"`                       | Codex riscrive il componente di classe, esegue `npm test` e mostra la diff.   |
| 2   | `codex "Genera migrazioni SQL per aggiungere una tabella utenti"`                      | Deduce il tuo ORM, crea file di migrazione ed esegue in un DB sandbox. |
| 3   | `codex "Scrivi test unitari per utils/date.ts"`                                    | Genera test, li esegue e itera finché non passano.              |
| 4   | `codex "Rinomina in blocco *.jpeg → *.jpg con git mv"`                                | Rinomina i file in modo sicuro e aggiorna importazioni/usi.                           |
| 5   | `codex "Spiega cosa fa questa regex: ^(?=.*[A-Z]).{8,}$"`                      | Produce una spiegazione passo-passo comprensibile.                                  |
| 6   | `codex "Esamina attentamente questo repository e proponi 3 PR ben definite ad alto impatto"` | Suggerisce PR di impatto nel codebase attuale.                            |
| 7   | `codex "Cerca vulnerabilità e crea un rapporto di revisione della sicurezza"`          | Trova e spiega bug di sicurezza.                                          |

---

## Installazione

<details open>
<summary><strong>Da npm (Consigliato)</strong></summary>

```bash
npm install -g @openai/codex
# o
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>Compila dal sorgente</strong></summary>

```bash
# Clona il repository e naviga nel pacchetto CLI
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# Installa le dipendenze e compila
npm install
npm run build

# Ottieni l'uso e le opzioni
node ./dist/cli.js --help

# Esegui direttamente il CLI compilato localmente
node ./dist/cli.js

# O collega il comando globalmente per comodità
npm link
```

</details>

---

## Configurazione

Codex cerca i file di configurazione in **`~/.codex/`**.

```yaml
# ~/.codex/config.yaml
model: o4-mini # Modello predefinito
fullAutoErrorMode: ask-user # o ignore-and-continue
```

Puoi anche definire istruzioni personalizzate:

```yaml
# ~/.codex/instructions.md
- Rispondi sempre con emoji
- Usa comandi git solo se lo menziono esplicitamente
```

---

## FAQ

<details>
<summary>OpenAI ha rilasciato un modello chiamato Codex nel 2021 - è correlato?</summary>

Nel 2021, OpenAI ha rilasciato Codex, un sistema AI progettato per generare codice da prompt in linguaggio naturale. Quel modello Codex originale è stato deprecato a marzo 2023 ed è separato dallo strumento CLI.

</details>

<details>
<summary>Come faccio a impedire a Codex di toccare il mio repository?</summary>

Codex esegue sempre prima in una **sandbox**. Se un comando proposto o una modifica al file sembra sospetta, puoi semplicemente rispondere **n** quando richiesto e nulla accadrà al tuo albero di lavoro.

</details>

<details>
<summary>Funziona su Windows?</summary>

Non direttamente. Richiede [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex è stato testato su macOS e Linux con Node ≥ 22.

</details>

<details>
<summary>Quali modelli sono supportati?</summary>

Qualsiasi modello disponibile con [Responses API](https://platform.openai.com/docs/api-reference/responses). Il predefinito è `o4-mini`, ma puoi passare `--model gpt-4o` o impostare `model: gpt-4o` nel file di configurazione per sovrascrivere.

</details>

---

## Opportunità di finanziamento

Siamo entusiasti di lanciare un'**iniziativa da 1 milione di dollari** per supportare progetti open source che utilizzano Codex CLI e altri modelli OpenAI.

- I finanziamenti sono concessi in incrementi di **25.000 dollari** di crediti API.
- Le candidature sono esaminate **su base continuativa**.

**Interessato? [Fai domanda qui](https://openai.com/form/codex-open-source-fund/).**

---

## Contribuire

Questo progetto è in fase di sviluppo attivo e il codice probabilmente cambierà significativamente. Aggiorneremo questo messaggio una volta completato!

In generale, accogliamo con favore i contributi – che tu stia aprendo la tua prima pull request o sia un mantenitore esperto. Allo stesso tempo, ci preoccupiamo della affidabilità e della manutenibilità a lungo termine, quindi la soglia per il merge del codice è intenzionalmente **alta**. Le linee guida qui sotto spiegano cosa significa "alta qualità" nella pratica e dovrebbero rendere il processo trasparente e amichevole.

### Flusso di lavoro di sviluppo

- Crea un _ramo tematico_ da `main` – es. `feat/interactive-prompt`.
- Mantieni le tue modifiche focalizzate. Correzioni multiple non correlate dovrebbero essere aperte come PR separate.
- Usa `npm run test:watch` durante lo sviluppo per feedback super veloci.
- Usiamo **Vitest** per i test unitari, **ESLint** + **Prettier** per lo stile e **TypeScript** per il controllo dei tipi.
- Prima di fare push, esegui l'intera suite di test/tipi/lint:

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- Se **non hai ancora** firmato l'Accordo di licenza per i contributori (CLA), aggiungi un commento alla PR con il testo esatto:

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  Il bot CLA-Assistant renderà lo stato della PR verde una volta che tutti gli autori avranno firmato.

```bash
# Modalità watch (i test vengono rieseguiti al cambio)
npm run test:watch

# Controllo dei tipi senza emissione di file
npm run typecheck

# Correggi automaticamente problemi di lint + prettier
npm run lint:fix
npm run format:fix
```

### Scrivere modifiche al codice ad alto impatto

1. **Inizia con un issue.** Apri uno nuovo o commenta una discussione esistente così possiamo concordare sulla soluzione prima che il codice venga scritto.
2. **Aggiungi o aggiorna i test.** Ogni nuova funzionalità o correzione di bug dovrebbe includere una copertura di test che fallisce prima della modifica e passa dopo. Non è richiesta una copertura del 100%, ma punta a asserzioni significative.
3. **Documenta il comportamento.** Se la tua modifica influisce sul comportamento visibile agli utenti, aggiorna il README, l'aiuto inline (`codex --help`) o i progetti di esempio pertinenti.
4. **Mantieni i commit atomici.** Ogni commit dovrebbe essere compilabile e i test dovrebbero passare. Questo rende le revisioni e i potenziali rollback più facili.

### Aprire una pull request

- Compila il modello di PR (o includi informazioni simili) – **Cosa? Perché? Come?**
- Esegui **tutti** i controlli localmente (`npm test && npm run lint && npm run typecheck`). I fallimenti CI che potevano essere rilevati localmente rallentano il processo.
- Assicurati che il tuo ramo sia aggiornato con `main` e che i conflitti di merge siano stati risolti.
- Contrassegna la PR come **Pronta per la revisione** solo quando credi che sia in uno stato unibile.

### Processo di revisione

1. Un mantenitore sarà assegnato come revisore primario.
2. Potremmo richiedere modifiche – per favore, non prenderla sul personale. Apprezziamo il lavoro, ma valorizziamo anche la coerenza e la manutenibilità a lungo termine.
3. Quando c'è consenso che la PR soddisfa gli standard, un mantenitore eseguirà uno squash-and-merge.

### Valori della comunità

- **Sii gentile e inclusivo.** Tratta gli altri con rispetto; seguiamo il [Contributor Covenant](https://www.contributor-covenant.org/).
- **Presupponi buone intenzioni.** La comunicazione scritta è difficile – opta per la generosità.
- **Insegna e impara.** Se noti qualcosa di confuso, apri un issue o una PR con miglioramenti.

### Ottenere aiuto

Se incontri problemi nella configurazione del progetto, desideri feedback su un'idea o vuoi semplicemente dire _ciao_ – per favore, apri una Discussione o partecipa all'issue pertinente. Siamo felici di aiutarti.

Insieme possiamo rendere Codex CLI uno strumento incredibile. **Buon hacking!** :rocket:

### Accordo di licenza per i contributori (CLA)

Tutti i contributori **devono** accettare il CLA. Il processo è leggero:

1. Apri la tua pull request.
2. Incolla il seguente commento (o rispondi `recheck` se hai già firmato in precedenza):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. Il bot CLA-Assistant registra la tua firma nel repository e contrassegna il controllo dello stato come superato.

Nessun comando Git speciale, allegati email o piè di pagina dei commit richiesti.

#### Correzioni rapide

| Scenario          | Comando                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Modifica ultimo commit | `git commit --amend -s --no-edit && git push -f`                                          |
| Solo UI GitHub    | Modifica il messaggio del commit nella PR → aggiungi<br>`Signed-off-by: Your Name <email@example.com>` |

Il controllo **DCO** blocca i merge finché ogni commit nella PR non contiene il piè di pagina (con lo squash è solo uno).

### Rilascio di `codex`

Per pubblicare una nuova versione del CLI, esegui gli script di rilascio definiti in `codex-cli/package.json`:

1. Apri la directory `codex-cli`
2. Assicurati di essere su un ramo come `git checkout -b bump-version`
3. Incrementa la versione e `CLI_VERSION` alla data/ora corrente: `npm run release:version`
4. Esegui il commit dell'incremento di versione (con firma DCO):
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. Copia il README, compila e pubblica su npm: `npm run release`
6. Esegui il push al ramo: `git push origin HEAD`

---

## Sicurezza e IA responsabile

Hai scoperto una vulnerabilità o hai preoccupazioni sull'output del modello? Invia un'email a **security@openai.com** e risponderemo prontamente.

---

## Licenza

Questo repository è concesso in licenza sotto la [Licenza Apache-2.0](LICENSE).