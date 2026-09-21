# Redirecciones de entrada/salida y filtros en Bash

## El flujo de texto en la shell

Muchas herramientas de Unix siguen una idea sencilla: leer texto, transformarlo y producir otro texto. Cada programa puede resolver una operación pequeña, mientras la shell conecta sus entradas y salidas para construir procesos más complejos.

Por ejemplo, una herramienta puede localizar archivos, otra contar líneas y otra ordenar resultados. Esta composición evita que cada programa necesite implementar todas las funciones posibles.

Un **filtro** es un programa que normalmente:

1. recibe datos mediante la entrada estándar;
2. los examina o transforma;
3. escribe el resultado en la salida estándar.

`sort`, `uniq`, `grep`, `tr`, `cut`, `head`, `tail` y `wc` pueden trabajar de esta manera.

## Los tres flujos estándar

Cuando la shell inicia un proceso, normalmente le proporciona tres canales abiertos:

| Descriptor | Nombre | Abreviatura | Destino predeterminado |
| :---: | --- | :---: | --- |
| `0` | Entrada estándar | `stdin` | Teclado o entrada de la terminal. |
| `1` | Salida estándar | `stdout` | Pantalla de la terminal. |
| `2` | Salida de error estándar | `stderr` | Pantalla de la terminal. |

La salida normal y los diagnósticos son flujos distintos, aunque ambos se vean en la misma terminal. Esta separación permite guardar resultados sin mezclar mensajes de error, o tratarlos por separado.

No todos los programas leen automáticamente de `stdin`: algunos esperan nombres de archivo como argumentos. La página de manual indica qué formas admite cada comando.

## Redirigir la salida estándar

### Sobrescribir con `>`

`>` envía `stdout` a un archivo:

```bash
ls -la > directory_listing.txt
```

La shell prepara el archivo antes de ejecutar `ls`:

- si no existe, lo crea;
- si existe, lo trunca a cero bytes;
- si no puede abrirlo, el comando no llega a ejecutarse.

El archivo de destino puede aparecer dentro de la propia operación que genera su contenido. Por ejemplo, al redirigir un listado del directorio actual (también llamado directorio de trabajo), la shell crea primero el archivo vacío y luego ejecuta `ls`: como el archivo ya existe en ese momento, su nombre puede aparecer en el propio listado que está por sobrescribirse.

### Anexar con `>>`

`>>` agrega la salida al final sin borrar el contenido anterior:

```bash
date >> activity.log
```

Si el archivo no existe, también se crea. La diferencia es el punto desde el que se escribe.

| Operador | Archivo inexistente | Archivo existente |
| :---: | --- | --- |
| `>` | Lo crea. | Lo trunca y escribe desde el comienzo. |
| `>>` | Lo crea. | Conserva el contenido y escribe al final. |

`sort` ordena las líneas de su entrada (alfabética o numéricamente, según las opciones) y escribe el resultado en la salida estándar.

No se debe redirigir con `>` hacia el mismo archivo que un programa está intentando leer:

```bash
sort data.txt > data.txt
```

La shell vacía `data.txt` antes de que `sort` pueda leerlo. Para reemplazar contenido de forma segura se utiliza un archivo temporal y, después de comprobar el resultado, se lo mueve sobre el original.

Anexar a un archivo mientras se lo lee también puede ser inseguro: algunos programas continúan observando los datos recién agregados y producen un crecimiento indefinido.

## Redirigir la entrada estándar

`<` hace que el descriptor `0` lea desde un archivo:

```bash
sort < names.txt
```

Puede combinarse con una redirección de salida:

```bash
sort < names.txt > sorted_names.txt
```

Algunas herramientas permiten expresar la misma operación pasando el archivo como argumento:

```bash
sort names.txt
```

Sin embargo, trabajar con `stdin` facilita la composición y permite que el mismo comando reciba datos desde un archivo, una tubería o la terminal.

### Here document

Un *here document* proporciona varias líneas como entrada:

```bash
cat <<'EOF'
first line
second line
EOF
```

Citar el delimitador, como en `'EOF'`, evita expansiones dentro del bloque. Sin citarlo, Bash puede expandir variables y sustituciones de comandos.

### Here string

Bash también puede convertir una cadena en entrada estándar:

```bash
tr '[:lower:]' '[:upper:]' <<< 'Hello world'
```

La construcción `<<<` agrega una nueva línea final. Es propia de shells como Bash y no forma parte del conjunto mínimo de sintaxis POSIX.

## Redirigir errores

Como `stderr` es el descriptor `2`, puede redirigirse de forma independiente:

```bash
find /srv -name '*.log' 2> errors.log
```

`find` busca archivos que cumplan ciertos criterios (nombre, tipo, tamaño, fecha) recorriendo un árbol de directorios; se detalla más adelante en su propia sección.

Para anexar diagnósticos:

```bash
find /srv -name '*.log' 2>> errors.log
```

Para guardar salida normal y errores en archivos separados:

```bash
command > output.log 2> errors.log
```

Para enviarlos al mismo destino:

```bash
command > combined.log 2>&1
```

`2>&1` significa: hacer que el descriptor `2` apunte al destino actual del descriptor `1`.

### El orden sí puede cambiar el resultado

Las redirecciones se procesan de izquierda a derecha:

```bash
command > combined.log 2>&1
```

Primero `stdout` pasa al archivo; después `stderr` copia ese destino. Ambos terminan en `combined.log`.

```bash
command 2>&1 > output.log
```

Aquí `stderr` copia primero el destino original de `stdout`, normalmente la terminal. Después solo `stdout` se mueve al archivo.

Bash admite una abreviatura para ambos flujos:

```bash
command &> combined.log
```

La forma `> file 2>&1` es más portable entre shells.

### Descartar salida

`/dev/null` acepta datos y los descarta:

```bash
command > /dev/null
command 2> /dev/null
command > /dev/null 2>&1
```

Ocultar errores sin comprenderlos dificulta el diagnóstico. Conviene hacerlo únicamente cuando esos mensajes son esperados y no contienen información necesaria.

## Tuberías

El operador `|` conecta `stdout` del comando izquierdo con `stdin` del derecho:

```bash
find . -type f -print | wc -l   # wc -l cuenta las líneas que recibe por la entrada
```

La shell crea la conexión y ejecuta los componentes de la tubería. Los programas pueden trabajar de manera concurrente: el segundo no necesita esperar a que el primero produzca toda su salida.

Las tuberías pueden encadenarse:

```bash
sort names.txt | uniq -c | sort -nr
```

Cada etapa debe tener una responsabilidad clara:

1. `sort` agrupa líneas iguales.
2. `uniq -c` cuenta cada grupo contiguo.
3. `sort -nr` ordena los conteos de mayor a menor.

### Una tubería transporta `stdout`

Por defecto, `stderr` no entra en la tubería:

```bash
command | filter
```

Para enviar también los diagnósticos:

```bash
command 2>&1 | filter
```

Bash ofrece la abreviatura:

```bash
command |& filter
```

Mezclar resultados y errores puede ser útil para registrar una sesión, pero no para filtros que esperan datos con un formato específico.

### Estado de salida de una tubería

El estado de salida es el número que un comando devuelve al terminar (0 indica éxito, cualquier otro valor indica error); `$?` guarda el de la última tubería o comando ejecutado.

Normalmente el estado de una tubería es el del último comando:

```bash
producer | filter
echo "$?"
```

Eso puede ocultar el fallo de una etapa anterior. En Bash:

```bash
set -o pipefail
```

hace que la tubería falle si falla alguno de sus componentes. El arreglo `PIPESTATUS` conserva los estados de todas las etapas inmediatamente después de ejecutarlas:

```bash
producer | filter
printf '%s\n' "${PIPESTATUS[*]}"
```

## Caracteres especiales y citado

Bash interpreta ciertos caracteres antes de ejecutar un programa. Para pasar un carácter literalmente hay que citarlo o escaparlo.

### Espacios en blanco

Los espacios separan palabras:

```bash
touch quarterly report.txt
```

Ese comando recibe dos nombres. Para representar uno solo:

```bash
touch 'quarterly report.txt'
```

Las tabulaciones y nuevas líneas también pueden separar palabras cuando no están protegidas.

### Comillas simples

Las comillas simples conservan literalmente todos los caracteres comprendidos entre ellas:

```bash
printf '%s\n' '$HOME * ? | >'
```

No es posible insertar una comilla simple dentro de un bloque entre comillas simples. Se puede cerrar el bloque, escapar la comilla y continuar:

```bash
printf '%s\n' 'It'\''s ready'
```

### Comillas dobles

Las comillas dobles evitan la separación de palabras y la expansión de nombres, pero permiten expansiones como `$variable` y `$(command)`:

```bash
printf '%s\n' "Usuario: $USER"
printf '%s\n' "Directorio: $(pwd)"
```

Como regla general, las expansiones de variables que representan una sola palabra deben citarse:

```bash
cat -- "$filename"
```

### Barra invertida

Fuera de comillas simples, `\` elimina el significado especial del carácter siguiente:

```bash
printf '%s\n' \$HOME
touch unusual\ name.txt
```

Una barra invertida seguida inmediatamente de una nueva línea permite continuar un comando sin incluir esa nueva línea en el argumento.

### Comentarios

En una shell no interactiva, un `#` no citado que comienza una palabra introduce un comentario hasta el final de la línea:

```bash
printf '%s\n' hello  # comentario explicativo
```

Dentro de comillas se conserva literalmente:

```bash
printf '%s\n' '#not-a-comment'
```

### Expansiones de nombres

Los patrones de la shell se expanden antes de ejecutar el comando:

| Patrón | Coincidencia |
| --- | --- |
| `*` | Cualquier secuencia de caracteres. |
| `?` | Un carácter. |
| `[abc]` | Uno de los caracteres indicados. |
| `[!abc]` | Un carácter que no esté en el conjunto. |

Ejemplo:

```bash
printf '%s\n' *.txt
```

El programa recibe la lista ya expandida. Para que `find` interprete el patrón y no la shell, debe citarse:

```bash
find . -type f -name '*.txt'
```

Por defecto, `*` no coincide con nombres que comienzan con `.`. `find`, en cambio, recorre también entradas ocultas si están bajo el punto de inicio.

### Otros caracteres importantes

| Carácter | Significado habitual sin citar |
| :---: | --- |
| `|` | Conecta una tubería. |
| `>` / `<` | Introduce una redirección. |
| `;` | Separa comandos. |
| `&` | Envía una operación al segundo plano: la shell no espera a que termine y continúa leyendo el siguiente comando mientras esa operación sigue corriendo. |
| `$` | Inicia una expansión. |
| `~` | Expande un directorio personal al comienzo de una palabra. |
| `()` | Crea un grupo en una subshell (una copia del proceso de la shell que ejecuta ese grupo de comandos de forma aislada: `(cd /tmp; pwd)` cambia de directorio solo dentro del paréntesis, y no afecta al `pwd` posterior) o participa en otras construcciones. |

Las comillas invertidas representan una forma antigua de sustitución de comandos: la shell ejecuta ese comando en un subproceso, espera a que termine y sustituye la construcción por lo que ese comando escribió en su salida estándar (sin las nuevas líneas finales). `$(command)` hace lo mismo, pero es más legible y permite anidamiento claro:

```bash
printf '%s\n' "Directorio actual: $(pwd)"
```

### Nombres que empiezan con `-`

Muchos programas aceptan `--` como final de las opciones:

```bash
cat -- -notes.txt
```

Otra posibilidad es incluir una ruta:

```bash
cat ./-notes.txt
```

## Producir texto: `echo` y `printf`

### `echo`

`echo` muestra sus argumentos separados por espacios y normalmente agrega una nueva línea:

```bash
echo 'Hello, World'
```

Su tratamiento de opciones y secuencias con barra invertida presenta diferencias históricas entre implementaciones. Es adecuado para salidas sencillas y controladas.

### `printf`

`printf` ofrece un formato predecible:

```bash
printf '%s\n' 'Hello, World'
printf 'name=%s count=%d\n' "$name" "$count"
```

El primer argumento es el formato. Los datos se pasan como argumentos posteriores, en lugar de incorporarlos directamente al formato:

```bash
printf '%s\n' "$user_input"
```

Así se evita interpretar por accidente caracteres `%` presentes en datos externos.

## Scripts ejecutables y nueva línea final

Un script Bash identifica su intérprete mediante el *shebang*:

```bash
#!/bin/bash
printf '%s\n' 'hello'
```

Después de guardarlo, puede habilitarse la ejecución directa:

```bash
chmod u+x script.sh   # otorga permiso de ejecución al usuario dueño del archivo
./script.sh
```

Los archivos de texto deben terminar normalmente con una nueva línea (también llamada salto de línea). Esto permite que herramientas orientadas a líneas procesen el último registro de la misma manera que los anteriores.

`wc -l` cuenta caracteres `\n`. Por eso, dos líneas visibles sin nueva línea después de la segunda pueden contarse como una sola. `od` (octal dump) muestra el contenido de datos en un formato distinto al texto plano, por ejemplo hexadecimal (`-t x1`); útil para inspeccionar bytes que no se ven en pantalla:

```bash
wc -l script.sh
tail -c 1 script.sh | od -An -t x1
```

El último comando mostrará habitualmente `0a` cuando el byte final sea una nueva línea Unix.

## Leer y concatenar con `cat`

`cat` escribe el contenido de uno o más archivos en orden:

```bash
cat first.txt second.txt
```

Sin nombres de archivo, copia `stdin` a `stdout`:

```bash
cat < input.txt
```

Opciones útiles de GNU `cat`:

| Opción | Efecto |
| :---: | --- |
| `-n` | Numera todas las líneas. |
| `-b` | Numera únicamente líneas no vacías. |
| `-E` | Muestra `$` al final de cada línea. |
| `-T` | Representa tabulaciones como `^I`. |
| `-A` | Hace visibles varios caracteres no imprimibles. |

`cat -E` ayuda a comprobar si una salida termina con nueva línea:

```bash
printf '%s\n' 'hello' | cat -E
```

```text
hello$
```

## Seleccionar líneas con `head` y `tail`

`head` muestra el comienzo de la entrada:

```bash
head -n 5 access.log
```

`tail` muestra el final:

```bash
tail -n 5 access.log
```

Si no se especifica una cantidad, ambos muestran normalmente diez líneas.

Para obtener una línea concreta pueden combinarse filtros:

```bash
head -n 7 access.log | tail -n 1
```

Las líneas vacías también cuentan.

Formas útiles adicionales:

```bash
tail -n +5 access.log
head -n -5 access.log
tail -f application.log
```

- `tail -n +5` comienza en la quinta línea.
- En GNU `head`, `-n -5` omite las últimas cinco líneas.
- `tail -f` sigue un archivo mientras crece; se detiene normalmente con `Ctrl+C`.

## Buscar archivos con `find`

La forma general es:

```text
find STARTING_POINT... EXPRESSION
```

Ejemplos de selección:

```bash
find . -type f
find . -type d
find . -type f -name '*.log'
find . -type f -iname '*.jpg'
find . -mindepth 1 -type d
find . -maxdepth 1 -type f
find . -empty
```

| Expresión | Significado |
| --- | --- |
| `-type f` | Archivo regular. |
| `-type d` | Directorio. |
| `-name PATTERN` | Coincidencia sensible a mayúsculas. |
| `-iname PATTERN` | Coincidencia sin distinguir mayúsculas. |
| `-mindepth N` | No aplica acciones antes del nivel indicado. |
| `-maxdepth N` | No desciende más allá del nivel indicado. |
| `-empty` | Archivo vacío o directorio sin entradas. |

El punto de inicio `.` también es un resultado potencial. `-mindepth 1` permite excluirlo manteniendo sus descendientes.

### Formatos de salida

`-print` muestra la ruta encontrada:

```bash
find . -type f -print
```

GNU `find` admite `-printf`:

```bash
find . -type f -printf '%f\n'
```

Algunas directivas:

| Directiva | Resultado |
| :---: | --- |
| `%p` | Ruta completa encontrada. |
| `%f` | Nombre base, sin directorios anteriores. |
| `%h` | Directorios anteriores al nombre. |
| `%s` | Tamaño en bytes. |
| `%T@` | Fecha de modificación en segundos desde la época Unix (1 de enero de 1970 UTC), útil para ordenar por antigüedad. |

Para ordenar resultados por fecha de modificación, una alternativa más directa es `ls -t` (lista de más reciente a más antiguo):

```bash
ls -t
```

### Eliminación segura

`-delete` borra directamente los resultados seleccionados:

```bash
find . -type f -name '*.tmp' -delete
```

Antes de borrar, debe ejecutarse la misma selección con `-print`:

```bash
find . -type f -name '*.tmp' -print
```

Las expresiones de `find` se evalúan de izquierda a derecha. Colocar `-delete` antes de los filtros puede eliminar objetos que todavía no fueron descartados. El punto inicial también debe revisarse cuidadosamente.

Para acciones más complejas:

```bash
find . -type f -name '*.log' -exec wc -l -- {} +
```

`{}` representa los nombres encontrados y `+` agrupa varios por ejecución.

## Contar con `wc`

`wc` cuenta elementos del texto:

| Opción | Cuenta |
| :---: | --- |
| `-l` | Nuevas líneas. |
| `-w` | Palabras. |
| `-c` | Bytes. |
| `-m` | Caracteres. |

Ejemplos:

```bash
wc -l access.log
find . -mindepth 1 -type d -print | wc -l
printf '%s\n' 'one two three' | wc -w
```

Cuando recibe un nombre de archivo, `wc` también imprime ese nombre. En una tubería muestra únicamente el conteo.

`wc -l` cuenta caracteres de nueva línea, no una noción abstracta de líneas. Un archivo no vacío cuya última línea no termina en `\n` puede producir un conteo menor de lo esperado.

## Ordenar con `sort`

`sort` ordena líneas completas:

```bash
sort names.txt
```

Opciones frecuentes:

| Opción | Efecto |
| :---: | --- |
| `-r` | Orden inverso. |
| `-n` | Comparación numérica. |
| `-f` | Ignora diferencias de mayúsculas y minúsculas. |
| `-u` | Conserva una sola línea de cada valor equivalente. |
| `-t CHAR` | Define el separador de campos. |
| `-k KEY` | Elige una clave o rango de campos. |

Ejemplos:

```bash
sort -n measurements.txt
sort -nr counts.txt
sort -t: -k1,1 accounts.txt
```

### Configuración regional

El orden depende de variables como `LC_COLLATE` y `LC_ALL`. Un orden lingüístico puede tratar acentos, signos y mayúsculas de manera diferente a un orden por valores de byte.

Para un resultado reproducible basado en bytes:

```bash
LC_ALL=C sort names.txt
LC_ALL=C sort -f names.txt
```

Definir la locale dentro de una tubería afecta únicamente al comando al que precede:

```bash
producer | LC_ALL=C sort -f
```

## Agrupar con `uniq`

`uniq` compara líneas adyacentes. No localiza duplicados dispersos por todo el archivo:

```bash
sort words.txt | uniq
```

Opciones comunes:

| Opción | Resultado |
| :---: | --- |
| `-c` | Anteponer el número de repeticiones. |
| `-d` | Mostrar valores repetidos. |
| `-u` | Mostrar valores que aparecen exactamente una vez. |
| `-i` | Ignorar diferencias entre mayúsculas y minúsculas. |

Ejemplos:

```bash
sort words.txt | uniq -u
sort words.txt | uniq -d
sort words.txt | uniq -c | sort -nr
```

`sort -u` conserva una copia de cada valor, pero no equivale a `uniq -u`: esta última descarta cualquier valor que aparezca más de una vez.

## Buscar patrones con `grep`

`grep` busca líneas que contienen un patrón de texto (literal o expresión regular) y las imprime; puede leer de uno o más archivos o de la entrada estándar.

La forma básica es:

```bash
grep 'pattern' file
```

Sin archivo, `grep` lee `stdin`:

```bash
producer | grep 'pattern'
```

Opciones útiles:

| Opción | Efecto |
| :---: | --- |
| `-i` | Ignora mayúsculas y minúsculas. |
| `-v` | Selecciona líneas que no coinciden. |
| `-c` | Cuenta líneas coincidentes. |
| `-n` | Agrega el número de línea. |
| `-A N` | Incluye `N` líneas posteriores. |
| `-B N` | Incluye `N` líneas anteriores. |
| `-C N` | Incluye contexto anterior y posterior. |
| `-F` | Interpreta el patrón como texto literal. |
| `-E` | Usa expresiones regulares extendidas. |

`grep -c` cuenta líneas que contienen al menos una coincidencia, no el número total de apariciones. Para contar cada aparición con GNU `grep`:

```bash
grep -o 'pattern' file | wc -l
```

### Anclas y clases de caracteres

En una expresión regular:

| Expresión | Significado |
| --- | --- |
| `^` | Comienzo de línea. |
| `$` | Final de línea. |
| `.` | Cualquier carácter individual. |
| `[abc]` | Un carácter del conjunto. |
| `[^abc]` | Un carácter fuera del conjunto. |
| `[[:alpha:]]` | Un carácter alfabético según la locale. |
| `[[:digit:]]` | Un dígito según la locale. |

Para seleccionar líneas que comienzan con una letra:

```bash
grep '^[[:alpha:]]' config.txt
```

Las comillas evitan que la shell interprete partes del patrón. Si se necesita buscar texto literal que contiene metacaracteres, `grep -F` suele ser más claro:

```bash
grep -F 'price=$5.00' catalog.txt
```

### Contexto y separadores

```bash
grep -A 3 'ERROR' application.log
```

Cuando existen grupos no contiguos, GNU `grep` puede separarlos con una línea `--`. Eso forma parte de la salida normal de las opciones de contexto.

### Patrones que comienzan con guion

Puede utilizarse `-e` o `--`:

```bash
grep -e '-draft' file
grep -- '-draft' file
```

## Transformar caracteres con `tr`

`tr` lee exclusivamente de `stdin`; no recibe un nombre de archivo como operando de entrada.

### Sustitución

```bash
tr 'abc' 'xyz' < input.txt
```

Cada carácter del primer conjunto se corresponde por posición con uno del segundo: `a` pasa a `x`, `b` a `y` y `c` a `z`.

Para convertir mayúsculas y minúsculas:

```bash
tr '[:lower:]' '[:upper:]' < input.txt
```

### Eliminación

`-d` elimina los caracteres indicados:

```bash
tr -d 'Cc' < input.txt
```

### Compactación

`-s` reduce repeticiones consecutivas:

```bash
tr -s '[:space:]' ' ' < input.txt
```

Esto transforma secuencias de espacios, tabulaciones o nuevas líneas en un solo espacio. Si las nuevas líneas tienen significado, debe utilizarse un conjunto más específico.

`tr` transforma caracteres, no palabras ni expresiones regulares. Su comportamiento con rangos y caracteres multibyte puede depender de la locale; `LC_ALL=C` resulta útil cuando se requiere procesamiento byte a byte.

## Invertir líneas con `rev`

`rev` invierte por separado los caracteres de cada línea:

```bash
printf '%s\n' 'Reverse' | rev
```

```text
esreveR
```

Puede ser útil para procesar componentes desde el final de una cadena, aunque no reemplaza una herramienta que comprenda rutas, extensiones o estructuras de datos.

## Extraer campos con `cut`

`cut` selecciona bytes, caracteres o campos de cada línea:

```bash
cut -c 1-8 file.txt
cut -d: -f1,6 accounts.txt
```

Opciones principales:

| Opción | Selección |
| :---: | --- |
| `-b LIST` | Bytes. |
| `-c LIST` | Caracteres. |
| `-d CHAR` | Delimitador de campos. |
| `-f LIST` | Campos. |
| `-s` | Omite líneas que no contienen el delimitador. |

El delimitador de `cut` es un solo carácter. Para datos TSV se utiliza una tabulación; Bash puede representarla con una cadena ANSI-C:

```bash
cut -f1 access.tsv
cut -d $'\t' -f1 access.tsv
```

Sin `-d`, `cut -f` usa tabulación de manera predeterminada.

### Eliminar la última extensión

Dividir directamente por `.` puede fallar con nombres ocultos o con varios puntos. Para un flujo controlado de nombres puede procesarse desde el final:

```bash
printf '%s\n' 'archive.tar.gz' | rev | cut -d. -f2- | rev
```

```text
archive.tar
```

Este procedimiento opera sobre texto; no interpreta rutas y no es seguro para nombres que contienen nuevas líneas.

## Unir líneas con `paste`

`paste` combina líneas de uno o más flujos. En GNU `paste`, el modo serial puede unir todas las líneas usando un delimitador indicado:

```bash
cut -c 1 poem.txt | paste -sd ''
```

El resultado conserva una nueva línea final. Para unir con comas:

```bash
paste -sd, values.txt
```

Esta herramienta es útil cuando una transformación necesita convertir varias líneas en una sola sin perder el terminador final.

## `/etc/passwd` y `/etc/shadow`

### Formato de `/etc/passwd`

Cada línea representa una cuenta mediante siete campos separados por `:`:

```text
login:password:uid:gid:gecos:home:shell
```

| Campo | Contenido |
| --- | --- |
| `login` | Nombre de la cuenta. |
| `password` | Normalmente `x`, indicando que el dato protegido está en otro archivo. |
| `uid` | Identificador numérico del usuario. |
| `gid` | Identificador numérico del grupo primario. |
| `gecos` | Información descriptiva. |
| `home` | Directorio personal. |
| `shell` | Programa de inicio de sesión. |

Puede extraerse una selección de campos:

```bash
cut -d: -f1,6 /etc/passwd
```

El contenido depende del sistema y de los paquetes instalados. No deben codificarse nombres de cuentas ni cantidades observadas en otro equipo.

### Formato de `/etc/shadow`

`/etc/shadow` almacena información protegida de autenticación y caducidad:

```text
login:password:lastchange:min:max:warn:inactive:expire:reserved
```

Su lectura está restringida normalmente a `root` y procesos autorizados. Los valores de contraseña son hashes o marcadores de estado, no contraseñas en texto plano.

No se debe copiar, publicar ni incorporar el contenido de `/etc/shadow` a registros o repositorios. Para conocer su formato se consulta:

```bash
man 5 shadow
```

## Procesar nombres de archivo de forma robusta

Las herramientas orientadas a líneas suponen que cada registro termina en `\n`. Linux permite nuevas líneas dentro de nombres de archivo, por lo que una tubería basada en `-print` puede volverse ambigua.

`xargs` toma los elementos recibidos por la entrada estándar y los pasa como argumentos a otro comando; `-0` indica que los registros están separados por byte nulo en lugar de nueva línea.

Para operaciones internas seguras, GNU `find` y otras herramientas admiten registros terminados en byte nulo:

```bash
find . -type f -print0 | xargs -0 command
```

Cuando sea posible, `-exec ... {} +` evita incluso esa conversión:

```bash
find . -type f -exec command -- {} +
```

Si el formato de salida exige un nombre por línea, los nombres que contienen nuevas líneas no pueden representarse sin escapar o establecer una convención adicional.

## Diseñar una tubería

Antes de escribir comandos conviene definir el contrato de cada etapa:

| Pregunta | Ejemplo |
| --- | --- |
| ¿Cuál es la fuente? | Archivo, `stdin`, listado de `find`. |
| ¿Cuál es la unidad? | Byte, carácter, palabra, línea, campo o archivo. |
| ¿Cuál es el delimitador? | Nueva línea, `:`, tabulación o byte nulo. |
| ¿Debe conservarse el orden? | Orden original, alfabético, numérico o temporal. |
| ¿Cómo se manejan duplicados? | Conservar, eliminar, contar o mostrar solo valores únicos. |
| ¿Dónde van los errores? | Terminal, archivo separado o flujo combinado. |

Una estrategia práctica es probar cada etapa con una entrada pequeña:

```bash
producer
producer | first_filter
producer | first_filter | second_filter
```

Después se agrega la redirección final. Esto permite localizar con claridad la transformación que introduce un resultado inesperado.

## Problemas frecuentes

### El archivo de salida queda vacío

Puede haberse usado `>` sobre el mismo archivo que se quería leer, o el programa no produjo `stdout`.

```bash
command > output.txt
printf 'status=%s\n' "$?"
wc -c output.txt
```

### Un mensaje sigue apareciendo en la pantalla

Probablemente pertenece a `stderr`. Debe decidirse si corresponde guardarlo, combinarlo o corregir su causa.

### `uniq` deja duplicados

Las líneas iguales no eran adyacentes. Se necesita ordenar o agrupar previamente:

```bash
sort input.txt | uniq
```

### Un patrón de `find` produce resultados inesperados

El patrón puede haber sido expandido por la shell. Debe citarse:

```bash
find . -name '*.log'
```

### El orden cambia entre equipos

La locale es diferente. Para un orden de bytes reproducible:

```bash
LC_ALL=C sort input.txt
```

### Un conteo no coincide

Debe confirmarse qué se cuenta: líneas, coincidencias, palabras, bytes, caracteres u objetos. `grep -c` y `wc -l` resuelven preguntas distintas.

### Una variable rompe un comando

La expansión no estaba citada:

```bash
cat -- "$filename"
```

## Hábitos seguros y útiles

- Consultar `man` o `--help` para conocer la versión instalada.
- Probar las selecciones de `find` con `-print` antes de usar `-delete` o `-exec` con una acción destructiva.
- Citar variables, rutas y patrones que no deba expandir la shell.
- Utilizar `--` cuando un operando pueda comenzar con `-`.
- Mantener separados datos y diagnósticos cuando el formato de una tubería sea importante.
- Usar `LC_ALL=C` cuando se requiera un orden de bytes reproducible.
- Evitar interpretar datos externos como fragmentos de shell.
- Comprobar la nueva línea final cuando la salida tenga un formato exacto.
- No asumir que `/etc/passwd`, `/etc/shadow` o los archivos de configuración tienen el mismo contenido en todos los equipos.

## Consultar la documentación instalada

Las páginas locales corresponden a las versiones reales del sistema:

```bash
help echo
help printf
man bash
man cat
man head
man tail
man find
man wc
man sort
man uniq
man grep
man tr
man rev
man cut
man paste
man 5 passwd
man 5 shadow
```

Dentro de `man`, `/pattern` busca texto, `n` repite la búsqueda y `q` sale.

## Recursos

### Introducción

- [Learning the Shell: I/O Redirection](https://linuxcommand.org/lc3_lts0070.php)
- [BashGuide: Special Characters](https://mywiki.wooledge.org/BashGuide/SpecialCharacters)

### Documentación principal

- [Bash Reference Manual: Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)
- [Bash Reference Manual: Pipelines](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html)
- [Bash Reference Manual: Quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/coreutils.html)
- [GNU Grep Manual](https://www.gnu.org/software/grep/manual/grep.html)
- [GNU Findutils Manual](https://www.gnu.org/software/findutils/manual/find.html)

### Referencia para Ubuntu 20.04 LTS

- [Ubuntu Manpage: `bash`](https://manpages.ubuntu.com/manpages/focal/man1/bash.1.html)
- [Ubuntu Manpage: `cat`](https://manpages.ubuntu.com/manpages/focal/man1/cat.1.html)
- [Ubuntu Manpage: `sort`](https://manpages.ubuntu.com/manpages/focal/man1/sort.1.html)
- [Ubuntu Manpage: `find`](https://manpages.ubuntu.com/manpages/focal/man1/find.1.html)
- [Ubuntu Manpage: `grep`](https://manpages.ubuntu.com/manpages/focal/man1/grep.1.html)
- [Ubuntu Manpage: `paste`](https://manpages.ubuntu.com/manpages/focal/man1/paste.1.html)
- [Ubuntu Manpage: `rev`](https://manpages.ubuntu.com/manpages/focal/man1/rev.1.html)
- [Ubuntu Manpage: `passwd(5)`](https://manpages.ubuntu.com/manpages/focal/man5/passwd.5.html)
- [Ubuntu Manpage: `shadow(5)`](https://manpages.ubuntu.com/manpages/focal/man5/shadow.5.html)

## Conclusión

Las redirecciones determinan de dónde lee un proceso y adónde escribe; las tuberías conectan procesos; y los filtros convierten flujos de texto mediante operaciones pequeñas y combinables. Comprender `stdin`, `stdout`, `stderr`, el orden de las redirecciones y el citado de caracteres permite predecir el comportamiento de la shell antes de ejecutar un comando.

El paso siguiente es pensar en términos de formatos: qué representa cada línea, qué delimitador separa los campos, qué orden se necesita y cómo se tratarán los errores. Con ese enfoque, comandos como `find`, `grep`, `sort`, `uniq`, `cut`, `tr`, `head`, `tail` y `wc` forman una caja de herramientas coherente para explorar y transformar datos.
