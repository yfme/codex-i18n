<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">Agent de programmation léger qui s'exécute dans votre terminal</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | [Español](./README.es.md) | [Português](./README.pt.md) | [Русский](./README.ru.md) | **Français** | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [繁體中文](./README.zh-TW.md) | [Bahasa Indonesia](./README.id.md) | [Italiano](./README.it.md)

![Codex demo GIF utilisant: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>Tableau des matières</strong></summary>

- [Avertissement sur la technologie expérimentale](#avertissement-sur-la-technologie-expérimentale)
- [Démarrage rapide](#démarrage-rapide)
- [Pourquoi Codex ?](#pourquoi-codex)
- [Modèle de sécurité et permissions](#modèle-de-sécurité-et-permissions)
  - [Détails sur le sandboxing de la plateforme](#détails-sur-le-sandboxing-de-la-plateforme)
- [Exigences système](#exigences-système)
- [Référence CLI](#référence-cli)
- [Mémoire et documents de projet](#mémoire-et-documents-de-projet)
- [Mode non interactif / CI](#mode-non-interactif--ci)
- [Recettes](#recettes)
- [Installation](#installation)
- [Configuration](#configuration)
- [FAQ](#faq)
- [Opportunité de financement](#opportunité-de-financement)
- [Contribuer](#contribuer)
  - [Flux de travail de développement](#flux-de-travail-de-développement)
  - [Rédiger des modifications de code à fort impact](#rédiger-des-modifications-de-code-à-fort-impact)
  - [Ouvrir une pull request](#ouvrir-une-pull-request)
  - [Processus de révision](#processus-de-révision)
  - [Valeurs de la communauté](#valeurs-de-la-communauté)
  - [Obtenir de l'aide](#obtenir-de-laide)
  - [Accord de licence des contributeurs (CLA)](#accord-de-licence-des-contributeurs-cla)
    - [Corrections rapides](#corrections-rapides)
  - [Publier `codex`](#publier-codex)
- [Sécurité et IA responsable](#sécurité-et-ia-responsable)
- [Licence](#licence)

</details>

---

## Avertissement sur la technologie expérimentale

Codex CLI est un projet expérimental en cours de développement actif. Il n'est pas encore stable, peut contenir des bugs, des fonctionnalités incomplètes ou subir des changements majeurs. Nous le construisons de manière ouverte avec la communauté et accueillons :

- Rapports de bugs
- Demandes de fonctionnalités
- Pull requests
- Bonne humeur

Aidez-nous à améliorer en signalant des problèmes ou en soumettant des PR (voir la section ci-dessous pour savoir comment contribuer) !

## Démarrage rapide

Installez globalement :

```shell
npm install -g @openai/codex
```

Ensuite, définissez votre clé API OpenAI en tant que variable d'environnement :

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **Note :** Cette commande définit la clé uniquement pour la session terminal en cours. Pour la rendre permanente, ajoutez la ligne `export` au fichier de configuration de votre shell (par exemple, `~/.zshrc`).

Exécutez de manière interactive :

```shell
codex
```

Ou exécutez avec une invite en entrée (et optionnellement en mode `Full Auto`) :

```shell
codex "expliquez-moi ce code"
```

```shell
codex --approval-mode full-auto "créez l'application de liste de tâches la plus sophistiquée"
```

C'est tout – Codex va générer un fichier, l'exécuter dans un sandbox, installer les dépendances manquantes et vous montrer le résultat en temps réel. Approuvez les changements et ils seront validés dans votre répertoire de travail.

---

## Pourquoi Codex ?

Codex CLI est conçu pour les développeurs qui **vivent déjà dans le terminal** et veulent un raisonnement au niveau de ChatGPT **plus** la capacité d'exécuter réellement du code, de manipuler des fichiers et d'itérer – tout cela sous contrôle de version. En bref, c'est un _développement guidé par le chat_ qui comprend et exécute votre dépôt.

- **Aucune configuration** — apportez votre clé API OpenAI et ça fonctionne immédiatement !
- **Approbation automatique complète, mais sécurisée** grâce à l'exécution sans réseau et dans un répertoire sandboxé
- **Multimodal** — passez des captures d'écran ou des diagrammes pour implémenter des fonctionnalités ✨

Et c'est **entièrement open-source**, donc vous pouvez voir et contribuer à son développement !

---

## Modèle de sécurité et permissions

Codex vous permet de décider _quel niveau d'autonomie_ accorder à l'agent et la politique d'approbation automatique via le flag `--approval-mode` (ou l'invite interactive d'intégration) :

| Mode                      | Ce que l'agent peut faire sans demander            | Nécessite encore une approbation                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **Suggérer** <br>(par défaut) | • Lire n'importe quel fichier dans le dépôt                     | • **Toutes** les écritures/patchs de fichiers <br>• **Toutes** les commandes shell/Bash |
| **Édition automatique**             | • Lire **et** appliquer des écritures de patchs aux fichiers      | • **Toutes** les commandes shell/Bash                                   |
| **Automatique complet**             | • Lire/écrire des fichiers <br>• Exécuter des commandes shell | —                                                               |

En mode **Automatique complet**, chaque commande est exécutée avec le **réseau désactivé** et confinée au répertoire de travail actuel (plus les fichiers temporaires) pour une défense en profondeur. Codex affichera également un avertissement/confirmation si vous démarrez en mode **édition automatique** ou **automatique complet** alors que le répertoire _n'est pas_ suivi par Git, vous offrant ainsi un filet de sécurité constant.

À venir prochainement : vous pourrez mettre sur liste blanche certaines commandes pour une exécution automatique avec le réseau activé, une fois que nous serons confiants dans des mesures de sécurité supplémentaires.

### Détails sur le sandboxing de la plateforme

Le mécanisme de durcissement utilisé par Codex dépend de votre système d'exploitation :

- **macOS 12+** – les commandes sont enveloppées avec **Apple Seatbelt** (`sandbox-exec`).

  - Tout est placé dans une prison en lecture seule, sauf un petit ensemble de racines inscriptibles (`$PWD`, `$TMPDIR`, `~/.codex`, etc.).
  - Le réseau sortant est _entièrement bloqué_ par défaut – même si un processus fils tente d'utiliser `curl` quelque part, il échouera.

- **Linux** – nous recommandons d'utiliser Docker pour le sandboxing, où Codex se lance lui-même dans une **image de conteneur minimale** et monte votre dépôt en _lecture/écriture_ au même chemin. Un script de pare-feu personnalisé `iptables`/`ipset` refuse tout trafic sortant sauf l'API OpenAI. Cela vous offre des exécutions déterministes et reproductibles sans nécessiter de privilèges root sur l'hôte. Vous pouvez en lire plus dans [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh).

Les deux approches sont _transparentes_ pour une utilisation quotidienne – vous exécutez toujours `codex` depuis la racine de votre dépôt et approuvez/rejetez les étapes comme d'habitude.

---

## Exigences système

| Exigence                 | Détails                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Systèmes d'exploitation           | macOS 12+, Ubuntu 20.04+/Debian 10+, ou Windows 11 **via WSL2** |
| Node.js                     | **22 ou plus récent** (LTS recommandé)                               |
| Git (facultatif, recommandé) | 2.23+ pour les assistants PR intégrés                                   |
| RAM                         | Minimum 4 Go (8 Go recommandé)                                 |

> Ne jamais exécuter `sudo npm install -g` ; corrigez plutôt les permissions npm.

---

## Référence CLI

| Commande                              | Objectif                             | Exemple                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | REPL interactif                    | `codex`                              |
| `codex "…"`                          | Invite initiale pour REPL interactif | `codex "corriger les erreurs de lint"`            |
| `codex -q "…"`                       | Mode non interactif "silencieux"        | `codex -q --json "expliquer utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | Imprimer le script de complétion shell       | `codex completion bash`              |

Drapeaux clés : `--model/-m`, `--approval-mode/-a`, et `--quiet/-q`.

---

## Mémoire et documents de projet

Codex fusionne les instructions Markdown dans cet ordre :

1. `~/.codex/instructions.md` – conseils globaux personnels
2. `codex.md` à la racine du dépôt – notes de projet partagées
3. `codex.md` dans le répertoire de travail actuel – spécificités des sous-paquets

Désactiver avec `--no-project-doc` ou `CODEX_DISABLE_PROJECT_DOC=1`.

---

## Mode non interactif / CI

Exécutez Codex sans interface dans les pipelines. Exemple d'étape GitHub Action :

```yaml
- name: Mettre à jour le changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "mettre à jour CHANGELOG pour la prochaine version"
```

Définissez `CODEX_QUIET_MODE=1` pour supprimer le bruit de l'interface utilisateur interactive.

---

## Recettes

Voici quelques exemples courts que vous pouvez copier-coller. Remplacez le texte entre guillemets par votre propre tâche. Consultez le [guide des prompts](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md) pour plus de conseils et de modèles d'utilisation.

| ✨  | Ce que vous tapez                                                                   | Ce qui se passe                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refactoriser le composant Dashboard en React Hooks"`                       | Codex réécrit le composant de classe, exécute `npm test`, et affiche la différence.   |
| 2   | `codex "Générer des migrations SQL pour ajouter une table utilisateurs"`                      | Déduit votre ORM, crée des fichiers de migration, et les exécute dans une DB sandboxée. |
| 3   | `codex "Écrire des tests unitaires pour utils/date.ts"`                                    | Génère des tests, les exécute, et itère jusqu'à ce qu'ils passent.              |
| 4   | `codex "Renommer en masse *.jpeg → *.jpg avec git mv"`                                | Renomme les fichiers en toute sécurité et met à jour les importations/usages.                           |
| 5   | `codex "Expliquer ce que fait cette regex : ^(?=.*[A-Z]).{8,}$"`                      | Fournit une explication étape par étape compréhensible par un humain.                                  |
| 6   | `codex "Examiner soigneusement ce dépôt, et proposer 3 PR bien définies à fort impact"` | Suggère des PR percutantes dans la base de code actuelle.                            |
| 7   | `codex "Rechercher des vulnérabilités et créer un rapport de revue de sécurité"`          | Trouve et explique les bugs de sécurité.                                          |

---

## Installation

<details open>
<summary><strong>Depuis npm (Recommandé)</strong></summary>

```bash
npm install -g @openai/codex
# ou
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>Construire à partir de la source</strong></summary>

```bash
# Cloner le dépôt et naviguer vers le paquet CLI
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# Installer les dépendances et construire
npm install
npm run build

# Obtenir l'utilisation et les options
node ./dist/cli.js --help

# Exécuter directement le CLI construit localement
node ./dist/cli.js

# Ou lier la commande globalement pour plus de commodité
npm link
```

</details>

---

## Configuration

Codex recherche les fichiers de configuration dans **`~/.codex/`**.

```yaml
# ~/.codex/config.yaml
model: o4-mini # Modèle par défaut
fullAutoErrorMode: ask-user # ou ignore-and-continue
```

Vous pouvez également définir des instructions personnalisées :

```yaml
# ~/.codex/instructions.md
- Toujours répondre avec des emojis
- N'utiliser les commandes git que si je le mentionne explicitement
```

---

## FAQ

<details>
<summary>OpenAI a publié un modèle appelé Codex en 2021 - est-ce lié ?</summary>

En 2021, OpenAI a publié Codex, un système IA conçu pour générer du code à partir de prompts en langage naturel. Ce modèle Codex original a été abandonné en mars 2023 et est distinct de l'outil CLI.

</details>

<details>
<summary>Comment empêcher Codex de toucher mon dépôt ?</summary>

Codex s'exécute toujours dans un **sandbox d'abord**. Si une commande ou une modification de fichier proposée semble suspecte, vous pouvez simplement répondre **n** lorsqu'on vous le demande, et rien ne se passe dans votre arbre de travail.

</details>

<details>
<summary>Est-ce que ça fonctionne sous Windows ?</summary>

Pas directement. Cela nécessite [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex a été testé sur macOS et Linux avec Node ≥ 22.

</details>

<details>
<summary>Quels modèles sont pris en charge ?</summary>

Tout modèle disponible avec [Responses API](https://platform.openai.com/docs/api-reference/responses). Le modèle par défaut est `o4-mini`, mais passez `--model gpt-4o` ou définissez `model: gpt-4o` dans votre fichier de configuration pour le remplacer.

</details>

---

## Opportunité de financement

Nous sommes ravis de lancer une **initiative de 1 million de dollars** pour soutenir les projets open-source utilisant Codex CLI et d'autres modèles OpenAI.

- Les subventions sont accordées par tranches de **25 000 $** de crédits API.
- Les candidatures sont examinées **en continu**.

**Intéressé ? [Postulez ici](https://openai.com/form/codex-open-source-fund/).**

---

## Contribuer

Ce projet est en cours de développement actif et le code risque de changer considérablement. Nous mettrons à jour ce message une fois cela terminé !

Plus largement, nous accueillons les contributions – que vous ouvriez votre toute première pull request ou que vous soyez un mainteneur expérimenté. En même temps, nous attachons de l'importance à la fiabilité et à la maintenabilité à long terme, donc le seuil pour fusionner le code est intentionnellement **élevé**. Les lignes directrices ci-dessous expliquent ce que signifie "haute qualité" en pratique et devraient rendre le processus transparent et convivial.

### Flux de travail de développement

- Créez une _branche thématique_ à partir de `main` – par exemple, `feat/interactive-prompt`.
- Gardez vos modifications ciblées. Plusieurs corrections non liées doivent être ouvertes en tant que PR séparées.
- Utilisez `npm run test:watch` pendant le développement pour un feedback ultra-rapide.
- Nous utilisons **Vitest** pour les tests unitaires, **ESLint** + **Prettier** pour le style, et **TypeScript** pour la vérification des types.
- Avant de pousser, exécutez la suite complète de tests/types/lint :

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- Si vous **n'avez pas encore** signé l'Accord de licence des contributeurs (CLA), ajoutez un commentaire à la PR contenant le texte exact :

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  Le bot CLA-Assistant passera le statut de la PR au vert une fois que tous les auteurs auront signé.

```bash
# Mode watch (les tests sont relancés à chaque changement)
npm run test:watch

# Vérification des types sans émettre de fichiers
npm run typecheck

# Corriger automatiquement les problèmes de lint + prettier
npm run lint:fix
npm run format:fix
```

### Rédiger des modifications de code à fort impact

1. **Commencez par un issue.** Ouvrez-en un nouveau ou commentez une discussion existante pour que nous puissions nous mettre d'accord sur la solution avant que le code ne soit écrit.
2. **Ajoutez ou mettez à jour les tests.** Chaque nouvelle fonctionnalité ou correction de bug doit être accompagnée d'une couverture de test qui échoue avant votre modification et passe après. Une couverture de 100 % n'est pas requise, mais visez des assertions significatives.
3. **Documentez le comportement.** Si votre modification affecte le comportement visible par l'utilisateur, mettez à jour le README, l'aide en ligne (`codex --help`), ou les projets d'exemple pertinents.
4. **Gardez les commits atomiques.** Chaque commit doit être compilable et les tests doivent passer. Cela facilite les révisions et les retours en arrière éventuels.

### Ouvrir une pull request

- Remplissez le modèle de PR (ou incluez des informations similaires) – **Quoi ? Pourquoi ? Comment ?**
- Exécutez **tous** les contrôles localement (`npm test && npm run lint && npm run typecheck`). Les échecs CI qui auraient pu être détectés localement ralentissent le processus.
- Assurez-vous que votre branche est à jour avec `main` et que les conflits de fusion sont résolus.
- Marquez la PR comme **Prête pour la révision** uniquement lorsque vous estimez qu'elle est dans un état fusionnable.

### Processus de révision

1. Un mainteneur sera assigné comme réviseur principal.
2. Nous pouvons demander des modifications – ne le prenez pas personnellement. Nous apprécions le travail, mais nous valorisons également la cohérence et la maintenabilité à long terme.
3. Lorsqu'il y a un consensus que la PR répond aux critères, un mainteneur effectuera un squash-and-merge.

### Valeurs de la communauté

- **Soyez gentil et inclusif.** Traitez les autres avec respect ; nous suivons le [Contributor Covenant](https://www.contributor-covenant.org/).
- **Présumez de bonnes intentions.** La communication écrite est difficile – penchez du côté de la générosité.
- **Enseignez et apprenez.** Si vous repérez quelque chose de confus, ouvrez un issue ou une PR avec des améliorations.

### Obtenir de l'aide

Si vous rencontrez des problèmes pour configurer le projet, souhaitez des retours sur une idée, ou voulez simplement dire _bonjour_ – ouvrez une Discussion ou plongez dans l'issue pertinente. Nous sommes heureux d'aider.

Ensemble, nous pouvons faire de Codex CLI un outil incroyable. **Bon hacking !** :rocket:

### Accord de licence des contributeurs (CLA)

Tous les contributeurs **doivent** accepter le CLA. Le processus est léger :

1. Ouvrez votre pull request.
2. Collez le commentaire suivant (ou répondez `recheck` si vous avez déjà signé) :

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. Le bot CLA-Assistant enregistre votre signature dans le dépôt et marque le contrôle de statut comme réussi.

Aucun commande Git spéciale, pièce jointe par email ou pied de page de commit requis.

#### Corrections rapides

| Scénario          | Commande                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Modifier le dernier commit | `git commit --amend -s --no-edit && git push -f`                                          |
| Uniquement via l'interface GitHub    | Modifiez le message du commit dans la PR → ajoutez<br>`Signed-off-by: Your Name <email@example.com>` |

Le contrôle **DCO** bloque les fusions tant que chaque commit dans la PR ne porte pas le pied de page (avec squash, il n'y en a qu'un).

### Publier `codex`

Pour publier une nouvelle version du CLI, exécutez les scripts de publication définis dans `codex-cli/package.json` :

1. Ouvrez le répertoire `codex-cli`
2. Assurez-vous d'être sur une branche comme `git checkout -b bump-version`
3. Augmentez la version et `CLI_VERSION` à la date/heure actuelle : `npm run release:version`
4. Validez l'augmentation de version (avec signature DCO) :
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. Copiez le README, construisez, et publiez sur npm : `npm run release`
6. Poussez vers la branche : `git push origin HEAD`

---

## Sécurité et IA responsable

Avez-vous découvert une vulnérabilité ou avez-vous des préoccupations concernant la sortie du modèle ? Veuillez envoyer un courriel à **security@openai.com** et nous répondrons rapidement.

---

## Licence

Ce dépôt est sous licence [Apache-2.0](LICENSE).