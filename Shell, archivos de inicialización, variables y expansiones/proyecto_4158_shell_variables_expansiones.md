# Archivos de inicialización, variables y expansiones en Bash

## De una línea escrita a un comando ejecutado

Bash no entrega literalmente cada carácter escrito al programa. Primero interpreta la línea, reconoce su estructura y transforma ciertas expresiones. De forma simplificada, el recorrido es:

1. lee una línea de entrada;
2. la divide en palabras y operadores, respetando las comillas;
3. reconoce alias y construcciones de la shell;
4. analiza la sintaxis;
5. realiza las expansiones correspondientes;
6. aplica las redirecciones;
7. resuelve el nombre del comando y lo ejecuta;
8. guarda su estado de salida y, si la sesión es interactiva, vuelve a mostrar el prompt.

El orden importa. Un alias se reconoce antes de resolver si el nombre corresponde a una función, un comando interno o un ejecutable. Las expansiones, por su parte, pueden convertir una sola palabra escrita en varios argumentos.

### Qué ocurre con `ls -l *.txt`

Supongamos que el directorio actual contiene `notes.txt`, `two words.txt` y `image.png`.

1. Bash reconoce las palabras `ls`, `-l` y `*.txt`.
2. Si `ls` es un alias, sustituye ese primer token por su valor.
3. La expansión de nombres de archivo compara `*.txt` con las entradas del directorio actual.
4. El patrón se transforma en dos argumentos: `notes.txt` y `two words.txt`. El espacio del segundo nombre no lo divide, porque el resultado de la expansión ya es un nombre de archivo completo.
5. Bash resuelve el comando. Busca una función con ese nombre, luego un comando interno y, finalmente, un ejecutable en los directorios de `PATH`.
6. El programa recibe una lista equivalente a `-l`, `notes.txt` y `two words.txt`; nunca necesita interpretar el asterisco.
7. Bash espera la finalización del proceso en una ejecución normal en primer plano, guarda su estado en `$?` y vuelve a expandir `PS1` para mostrar el prompt.

Por defecto, `*` no incluye nombres que comienzan con `.`. Si no existe ninguna coincidencia, Bash conserva normalmente el patrón literal. Las opciones `nullglob`, `failglob` y `dotglob` permiten cambiar esos comportamientos:

```bash
shopt -s nullglob   # un patrón sin coincidencias desaparece
shopt -s failglob   # un patrón sin coincidencias produce un error
shopt -s dotglob    # * también puede coincidir con nombres ocultos
```

Las opciones activas se consultan con `shopt`. Un script que dependa de alguna de ellas debe establecerla explícitamente, en lugar de asumir la configuración de la sesión del usuario.

## Sesiones y archivos de inicialización

Los archivos que Bash lee dependen de dos propiedades diferentes:

- una shell **interactiva** acepta comandos desde una terminal y muestra un prompt;
- una shell **de inicio de sesión** representa el comienzo de una sesión autenticada o fue invocada con la opción `--login`.

Una shell puede ser interactiva sin ser de inicio de sesión. Es el caso habitual al abrir una terminal dentro de un entorno gráfico.

### Shell interactiva de inicio de sesión

Bash lee, en este orden:

1. `/etc/profile`;
2. el primero que exista entre `~/.bash_profile`, `~/.bash_login` y `~/.profile`.

No lee los tres archivos personales: se detiene en el primero disponible. Es frecuente que `~/.profile` o `~/.bash_profile` cargue expresamente `~/.bashrc` para compartir la configuración interactiva:

```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

Al terminar una shell de inicio de sesión, Bash intenta leer `~/.bash_logout`. Algunos sistemas agregan archivos globales propios.

### Shell interactiva que no es de inicio de sesión

Bash lee `~/.bashrc`. Allí suelen definirse:

- alias y funciones de uso interactivo;
- el prompt;
- opciones de Bash y de historial;
- completado y otras personalizaciones de la terminal.

En Ubuntu también existe `/etc/bash.bashrc` como configuración interactiva global. Su participación es una decisión de la distribución y no debe confundirse con las reglas generales de Bash.

### Shell no interactiva

Un script normal no lee `~/.bashrc`. Para una shell no interactiva, Bash consulta la variable `BASH_ENV`; si contiene el nombre de un archivo, lo expande y lo carga antes de ejecutar el script.

Depender de la configuración personal vuelve frágil a un script. Las variables, opciones y funciones imprescindibles deben declararse en el propio script o recibirse mediante una interfaz documentada.

### `/etc/profile.d` y `/etc/inputrc`

El directorio `/etc/profile.d` no es leído automáticamente por una regla interna de Bash. Habitualmente `/etc/profile` recorre los archivos `.sh` de ese directorio para dividir la configuración global en piezas mantenibles.

`/etc/inputrc` cumple otra función: configura **Readline**, la biblioteca que ofrece edición de línea, historial y atajos de teclado. Su contenido no es código Bash y no debe cargarse con `source`.

### Averiguar qué clase de shell está activa

`case` compara un valor contra una lista de patrones y ejecuta el bloque del primero que coincide, terminado en `;;`; `esac` cierra la construcción:

```bash
case $- in
    *i*) printf '%s\n' 'interactive' ;;
    *)   printf '%s\n' 'non-interactive' ;;
esac

shopt -q login_shell
printf 'status: %d\n' "$?"
```

`$-` contiene las opciones activas de la shell; la letra `i` indica interactividad. `shopt -q login_shell` devuelve éxito si se trata de una shell de inicio de sesión.

## Ejecutar un archivo o cargarlo en la sesión actual

Estas operaciones no son equivalentes:

```bash
./settings.sh
source settings.sh
. settings.sh
```

`./settings.sh` crea otro proceso de shell. Sus cambios de directorio, variables no exportadas, alias y funciones desaparecen cuando termina.

`source` y `.` leen el archivo en la shell actual. Por eso pueden modificar la sesión desde la que se invocan. `source` es una forma propia de Bash y otras shells; `.` es la forma especificada por POSIX.

Antes de cargar cambios de configuración conviene revisar la sintaxis:

```bash
bash -n ~/.bashrc
```

`bash -n` analiza sin ejecutar. Después se puede abrir una terminal nueva para probar la configuración sin arriesgar la sesión que todavía funciona. Si se utiliza `source`, cualquier `exit`, redirección persistente o cambio de directorio del archivo afecta inmediatamente a la shell actual.

## Variables de shell y variables de entorno

Una variable de shell pertenece a una instancia de Bash:

```bash
course='Linux fundamentals'
count=3
```

No puede haber espacios alrededor de `=`. Con espacios, Bash interpreta la primera palabra como un comando.

```bash
name = value   # intenta ejecutar un comando llamado name
```

Una variable de entorno es una variable marcada para ser heredada por los procesos hijos:

```bash
export EDITOR='vim'
export course
```

También se puede crear y exportar en una sola operación:

```bash
export LANG='en_US.UTF-8'
```

Un proceso hijo recibe una copia del entorno. Puede modificar su copia, pero no puede alterar el entorno de su padre. Por eso ejecutar un script no cambia las variables de la terminal que lo inició; cargarlo con `source` sí puede hacerlo.

### Dos significados de “local”

En explicaciones introductorias, *variable local* suele significar una variable de shell no exportada. Bash también posee el comando interno `local`, que crea una variable con alcance limitado a una función:

```bash
show_user() {
    local label='current user'
    printf '%s: %s\n' "$label" "$USER"
}
```

Son conceptos relacionados, pero no idénticos. Una variable de shell definida fuera de una función continúa disponible en toda esa instancia de Bash aunque no forme parte del entorno.

### Modificar, borrar y proteger variables

```bash
color='blue'
color='green'
unset color

readonly API_VERSION='1'
```

`unset` elimina una variable o función de la shell actual. `readonly` impide modificar o eliminar un nombre durante esa sesión.

Los nombres habituales usan letras, números y guion bajo, sin comenzar con un número. Las variables internas y de entorno suelen escribirse en mayúsculas; usar minúsculas para variables propias reduce el riesgo de sobrescribir nombres con significado especial.

### Inspeccionar el estado

| Comando | Información principal |
| --- | --- |
| `printenv` | Variables exportadas que forman el entorno. |
| `env` | Entorno o ejecución de un comando con un entorno modificado. |
| `set` | Variables de shell, variables exportadas y funciones, además de otras salidas dependientes de Bash. |
| `declare -p` | Declaraciones de variables con atributos y valores reutilizables. |
| `export -p` | Variables marcadas para exportación. |
| `unset name` | Elimina la variable `name`. |

`set` sin argumentos puede producir una salida extensa y también revelar valores sensibles. No conviene publicarla completa en registros o capturas.

## Variables y parámetros especiales

Bash mantiene nombres cuyo valor tiene una función definida (también llamadas variables reservadas). Algunos de los más frecuentes son:

| Nombre | Función |
| --- | --- |
| `HOME` | Directorio personal del usuario. |
| `PATH` | Lista ordenada de directorios donde se buscan ejecutables. |
| `PWD` | Directorio de trabajo actual. |
| `OLDPWD` | Directorio anterior usado por `cd -`. |
| `PS1` | Formato del prompt principal en una shell interactiva. |
| `SHELL` | Shell de inicio configurada para el usuario; no garantiza cuál está ejecutándose ahora. |
| `USER` | Nombre de usuario, cuando el entorno lo proporciona (no siempre está disponible; `whoami` o `id -un` son alternativas más confiables). |
| `IFS` | Caracteres usados en ciertas operaciones de separación de palabras. |
| `BASH_VERSION` | Versión de Bash activa. |
| `BASH_SOURCE` | Archivos de origen de funciones y scripts en la pila de llamadas (la secuencia de funciones/scripts que se invocaron unos a otros hasta llegar al punto actual). |

`PS1` es una variable de shell. No necesita estar exportada y normalmente no lo está. Puede incluir escapes como `\u` para el usuario, `\h` para el host y `\w` para el directorio actual.

Los parámetros especiales representan información dinámica:

| Parámetro | Significado |
| :---: | --- |
| `$?` | Estado de salida de la última tubería ejecutada en primer plano. |
| `$$` | PID de la shell. |
| `$!` | PID del último proceso en segundo plano. |
| `$#` | Cantidad de argumentos posicionales. |
| `$0` | Nombre usado para invocar la shell o el script. |
| `$1`…`$9` | Argumentos posicionales individuales. |
| `"$@"` | Todos los argumentos, conservando uno por elemento. |
| `"$*"` | Todos los argumentos unidos en una sola palabra. |
| `$_` | Último argumento del comando anterior, con detalles que dependen del contexto. |

El valor de `$?` cambia después de cada comando, por lo que debe guardarse inmediatamente si se necesita más tarde:

```bash
some_command
status=$?
printf 'exit status: %d\n' "$status"
```

Por convención, `0` significa éxito y un valor distinto de cero indica que la operación no se completó normalmente.

## `PATH`: búsqueda de comandos

`PATH` contiene directorios separados por dos puntos:

```text
/usr/local/bin:/usr/bin:/bin
```

Bash los consulta de izquierda a derecha. Si dos directorios contienen un ejecutable con el mismo nombre, gana el primero. Para añadir una ubicación sin perder el contenido existente:

```bash
PATH="$PATH:$HOME/bin"     # la añade al final
PATH="$HOME/bin:$PATH"     # la añade al principio
export PATH
```

Usar `$HOME/bin` es preferible a escribir `~/bin` dentro de comillas: la expansión de `~` no ocurre dentro de comillas dobles.

### Entradas vacías y seguridad

En `PATH`, una entrada vacía representa el directorio actual. Aparece con `::`, con `:` al comienzo o con `:` al final:

```text
/usr/bin::/bin:
```

Esto puede provocar que un programa del directorio actual se ejecute antes que el esperado. Es más claro y seguro usar rutas explícitas y evitar elementos vacíos. Agregar `.` al principio tiene el mismo riesgo; si fuera imprescindible, el final es menos peligroso que el principio.

Para inspeccionar solo las entradas no vacías de un valor convencional:

```bash
printf '%s\n' "$PATH" | tr -s ':' '\n' | grep -c .
```

Este enfoque trata el contenido como texto delimitado por `:`. No refleja que una entrada vacía tiene el significado especial de directorio actual: con `PATH="/usr/bin::/bin"`, `tr -s` colapsa el `::` y el conteo no distingue ese caso del de un `PATH` sin entradas vacías, aunque Bash sí trataría esa entrada vacía como `.` al buscar un ejecutable.

Para saber qué se ejecutaría:

```bash
type -a python3
command -v python3
```

`type -a` también muestra alias, funciones, comandos internos y todas las coincidencias del `PATH`. Bash conserva en una tabla algunas búsquedas anteriores; `hash -r` descarta esa caché tras instalar, mover o reemplazar ejecutables.

## Alias

Un alias sustituye una palabra de comando por otro texto durante la lectura de una orden interactiva:

```bash
alias ll='ls -alF'
alias
alias ll
```

No se permiten espacios alrededor de `=`. Las comillas simples retrasan las expansiones hasta que se utiliza el alias:

```bash
alias where='printf "%s\n" "$PWD"'
```

Con comillas dobles, `$PWD` se expandiría al definir el alias y quedaría fijado al directorio de ese momento.

### Desactivar o eliminar un alias

```bash
\ls
command ls
unalias ls
unalias -a
```

La barra invertida evita la expansión de ese alias una vez. `command` omite funciones y continúa con comandos internos o externos. `unalias` cambia únicamente la sesión actual; si la definición está en `~/.bashrc`, reaparecerá al cargar nuevamente ese archivo hasta que se elimine allí.

### Límites y seguridad

Un alias puede ocultar un comando conocido, incluso con una operación destructiva. Antes de ejecutar una orden sensible, `type nombre` permite verificar qué resolverá Bash.

Los alias se expanden normalmente en shells interactivas. En una shell no interactiva la expansión está desactivada, salvo que se habilite `expand_aliases`. Para lógica con argumentos, condiciones o varias operaciones, una función resulta más clara y fiable:

```bash
mkcd() {
    mkdir -p -- "$1"
    cd -- "$1" || return
}
```

## Expansiones de Bash

Una expansión transforma palabras antes de ejecutar el comando. El orden general es:

1. expansión de llaves;
2. expansión de tilde;
3. expansión de parámetros y variables;
4. expansión aritmética;
5. sustitución de comandos;
6. separación de palabras;
7. expansión de nombres de archivo;
8. eliminación de comillas.

La expansión de parámetros, la sustitución de comandos y la expansión aritmética se realizan en la misma fase, de izquierda a derecha. Ciertas construcciones, como `"$@"` y los arreglos (listas de valores bajo un mismo nombre, por ejemplo `arr=(a b c)`), son excepciones capaces de producir varios resultados sin separación de palabras.

### Expansión de llaves

Genera texto combinatorio antes que las demás expansiones:

```bash
printf '%s\n' file.{txt,csv}
printf '%s\n' {a..d}{1..3}
printf '%s\n' image_{01..05}.png
```

La segunda línea genera doce palabras: `a1`, `a2`, `a3`, hasta `d3`. Las llaves no consultan el sistema de archivos. Para descartar del resultado combinaciones puntuales no deseadas, puede filtrarse la salida con `grep -v`.

Como esta expansión ocurre antes que la expansión de variables, los límites almacenados en variables no forman rangos de la manera que podría parecer:

```bash
start=1
end=3
printf '%s\n' {$start..$end}   # no genera 1, 2 y 3
```

### Expansión de tilde

Al comienzo de una palabra, `~` representa el directorio personal:

```bash
printf '%s\n' ~
printf '%s\n' ~root
```

No ocurre dentro de comillas dobles. Para construir rutas en código, `"$HOME/file"` suele ser más predecible que `"~/file"`.

### Expansión de parámetros

La forma básica obtiene el valor de una variable:

```bash
name='Ada'
printf '%s\n' "$name"
printf '%s\n' "${name}_notes"
```

Las llaves delimitan el nombre. También permiten establecer valores alternativos:

| Forma | Resultado cuando `var` está vacía o no definida |
| --- | --- |
| `${var:-default}` | Usa `default` sin modificar `var`. |
| `${var:=default}` | Asigna `default` y lo usa. |
| `${var:?message}` | Informa el error y detiene una shell no interactiva. |
| `${var:+alternate}` | Usa `alternate` solamente si `var` tiene valor. |

Otras operaciones útiles:

```bash
text='report.final.txt'
printf '%s\n' "${#text}"       # longitud
printf '%s\n' "${text#*.}"    # elimina la coincidencia inicial más corta
printf '%s\n' "${text##*.}"   # elimina la coincidencia inicial más larga
printf '%s\n' "${text%.*}"    # elimina la coincidencia final más corta
printf '%s\n' "${text:0:6}"   # subcadena en Bash
```

La regla general: `#`/`##` buscan el patrón desde el principio de la cadena y `%`/`%%` desde el final; la forma simple (`#`, `%`) quita la coincidencia más corta posible y la doble (`##`, `%%`) la más larga. El patrón usa la misma sintaxis que la expansión de nombres de archivo (comodines), no una expresión regular.

La expansión no valida automáticamente que un valor sea seguro, numérico o no vacío. Esa validación corresponde al código que lo consume.

### Sustitución de comandos

`command` se ejecuta en un subproceso independiente; Bash captura todo lo que ese subproceso escribe en su salida estándar mientras corre, y `$(command)` se sustituye por ese texto capturado (sin las nuevas líneas finales) una vez que termina. Por tratarse de un subproceso, una variable o un `cd` que ocurra dentro de `command` no persiste en la shell que hizo la sustitución.

```bash
kernel=$(uname -r)
printf 'kernel: %s\n' "$kernel"
```

Bash elimina todas las nuevas líneas finales del resultado. Las nuevas líneas internas permanecen. Si la sustitución queda sin comillas, su salida también puede sufrir separación de palabras y expansión de nombres de archivo:

```bash
printf '<%s>\n' "$(printf 'one two\n')"  # un argumento
printf '<%s>\n' $(printf 'one two\n')    # dos argumentos
```

La sintaxis histórica con comillas invertidas, `` `command` ``, todavía funciona, pero `$()` se anida con mayor claridad y tiene reglas de escape más comprensibles.

### Expansión de nombres de archivo

Después de la separación de palabras, Bash interpreta patrones no citados:

| Patrón | Coincidencia |
| --- | --- |
| `*` | Cero o más caracteres. |
| `?` | Exactamente un carácter. |
| `[abc]` | Un carácter del conjunto. |
| `[a-z]` | Un carácter del rango, sujeto a la configuración regional. |
| `[[:digit:]]` | Un carácter de la clase indicada. |

Para pasar un patrón literalmente, se lo cita:

```bash
printf '%s\n' '*.txt'
```

## Comillas y barra invertida

### Sin comillas

Una expansión sin comillas puede producir varios argumentos y activar patrones de nombres de archivo:

```bash
value='one two'
printf '<%s>\n' $value
```

Por eso la regla práctica es citar las expansiones que deben conservarse como un único argumento:

```bash
printf '<%s>\n' "$value"
```

### Comillas simples

Las comillas simples conservan literalmente todos los caracteres interiores:

```bash
printf '%s\n' '$HOME $(date) *.txt'
```

No se puede incluir una comilla simple dentro de una cadena delimitada por comillas simples. Se cierran las comillas, se agrega una comilla escapada y se vuelven a abrir:

```bash
printf '%s\n' 'It'\''s Bash'
```

### Comillas dobles

Las comillas dobles permiten expansión de parámetros, sustitución de comandos y expansión aritmética, pero impiden la separación de palabras y la expansión de nombres de archivo:

```bash
printf '%s\n' "$HOME" "$(pwd)" "$((6 * 7))"
```

Dentro de ellas, la barra invertida conserva un significado especial solo ante ciertos caracteres, entre ellos `$`, `` ` ``, `"`, `\` y una nueva línea.

### Barra invertida

Fuera de comillas, `\` elimina el significado especial del carácter siguiente:

```bash
printf '%s\n' \$HOME
printf '%s\n' file\ name.txt
```

Una barra invertida seguida inmediatamente por una nueva línea permite continuar una orden en la línea siguiente; ambos caracteres se eliminan antes del análisis.

## Aritmética entera

La expansión aritmética devuelve el resultado como texto:

```bash
printf '%d\n' "$((8 + 5))"
```

Dentro de `((...))` se pueden usar nombres de variables sin `$`:

```bash
width=8
height=5
printf '%d\n' "$((width * height))"
```

Los operadores principales son:

| Operador | Operación |
| :---: | --- |
| `+`, `-`, `*` | Suma, resta y multiplicación. |
| `/`, `%` | División entera y resto. |
| `**` | Potencia. |
| `++`, `--` | Incremento y decremento. |
| `<<`, `>>` | Desplazamiento de bits. |
| `&`, `^`, `\|` | Operaciones bit a bit. |
| `==`, `!=`, `<`, `<=`, `>`, `>=` | Comparaciones numéricas. |
| `&&`, `\|\|`, `!` | Operaciones lógicas dentro del contexto aritmético. |
| `=`, `+=`, `-=`, `*=`, `/=` | Asignaciones. |

Bash trabaja con enteros de ancho fijo propio de la plataforma (normalmente 64 bits). Al desbordarse, el valor da la vuelta (wraps) a un número fuera del rango esperado en lugar de producir un error, y no de manera portable entre plataformas. La división descarta la parte fraccionaria:

```bash
printf '%d\n' "$((7 / 2))"   # 3
printf '%d\n' "$((7 % 2))"   # 1
printf '%d\n' "$((2 ** 10))" # 1024
```

Dividir entre cero produce un error. Una variable vacía suele evaluarse como cero dentro de una expresión aritmética, lo que puede ocultar datos faltantes; es preferible validar las entradas.

### Bases numéricas

Bash acepta constantes con la forma `base#número`, para bases entre 2 y 64:

```bash
printf '%d\n' "$((2#101010))" # 42
printf '%d\n' "$((16#2A))"    # 42
```

Los dígitos se ordenan como `0-9`, `a-z`, `A-Z`, `@` y `_`. Para bases de hasta 36 no se distingue entre mayúsculas y minúsculas. Un literal que comienza con `0` puede interpretarse como octal, por lo que `08` es problemático: 8 no es un dígito octal válido (van de 0 a 7). Para datos decimales con ceros iniciales puede forzarse la base:

```bash
value='008'
printf '%d\n' "$((10#$value))"
```

### Alfabetos numéricos personalizados

Cuando los datos de entrada usan un alfabeto propio en vez de dígitos convencionales (por ejemplo, letras en lugar de `0-9`), no alcanza con `base#número`: una base nombrada con símbolos propios necesita una traducción previa. Si el alfabeto `abcd` representa los dígitos `0`, `1`, `2` y `3`, el flujo general es:

1. traducir `abcd` a `0123`;
2. interpretar el resultado con `4#...`;
3. realizar la operación;
4. convertir el resultado a la base de destino;
5. traducir sus dígitos al alfabeto de salida.

`tr` traduce o elimina caracteres de su entrada según dos conjuntos que se corresponden por posición (el carácter en la posición N del primer conjunto se reemplaza por el de la posición N del segundo); sirve, por ejemplo, para una correspondencia carácter por carácter:

```bash
encoded='bcda'
digits=$(printf '%s' "$encoded" | tr 'abcd' '0123')
value=$((4#$digits))
printf '%d\n' "$value"
```

Para bases comunes, `printf` genera representaciones octales o hexadecimales:

```bash
printf '%o\n' 42   # 52
printf '%x\n' 42   # 2a
printf '%X\n' 42   # 2A
```

`printf` no ofrece un especificador general para cualquier base; fuera de octal y hexadecimal se necesita un algoritmo de divisiones sucesivas o una herramienta apropiada.

## Formato numérico con `printf`

`printf` separa el formato de los datos y produce resultados predecibles:

```bash
printf '%s\n' 'plain text'
printf '%d\n' 42
printf '%08d\n' 42
printf '%.2f\n' 3.14159
printf '%x\n' 255
```

| Especificador | Uso |
| :---: | --- |
| `%s` | Cadena. |
| `%d` | Entero decimal. |
| `%o` | Entero octal. |
| `%x`, `%X` | Entero hexadecimal en minúsculas o mayúsculas. |
| `%f` | Número de punto flotante para formateo. |

`%.2f` redondea y muestra dos cifras decimales, pero esto no convierte la aritmética de Bash en aritmética de punto flotante. Bash puede delegar el formateo a `printf`; expresiones como `$((3.5 + 1))` siguen siendo inválidas.

Conviene mantener el formato como argumento constante y pasar los datos después:

```bash
printf '%s\n' "$user_input"
```

Usar una entrada desconocida como cadena de formato permitiría que secuencias como `%s` o `%q` alterasen la salida: por ejemplo, `printf "$user_input\n"` con `user_input='%s'` hace que `printf` intente consumir un argumento adicional que no se le pasó, produciendo una salida distinta a la esperada (o vacía) en vez del texto literal `%s`.

## Transformaciones de texto relacionadas

### ROT13 con `tr`

ROT13 sustituye cada letra por la que se encuentra trece posiciones más adelante. Aplicarlo dos veces recupera el texto original. Los caracteres que no son letras ASCII permanecen iguales:

```bash
printf '%s\n' 'Hello, World!' | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

El resultado es `Uryyb, Jbeyq!`. ROT13 es una codificación reversible, no un cifrado seguro.

### Agrupar líneas con `paste`

`paste` une líneas de una o más fuentes, colocando entre ellas un delimitador (por defecto, una tabulación); con `-` como fuente, lee de la entrada estándar en lugar de un archivo, y puede leer varias líneas consecutivas de la misma entrada usando `-` más de una vez:

```bash
printf '%s\n' one two three four five | paste - -
```

La salida contiene pares separados por tabulaciones. Al seleccionar el primer campo de cada par se conservan las líneas impares:

```bash
printf '%s\n' one two three four five | paste - - | cut -f1
```

Este método presupone que el tabulador no forma parte de las líneas originales. Cuando los datos pueden contener cualquier carácter, una estructura que lea líneas individualmente es más robusta.

## Scripts pequeños y reproducibles

El *shebang* indica qué intérprete debe usar el sistema al ejecutar un archivo directamente:

```bash
#!/bin/bash
```

El archivo necesita permiso de ejecución:

```bash
chmod u+x script_name
```

También debe terminar con una nueva línea. Esta convención facilita el procesamiento de texto, evita que el prompt quede unido a la última línea y permite que distintas herramientas concatenen archivos sin fusionar accidentalmente sus últimas líneas.

Para comprobar sintaxis, permisos, cantidad de líneas y el carácter final. `od` (octal dump) muestra el contenido de datos en un formato distinto al texto plano, por ejemplo hexadecimal (`-t x1`); útil para inspeccionar bytes que no se ven en pantalla:

```bash
bash -n script_name
test -x script_name
wc -l script_name
tail -c 1 script_name | od -An -t x1
```

El byte `0a` es la nueva línea en sistemas Unix.

## Buenas prácticas cotidianas

- Usar comillas dobles alrededor de expansiones que deban conservarse como un solo argumento.
- Preferir `printf` a `echo` cuando el formato importa.
- Consultar `type -a` antes de asumir qué comando se ejecutará.
- No colocar el directorio actual al comienzo de `PATH` y evitar entradas vacías.
- Mantener alias interactivos en `~/.bashrc`; usar funciones o scripts para lógica reutilizable.
- Comprobar archivos de configuración con `bash -n` antes de cargarlos.
- No imprimir el entorno completo en registros públicos: puede contener secretos.
- Validar datos antes de usarlos en aritmética, rutas, comandos o nombres de variables.
- Capturar `$?` inmediatamente si se necesita conservar el estado.
- Probar casos con valores vacíos, espacios, comodines, ceros iniciales y ausencia de coincidencias.

## Referencias

- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/)
- [Bash Startup Files](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html)
- [Shell Expansions](https://www.gnu.org/software/bash/manual/html_node/Shell-Expansions.html)
- [Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)
- [Shell Arithmetic](https://www.gnu.org/software/bash/manual/html_node/Shell-Arithmetic.html)
- [Aliases](https://www.gnu.org/software/bash/manual/html_node/Aliases.html)
- [GNU Coreutils: `tr`](https://www.gnu.org/software/coreutils/manual/html_node/tr-invocation.html)
- [GNU Coreutils: `paste`](https://www.gnu.org/software/coreutils/manual/html_node/paste-invocation.html)
- [GNU Coreutils: `cut`](https://www.gnu.org/software/coreutils/manual/html_node/cut-invocation.html)
- [Learning the Shell: Expansion](https://linuxcommand.org/lc3_lts0080.php)
- [Bash Guide for Beginners: Variables](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_02.html)
- [Bash Guide for Beginners: Shell initialization files](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_01.html)
