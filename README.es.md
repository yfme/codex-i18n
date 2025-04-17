<h1 align="center">OpenAI Codex CLI</h1>
<p align="center">Agente de programación ligero que se ejecuta en tu terminal</p>

<p align="center"><code>npm i -g @openai/codex</code></p>

[English](./README.md) | [简体中文](./README.zh-CN.md) | **Español** | [Português](./README.pt.md) | [Русский](./README.ru.md) | [Français](./README.fr.md) | [Deutsch](./README.de.md) | [日本語](./README.ja.md) | [한국어](./README.ko.md) | [繁體中文](./README.zh-TW.md) | [Bahasa Indonesia](./README.id.md) | [Italiano](./README.it.md)

![Codex demo GIF usando: codex "explain this codebase to me"](./.github/demo.gif)

---

<details>
<summary><strong>Tabla de Contenidos</strong></summary>

- [Descargo de responsabilidad sobre tecnología experimental](#descargo-de-responsabilidad-sobre-tecnología-experimental)
- [Inicio rápido](#inicio-rápido)
- [¿Por qué Codex?](#por-qué-codex)
- [Modelo de seguridad y permisos](#modelo-de-seguridad-y-permisos)
  - [Detalles del sandboxing de la plataforma](#detalles-del-sandboxing-de-la-plataforma)
- [Requisitos del sistema](#requisitos-del-sistema)
- [Referencia de CLI](#referencia-de-cli)
- [Memoria y documentos del proyecto](#memoria-y-documentos-del-proyecto)
- [Modo no interactivo / CI](#modo-no-interactivo--ci)
- [Recetas](#recetas)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Preguntas frecuentes](#preguntas-frecuentes)
- [Oportunidad de financiación](#oportunidad-de-financiación)
- [Contribuir](#contribuir)
  - [Flujo de trabajo de desarrollo](#flujo-de-trabajo-de-desarrollo)
  - [Escribir cambios de código de alto impacto](#escribir-cambios-de-código-de-alto-impacto)
  - [Abrir una pull request](#abrir-una-pull-request)
  - [Proceso de revisión](#proceso-de-revisión)
  - [Valores de la comunidad](#valores-de-la-comunidad)
  - [Obtener ayuda](#obtener-ayuda)
  - [Acuerdo de Licencia de Contribuyentes (CLA)](#acuerdo-de-licencia-de-contribuyentes-cla)
    - [Correcciones rápidas](#correcciones-rápidas)
  - [Lanzamiento de `codex`](#lanzamiento-de-codex)
- [Seguridad e IA responsable](#seguridad-e-ia-responsable)
- [Licencia](#licencia)

</details>

---

## Descargo de responsabilidad sobre tecnología experimental

Codex CLI es un proyecto experimental en desarrollo activo. Aún no es estable, puede contener errores, funcionalidades incompletas o sufrir cambios importantes. Lo estamos construyendo de forma abierta con la comunidad y damos la bienvenida a:

- Informes de errores
- Solicitudes de funcionalidades
- Pull requests
- Buen ambiente

¡Ayúdanos a mejorar reportando problemas o enviando PRs (consulta la sección a continuación para saber cómo contribuir)!

## Inicio rápido

Instala globalmente:

```shell
npm install -g @openai/codex
```

A continuación, configura tu clave API de OpenAI como una variable de entorno:

```shell
export OPENAI_API_KEY="your-api-key-here"
```

> **Nota:** Este comando configura la clave solo para la sesión actual del terminal. Para hacerla permanente, agrega la línea `export` al archivo de configuración de tu shell (por ejemplo, `~/.zshrc`).

Ejecuta de forma interactiva:

```shell
codex
```

O ejecuta con un prompt como entrada (y opcionalmente en modo `Full Auto`):

```shell
codex "explícame este código"
```

```shell
codex --approval-mode full-auto "crea la aplicación de lista de tareas más sofisticada"
```

Eso es todo – Codex generará un archivo, lo ejecutará dentro de un sandbox, instalará cualquier dependencia faltante y te mostrará el resultado en tiempo real. Aprueba los cambios y se confirmarán en tu directorio de trabajo.

---

## ¿Por qué Codex?

Codex CLI está diseñado para desarrolladores que ya **viven en el terminal** y desean un razonamiento al nivel de ChatGPT **más** el poder de ejecutar código, manipular archivos e iterar – todo bajo control de versiones. En resumen, es un _desarrollo guiado por chat_ que comprende y ejecuta tu repositorio.

- **Sin configuración** — trae tu clave API de OpenAI y funciona de inmediato.
- **Aprobación automática completa, pero segura** al ejecutarse sin red y en un directorio sandboxed
- **Multimodal** — pasa capturas de pantalla o diagramas para implementar funcionalidades ✨

Y es **completamente de código abierto**, por lo que puedes ver y contribuir a su desarrollo.

---

## Modelo de seguridad y permisos

Codex te permite decidir _cuánta autonomía_ recibe el agente y la política de aprobación automática a través del flag `--approval-mode` (o el prompt interactivo de incorporación):

| Modo                      | Qué puede hacer el agente sin preguntar            | Todavía requiere aprobación                                         |
| ------------------------- | ----------------------------------------------- | --------------------------------------------------------------- |
| **Sugerir** <br>(predeterminado) | • Leer cualquier archivo en el repositorio                     | • **Todas** las escrituras/patches de archivos <br>• **Todos** los comandos shell/Bash |
| **Edición automática**             | • Leer **y** aplicar escrituras de parches a archivos      | • **Todos** los comandos shell/Bash                                   |
| **Automático completo**             | • Leer/escribir archivos <br>• Ejecutar comandos shell | —                                                               |

En modo **Automático completo**, cada comando se ejecuta con la **red desactivada** y confinado al directorio de trabajo actual (más archivos temporales) para una defensa en profundidad. Codex también mostrará una advertencia/confirmación si inicias en modo **edición automática** o **automático completo** mientras el directorio _no_ está rastreado por Git, asegurándote siempre una red de seguridad.

Próximamente: podrás incluir en una lista blanca comandos específicos para ejecutarse automáticamente con la red habilitada, una vez que estemos seguros de medidas de seguridad adicionales.

### Detalles del sandboxing de la plataforma

El mecanismo de endurecimiento que usa Codex depende de tu sistema operativo:

- **macOS 12+** – los comandos están envueltos con **Apple Seatbelt** (`sandbox-exec`).

  - Todo se coloca en una cárcel de solo lectura, excepto un pequeño conjunto de raíces escribibles (`$PWD`, `$TMPDIR`, `~/.codex`, etc.).
  - La red saliente está _completamente bloqueada_ por defecto – incluso si un proceso hijo intenta hacer `curl` a algún lugar, fallará.

- **Linux** – recomendamos usar Docker para el sandboxing, donde Codex se lanza a sí mismo dentro de una **imagen de contenedor mínima** y monta tu repositorio en _lectura/escritura_ en la misma ruta. Un script de firewall personalizado `iptables`/`ipset` deniega todo el tráfico saliente excepto la API de OpenAI. Esto te proporciona ejecuciones deterministas y reproducibles sin necesidad de root en el host. Puedes leer más en [`run_in_container.sh`](./codex-cli/scripts/run_in_container.sh).

Ambos enfoques son _transparentes_ para el uso diario – sigues ejecutando `codex` desde la raíz de tu repositorio y apruebas/rechazas pasos como de costumbre.

---

## Requisitos del sistema

| Requisito                 | Detalles                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Sistemas operativos           | macOS 12+, Ubuntu 20.04+/Debian 10+, o Windows 11 **vía WSL2** |
| Node.js                     | **22 o más reciente** (LTS recomendado)                               |
| Git (opcional, recomendado) | 2.23+ para ayudantes de PR integrados                                   |
| RAM                         | Mínimo 4 GB (8 GB recomendado)                                 |

> Nunca ejecutes `sudo npm install -g`; en su lugar, corrige los permisos de npm.

---

## Referencia de CLI

| Comando                              | Propósito                             | Ejemplo                              |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `codex`                              | REPL interactivo                    | `codex`                              |
| `codex "…"`                          | Prompt inicial para REPL interactivo | `codex "corregir errores de lint"`            |
| `codex -q "…"`                       | Modo no interactivo "silencioso"        | `codex -q --json "explicar utils.ts"` |
| `codex completion <bash\|zsh\|fish>` | Imprimir script de completado de shell       | `codex completion bash`              |

Banderas clave: `--model/-m`, `--approval-mode/-a`, y `--quiet/-q`.

---

## Memoria y documentos del proyecto

Codex combina las instrucciones de Markdown en este orden:

1. `~/.codex/instructions.md` – guía global personal
2. `codex.md` en la raíz del repositorio – notas compartidas del proyecto
3. `codex.md` en el directorio de trabajo actual – especificaciones de subpaquetes

Desactiva con `--no-project-doc` o `CODEX_DISABLE_PROJECT_DOC=1`.

---

## Modo no interactivo / CI

Ejecuta Codex sin interfaz en pipelines. Ejemplo de paso de GitHub Action:

```yaml
- name: Actualizar changelog vía Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex -a auto-edit --quiet "actualizar CHANGELOG para la próxima versión"
```

Configura `CODEX_QUIET_MODE=1` para silenciar el ruido de la interfaz interactiva.

---

## Recetas

A continuación, algunos ejemplos cortos que puedes copiar y pegar. Reemplaza el texto entre comillas con tu propia tarea. Consulta la [guía de prompts](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md) para más consejos y patrones de uso.

| ✨  | Qué escribes                                                                   | Qué pasa                                                               |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refactorizar el componente Dashboard a React Hooks"`                       | Codex reescribe el componente de clase, ejecuta `npm test` y muestra la diferencia.   |
| 2   | `codex "Generar migraciones SQL para añadir una tabla de usuarios"`                      | Deduce tu ORM, crea archivos de migración y los ejecuta en una DB sandboxed. |
| 3   | `codex "Escribir pruebas unitarias para utils/date.ts"`                                    | Genera pruebas, las ejecuta e itera hasta que pasen.              |
| 4   | `codex "Renombrar en masa *.jpeg → *.jpg con git mv"`                                | Renombra archivos de forma segura y actualiza importaciones/usos.                           |
| 5   | `codex "Explicar qué hace esta regex: ^(?=.*[A-Z]).{8,}$"`                      | Proporciona una explicación paso a paso comprensible para humanos.                                  |
| 6   | `codex "Revisar cuidadosamente este repositorio y proponer 3 PRs bien definidas de alto impacto"` | Sugiere PRs impactantes en la base de código actual.                            |
| 7   | `codex "Buscar vulnerabilidades y crear un informe de revisión de seguridad"`          | Encuentra y explica errores de seguridad.                                          |

---

## Instalación

<details open>
<summary><strong>Desde npm (Recomendado)</strong></summary>

```bash
npm install -g @openai/codex
# o
yarn global add @openai/codex
```

</details>

<details>
<summary><strong>Construir desde el código fuente</strong></summary>

```bash
# Clonar el repositorio y navegar al paquete CLI
git clone https://github.com/openai/codex.git
cd codex/codex-cli

# Instalar dependencias y construir
npm install
npm run build

# Obtener el uso y las opciones
node ./dist/cli.js --help

# Ejecutar directamente el CLI construido localmente
node ./dist/cli.js

# O enlazar el comando globalmente para mayor comodidad
npm link
```

</details>

---

## Configuración

Codex busca archivos de configuración en **`~/.codex/`**.

```yaml
# ~/.codex/config.yaml
model: o4-mini # Modelo predeterminado
fullAutoErrorMode: ask-user # o ignore-and-continue
```

También puedes definir instrucciones personalizadas:

```yaml
# ~/.codex/instructions.md
- Siempre responde con emojis
- Usa comandos git solo si lo menciono explícitamente
```

---

## Preguntas frecuentes

<details>
<summary>OpenAI lanzó un modelo llamado Codex en 2021 - ¿está relacionado?</summary>

En 2021, OpenAI lanzó Codex, un sistema de IA diseñado para generar código a partir de prompts en lenguaje natural. Ese modelo Codex original fue descontinuado en marzo de 2023 y es independiente de la herramienta CLI.

</details>

<details>
<summary>¿Cómo evito que Codex toque mi repositorio?</summary>

Codex siempre se ejecuta primero en un **sandbox**. Si un comando o cambio de archivo propuesto parece sospechoso, simplemente puedes responder **n** cuando se te solicite y no pasará nada en tu árbol de trabajo.

</details>

<details>
<summary>¿Funciona en Windows?</summary>

No directamente. Requiere [Windows Subsystem for Linux (WSL2)](https://learn.microsoft.com/en-us/windows/wsl/install) – Codex ha sido probado en macOS y Linux con Node ≥ 22.

</details>

<details>
<summary>¿Qué modelos son compatibles?</summary>

Cualquier modelo disponible con [Responses API](https://platform.openai.com/docs/api-reference/responses). El predeterminado es `o4-mini`, pero pasa `--model gpt-4o` o configura `model: gpt-4o` en tu archivo de configuración para sobrescribirlo.

</details>

---

## Oportunidad de financiación

Estamos emocionados de lanzar una **iniciativa de 1 millón de dólares** para apoyar proyectos de código abierto que usen Codex CLI y otros modelos de OpenAI.

- Las subvenciones se otorgan en incrementos de **25,000 $** en créditos de API.
- Las solicitudes se revisan **de forma continua**.

**¿Interesado? [Solicita aquí](https://openai.com/form/codex-open-source-fund/).**

---

## Contribuir

Este proyecto está en desarrollo activo y el código probablemente cambiará significativamente. Actualizaremos este mensaje una vez que esté completo.

En general, damos la bienvenida a las contribuciones – ya sea que estés abriendo tu primera pull request o seas un mantenedor experimentado. Al mismo tiempo, nos preocupamos por la fiabilidad y la mantenibilidad a largo plazo, por lo que el umbral para fusionar código es intencionalmente **alto**. Las directrices a continuación explican qué significa "alta calidad" en la práctica y deberían hacer que el proceso sea transparente y amigable.

### Flujo de trabajo de desarrollo

- Crea una _rama temática_ desde `main` – por ejemplo, `feat/interactive-prompt`.
- Mantén tus cambios enfocados. Múltiples correcciones no relacionadas deben abrirse como PRs separadas.
- Usa `npm run test:watch` durante el desarrollo para obtener retroalimentación súper rápida.
- Usamos **Vitest** para pruebas unitarias, **ESLint** + **Prettier** para estilo, y **TypeScript** para verificación de tipos.
- Antes de hacer push, ejecuta la suite completa de pruebas/tipos/lint:

  ```bash
  npm test && npm run lint && npm run typecheck
  ```

- Si **aún no has** firmado el Acuerdo de Licencia de Contribuyentes (CLA), agrega un comentario a la PR con el texto exacto:

  ```text
  I have read the CLA Document and I hereby sign the CLA
  ```

  El bot CLA-Assistant cambiará el estado de la PR a verde una vez que todos los autores hayan firmado.

```bash
# Modo watch (las pruebas se ejecutan al cambiar)
npm run test:watch

# Verificación de tipos sin emitir archivos
npm run typecheck

# Corregir automáticamente problemas de lint + prettier
npm run lint:fix
npm run format:fix
```

### Escribir cambios de código de alto impacto

1. **Comienza con un issue.** Abre uno nuevo o comenta en una discusión existente para que podamos acordar la solución antes de que se escriba el código.
2. **Agrega o actualiza pruebas.** Cada nueva funcionalidad o corrección de errores debe venir con una cobertura de pruebas que falle antes de tu cambio y pase después. No se requiere una cobertura del 100 %, pero apunta a aserciones significativas.
3. **Documenta el comportamiento.** Si tu cambio afecta el comportamiento visible para el usuario, actualiza el README, la ayuda en línea (`codex --help`), o los proyectos de ejemplo relevantes.
4. **Mantén los commits atómicos.** Cada commit debe compilarse y las pruebas deben pasar. Esto facilita las revisiones y posibles reversiones.

### Abrir una pull request

- Completa la plantilla de PR (o incluye información similar) – **¿Qué? ¿Por qué? ¿Cómo?**
- Ejecuta **todos** los controles localmente (`npm test && npm run lint && npm run typecheck`). Los fallos de CI que podrían haberse detectado localmente ralentizan el proceso.
- Asegúrate de que tu rama esté actualizada con `main` y que los conflictos de fusión estén resueltos.
- Marca la PR como **Lista para revisión** solo cuando creas que está en un estado fusionable.

### Proceso de revisión

1. Un mantenedor será asignado como revisor principal.
2. Podemos pedir cambios – por favor, no lo tomes como algo personal. Valoramos el trabajo, pero también valoramos la consistencia y la mantenibilidad a largo plazo.
3. Cuando haya consenso de que la PR cumple con los estándares, un mantenedor hará un squash-and-merge.

### Valores de la comunidad

- **Sé amable e inclusivo.** Trata a los demás con respeto; seguimos el [Contributor Covenant](https://www.contributor-covenant.org/).
- **Asume buena intención.** La comunicación escrita es difícil – inclínate por la generosidad.
- **Enseña y aprende.** Si encuentras algo confuso, abre un issue o una PR con mejoras.

### Obtener ayuda

Si encuentras problemas al configurar el proyecto, quieres retroalimentación sobre una idea, o simplemente quieres decir _hola_ – por favor, abre una Discusión o participa en el issue relevante. Estamos encantados de ayudar.

Juntos podemos hacer de Codex CLI una herramienta increíble. **¡Feliz hacking!** :rocket:

### Acuerdo de Licencia de Contribuyentes (CLA)

Todos los contribuyentes **deben** aceptar el CLA. El proceso es ligero:

1. Abre tu pull request.
2. Pega el siguiente comentario (o responde `recheck` si ya has firmado antes):

   ```text
   I have read the CLA Document and I hereby sign the CLA
   ```

3. El bot CLA-Assistant registra tu firma en el repositorio y marca el control de estado como aprobado.

No se requieren comandos Git especiales, archivos adjuntos por correo electrónico ni pies de página de commit.

#### Correcciones rápidas

| Escenario          | Comando                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Modificar el último commit | `git commit --amend -s --no-edit && git push -f`                                          |
| Solo interfaz de GitHub    | Edita el mensaje del commit en la PR → agrega<br>`Signed-off-by: Your Name <email@example.com>` |

El control **DCO** bloquea las fusiones hasta que cada commit en la PR lleve el pie de página (con squash, esto es solo uno).

### Lanzamiento de `codex`

Para publicar una nueva versión del CLI, ejecuta los scripts de lanzamiento definidos en `codex-cli/package.json`:

1. Abre el directorio `codex-cli`
2. Asegúrate de estar en una rama como `git checkout -b bump-version`
3. Aumenta la versión y `CLI_VERSION` a la fecha/hora actual: `npm run release:version`
4. Confirma el aumento de versión (con firma DCO):
   ```bash
   git add codex-cli/src/utils/session.ts codex-cli/package.json
   git commit -s -m "chore(release): codex-cli v$(node -p \"require('./codex-cli/package.json').version\")"
   ```
5. Copia el README, construye y publica en npm: `npm run release`
6. Haz push a la rama: `git push origin HEAD`

---

## Seguridad e IA responsable

¿Has descubierto una vulnerabilidad o tienes preocupaciones sobre la salida del modelo? Por favor, envía un correo a **security@openai.com** y responderemos rápidamente.

---

## Licencia

Este repositorio está licenciado bajo la [Licencia Apache-2.0](LICENSE).