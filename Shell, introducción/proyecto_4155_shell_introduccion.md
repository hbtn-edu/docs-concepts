# Introducción a la shell de Linux

## Terminal, shell y línea de comandos

Una **terminal** es el programa que permite escribir texto y mostrar la salida de otros programas. La **shell** es el intérprete que recibe los comandos, analiza su sintaxis y solicita al sistema operativo que los ejecute.

En muchos sistemas Linux la shell predeterminada es **Bash**, sigla de *Bourne Again Shell*. También existen otras shells, como `sh`, `zsh`, `ksh` y `fish`. Aunque comparten muchos comandos, su sintaxis y funcionalidades no son idénticas.

Un emulador de terminal puede ejecutar distintas shells. Por eso, terminal y shell no son sinónimos:

| Elemento | Función |
| --- | --- |
| Terminal | Proporciona la interfaz de entrada y salida. |
| Shell | Interpreta y ejecuta comandos. |
| Comando | Indica una operación que debe realizarse. |
| Programa | Archivo ejecutable que puede ser invocado por la shell. |

Para identificar la shell asociada con la sesión pueden consultarse:

```bash
ps -p $$
echo "$SHELL"
```

`$$` es una variable especial que contiene el PID del proceso actual (la propia shell, en este caso). `$SHELL` suele indicar la shell configurada para iniciar sesión, que no necesariamente coincide con la que se está ejecutando en ese instante.

## El prompt

El **prompt** es el texto que indica que la shell está esperando un comando. Su aspecto depende de la configuración. Puede mostrar el usuario, el nombre del equipo y el directorio actual:

```text
student@ubuntu:~/work$
```

Por convención:

- `$` suele representar una sesión de usuario normal.
- `#` suele representar una sesión con privilegios de superusuario.

El símbolo forma parte del prompt y no se escribe al copiar un comando de la documentación.

Trabajar como `root` aumenta el impacto de cualquier error: una ruta o comodín incorrecto puede modificar archivos de todo el sistema. Los privilegios elevados deben usarse únicamente cuando sean necesarios.

## Anatomía de un comando

La forma general es:

```text
command [options] [arguments]
```

Por ejemplo:

```bash
ls -l /etc
```

- `ls` es el comando.
- `-l` es una opción que modifica su comportamiento.
- `/etc` es el argumento sobre el que trabaja.

Las opciones cortas compatibles suelen poder agruparse:

```bash
ls -l -a
ls -la
```

No todos los comandos aceptan las mismas opciones. Antes de asumir su significado se debe consultar la ayuda correspondiente.

### Nombres y caracteres especiales

Linux distingue entre mayúsculas y minúsculas. `Report.txt`, `report.txt` y `REPORT.txt` son nombres diferentes.

Los nombres pueden contener espacios, pero deben citarse o escaparse:

```bash
ls "My Documents"
ls My\ Documents
```

Para evitar que un nombre comenzado con `-` se interprete como una opción, muchos comandos aceptan `--` para marcar el final de las opciones:

```bash
rm -- -strange-name
```

## Tipos de comandos

Lo que se escribe como un comando puede resolverse de distintas maneras:

| Tipo | Descripción | Ejemplo habitual |
| --- | --- | --- |
| Programa ejecutable | Archivo binario o script localizado normalmente mediante `PATH`. | `cp`, `ls` |
| Builtin de la shell | Operación implementada dentro de Bash. | `cd`, `help`, `type` |
| Función de shell | Bloque de comandos definido en la sesión o configuración. | Función personalizada |
| Alias | Nombre corto que la shell expande antes de ejecutar. | `ll` como alias de `ls -l` |

`type` muestra cómo Bash interpreta un nombre:

```bash
type cd
type ls
type -a ls
```

`which` busca ejecutables en los directorios de `PATH`, pero puede no describir correctamente builtins, funciones o aliases. Para comprender qué ejecutará Bash, `type` o `command -V` suelen ser más completos:

```bash
command -V pwd
```

### Aliases

Un alias permite abreviar una combinación usada con frecuencia:

```bash
alias ll='ls -la'
alias
unalias ll
```

Los aliases definidos de esta forma existen solamente en la sesión actual. Pueden guardarse en la configuración de Bash, pero no conviene depender de aliases interactivos dentro de scripts.

## Buscar ayuda

La documentación instalada describe el comportamiento real de las herramientas disponibles en el sistema.

### `help`

`help` documenta los builtins de Bash:

```bash
help cd
help type
help help
```

### `man`

`man` abre páginas de manual de programas, funciones y componentes del sistema:

```bash
man ls
man cp
man man
```

Las secciones más importantes al comenzar son:

| Sección | Contenido |
| ---: | --- |
| 1 | Programas ejecutables y comandos de usuario. |
| 2 | Llamadas al sistema proporcionadas por el kernel. |
| 3 | Funciones de bibliotecas. |
| 4 | Archivos especiales y dispositivos. |
| 5 | Formatos de archivo y convenciones. |
| 7 | Conceptos y convenciones generales. |
| 8 | Comandos de administración. |

Si un nombre aparece en varias secciones, se puede elegir una explícitamente:

```bash
man 1 printf
man 3 printf
```

Las páginas suelen contener `NAME`, `SYNOPSIS`, `DESCRIPTION`, `OPTIONS`, `EXAMPLES` y `SEE ALSO`. En `SYNOPSIS`:

- El texto literal se escribe como aparece.
- Los elementos entre corchetes son opcionales.
- Los puntos suspensivos indican que un argumento puede repetirse.
- Una barra vertical separa alternativas.

Comandos relacionados:

```bash
whatis ls
apropos "copy files"
ls --help
```

Dentro de `man` y `less` pueden usarse `/texto` para buscar, `n` para repetir la búsqueda y `q` para salir.

### RTFM

**RTFM** es un acrónimo informal que originalmente significa *Read The F… Manual*. En contextos educativos o profesionales suele suavizarse como *Read The Fine Manual*.

La idea útil detrás de la expresión es consultar la documentación antes de adivinar. Sin embargo, emplearla como respuesta puede resultar despectivo. Es más constructivo señalar qué manual o sección ayuda a resolver la duda.

## Historial y edición de la línea

Bash conserva un historial de comandos y utiliza la biblioteca GNU Readline para editar la línea actual.

```bash
history
```

Atajos frecuentes:

| Atajo | Acción habitual |
| --- | --- |
| `Up` / `Down` | Recorrer comandos anteriores y posteriores. |
| `Ctrl+R` | Buscar hacia atrás en el historial. |
| `Ctrl+G` | Cancelar una búsqueda del historial. |
| `Ctrl+A` | Ir al comienzo de la línea. |
| `Ctrl+E` | Ir al final de la línea. |
| `Alt+B` / `Alt+F` | Retroceder o avanzar una palabra. |
| `Ctrl+U` | Cortar desde el cursor hasta el comienzo. |
| `Ctrl+K` | Cortar desde el cursor hasta el final. |
| `Ctrl+W` | Cortar la palabra anterior. |
| `Ctrl+Y` | Pegar el último texto cortado. |
| `Ctrl+L` | Limpiar visualmente la pantalla. |
| `Ctrl+C` | Interrumpir el proceso en primer plano. |
| `Ctrl+D` | Enviar fin de archivo; en un prompt vacío puede cerrar la shell. |
| `Tab` | Completar comandos y rutas. |

La expansión `!!` vuelve a ejecutar inmediatamente el comando anterior. Antes de reutilizar un comando destructivo es más seguro recuperarlo con `Up` o `Ctrl+R`, revisarlo y recién entonces ejecutarlo.

Los atajos pueden variar según el modo de edición de Bash, la configuración de Readline y el emulador de terminal.

## El sistema de archivos

Linux organiza archivos y directorios en un único árbol. La parte superior es el directorio raíz, representado por `/`. Los discos y otros sistemas de archivos se conectan a diferentes puntos de ese árbol en lugar de utilizar letras de unidad.

### Directorios habituales

| Ruta | Uso general |
| --- | --- |
| `/` | Raíz de toda la jerarquía. |
| `/bin` | Comandos esenciales o enlace hacia su ubicación actual. |
| `/boot` | Archivos necesarios para el arranque. |
| `/dev` | Representaciones de dispositivos. |
| `/etc` | Configuración del sistema. |
| `/home` | Directorios personales de usuarios habituales. |
| `/root` | Directorio personal del superusuario `root`. |
| `/tmp` | Archivos temporales; su contenido puede eliminarse. |
| `/usr/bin` | Gran parte de los programas disponibles para usuarios. |
| `/usr/local` | Software y datos instalados localmente. |
| `/var` | Datos que cambian durante el funcionamiento, como logs y cachés. |
| `/proc` | Vista virtual de procesos e información del kernel. |
| `/sys` | Información estructurada sobre kernel y dispositivos. |
| `/opt` | Software adicional instalado como paquetes independientes. |

La distribución exacta puede variar entre versiones. En sistemas modernos, rutas históricas como `/bin` o `/sbin` pueden ser enlaces simbólicos hacia directorios bajo `/usr`. Conviene observar el sistema real con `ls -ld` y consultar `man 7 hier`.

El directorio raíz `/` y el directorio personal de `root`, `/root`, son ubicaciones diferentes.

## Rutas y navegación

### Directorio de trabajo

Cada proceso tiene un **directorio de trabajo actual**. Los nombres relativos se interpretan desde ese lugar.

```bash
pwd
```

`pwd` imprime su ruta absoluta. Si el recorrido contiene enlaces simbólicos, `pwd -P` muestra la ubicación física resuelta:

```bash
pwd -P
```

### Rutas absolutas y relativas

Una ruta absoluta comienza con `/`:

```text
/var/log/syslog
```

Una ruta relativa comienza desde el directorio actual:

```text
docs/guide.md
../archive
```

Referencias especiales:

| Referencia | Significado |
| --- | --- |
| `.` | Directorio actual. |
| `..` | Directorio padre. |
| `~` | Directorio personal del usuario actual. |
| `~user` | Directorio personal del usuario indicado. |

### Cambiar de directorio

```bash
cd /var/log
cd ..
cd
cd ~
cd -
```

- `cd` sin argumentos lleva al directorio personal.
- `cd -` alterna entre el directorio actual y el anterior y normalmente imprime el destino.

`cd` es un builtin porque debe cambiar el directorio de la propia shell. Un proceso hijo no puede modificar el directorio de trabajo de su proceso padre.

## Listar contenido con `ls`

```bash
ls
ls /etc
ls -l
ls -la
ls -lan
ls -la . .. /boot
```

Opciones importantes:

| Opción | Efecto |
| --- | --- |
| `-l` | Formato largo. |
| `-a` | Incluye entradas cuyo nombre comienza con `.`, además de `.` y `..`. |
| `-A` | Incluye archivos ocultos, pero omite `.` y `..`. |
| `-n` | Muestra identificadores numéricos de usuario y grupo en formato largo. |
| `-h` | Usa tamaños legibles cuando se combina con `-l`. |
| `-d` | Muestra el directorio como entrada en lugar de listar su contenido. |

Un archivo oculto en Linux es simplemente un archivo cuyo nombre comienza con `.`. No tiene un atributo especial de ocultación y los comodines comunes tampoco suelen seleccionarlo.

### Interpretar el formato largo

Una línea típica puede verse así:

```text
-rw-r--r-- 1 student developers 128 Sep  9 10:30 notes.txt
```

Sus campos representan, de izquierda a derecha:

1. Tipo y permisos.
2. Cantidad de enlaces duros.
3. Propietario.
4. Grupo.
5. Tamaño.
6. Fecha de modificación.
7. Nombre.

El primer carácter permite reconocer algunos tipos:

- `-`: archivo regular.
- `d`: directorio.
- `l`: enlace simbólico.

El formato y los nombres pueden variar según locale, sistema y opciones. No conviene asumir que una captura de otra máquina será idéntica.

## Examinar archivos

### `file`

`file` inspecciona el contenido y otros indicadores para clasificar un archivo:

```bash
file /bin/ls
file image.jpg
file script.sh
```

La extensión puede orientar a una aplicación, pero Linux no depende de ella para decidir qué contiene el archivo. La salida exacta de `file` puede variar según la versión y el contenido analizado.

### `less`

`less` permite leer archivos de texto sin cargarlos en un editor:

```bash
less /etc/services
```

Controles útiles:

| Tecla | Acción |
| --- | --- |
| `Space` o `Page Down` | Avanzar una pantalla. |
| `b` o `Page Up` | Retroceder una pantalla. |
| `g` | Ir al comienzo. |
| `G` | Ir al final. |
| `/texto` | Buscar hacia adelante. |
| `n` | Repetir la búsqueda. |
| `q` | Salir. |

No se debe abrir como texto un archivo binario desconocido antes de clasificarlo con `file`.

## Manipular archivos y directorios

### Crear directorios

```bash
mkdir reports
mkdir -p archive/2026/september
```

`mkdir -p` crea los directorios padre que falten y no produce error si la ruta ya existe como directorio.

### Copiar

```bash
cp source.txt destination.txt
cp source.txt backup/
cp -r source-directory/ destination-directory/
```

`cp` conserva el original y crea otra entrada en el destino. Puede sobrescribir archivos existentes, por lo que conviene inspeccionar antes o usar `-i` cuando se desea confirmación interactiva.

Para copiar solamente cuando el destino no existe o es más antiguo:

```bash
cp -u -- *.html ../archive/
```

La opción `-u` corresponde a una política de actualización. Su comportamiento exacto debe comprobarse con `cp --help` o `man cp` en la versión instalada.

### Mover o renombrar

```bash
mv draft.txt final.txt
mv final.txt archive/
```

`mv` cambia el nombre o la ubicación. También puede reemplazar un destino existente. La opción `-i` pide confirmación antes de sobrescribir.

### Eliminar

```bash
rm obsolete.txt
rmdir empty-directory
rm -r old-directory
```

- `rm` elimina archivos.
- `rmdir` elimina directorios vacíos.
- `rm -r` recorre y elimina un directorio con su contenido.

La eliminación desde la terminal normalmente no utiliza la papelera. Antes de usar `rm`, y especialmente `rm -r`, se deben revisar el directorio actual, la ruta y cualquier comodín:

```bash
pwd
printf '%s\n' -- *~
```

`rm -f` evita avisos por nombres inexistentes, pero también reduce las protecciones. No debe utilizarse como sustituto de revisar el patrón.

## Comodines o globs

Los comodines son patrones que la shell expande **antes** de ejecutar el comando. El programa recibe una lista de nombres ya resuelta.

| Patrón | Coincide con |
| --- | --- |
| `*` | Cero o más caracteres. |
| `?` | Un carácter cualquiera. |
| `[abc]` | Un carácter incluido en el conjunto. |
| `[a-z]` | Un carácter dentro del rango según el locale. |
| `[!0-9]` | Un carácter que no pertenece al conjunto. |
| `[[:upper:]]` | Una letra mayúscula según la clase POSIX. |
| `[[:digit:]]` | Un dígito. |

Ejemplos:

```bash
ls *.html
ls report?.txt
ls [[:upper:]]*
printf '%s\n' -- *~
```

Aspectos importantes:

- `*` normalmente no incluye nombres que comienzan con `.`.
- Citar el patrón impide su expansión: `"*.html"` es texto literal.
- Si no existe ninguna coincidencia, Bash normalmente deja el patrón sin expandir, a menos que se haya activado una opción como `nullglob`.
- Los rangos como `[A-Z]` pueden depender del locale; las clases como `[[:upper:]]` expresan mejor la intención.
- Es recomendable previsualizar un patrón antes de usarlo con `mv` o `rm`.

Cuando un comodín coincide con varios nombres, cada resultado se entrega como un argumento independiente, incluso si alguno contiene espacios.

## Enlaces duros y simbólicos

### Enlace duro

Un enlace duro es otro nombre para el mismo inode y contenido:

```bash
ln original.txt second-name.txt
```

Características habituales:

- Ambos nombres apuntan a los mismos datos.
- Eliminar uno no elimina el contenido mientras exista otro enlace.
- Normalmente no se crean sobre directorios.
- No pueden atravesar sistemas de archivos diferentes.

### Enlace simbólico

Un enlace simbólico es un archivo especial que almacena una ruta hacia otro objeto:

```bash
ln -s /usr/bin/example example-link
ls -l example-link
readlink example-link
```

Puede apuntar a archivos o directorios y cruzar sistemas de archivos. Si el destino desaparece o la ruta deja de ser válida, el enlace queda roto.

Un destino absoluto comienza con `/`. Un destino relativo se interpreta desde el directorio que contiene el enlace, no desde el directorio desde el cual se accede a él.

| Característica | Enlace duro | Enlace simbólico |
| --- | --- | --- |
| Comparte inode con el destino | Sí | No |
| Puede apuntar a un directorio | Generalmente no | Sí |
| Puede cruzar sistemas de archivos | No | Sí |
| Puede quedar roto | No mientras exista algún enlace | Sí |

## Scripts Bash

Un script es un archivo de texto que contiene comandos. Una forma mínima es:

```bash
#!/bin/bash
command
```

### Shebang

La primera línea que comienza con `#!` se denomina **shebang**. Cuando el archivo se ejecuta directamente, el sistema utiliza la ruta indicada para elegir el intérprete:

```text
#!/bin/bash
```

El shebang debe ocupar la primera línea y comenzar en el primer carácter. En un entorno donde Bash está garantizado en `/bin/bash`, esta forma selecciona explícitamente ese intérprete.

### Permiso y ejecución

Para otorgar permiso de ejecución al propietario:

```bash
chmod u+x script_name
./script_name
```

`./` especifica una ruta al archivo del directorio actual. La shell normalmente no busca comandos en `.` porque ese directorio no suele estar incluido en `PATH`.

También puede invocarse Bash directamente:

```bash
bash script_name
```

En este caso no se necesita el permiso de ejecución y el intérprete ya fue elegido en el comando.

### Ejecutar frente a cargar con `source`

Estas operaciones no son equivalentes:

```bash
./script_name
source script_name
. script_name
```

- `./script_name` ejecuta el archivo como otro proceso de Bash.
- `source script_name` y `. script_name` leen los comandos dentro de la shell actual.

Un cambio de directorio ejecutado en un proceso hijo desaparece cuando ese proceso termina. En cambio, un `cd` cargado mediante `source` modifica la shell actual y permanece después de finalizar el archivo.

El shebang y el permiso ejecutable intervienen en la ejecución directa, pero no seleccionan un intérprete cuando el contenido se carga con `source`.

### Líneas y nueva línea final

Los archivos de texto deberían terminar con una nueva línea. Para comprobar la estructura:

```bash
wc -l script_name
head -n 1 script_name
file script_name
bash -n script_name
```

`wc -l` cuenta caracteres de nueva línea, no líneas visuales. Si falta la nueva línea final, el resultado puede ser menor de lo esperado. `bash -n` comprueba la sintaxis sin ejecutar los comandos, pero no demuestra que su comportamiento sea correcto.

### Estado de salida

Cada comando devuelve un código. Por convención, `0` indica éxito y un valor distinto de cero indica algún error:

```bash
echo $?
```

El valor corresponde al comando ejecutado inmediatamente antes. Es útil para diagnosticar scripts, pero se reemplaza en cuanto se ejecuta otro comando.

Las comillas invertidas son una sintaxis antigua de sustitución de comandos. En scripts modernos se prefiere `$(command)` porque es más legible y fácil de anidar.

## Ubuntu LTS

**LTS** significa *Long Term Support*, o soporte a largo plazo. Las versiones LTS de Ubuntu aparecen cada dos años y priorizan estabilidad y mantenimiento prolongado frente a la incorporación frecuente de cambios mayores.

Canonical indica cinco años de mantenimiento de seguridad estándar para paquetes de `Main`. Ubuntu Pro puede ampliar la cobertura de seguridad, y existen opciones comerciales adicionales para períodos más largos. El alcance depende del repositorio de paquetes y del tipo de suscripción.

Una versión LTS no permanece congelada: recibe correcciones de seguridad y mantenimiento dentro de su ciclo de vida. Los comandos, paquetes y rutas observados pueden diferir entre distintas versiones LTS.

## Hábitos seguros para el trabajo diario

### Antes de modificar o eliminar

```bash
pwd
ls -la
```

- Confirmar el directorio actual.
- Leer el comando completo antes de presionar Enter.
- Previsualizar los comodines.
- Usar rutas absolutas cuando el destino no deba depender del directorio actual.
- Añadir `--` cuando los nombres puedan comenzar con `-`.
- Evitar privilegios de superusuario si no son imprescindibles.

### Antes de ejecutar un script

```bash
head -n 2 script_name
bash -n script_name
ls -l script_name
```

- Confirmar el intérprete.
- Revisar rutas y operaciones destructivas.
- Verificar permisos.
- Probar primero con archivos temporales que puedan recrearse.

### Al consultar documentación

1. Determinar si el comando es un builtin con `type`.
2. Usar `help` para builtins de Bash.
3. Usar `man` o `--help` para programas externos.
4. Leer `SYNOPSIS`, `DESCRIPTION`, `OPTIONS` y `EXAMPLES`.
5. Confirmar la versión instalada si el comportamiento es relevante.

## Problemas frecuentes

| Situación | Causa probable | Qué comprobar |
| --- | --- | --- |
| `command not found` | Nombre incorrecto o programa ausente de `PATH`. | `type`, ortografía y valor de `PATH`. |
| `No such file or directory` | Ruta incorrecta, destino inexistente o intérprete del shebang ausente. | `pwd`, `ls`, mayúsculas y primera línea. |
| `Permission denied` | Falta permiso de ejecución o acceso a algún componente de la ruta. | `ls -l` y permisos del archivo y directorios. |
| Un script con `cd` no cambia la terminal | Se ejecutó en un proceso hijo. | Diferencia entre ejecución directa y `source`. |
| `Directory not empty` con `rmdir` | El directorio todavía contiene entradas. | `ls -la <directorio>` antes de decidir cómo actuar. |
| Un comodín aparece literalmente en el error | No hubo coincidencias o el patrón estaba citado. | Previsualizar el patrón y revisar las comillas. |
| `cp` o `mv` reemplazó un archivo | Ya existía un nombre igual en el destino. | Usar `-i` o inspeccionar el destino antes. |
| Un enlace simbólico está roto | La ruta almacenada ya no conduce al destino. | `readlink`, `ls -l` y rutas relativas. |
| La salida no coincide con otra máquina | Diferencias de versión, locale, permisos o contenido. | Consultar la ayuda local y comparar el entorno. |

## Recursos

### Recorrido introductorio

- [¿Qué es la shell?](https://linuxcommand.org/lc3_lts0010.php)
- [Navegación](https://linuxcommand.org/lc3_lts0020.php)
- [Mirando alrededor](https://linuxcommand.org/lc3_lts0030.php)
- [Un recorrido guiado por el sistema](https://linuxcommand.org/lc3_lts0040.php)
- [Manipulación de archivos](https://linuxcommand.org/lc3_lts0050.php)
- [Trabajar con comandos](https://linuxcommand.org/lc3_lts0060.php)
- [Página de manual de `man`](https://linuxcommand.org/lc3_man_pages/man1.html)

### Recursos complementarios

- [Atajos de teclado para Bash](https://www.howtogeek.com/181/keyboard-shortcuts-for-bash-command-shell-for-ubuntu-debian-suse-redhat-linux-etc/)
- [Información sobre Ubuntu LTS](https://wiki.ubuntu.com/LTS)
- [Shebang en sistemas Unix](https://en.wikipedia.org/wiki/Shebang_%28Unix%29)
- [Explicación del sistema de archivos de Linux](https://www.linuxfoundation.org/blog/blog/classic-sysadmin-the-linux-filesystem-explained)

### Documentación primaria recomendada

- [Manual de referencia de Bash](https://www.gnu.org/software/bash/manual/bash.html)
- [Edición de línea con GNU Readline](https://www.gnu.org/software/bash/manual/html_node/Readline-Interaction.html)
- [Manual de GNU Coreutils](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [Ciclo de versiones de Ubuntu](https://ubuntu.com/about/release-cycle)
- [`execve(2)` e interpretación del shebang](https://man7.org/linux/man-pages/man2/execve.2.html)
- [Jerarquía del sistema de archivos en Ubuntu 22.04](https://manpages.ubuntu.com/manpages/jammy/en/man7/hier.7.html)

## Conclusión

La shell convierte texto en operaciones precisas sobre procesos y archivos. Para trabajar con seguridad es necesario saber dónde se está, cómo se resolverá cada ruta, qué nombres seleccionará un comodín y qué tipo de comando ejecutará Bash.

Los comandos básicos son pocos, pero pueden combinarse de muchas maneras. Consultar `help`, `man` y la ayuda local, revisar el estado antes de actuar y comprender la diferencia entre ejecutar y cargar un script son hábitos más valiosos que memorizar secuencias aisladas.
