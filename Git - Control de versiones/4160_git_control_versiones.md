# Git y GitHub: control de versiones

## Introducción

Git es un sistema de control de versiones distribuido: registra cambios en archivos, permite recuperar estados anteriores y facilita que varias personas trabajen sobre un mismo proyecto. GitHub es un servicio de alojamiento y colaboración que agrega repositorios remotos, *pull requests*, revisiones y otras herramientas alrededor de Git.

Aunque suelen utilizarse juntos, no son lo mismo:

| Herramienta | Función principal |
| --- | --- |
| **Git** | Administra versiones y ramas en el equipo local. |
| **GitHub** | Aloja una copia remota y facilita la colaboración. |

El recorrido habitual de un cambio es:

```text
Directorio de trabajo → Área de preparación → Repositorio local → Repositorio remoto
       editar              git add             git commit          git push
```

> Los comandos de este documento se ejecutan desde una terminal. Sustituye los valores entre `< >` por los datos reales y no escribas los símbolos `<` y `>`.

## 1. Configuración inicial

### Comprobar Git

```bash
git --version
```

### Configurar la identidad

Git guarda un autor y un correo en cada *commit*. Se configuran una sola vez para el usuario del equipo:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@example.com"
```

Para comprobarlos:

```bash
git config --global user.name
git config --global user.email
```

Conviene usar un correo asociado con GitHub o una dirección `noreply` proporcionada por GitHub si se desea ocultar el correo personal.

### Crear un repositorio local

```bash
mkdir -p /root/git-intro
cd /root/git-intro
git init
git status
```

`git init` crea el directorio interno `.git`, donde se almacena el historial y la configuración local. No se debe editar su contenido manualmente.

### Conectar un repositorio remoto

Después de crear en GitHub un repositorio vacío —sin README, licencia ni `.gitignore` iniciales— se agrega su URL:

```bash
git remote add origin https://github.com/<usuario>/<repositorio>.git
git remote -v
```

`origin` es el nombre convencional del remoto principal; no es una palabra reservada. Si se agregó una URL incorrecta, puede corregirse con:

```bash
git remote set-url origin https://github.com/<usuario>/<repositorio>.git
```

## 2. Estados de los archivos y primeros commits

Un archivo puede atravesar estos estados:

| Estado | Significado |
| --- | --- |
| **Untracked** | Git todavía no registra el archivo. |
| **Modified** | El archivo rastreado cambió desde el último commit. |
| **Staged** | La versión actual está preparada para el próximo commit. |
| **Committed** | La versión está guardada en el historial local. |

Un primer cambio puede registrarse así:

```bash
printf '# Introducción a Git\n' > README.md
git status
git add README.md
git commit -m "Add project README"
git log --oneline
```

`git add` prepara una **instantánea** del archivo. Si el archivo se vuelve a modificar después, esas nuevas modificaciones no entran en el commit hasta ejecutar `git add` otra vez.

Antes de confirmar, resulta útil inspeccionar los cambios:

```bash
git diff
git diff --staged
```

- `git diff` muestra cambios todavía no preparados.
- `git diff --staged` muestra exactamente qué se incluirá en el próximo commit.

Un buen commit es pequeño, coherente y tiene un mensaje que explica la intención. Por ejemplo, `Add greeting script` es más informativo que `changes`.

## 3. Ignorar archivos con `.gitignore`

`.gitignore` contiene patrones de archivos que Git no debe comenzar a rastrear. Para ignorar todos los archivos con extensión `.log`:

```gitignore
*.log
```

Después se registra la propia configuración:

```bash
git add .gitignore
git commit -m "Ignore log files"
touch test.log
git status
```

`test.log` no debería aparecer como archivo sin seguimiento. Puede verificarse qué regla lo ignora con:

```bash
git check-ignore -v test.log
```

`.gitignore` no deja de rastrear un archivo que ya fue confirmado. En ese caso se retira únicamente del índice y luego se confirma el cambio:

```bash
git rm --cached archivo.log
git commit -m "Stop tracking generated log"
```

Nunca deben confirmarse contraseñas, tokens, claves privadas o archivos `.env`. Agregarlos luego a `.gitignore` no elimina el secreto del historial; si ocurre, se debe revocar o rotar inmediatamente.

## 4. Ramas, remotos y autenticación

### Usar `main` como rama principal

```bash
git branch -M main
```

La opción `-M` renombra la rama actual, incluso si ya existe una referencia con ese nombre.

### Autenticarse con un token

GitHub no acepta la contraseña de la cuenta para operaciones Git por HTTPS. Cuando Git solicite credenciales:

- En **Username**, se escribe el nombre de usuario de GitHub.
- En **Password**, se pega el *personal access token* (PAT), no la contraseña.

GitHub recomienda utilizar un token **fine-grained** cuando sea compatible, restringido al repositorio necesario y con el permiso mínimo. Para este ejercicio normalmente basta acceso de lectura y escritura a **Contents**. Si la consigna exige un token clásico con alcance `repo`, conviene asignarle una caducidad corta y revocarlo al terminar, porque ese alcance puede dar acceso amplio a repositorios.

Reglas de seguridad:

- No incluir el token en un comando, archivo, captura de pantalla o URL remota.
- No confirmarlo en Git.
- No compartirlo con otras personas.
- Revocarlo de inmediato si queda expuesto.

En un entorno temporal puede guardarse en memoria durante dos horas:

```bash
git config --global credential.helper 'cache --timeout=7200'
```

La caché se pierde al finalizar su proceso o reiniciar el entorno. Para vaciarla antes del tiempo configurado:

```bash
git credential-cache exit
```

### Enviar y recibir cambios

El primer envío vincula `main` local con `origin/main`:

```bash
git push -u origin main
```

Gracias a `-u`, los siguientes envíos pueden hacerse con `git push`. Para incorporar cambios remotos:

```bash
git pull --ff-only origin main
```

`git pull` ejecuta primero una descarga (*fetch*) y luego integra los cambios. `--ff-only` es una opción prudente al sincronizar una rama principal en la que no deberían existir commits locales independientes: actualiza únicamente si puede avanzar en línea recta y se detiene si las historias divergieron.

Para inspeccionar cambios remotos antes de integrarlos:

```bash
git fetch origin
git log --oneline --graph --decorate --all
```

## 5. Trabajar con ramas

Una rama permite aislar un cambio sin alterar `main` mientras se desarrolla.

Una forma habitual de crear y cambiar a la vez es usar `checkout`:

```bash
git checkout -b feature-greeting
```

La alternativa moderna y más explícita es:

```bash
git switch -c feature-greeting
```

Después se edita, prepara y confirma el trabajo:

```bash
git add greeting.txt
git commit -m "Add greeting message"
```

Para regresar a la rama principal:

```bash
git switch main
```

Comandos útiles para orientarse:

```bash
git branch
git status
git log --oneline --graph --decorate --all
```

Cada cambio independiente debe comenzar en una rama nueva creada desde un `main` actualizado. Reutilizar para otra tarea una rama ya fusionada dificulta revisar el historial y contradice el flujo recomendado por GitHub.

## 6. Fusionar ramas y resolver conflictos

Para incorporar una rama de trabajo a `main` desde la terminal:

```bash
git switch main
git merge feature-greeting
```

Git puede hacer una fusión automática. Si las dos ramas modificaron de forma incompatible las mismas líneas, detiene el proceso y marca el archivo:

```text
<<<<<<< HEAD
contenido de la rama actual
=======
contenido de la rama que se intenta fusionar
>>>>>>> feature-greeting
```

Para resolverlo:

1. Ejecuta `git status` para identificar los archivos afectados.
2. Abre cada archivo y decide qué contenido debe conservarse.
3. Elimina las tres líneas de marcadores (`<<<<<<<`, `=======`, `>>>>>>>`).
4. Comprueba que el archivo final sea válido.
5. Prepara y confirma la resolución.

```bash
git add message.txt
git commit -m "Resolve message merge conflict"
git log --oneline --graph --decorate --all
```

Si todavía no se desea resolver y no se ha confirmado la fusión, puede cancelarse:

```bash
git merge --abort
```

No se debe elegir automáticamente “la versión local” o “la remota”: la solución correcta puede combinar ambas y depende del resultado que se busca.

## 7. Etiquetas y formas de deshacer cambios

Una etiqueta asigna un nombre estable a un commit. Para marcar una versión:

```bash
git tag -a v1 -m "Stable version"
git show v1
```

Las operaciones para deshacer no son equivalentes:

| Necesidad | Comando recomendado | Efecto |
| --- | --- | --- |
| Descartar cambios no preparados de un archivo | `git restore archivo` | Restaura el archivo desde el índice. |
| Quitar un archivo del área de preparación | `git restore --staged archivo` | Conserva el contenido, pero lo saca del próximo commit. |
| Deshacer un commit ya compartido | `git revert <hash>` | Crea un commit nuevo con el cambio inverso. |
| Mover una rama local a otro commit | `git reset <commit>` | Reescribe la posición de la rama. |
| Mover la rama y descartar índice y archivos | `git reset --hard <commit>` | Elimina cambios locales no guardados. |

### `git revert`: opción segura para historial compartido

```bash
git log --oneline
git revert <hash-del-commit>
```

El commit original sigue en el historial y aparece un nuevo commit que lo revierte. Por eso es apropiado para cambios que ya llegaron a GitHub.

Si el objetivo es revertir un **commit de fusión** desde la terminal, Git necesita saber qué padre representa la línea principal; habitualmente sería:

```bash
git revert -m 1 <hash-del-merge>
```

No se debe usar `-m 1` a ciegas: primero hay que confirmar que el hash corresponde realmente a un merge y cuál es su padre principal. Para una pull request fusionada, GitHub también puede crear una nueva PR de reversión desde su interfaz.

### `git reset`: solo cuando reescribir la historia sea aceptable

Antes de un reinicio destructivo:

```bash
git status
git log --oneline --decorate -5
git show v1 --stat
```

Después, si se comprobó el objetivo y el ejercicio exige descartar el trabajo local:

```bash
git reset --hard v1
```

`--hard` borra del directorio de trabajo y del índice los cambios rastreados posteriores al destino. No debe usarse para reescribir una rama compartida ni cuando existan cambios locales que se quieran conservar.

> Si se practican ambas estrategias en secuencia (`revert` y luego `reset --hard` hacia un commit anterior), conviene notar que son dos formas distintas de lograr algo similar, no complementarias: al hacer `reset` después de un `revert`, el commit de reversión deja de ser visible desde la rama actual. En un caso real normalmente se elegiría una de las dos según si el historial ya fue compartido.

## 8. GitHub Flow y pull requests

GitHub Flow es un proceso ligero basado en ramas:

1. Actualizar `main`.
2. Crear una rama descriptiva para un solo cambio.
3. Realizar commits pequeños y claros.
4. Publicar la rama en GitHub.
5. Abrir una *pull request* (PR) con contexto y pruebas realizadas.
6. Revisar, conversar y agregar nuevos commits si es necesario.
7. Resolver conflictos y fusionar la PR.
8. Sincronizar `main` local y eliminar la rama terminada.

Ejemplo:

```bash
git switch main
git pull --ff-only origin main
git switch -c feature-improvement

# editar y comprobar el trabajo
git add <archivos>
git commit -m "Improve project message"
git push -u origin feature-improvement
```

Después de fusionar y eliminar la rama remota en GitHub:

```bash
git switch main
git pull --ff-only origin main
git branch -d feature-improvement
```

Si `git branch -d` se niega a eliminar la rama, primero debe comprobarse si el trabajo fue fusionado. `-D` fuerza la eliminación local y puede ocultar commits no integrados.

### Qué debe contener una buena PR

- Un título preciso.
- Un resumen del problema y la solución.
- Los pasos empleados para comprobar el resultado.
- Riesgos, limitaciones o tareas pendientes.
- Capturas solo cuando ayuden a revisar un cambio visual.

### Métodos de fusión

| Método | Resultado |
| --- | --- |
| **Merge commit** | Conserva los commits de la rama y agrega un commit de fusión. |
| **Squash and merge** | Combina los commits de la rama en un único commit sobre `main`. |
| **Rebase and merge** | Reproduce los commits sobre `main` sin crear un commit de fusión. |

La disponibilidad depende de la configuración del repositorio. Para principiantes, lo importante es comprender qué historial produce cada método y seguir la convención del equipo.

## 9. Errores comunes al practicar el flujo completo de GitHub

### Olvidar sincronizar después de una fusión en GitHub

Cuando una PR se fusiona desde el sitio web, el `main` remoto avanza, pero el `main` local no se actualiza solo. Antes de hacer un nuevo commit sobre `main` o buscar un commit para revertir, se necesita:

```bash
git switch main
git pull --ff-only origin main
```

Omitir esta sincronización es un error frecuente: sin ella, el siguiente `git push origin main` puede ser rechazado como **non-fast-forward**, y el `git log` local puede no contener la fusión realizada en GitHub.

### Reutilizar una rama ya fusionada para demostrar un conflicto

Reutilizar una rama después de haberla fusionado y eliminado en GitHub suele generar confusión, porque GitHub ya no la reconoce como una rama activa comparable. Un flujo más claro es crear una rama nueva dedicada a la demostración:

```bash
# después de sincronizar main
git switch -c conflict-demo

# volver a main, modificar una línea y publicar el cambio
git switch main
# editar message.txt
git add message.txt
git commit -m "Update message on main"
git push origin main

# modificar la misma línea de otra forma en la rama
git switch conflict-demo
# editar message.txt
git add message.txt
git commit -m "Propose alternative message"
git push -u origin conflict-demo
```

Al abrir la PR de `conflict-demo` hacia `main`, GitHub podrá detectar el conflicto. Después de resolverlo y fusionarlo, se vuelve a sincronizar `main` local antes de ejecutar `git log` o `git revert`.

### Revertir el cambio correcto

Antes de revertir:

```bash
git switch main
git pull --ff-only origin main
git log --oneline --graph --decorate -10
```

Se debe identificar si se quiere revertir un commit normal o una PR completa. Para una PR, la opción **Revert** de GitHub crea otra PR y conserva la revisión del cambio inverso. Si no está disponible o aparecen conflictos, la reversión deberá hacerse localmente con cuidado.

## 10. Diagnóstico de problemas frecuentes

| Problema | Causa probable | Qué hacer |
| --- | --- | --- |
| `src refspec main does not match any` | Aún no existe un commit o la rama tiene otro nombre. | Crear el primer commit y comprobar `git branch`. |
| `remote origin already exists` | Ya hay un remoto llamado `origin`. | Revisar `git remote -v` y usar `git remote set-url`. |
| `Authentication failed` | Token incorrecto, vencido o sin permisos. | Revisar permisos y caducidad; vaciar la caché y volver a autenticarse. |
| `non-fast-forward` al hacer push | El remoto contiene commits que no están localmente. | Ejecutar `git fetch`, inspeccionar el historial y sincronizar; no forzar el push. |
| `CONFLICT (content)` | Dos ramas cambiaron las mismas líneas. | Usar `git status`, resolver marcadores, probar, agregar y confirmar. |
| `pathspec ... did not match` | El nombre de rama o archivo es incorrecto. | Comprobar `git branch --all` o `git status`. |
| Estado `detached HEAD` | Se cambió directamente a un commit o etiqueta. | Crear una rama con `git switch -c <nombre>` si se desea conservar cambios. |

Evita `git push --force` como solución automática. En una rama compartida puede borrar trabajo remoto. Si una reescritura autorizada requiere forzar el envío, `--force-with-lease` ofrece una comprobación adicional, pero tampoco sustituye revisar el historial y coordinar con el equipo.

## 11. Comandos de consulta rápida

```bash
# Estado y diferencias
git status
git diff
git diff --staged

# Historial
git log --oneline --graph --decorate --all
git show <commit-o-etiqueta>

# Ramas
git branch
git switch -c <rama-nueva>
git switch main

# Remotos
git remote -v
git fetch origin
git pull --ff-only origin main
git push -u origin <rama>

# Preparar y confirmar
git add <archivo>
git commit -m "Mensaje descriptivo"

# Recuperación
git restore <archivo>
git restore --staged <archivo>
git revert <commit>
```

## 12. Información adicional útil

### Git no es una copia de seguridad automática

Un commit existe primero solo en el repositorio local. Para contar con una copia remota se debe ejecutar `git push`. Tampoco conviene confirmar archivos grandes, binarios generados o secretos únicamente “para guardarlos”.

### `HEAD`, hashes y referencias

- `HEAD` señala la rama o commit actualmente seleccionado.
- Un hash identifica de manera única un commit dentro del repositorio.
- `main`, `origin/main` y `v1` son referencias legibles que apuntan a commits.
- `main` y `origin/main` pueden apuntar a commits distintos hasta sincronizar el repositorio.

### Revisar antes de cada commit

Una rutina breve previene muchos errores:

```bash
git status
git diff
git diff --staged
```

Luego de confirmar:

```bash
git show --stat --oneline HEAD
```

### Ayuda incorporada

La documentación instalada puede consultarse sin abandonar la terminal:

```bash
git help commit
git help pull
git <comando> -h
```

## Recursos oficiales

### Guías generales

- [Pro Git, segunda edición](https://git-scm.com/book/en/v2): libro conceptual y práctico. También está disponible la [traducción oficial al español](https://git-scm.com/book/es/v2).
- [Referencia oficial de comandos Git](https://git-scm.com/docs): documentación precisa de cada comando; es más útil como consulta que como recorrido inicial.
- [Introducción a GitHub](https://docs.github.com/en/get-started): portal general de conceptos, configuración y primeros pasos.
- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow): flujo completo de ramas, commits, pull requests, revisión y fusión.
- [Fusionar y cerrar pull requests](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests): índice de las acciones relacionadas con fusión, conflictos y reversión de PR.

### Documentación directa recomendada

- [Registrar cambios en el repositorio](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)
- [Trabajar con repositorios remotos](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes)
- [Ramificaciones y fusiones básicas](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- [Referencia de `.gitignore`](https://git-scm.com/docs/gitignore)
- [Referencia de `git pull`](https://git-scm.com/docs/git-pull)
- [Referencia de `git reset`](https://git-scm.com/docs/git-reset)
- [Referencia de `git revert`](https://git-scm.com/docs/git-revert)
- [Administrar personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [Caché de credenciales de Git](https://git-scm.com/docs/git-credential-cache)
- [Resolver un conflicto de fusión desde la terminal](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-using-the-command-line)
- [Revertir una pull request](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/reverting-a-pull-request)

## Conclusión

El objetivo no es memorizar todos los comandos, sino comprender dónde está cada cambio y qué operación modifica el directorio de trabajo, el área de preparación, el historial local o el repositorio remoto. `git status`, `git diff` y un historial gráfico permiten verificar ese estado antes de actuar. Para trabajo compartido, las ramas pequeñas, las pull requests y `git revert` ofrecen un historial más claro y recuperable; los comandos destructivos como `reset --hard` deben reservarse para situaciones controladas.
