# Fundamentos de la shell: navegación y manipulación de archivos

## Introducción

En Linux, los archivos y directorios se organizan en una estructura jerárquica con forma de árbol. La parte superior de esta estructura es el directorio raíz, representado por `/`. A diferencia de Windows, Linux no organiza cada dispositivo mediante una letra de unidad: todos los dispositivos forman parte de un único árbol de directorios.

La shell permite recorrer esta estructura, consultar archivos y directorios, identificar tipos de archivos y realizar operaciones como copiar, mover, renombrar, crear o eliminar elementos.

## Navegación por el sistema de archivos

### Directorio de trabajo

La shell siempre se encuentra ubicada en un directorio, denominado **directorio de trabajo actual**. Para mostrar su ruta se utiliza `pwd`:

```bash
pwd
```

Al iniciar una sesión, normalmente el directorio de trabajo es el directorio personal del usuario, cuya ruta suele seguir el formato `/home/username`.

### Listar el contenido de un directorio

El comando `ls` muestra los archivos y subdirectorios contenidos en el directorio actual:

```bash
ls
```

También puede recibir la ruta de otro directorio:

```bash
ls /bin
```

### Cambiar de directorio

El comando `cd` permite cambiar el directorio de trabajo:

```bash
cd /usr/bin
```

Las rutas pueden ser de dos tipos:

- **Ruta absoluta:** comienza en el directorio raíz `/` e indica la ubicación completa, por ejemplo, `/usr/bin`.
- **Ruta relativa:** comienza en el directorio de trabajo actual, por ejemplo, `documents/reports`.

En las rutas relativas se utilizan dos referencias especiales:

- `.` representa el directorio actual.
- `..` representa el directorio padre.

Por ejemplo, el siguiente comando sube un nivel en la estructura de directorios:

```bash
cd ..
```

En la mayoría de los casos, `./` puede omitirse. Por ejemplo, `cd ./bin` y `cd bin` producen el mismo resultado si `bin` se encuentra dentro del directorio actual.

### Atajos de `cd`

| Comando | Resultado |
| --- | --- |
| `cd` | Regresa al directorio personal del usuario. |
| `cd ~username` | Accede al directorio personal del usuario indicado. |
| `cd -` | Regresa al directorio de trabajo anterior. |

## Inspección de archivos y directorios

### Opciones útiles de `ls`

La estructura general de muchos comandos de Linux es:

```text
command -options arguments
```

Las opciones modifican el comportamiento del comando y los argumentos indican los elementos sobre los que debe actuar.

| Comando | Resultado |
| --- | --- |
| `ls` | Lista el contenido del directorio actual. |
| `ls -l` | Muestra una lista detallada. |
| `ls -a` | Incluye los archivos ocultos. |
| `ls -la ..` | Muestra en formato detallado todo el contenido del directorio padre, incluidos los archivos ocultos. |
| `ls -l /etc /bin` | Muestra en formato detallado el contenido de ambos directorios. |

La salida de `ls -l` incluye, entre otros datos:

- Tipo de archivo y permisos.
- Usuario propietario.
- Grupo propietario.
- Tamaño en bytes.
- Fecha de la última modificación.
- Nombre del archivo o directorio.

El primer carácter permite distinguir algunos tipos de elementos: `-` representa un archivo regular y `d` representa un directorio.

### Visualizar archivos de texto con `less`

`less` permite consultar un archivo de texto una página a la vez sin modificarlo:

```bash
less text_file
```

Dentro de `less` se pueden utilizar los siguientes controles:

| Tecla o comando | Acción |
| --- | --- |
| `Page Up` o `b` | Retrocede una página. |
| `Page Down` o `Space` | Avanza una página. |
| `G` | Va al final del archivo. |
| `1G` | Va al comienzo del archivo. |
| `/text` | Busca `text` hacia adelante. |
| `n` | Repite la búsqueda anterior. |
| `h` | Muestra la ayuda. |
| `q` | Cierra `less`. |

### Identificar el tipo de un archivo

La extensión no determina necesariamente el contenido de un archivo en Linux. El comando `file` examina sus datos e informa qué tipo de archivo es:

```bash
file name_of_file
```

Puede identificar, entre otros, archivos de texto, scripts de Bash, ejecutables, bibliotecas compartidas, archivos comprimidos, documentos HTML e imágenes.

## Características importantes de los nombres de archivo

- Los nombres que comienzan con `.` se consideran ocultos y normalmente se muestran con `ls -a`.
- Linux distingue entre mayúsculas y minúsculas: `File1` y `file1` son nombres diferentes.
- Linux no depende de las extensiones para reconocer el tipo de archivo, aunque algunas aplicaciones sí las utilizan.
- Los nombres pueden contener espacios y signos de puntuación, pero es recomendable utilizar letras, números, puntos, guiones y guiones bajos para evitar problemas al escribir comandos.

## Comodines

Los comodines permiten seleccionar grupos de archivos cuyos nombres coinciden con un patrón.

| Patrón | Coincidencia |
| --- | --- |
| `*` | Cualquier cantidad de caracteres. |
| `?` | Un único carácter. |
| `[abc]` | Un carácter incluido en el conjunto indicado. |
| `[!abc]` | Un carácter que no pertenece al conjunto indicado. |
| `[[:digit:]]` | Un dígito. |
| `[[:upper:]]` | Una letra mayúscula. |
| `[[:lower:]]` | Una letra minúscula. |

Algunos ejemplos:

| Patrón | Archivos seleccionados |
| --- | --- |
| `*.txt` | Todos los nombres que terminan en `.txt`. |
| `g*` | Todos los nombres que comienzan con `g`. |
| `Data???` | Nombres que comienzan con `Data` y terminan con exactamente tres caracteres adicionales. |
| `[abc]*` | Nombres que comienzan con `a`, `b` o `c`. |

## Manipulación de archivos y directorios

### Crear archivos vacíos con `touch`

`touch` crea uno o varios archivos vacíos:

```bash
touch file
```

Si `file` ya existe, `touch` no borra ni modifica su contenido: solo actualiza su fecha de modificación.

### Crear directorios con `mkdir`

`mkdir` crea uno o varios directorios:

```bash
mkdir directory
```

### Copiar con `cp`

`cp` permite copiar archivos y directorios.

| Comando | Resultado |
| --- | --- |
| `cp file1 file2` | Copia `file1` en `file2`; si `file2` existe, se sobrescribe. |
| `cp -i file1 file2` | Solicita confirmación antes de sobrescribir `file2`. |
| `cp file1 dir1` | Copia `file1` dentro de `dir1`. |
| `cp -R dir1 dir2` | Copia recursivamente el directorio `dir1`. |
| `cp *.txt text_files` | Copia en `text_files` todos los archivos del directorio actual que terminan en `.txt`. |

### Mover o renombrar con `mv`

`mv` sirve tanto para mover como para renombrar archivos y directorios.

| Comando | Resultado |
| --- | --- |
| `mv file1 file2` | Renombra `file1` como `file2`; si el destino existe, puede sobrescribirlo. |
| `mv -i file1 file2` | Solicita confirmación antes de sobrescribir el destino. |
| `mv file1 file2 dir1` | Mueve ambos archivos al directorio `dir1`. |
| `mv dir1 dir2` | Renombra `dir1` si `dir2` no existe; si existe, mueve `dir1` dentro de `dir2`. |

### Eliminar con `rm`

`rm` elimina archivos:

```bash
rm file1 file2
```

Para eliminar un directorio y todo su contenido se utiliza la opción recursiva:

```bash
rm -r directory
```

La opción `-i` solicita confirmación antes de eliminar cada elemento:

```bash
rm -i file1 file2
```

> **Advertencia:** lo eliminado con `rm` no se envía a una papelera ni puede recuperarse mediante un comando de deshacer. Antes de combinar `rm` con comodines, se recomienda ejecutar primero el mismo patrón con `ls` para comprobar exactamente qué archivos seleccionará.

Por ejemplo, antes de ejecutar:

```bash
rm *.txt
```

se puede verificar la selección con:

```bash
ls *.txt
```

### Eliminar directorios vacíos con `rmdir`

`rmdir` elimina un directorio, pero únicamente si está vacío:

```bash
rmdir directory
```

Si el directorio contiene archivos u otros directorios, `rmdir` no lo elimina y muestra un error. Para eliminar un directorio que no está vacío se necesita `rm -r` (ver la sección anterior).

## Resumen de comandos

| Comando | Función principal |
| --- | --- |
| `pwd` | Muestra el directorio de trabajo actual. |
| `cd` | Cambia el directorio de trabajo. |
| `ls` | Lista archivos y directorios. |
| `less` | Permite consultar archivos de texto. |
| `file` | Identifica el tipo de contenido de un archivo. |
| `touch` | Crea archivos vacíos. |
| `mkdir` | Crea directorios. |
| `cp` | Copia archivos y directorios. |
| `mv` | Mueve o renombra archivos y directorios. |
| `rm` | Elimina archivos y directorios. |
| `rmdir` | Elimina directorios vacíos. |

## Fuentes

- [Navigation — LinuxCommand.org](https://linuxcommand.org/lc3_lts0020.php)
- [Looking Around — LinuxCommand.org](https://linuxcommand.org/lc3_lts0030.php)
- [Manipulating Files — LinuxCommand.org](https://linuxcommand.org/lc3_lts0050.php)
