# GitHub Flow: trabajo colaborativo

## Introducción

GitHub Flow es un flujo de colaboración ligero basado en ramas. Cada cambio se desarrolla de forma aislada, se publica, se revisa mediante una *pull request* y se integra en la rama principal cuando está listo.

Este proceso puede aplicarse tanto a código como a documentación, configuraciones y otros archivos de texto. Su valor no está solamente en evitar errores: también permite saber quién propuso cada cambio, por qué se hizo, qué conversaciones produjo y en qué versión se publicó.

El ciclo general es:

1. Sincronizar la rama principal.
2. Crear una rama para un cambio concreto.
3. Editar, comprobar y realizar commits pequeños.
4. Publicar la rama en GitHub.
5. Abrir una pull request.
6. Revisar y mejorar la propuesta.
7. Resolver posibles conflictos.
8. Fusionar la pull request.
9. Actualizar la copia local y eliminar la rama terminada.
10. Cuando corresponda, etiquetar y publicar una versión.

## Conceptos fundamentales

| Concepto | Significado |
| --- | --- |
| **Repositorio upstream** | Repositorio original del que procede un fork. |
| **Fork** | Copia de un repositorio alojada en otra cuenta de GitHub y vinculada con el original. |
| **Clone** | Copia local de un repositorio para trabajar desde el equipo. |
| **Remote** | Nombre local asociado con la URL de otro repositorio Git. |
| **Branch** | Referencia móvil que permite desarrollar cambios sin modificar directamente la rama principal. |
| **Commit** | Instantánea identificada del contenido preparado. |
| **Pull request** | Propuesta para integrar los commits de una rama en otra. |
| **Merge** | Integración de historiales o cambios procedentes de ramas diferentes. |
| **Tag** | Nombre estable que señala un commit determinado. |
| **Release** | Publicación de GitHub asociada con una etiqueta, acompañada de notas y, opcionalmente, archivos descargables. |

## Fork, clone y remotos

### Diferencia entre fork y clone

Un **fork** existe en GitHub. Permite experimentar o contribuir sin modificar directamente el repositorio original. Un **clone** existe en el entorno local y contiene los archivos, ramas y el historial necesarios para usar Git desde la terminal.

Al crear un fork desde la interfaz de GitHub se puede elegir su propietario, descripción y, si es necesario, un nombre diferente. También se puede decidir si se copia únicamente la rama predeterminada o todas las ramas.

Para clonar el fork:

```bash
git clone https://github.com/<usuario>/<fork>.git
cd <fork>
```

Siempre conviene comprobar el directorio actual y el estado antes de continuar:

```bash
pwd
git status
```

### `origin` y `upstream`

Al clonar un repositorio, Git crea normalmente el remoto `origin`. Si se clonó el fork, `origin` apunta al fork y es el destino habitual de los pushes.

Para mantener la relación con el repositorio original se agrega un segundo remoto:

```bash
git remote add upstream https://github.com/<propietario-original>/<repositorio-original>.git
git remote -v
```

La configuración esperada es:

| Remoto | Apunta a | Uso habitual |
| --- | --- | --- |
| `origin` | Fork personal o fork de integración | Publicar ramas y actualizar la copia en GitHub. |
| `upstream` | Repositorio original | Recibir cambios de la fuente principal. |

Comandos de diagnóstico:

```bash
git remote get-url origin
git remote get-url upstream
git remote show origin
git remote show upstream
```

Agregar `upstream` no descarga sus cambios automáticamente. Tampoco modifica `origin`.

### Modelos de colaboración con forks

Antes de comenzar en pareja o equipo conviene acordar dónde se integrará el trabajo:

| Modelo | Funcionamiento | Consideraciones |
| --- | --- | --- |
| **Fork compartido** | Una persona crea el fork de integración y agrega a las demás como colaboradoras. Todas publican ramas en ese fork. | Simplifica las PR internas, pero requiere permisos de escritura y una única cuenta responsable del fork. |
| **Fork por persona** | Cada participante publica en su propio fork y abre PR hacia un fork o repositorio acordado como destino. | Se parece más a una contribución externa, pero exige revisar cuidadosamente repositorio y rama de origen y destino. |

En ambos modelos cada persona debe usar sus propias credenciales. Los tokens, contraseñas y claves SSH nunca se comparten.

## Sincronizar un fork

Un fork no recibe automáticamente todos los cambios futuros del repositorio original. Además, cuando una pull request se fusiona desde la web, el remoto cambia pero el clone local no se actualiza solo.

Una sincronización explícita permite distinguir las dos fuentes:

```bash
git fetch --prune origin
git fetch --prune upstream
git switch main
git pull --ff-only origin main
git merge upstream/main
git push origin main
```

La secuencia realiza lo siguiente:

1. Actualiza las referencias conocidas de `origin` y elimina referencias remotas obsoletas.
2. Actualiza las referencias conocidas de `upstream`.
3. Incorpora en el `main` local las fusiones realizadas previamente en el fork.
4. Integra los cambios nuevos del repositorio original.
5. Publica el resultado en el fork.

Si se sabe que `main` no tiene commits propios y solo debe avanzar en línea recta, se puede exigir un *fast-forward*:

```bash
git merge --ff-only upstream/main
```

Si falla, las historias no pueden integrarse mediante un simple avance. Antes de decidir entre merge o rebase (esta última se describe más adelante) se debe inspeccionar la situación:

```bash
git status
git log --oneline --graph --decorate --all -15
```

`reset --hard` descarta cambios locales de forma difícil de revertir, y `push --force` sobrescribe el historial en el remoto: ninguno de los dos debe usarse únicamente para silenciar el error. Primero se debe entender qué commits existen en cada rama y si alguno pertenece a otra persona.

## Ramas de funcionalidad

Internamente, una rama es un nombre que apunta a un commit y avanza con cada nuevo commit. Crear una rama es una operación rápida: no duplica todos los archivos del repositorio.

### Crear una rama desde un `main` actualizado

```bash
git switch main
git pull --ff-only origin main
git switch -c feature/short-description
```

El nombre debería comunicar la intención, por ejemplo:

- `feature/add-installation-guide`
- `docs/update-history`
- `fix/broken-link`

Cada conjunto de cambios no relacionados necesita una rama distinta. Para crear dos ramas hermanas, ambas deben partir de `main`, no una de la otra:

```bash
git switch main
git switch -c docs/first-update
git push -u origin docs/first-update

git switch main
git switch -c docs/second-update
git push -u origin docs/second-update
```

La opción `-u` establece la rama remota de seguimiento. Después, normalmente bastan `git push` y `git pull` mientras se permanezca en esa rama.

Para comprobar ramas locales, remotas y su seguimiento:

```bash
git branch
git branch -r
git branch -vv
git log --oneline --graph --decorate --all
```

## Cambios y commits enfocados

Antes de confirmar un cambio:

```bash
git status
git diff
git add <archivo-especifico>
git diff --staged
git commit -m "Describe the change clearly"
git push
```

Cuando solo debe cambiar un archivo, agregarlo por su nombre es más seguro que usar `git add .`. `git diff --staged` permite confirmar exactamente qué contenido entrará en el commit.

Un commit útil:

- Representa un cambio completo y coherente.
- No mezcla correcciones sin relación.
- Incluye únicamente los archivos necesarios.
- Usa un mensaje breve y descriptivo en modo imperativo.
- Deja el árbol de trabajo en un estado conocido.

Después de confirmar:

```bash
git show --stat --oneline HEAD
git status
```

Un árbol limpio significa que los archivos rastreados no contienen modificaciones pendientes. No significa que el commit ya esté en GitHub; para eso debe publicarse con `git push`.

## Pull requests

Una pull request no es solamente una solicitud de fusión. Es el espacio donde se presenta el cambio, se revisa el diff, se discuten alternativas y queda registrada la decisión.

### Rama base y rama de comparación

Al abrir una PR deben verificarse cuatro elementos:

| Elemento | Pregunta que responde |
| --- | --- |
| **Base repository** | ¿En qué repositorio se integrará el cambio? |
| **Base branch** | ¿En qué rama de ese repositorio terminará? |
| **Head repository** | ¿En qué repositorio está la propuesta? |
| **Compare branch** | ¿Qué rama contiene los commits propuestos? |

La dirección es siempre:

```text
compare/head → base
```

GitHub puede seleccionar automáticamente el repositorio original como base al trabajar con forks. Antes de crear la PR hay que comprobar el repositorio completo, no solamente que la rama base se llame `main`.

### Contenido de una buena PR

El título debe resumir el cambio y la descripción debe permitir entenderlo sin reconstruir toda la historia. Una plantilla sencilla es:

```markdown
## Summary

- Describe the relevant change.
- Explain why it is useful.

## Validation

- Describe how the result was checked.
```

También pueden agregarse enlaces, imágenes o referencias a incidencias cuando aporten contexto. Si el trabajo todavía no está listo, una **draft pull request** permite solicitar comentarios sin presentarlo como fusionable.

Los nuevos commits enviados a la rama de comparación aparecen automáticamente en la PR; no es necesario crear otra.

## Revisión colaborativa

Una revisión útil examina el contenido en la pestaña **Files changed** y ofrece observaciones concretas. GitHub permite:

- **Comment:** enviar comentarios sin aprobar ni bloquear la propuesta.
- **Approve:** indicar que el cambio está listo para integrarse.
- **Request changes:** solicitar modificaciones antes de aprobar.
- Agregar comentarios generales o vinculados con líneas específicas.

Una observación de calidad explica tanto lo que se ve como su impacto:

- Poco útil: `Looks good`.
- Más útil: `This paragraph makes the setup sequence clearer because it identifies the required remote before the first push.`

La persona autora no puede aprobar su propia pull request. En una práctica individual puede dejar comentarios y verificar el proceso, pero una aprobación real requiere otra cuenta. Además, **Request changes** solo bloquea una fusión cuando las reglas del repositorio exigen revisiones y la persona revisora tiene los permisos requeridos.

Después de recibir comentarios:

1. Responder o pedir aclaraciones cuando sea necesario.
2. Aplicar las mejoras en la misma rama.
3. Volver a comprobar el contenido.
4. Crear y publicar un commit adicional.
5. Solicitar una nueva revisión si el cambio fue importante.

No conviene reescribir continuamente commits que otras personas ya revisaron. Los commits adicionales hacen visible cómo evolucionó la propuesta; el historial puede simplificarse al fusionar mediante *squash* si el equipo lo prefiere.

## Fusionar y cerrar el ciclo

Una PR debe fusionarse cuando el resultado es correcto, las conversaciones están resueltas y se cumplen las comprobaciones o reglas del repositorio.

GitHub puede ofrecer tres métodos:

| Método | Resultado en la rama base |
| --- | --- |
| **Merge commit** | Conserva los commits y agrega un commit de fusión. |
| **Squash and merge** | Combina los commits de la PR en un único commit. |
| **Rebase and merge** | Reproduce los commits sobre la rama base sin un commit de fusión y genera hashes nuevos. |

El método debe seguir la convención del repositorio. No todos están habilitados en todos los proyectos.

Después de fusionar desde GitHub, se actualiza el entorno local:

```bash
git switch main
git pull --ff-only origin main
git branch -d <rama-fusionada>
git fetch --prune origin
```

Eliminar la rama no borra la pull request ni los commits ya integrados. `git branch -d` se detiene si Git no reconoce la rama como fusionada; `-D` fuerza la eliminación y puede ocultar trabajo no incorporado.

## Conflictos de merge

Un conflicto aparece cuando Git no puede decidir automáticamente cómo combinar cambios. Es frecuente cuando dos ramas modifican las mismas líneas o cuando una elimina un archivo que la otra editó.

La existencia de un conflicto no implica que una persona haya trabajado mal. Significa que la decisión necesita contexto humano.

### Incorporar la rama base de forma explícita

Para actualizar una rama con el estado remoto de `main` y observar claramente cada operación:

```bash
git switch <rama-de-trabajo>
git fetch origin
git merge origin/main
```

Esta forma separa la descarga de la integración. Evita la ambigüedad de `git pull` cuando no se ha configurado si las ramas divergentes deben fusionarse o reorganizarse mediante rebase.

Si Git detecta un conflicto:

```bash
git status
```

Los archivos afectados contienen marcadores similares a estos:

```text
<<<<<<< HEAD
contenido de la rama actual
=======
contenido procedente de la otra rama
>>>>>>> origin/main
```

Para resolverlo:

1. Leer ambas versiones y comprender la intención.
2. Redactar el contenido final correcto; puede ser una versión, la otra o una combinación nueva.
3. Eliminar todos los marcadores.
4. Revisar el archivo completo y ejecutar las validaciones disponibles.
5. Marcar el archivo como resuelto y confirmar la integración.

```bash
git add <archivo-resuelto>
git commit -m "Resolve merge conflict"
git push
```

Antes del commit puede comprobarse que no queden marcadores:

```bash
git diff --check
git status
```

Para cancelar una fusión que todavía no fue confirmada:

```bash
git merge --abort
```

Una vez publicada la resolución, GitHub actualiza la PR y vuelve a calcular si puede fusionarse.

## Etiquetas y releases

### Diferencia entre tag y release

Una **tag** es una referencia Git que señala un commit. Una **release** es un objeto de GitHub basado en una tag y pensado para comunicar o distribuir una versión.

| Elemento | Vive en Git | Puede incluir notas | Puede incluir archivos descargables |
| --- | ---: | ---: | ---: |
| Tag | Sí | Solo mensaje de etiqueta anotada | No |
| GitHub Release | Asociada con una tag | Sí | Sí |

Desde la terminal puede crearse y publicarse una etiqueta anotada:

```bash
git switch main
git pull --ff-only origin main
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

Desde la sección **Releases** de GitHub también se puede seleccionar una tag existente o crear una nueva. Cuando se crea allí, es imprescindible elegir como destino la rama o commit que contiene todo el contenido que se desea publicar.

Para comprobar localmente que una tag remota y `origin/main` apuntan al mismo commit:

```bash
git fetch origin --tags
git rev-parse v1.0.0^{commit}
git rev-parse origin/main
```

Los dos hashes deben coincidir si la versión se creó desde la punta actual de `main`.

Una versión ya publicada no debería modificarse moviendo su tag. Si hace falta corregirla, se crea una versión nueva.

### Notas de la release

Unas buenas notas responden:

- ¿Qué cambió desde la versión anterior?
- ¿Qué impacto tiene para quien usa el proyecto?
- ¿Existen incompatibilidades, pasos de migración o problemas conocidos?
- ¿Qué personas contribuyeron?

GitHub puede generar notas automáticamente a partir de PR y colaboradores, pero deben revisarse antes de publicar.

## Versionado Semántico

Semantic Versioning 2.0.0 usa el formato:

```text
MAJOR.MINOR.PATCH
```

| Componente | Se incrementa cuando… | Ejemplo |
| --- | --- | --- |
| `MAJOR` | Se introduce un cambio incompatible en la API pública. | `1.4.2 → 2.0.0` |
| `MINOR` | Se agrega funcionalidad compatible con versiones anteriores. | `1.4.2 → 1.5.0` |
| `PATCH` | Se corrige comportamiento de forma compatible. | `1.4.2 → 1.4.3` |

El prefijo `v` usado en tags como `v1.5.0` es una convención habitual, pero no forma parte del número SemVer propiamente dicho.

También existen:

- Preversiones: `2.0.0-alpha.1`, `2.0.0-rc.1`.
- Metadatos de compilación: `2.0.0+20260909`.

SemVer presupone que el software declara una **API pública**. Por eso, antes de clasificar un cambio se debe preguntar qué comportamiento público cambió, no cuántos archivos o commits hubo.

Las modificaciones exclusivas de documentación no cambian por sí mismas una API. Según la política del producto, pueden justificar un incremento `PATCH`, no generar una nueva versión del software o utilizar un esquema propio. Un producto documental también puede considerar contenido sustancial nuevo como `MINOR`, pero esa interpretación debe establecerse explícitamente en su política de versiones.

Después de publicar `1.0.0`:

- Nueva funcionalidad compatible: `1.1.0`.
- Corrección compatible: `1.0.1`.
- Cambio incompatible: `2.0.0`.

El número no se elige por cantidad de trabajo ni por importancia comercial.

## Mantener un changelog

Un changelog es una selección de cambios relevantes para las personas usuarias. No sustituye el historial de Git:

| Registro | Audiencia | Contenido |
| --- | --- | --- |
| Historial de commits | Personas que desarrollan y mantienen | Detalle técnico de cada cambio. |
| Pull requests | Equipo de colaboración | Propuesta, revisión y decisión. |
| `CHANGELOG.md` | Personas usuarias y mantenedoras | Cambios relevantes agrupados por versión. |
| Release notes | Personas que consumen una publicación | Resumen y consideraciones de una versión concreta. |

Una estructura habitual conserva una sección `Unreleased` para el trabajo que todavía no tiene versión:

```markdown
# Changelog

## Unreleased

### Added

- Describe new user-visible content or functionality.

### Fixed

- Describe a corrected behavior or problem.

## 1.1.0 - 2026-09-09

### Changed

- Describe the relevant changes included in this release.
```

Al preparar una versión:

1. Crear una rama para actualizar el changelog.
2. Mover las entradas correspondientes desde `Unreleased` a una sección con versión y fecha.
3. Revisar y fusionar el cambio.
4. Sincronizar `main` local.
5. Crear la tag sobre el commit que ya contiene el changelog final.
6. Crear la release a partir de esa tag.

Conviene conservar `Unreleased` vacío para los siguientes cambios en lugar de eliminarlo por completo.

Si un commit ya fue publicado en `main`, corregir el changelog con un commit nuevo es más seguro que usar `git commit --amend`. Enmendar un commit publicado reescribe su hash y normalmente exige un push forzado.

## Rutinas útiles de trabajo

### Antes de comenzar un cambio

```bash
git switch main
git fetch --prune origin
git pull --ff-only origin main
git status
git switch -c <rama-descriptiva>
```

Si se trabaja con un fork que sigue recibiendo cambios del repositorio original, también se actualiza `upstream` antes de crear la rama.

### Antes de publicar

```bash
git status
git diff
git diff --staged
git log --oneline --decorate -5
```

### Antes de fusionar una PR

- Confirmar los repositorios y ramas `base` y `compare`.
- Leer el diff completo.
- Resolver conversaciones pendientes.
- Revisar las comprobaciones automáticas.
- Verificar que la rama incluye el estado reciente de la base cuando sea necesario.
- Elegir conscientemente el método de fusión.

### Después de fusionar

```bash
git switch main
git pull --ff-only origin main
git branch -d <rama-terminada>
git fetch --prune origin
```

## Problemas frecuentes

| Situación | Causa probable | Acción recomendada |
| --- | --- | --- |
| `remote upstream already exists` | El remoto ya fue configurado. | Revisar `git remote -v`; corregir con `git remote set-url upstream <url>`. |
| Push enviado al repositorio equivocado | Se confundieron `origin` y `upstream`. | Comprobar `git remote get-url <nombre>` antes de publicar. |
| La segunda rama contiene commits de la primera | Fue creada desde otra rama de funcionalidad. | Crear ramas independientes desde un `main` actualizado. |
| La PR apunta al repositorio original | GitHub eligió automáticamente otra base. | Corregir **base repository** y **base branch** antes de crearla. |
| `non-fast-forward` al hacer push | El remoto contiene commits ausentes localmente. | Ejecutar `git fetch`, inspeccionar el gráfico y sincronizar; no forzar el envío. |
| `Need to specify how to reconcile divergent branches` | `git pull` encontró historias divergentes sin estrategia configurada. | Separar con `git fetch` y luego elegir conscientemente `git merge` o `git rebase`. |
| GitHub no permite aprobar la PR | La cuenta es autora de la propuesta. | Solicitar la revisión de otra persona; la autorrevisión solo permite comentar. |
| La PR no se puede fusionar | Hay conflictos, checks fallidos, revisión pendiente o reglas de rama. | Leer el estado mostrado por GitHub y resolver la causa concreta. |
| Una tag apunta al commit incorrecto | Fue creada antes de integrar o sincronizar todos los cambios. | No mover una versión publicada; corregir con una nueva versión. |
| El push exige `--force` después de amend | Se reescribió un commit ya publicado. | Preferir un commit correctivo; evitar reescribir `main`. |

## Prácticas recomendadas para el trabajo diario

- Proteger `main` y exigir pull requests cuando el repositorio sea compartido.
- Usar una rama por cambio independiente.
- Mantener las ramas cortas y sincronizarlas con frecuencia.
- Revisar archivos específicos y no solamente el resumen general de la PR.
- No compartir credenciales entre colaboradores.
- No confirmar tokens, contraseñas, claves o archivos `.env`.
- Evitar pushes forzados sobre ramas compartidas.
- No reutilizar una rama ya fusionada para un cambio diferente.
- Actualizar `main` local inmediatamente después de cada fusión web.
- Publicar tags únicamente desde estados comprobados y reproducibles.
- Documentar decisiones importantes en la PR y los cambios relevantes en el changelog.

## Recursos oficiales

### Lecturas principales

- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)
- [Hacer fork de un repositorio](https://docs.github.com/en/pull-requests/how-tos/work-with-forks/fork-a-repo)
- [Ramas en Git](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [Documentación de pull requests](https://docs.github.com/en/pull-requests)
- [Editar archivos desde GitHub](https://docs.github.com/en/repositories/working-with-files/managing-files/editing-files)
- [Fusionar y cerrar pull requests](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [Publicar proyectos en GitHub](https://docs.github.com/en/repositories/releasing-projects-on-github)

### Consultas directas recomendadas

- [Sincronizar un fork](https://docs.github.com/en/pull-requests/how-tos/work-with-forks/syncing-a-fork)
- [Crear una pull request](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/creating-a-pull-request)
- [Aprobar y revisar una pull request](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews)
- [Fusionar una pull request](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/merging-a-pull-request)
- [Resolver conflictos desde la terminal](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-using-the-command-line)
- [Administrar releases](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository)
- [Etiquetas en Git](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
- [Pro Git en español](https://git-scm.com/book/es/v2)

## Conclusión

GitHub Flow organiza el trabajo alrededor de cambios pequeños, ramas temporales y decisiones revisables. Un fork aporta independencia; los remotos mantienen conectadas las distintas copias; una pull request convierte un conjunto de commits en una conversación; y una tag fija el estado exacto que una release comunica.

La práctica más importante es comprobar siempre el estado antes de actuar: rama actual, archivos modificados, commits locales, remotos y dirección de la pull request. Esa disciplina reduce errores y permite colaborar sin depender de memorizar secuencias rígidas.
