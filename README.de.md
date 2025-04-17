<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">Leichter Programmieragent, der in Ihrem Terminal läuft</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | [Español](./README.es.md) | [Português](./README.pt.md) | [Русский](./README.ru.md) | [Français](./README.fr.md) | **Deutsch** | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [繁體中文](./README.zh-TW.md) | [Bahasa Indonesia](./README.id.md) | [Italiano](./README.it.md)

![Codex demo GIF using: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>Inhaltsverzeichnis</strong></summary>

- [Haftungsausschluss für experimentelle Technologie](#haftungsausschluss-für-experimentelle-technologie)
- [Schnellstart](#schnellstart)
- [Warum Codex?](#warum-codex)
- [Sicherheitsmodell & Berechtigungen](#sicherheitsmodell-und-berechtigungen)
  - [Details zur Plattform-Sandbox](#details-zur-plattform-sandbox)
- [Systemanforderungen](#systemanforderungen)
- [CLI-Referenz](#cli-referenz)
- [Speicher & Projektdokumentation](#speicher-und-projektdokumentation)
- [Nicht-interaktiver / CI-Modus](#nicht-interaktiver--ci-modus)
- [Rezepte](#rezepte)
- [Installation](#installation)
- [Konfiguration](#konfiguration)
- [FAQ](#faq)
- [Finanzierungsmöglichkeit](#finanzierungsmöglichkeit)
- [Mitwirken](#mitwirken)
  - [Entwicklungsablauf](#entwicklungsablauf)
  - [Schreiben von wirkungsvollen Code-Änderungen](#schreiben-von-wirkungsvollen-code-änderungen)
  - [Öffnen eines Pull-Requests](#öffnen-eines-pull-requests)
  - [Überprüfungsprozess](#überprüfungsprozess)
  - [Gemeinschaftswerte](#gemeinschaftswerte)
  - [Hilfe bekommen](#hilfe-bekommen)
  - [Mitwirkenden-Lizenzvereinbarung (CLA)](#mitwirkenden-lizenzvereinbarung-cla)
    - [Schnelle Korrekturen](#schnelle-korrekturen)
  - [Veröffentlichen von `codex`](#veröffentlichen-von-codex)
- [Sicherheit & verantwortungsvolle KI](#sicherheit-und-verantwortungsvolle-ki)
- [Lizenz](#lizenz)

</details>

---

## Haftungsausschluss für experimentelle Technologie

Codex CLI ist ein experimentelles Projekt in aktiver Entwicklung. Es ist noch nicht stabil, kann Fehler enthalten, unvollständige Funktionen bieten oder größeren Änderungen unterliegen. Wir entwickeln es offen mit der Community und begrüßen:

- Fehlerberichte
- Funktionswünsche
- Pull-Requests
- Positive Stimmung

Helfen Sie uns, es zu verbessern, indem Sie Probleme melden oder PRs einreichen (siehe den Abschnitt unten, wie Sie beitragen können)!

## Schnellstart

Global installieren:

```shell
npm install -g @openai/codex
```

Als Nächstes setzen Sie Ihren OpenAI-API-Schlüssel als Umgebungsvariable:

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **Hinweis:** Dieser Befehl setzt den Schlüssel nur für Ihre aktuelle Terminal-Sitzung. Um ihn dauerhaft zu machen, fügen Sie die `export`-Zeile zur Konfigurationsdatei Ihrer Shell hinzu (z. B. `~/.zshrc`).

Interaktiv ausführen:

```shell
codex
```

Oder mit einem Prompt als Eingabe ausführen (und optional im `Full Auto`-Modus):

```shell
codex "erkläre mir diesen Code"
```

```shell
codex --approval-mode full-auto "erstelle die ausgefallenste To-Do-Listen-App"
```

Das war’s – Codex erstellt eine Datei, führt sie in einer Sandbox aus, installiert fehlende Abhängigkeiten und zeigt Ihnen das Ergebnis live an. Genehmigen Sie die Änderungen, und sie werden in Ihr Arbeitsverzeichnis übernommen.

---

## Warum Codex?

Codex CLI ist für Entwickler gedacht, die bereits **im Terminal leben** und eine ChatGPT-ähnliche Denkfähigkeit **plus** die Möglichkeit wünschen, Code tatsächlich auszuführen, Dateien zu manipulieren und zu iterieren – alles unter Versionskontrolle. Kurz gesagt, es ist _chatgesteuerte Entwicklung_, die Ihr Repository versteht und ausführt.

- **Keine Einrichtung** – Bringen Sie Ihren OpenAI-API-Schlüssel mit, und es funktioniert sofort!
- **Vollautomatische Genehmigung, aber sicher** durch Ausführung ohne Netzwerk und in einem sandgeboxten Verzeichnis
- **Multimodal** – Übergeben Sie Screenshots oder Diagramme, um Funktionen zu implementieren ✨

Und es ist **vollständig Open-Source**, sodass Sie sehen und dazu beitragen können, wie es sich entwickelt!

---

## Sicherheitsmodell & Berechtigungen

Codex lässt Sie entscheiden, _wie viel Autonomie_ der Agent erhält und welche automatische Genehmigungsrichtlinie über den `--approval-mode`-Flag (oder das interaktive Onboarding-Prompt) gilt:

| Modus                      | Was der Agent ohne Nachfrage tun darf            | Erfordert weiterhin Genehmigung                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **Vorschlagen** <br>(Standard) | • Jede Datei im Repository lesen                     | • **Alle** Dateischreibvorgänge/Patches <br>• **Alle** Shell/Bash-Befehle |
| **Automatische Bearbeitung**             | • Dateien lesen **und** Schreib-Patches anwenden      | • **Alle** Shell/Bash-Befehle                                   |
| **Vollautomatisch**             | • Dateien lesen/schreiben <br>• Shell-Befehle ausführen | –                                                               |

Im **Vollautomatisch**-Modus wird jeder Befehl **ohne Netzwerk** ausgeführt und auf das aktuelle Arbeitsverzeichnis (plus temporäre Dateien) beschränkt, um eine mehrschichtige Verteidigung zu gewährleisten. Codex zeigt auch eine Warnung/Bestätigung an, wenn Sie im **Automatische Bearbeitung**- oder **Vollautomatisch**-Modus starten, während das Verzeichnis _nicht_ von Git getrackt wird, sodass Sie immer ein Sicherheitsnetz haben.

Bald verfügbar: Sie können bestimmte Befehle auf eine Whitelist setzen, um sie automatisch mit aktiviertem Netzwerk auszuführen, sobald wir von zusätzlichen Sicherheitsmaßnahmen überzeugt sind.

### Details zur Plattform-Sandbox

Der von Codex verwendete Härtungsmechanismus hängt von Ihrem Betriebssystem ab:

- **macOS 12+** – Befehle werden mit **Apple Seatbelt** (`sandbox-exec`) umschlossen.

  - Alles wird in einen schreibgeschützten Jail gesetzt, außer einem kleinen Satz schreibbarer Wurzeln (`$PWD`, `$TMPDIR`, `~/.codex`, etc.).
  - Ausgehender Netzwerkverkehr ist standardmäßig _vollständig blockiert_ – selbst wenn ein Kindprozess versucht, irgendwo `curl` auszuführen, wird es fehlschlagen.

- **Linux** – Wir empfehlen die Verwendung von Docker für die Sandbox, wo Codex sich selbst in einem **minimalen Container-Image** startet und Ihr Repository im selben Pfad mit _Lese-/Schreibrechten_ einbindet. Ein benutzerdefiniertes `iptables`/`ipset`-Firewall-Skript blockiert jeglichen ausgehenden Verkehr außer der OpenAI-API. Dies ermöglicht deterministische, reproduzierbare Ausführungen ohne Root-Rechte auf dem Host. Mehr dazu in [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh).

Beide Ansätze sind für den täglichen Gebrauch _transparent_ – Sie führen `codex` weiterhin von der Wurzel Ihres Repositories aus und genehmigen/ablehnen Schritte wie gewohnt.

---

## Systemanforderungen

| Anforderung                 | Details                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Betriebssysteme           | macOS 12+, Ubuntu 20.04+/Debian 10+, oder Windows 11 **via WSL2** |
| Node.js                     | **22 oder neuer** (LTS empfohlen)                               |
| Git (optional, empfohlen) | 2.23+ für eingebaute PR-Helfer                                   |
| RAM                         | Mindestens 4 GB (8 GB empfohlen)                                 |

> Führen Sie niemals `sudo npm install -g` aus; beheben Sie stattdessen die npm-Berechtigungen.

---

## CLI-Referenz

| Befehl                              | Zweck                             | Beispiel                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | Interaktives REPL                    | `codex`                              |
| `codex "…"`                          | Anfangsprompt für interaktives REPL | `codex "Lint-Fehler beheben"`            |
| `codex -q "…"`                       | Nicht-interaktiver "leiser Modus"        | `codex -q --json "utils.ts erklären"` |
| `codex completion <bash\|zsh\|fish>` | Shell-Vervollständigungsskript ausgeben       | `codex completion bash`              |

Wichtige Flags: `--model/-m`, `--approval-mode/-a`, und `--quiet/-q`.

---

## Speicher & Projektdokumentation

Codex kombiniert Markdown-Anweisungen in dieser Reihenfolge:

1. `~/.codex/instructions.md` – persönliche globale Anleitungen
2. `codex.md` im Repository-Wurzelverzeichnis – gemeinsame Projektdokumentation
3. `codex.md` im aktuellen Arbeitsverzeichnis – spezifische Unterpaket-Details

Deaktivieren Sie dies mit `--no-project-doc` oder `CODEX_DISABLE_PROJECT_DOC=1`.

---

## Nicht-interaktiver / CI-Modus

Führen Sie Codex ohne Benutzeroberfläche in Pipelines aus. Beispiel für einen GitHub Action-Schritt:

```yaml
- name: Changelog über Codex aktualisieren
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "CHANGELOG für die nächste Version aktualisieren"
```

Setzen Sie `CODEX_QUIET_MODE=1`, um interaktive UI-Geräusche zu unterdrücken.

---

## Rezepte

Hier sind einige kurze Beispiele, die Sie kopieren und einfügen können. Ersetzen Sie den Text in Anführungszeichen durch Ihre eigene Aufgabe. Weitere Tipps und Nutzungsmuster finden Sie im [Prompting Guide](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md).

| ✨  | Was Sie eingeben                                                                   | Was passiert                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Die Dashboard-Komponente auf React Hooks umstellen"`                       | Codex schreibt die Klassenkomponente um, führt `npm test` aus und zeigt die Änderungen an.   |
| 2   | `codex "SQL-Migrationen für das Hinzufügen einer Benutzertabelle generieren"`                      | Erkennt Ihr ORM, erstellt Migrationsdateien und führt sie in einer sandgeboxten DB aus. |
| 3   | `codex "Unit-Tests für utils/date.ts schreiben"`                                    | Generiert Tests, führt sie aus und iteriert, bis sie bestehen.              |
| 4   | `codex "Massenumbenennung von *.jpeg → *.jpg mit git mv"`                                | Benennt Dateien sicher um und aktualisiert Importe/Verwendungen.                           |
| 5   | `codex "Erklären, was dieser Regex macht: ^(?=.*[A-Z]).{8,}$"`                      | Gibt eine schrittweise, für Menschen verständliche Erklärung aus.                                  |
| 6   | `codex "Dieses Repository sorgfältig prüfen und 3 gut abgegrenzte, wirkungsvolle PRs vorschlagen"` | Schlägt wirkungsvolle PRs in der aktuellen Codebasis vor.                            |
| 7   | `codex "Nach Schwachstellen suchen und einen Sicherheitsprüfungsbericht erstellen"`          | Findet und erklärt Sicherheitsfehler.                                          |

---

## Installation

<details open>
<summary><strong>Von npm (Empfohlen)</strong></summary>

```bash
npm install -g @openai/codex
# oder
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>Von Quelle bauen</strong></summary>

```bash
# Repository klonen und zum CLI-Paket navigieren
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# Abhängigkeiten installieren und bauen
npm install
npm run build

# Nutzung und Optionen anzeigen
node ./dist/cli.js --help

# Das lokal gebaute CLI direkt ausführen
node ./dist/cli.js

# Oder den Befehl global verknüpfen für mehr Komfort
npm link
```

</details>

---

## Konfiguration

Codex sucht nach Konfigurationsdateien in **`~/.codex/`**.

```yaml
# ~/.codex/config.yaml
model: o4-mini # Standardmodell
fullAutoErrorMode: ask-user # oder ignore-and-continue
```

Sie können auch benutzerdefinierte Anweisungen festlegen:

```yaml
# ~/.codex/instructions.md
- Immer mit Emojis antworten
- Git-Befehle nur verwenden, wenn ich dies ausdrücklich erwähne
```

---

## FAQ

<details>
<summary>OpenAI hat 2021 ein Modell namens Codex veröffentlicht – ist dies damit verwandt?</summary>

Im Jahr 2021 veröffentlichte OpenAI Codex, ein KI-System, das entwickelt wurde, um Code aus natürlichen Sprachprompts zu generieren. Dieses ursprüngliche Codex-Modell wurde im März 2023 eingestellt und ist vom CLI-Tool getrennt.

</details>

<details>
<summary>Wie verhindere ich, dass Codex mein Repository verändert?</summary>

Codex läuft immer zuerst in einer **Sandbox**. Wenn ein vorgeschlagener Befehl oder eine Dateiänderung verdächtig aussieht, können Sie einfach mit **n** antworten, wenn Sie aufgefordert werden, und es passiert nichts mit Ihrem Arbeitsbaum.

</details>

<details>
<summary>Funktioniert es unter Windows?</summary>

Nicht direkt. Es erfordert [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex wurde auf macOS und Linux mit Node ≥ 22 getestet.

</details>

<details>
<summary>Welche Modelle werden unterstützt?</summary>

Jedes Modell, das mit der [Responses API](https://platform.openai.com/docs/api-reference/responses) verfügbar ist. Der Standard ist `o4-mini`, aber übergeben Sie `--model gpt-4o` oder setzen Sie `model: gpt-4o` in Ihrer Konfigurationsdatei, um es zu überschreiben.

</details>

---

## Finanzierungsmöglichkeit

Wir freuen uns, eine **1-Millionen-Dollar-Initiative** zu starten, um Open-Source-Projekte zu unterstützen, die Codex CLI und andere OpenAI-Modelle verwenden.

- Zuschüsse werden in Schritten von **25.000 $** API-Guthaben vergeben.
- Anträge werden **laufend geprüft**.

**Interessiert? [Hier bewerben](https://openai.com/form/codex-open-source-fund/).**

---

## Mitwirken

Dieses Projekt befindet sich in aktiver Entwicklung, und der Code wird sich wahrscheinlich erheblich ändern. Wir werden diese Nachricht aktualisieren, sobald dies abgeschlossen ist!

Allgemein begrüßen wir Beiträge – egal, ob Sie Ihren allerersten Pull-Request öffnen oder ein erfahrener Maintainer sind. Gleichzeitig legen wir Wert auf Zuverlässigkeit und langfristige Wartbarkeit, weshalb die Hürde für das Zusammenführen von Code absichtlich **hoch** ist. Die folgenden Richtlinien erläutern, was "hohe Qualität" in der Praxis bedeutet und sollten den Prozess transparent und freundlich gestalten.

### Entwicklungsablauf

- Erstellen Sie einen _Themenzweig_ von `main` – z. B. `feat/interactive-prompt`.
- Halten Sie Ihre Änderungen fokussiert. Mehrere unzusammenhängende Korrekturen sollten als separate PRs geöffnet werden.
- Verwenden Sie `npm run test:watch` während der Entwicklung für superschnelles Feedback.
- Wir verwenden **Vitest** für Unit-Tests, **ESLint** + **Prettier** für Stil und **TypeScript** für Typprüfung.
- Führen Sie vor dem Push die gesamte Test-/Typ-/Lint-Suite aus:

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- Wenn Sie die Mitwirkenden-Lizenzvereinbarung (CLA) **noch nicht** unterzeichnet haben, fügen Sie einen PR-Kommentar mit dem exakten Text hinzu:

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  Der CLA-Assistant-Bot ändert den PR-Status auf grün, sobald alle Autoren unterschrieben haben.

```bash
# Watch-Modus (Tests werden bei Änderungen erneut ausgeführt)
npm run test:watch

# Typprüfung ohne Dateien zu erzeugen
npm run typecheck

# Lint- und Prettier-Probleme automatisch beheben
npm run lint:fix
npm run format:fix
```

### Schreiben von wirkungsvollen Code-Änderungen

1. **Beginnen Sie mit einem Issue.** Öffnen Sie ein neues oder kommentieren Sie eine bestehende Diskussion, damit wir uns auf die Lösung einigen können, bevor Code geschrieben wird.
2. **Tests hinzufügen oder aktualisieren.** Jede neue Funktion oder Fehlerbehebung sollte mit einer Testabdeckung geliefert werden, die vor Ihrer Änderung fehlschlägt und danach besteht. Eine 100 %-Abdeckung ist nicht erforderlich, aber streben Sie nach sinnvollen Aussagen.
3. **Verhalten dokumentieren.** Wenn Ihre Änderung das benutzerseitige Verhalten beeinflusst, aktualisieren Sie das README, die Inline-Hilfe (`codex --help`) oder relevante Beispielprojekte.
4. **Commits atomar halten.** Jeder Commit sollte kompilierbar sein und die Tests bestehen. Dies erleichtert Überprüfungen und mögliche Rücksetzungen.

### Öffnen eines Pull-Requests

- Füllen Sie die PR-Vorlage aus (oder geben Sie ähnliche Informationen an) – **Was? Warum? Wie?**
- Führen Sie **alle** Prüfungen lokal aus (`npm test && npm run lint && npm run typecheck`). CI-Fehler, die lokal hätten erkannt werden können, verlangsamen den Prozess.
- Stellen Sie sicher, dass Ihr Branch mit `main` auf dem neuesten Stand ist und Merge-Konflikte gelöst sind.
- Markieren Sie den PR als **Bereit zur Überprüfung**, nur wenn Sie glauben, dass er in einem zusammenführbaren Zustand ist.

### Überprüfungsprozess

1. Ein Maintainer wird als primärer Reviewer zugewiesen.
2. Wir können Änderungen anfordern – nehmen Sie dies bitte nicht persönlich. Wir schätzen die Arbeit, legen aber auch Wert auf Konsistenz und langfristige Wartbarkeit.
3. Sobald Einigkeit besteht, dass der PR die Anforderungen erfüllt, wird ein Maintainer ihn squashen und mergen.

### Gemeinschaftswerte

- **Seien Sie freundlich und inklusiv.** Behandeln Sie andere mit Respekt; wir folgen dem [Contributor Covenant](https://www.contributor-covenant.org/).
- **Unterstellen Sie gute Absichten.** Schriftliche Kommunikation ist schwierig – seien Sie großzügig.
- **Lehren und lernen.** Wenn Sie etwas Verwirrendes entdecken, öffnen Sie ein Issue oder einen PR mit Verbesserungen.

### Hilfe bekommen

Wenn Sie Probleme beim Einrichten des Projekts haben, Feedback zu einer Idee wünschen oder einfach nur _Hallo_ sagen möchten – öffnen Sie bitte eine Diskussion oder beteiligen Sie sich an einem relevanten Issue. Wir helfen gerne.

Zusammen können wir Codex CLI zu einem großartigen Werkzeug machen. **Viel Spaß beim Hacken!** :rocket:

### Mitwirkenden-Lizenzvereinbarung (CLA)

Alle Mitwirkenden **müssen** die CLA akzeptieren. Der Prozess ist unkompliziert:

1. Öffnen Sie Ihren Pull-Request.
2. Fügen Sie den folgenden Kommentar ein (oder antworten Sie mit `recheck`, wenn Sie bereits unterschrieben haben):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. Der CLA-Assistant-Bot zeichnet Ihre Unterschrift im Repository auf und markiert den Statuscheck als bestanden.

Keine speziellen Git-Befehle, E-Mail-Anhänge oder Commit-Fußzeilen erforderlich.

#### Schnelle Korrekturen

| Szenario          | Befehl                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Letzten Commit anpassen | `git commit --amend -s --no-edit && git push -f`                                          |
| Nur GitHub-UI    | Bearbeiten Sie die Commit-Nachricht im PR → fügen Sie hinzu<br>`Signed-off-by: Your Name <email@example.com>` |

Die **DCO-Prüfung** blockiert Merges, bis jeder Commit im PR die Fußzeile trägt (mit Squash ist dies nur einer).

### Veröffentlichen von `codex`

Um eine neue Version des CLI zu veröffentlichen, führen Sie die in `codex-cli/package.json` definierten Veröffentlichungsskripte aus:

1. Öffnen Sie das Verzeichnis `codex-cli`
2. Stellen Sie sicher, dass Sie sich auf einem Branch wie `git checkout -b bump-version` befinden
3. Erhöhen Sie die Version und `CLI_VERSION` auf das aktuelle Datum/Uhrzeit: `npm run release:version`
4. Bestätigen Sie die Versionserhöhung (mit DCO-Signatur):
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. Kopieren Sie das README, bauen Sie und veröffentlichen Sie auf npm: `npm run release`
6. Pushen Sie zum Branch: `git push origin HEAD`

---

## Sicherheit & verantwortungsvolle KI

Haben Sie eine Schwachstelle entdeckt oder Bedenken bezüglich der Modellausgabe? Bitte senden Sie eine E-Mail an **security@openai.com**, und wir werden umgehend antworten.

---

## Lizenz

Dieses Repository ist unter der [Apache-2.0-Lizenz](LICENSE) lizenziert.