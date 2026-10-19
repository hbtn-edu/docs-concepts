# Manejo de archivos en Python

## Archivos y persistencia

Las variables conservan información mientras un programa está en ejecución. Un archivo permite almacenar datos fuera del proceso para recuperarlos más tarde, compartirlos con otras herramientas o utilizarlos como entrada de nuevas ejecuciones.

El trabajo con un archivo suele seguir esta secuencia:

1. abrirlo con el modo y la codificación adecuados;
2. leer o escribir mediante el objeto de archivo;
3. cerrar el recurso, incluso si ocurre un error.

Python reúne estas operaciones alrededor de `open()` y de los objetos que esta función devuelve.

## Abrir un archivo con `open()`

La forma general es:

```python
file = open(filename, mode, encoding="utf-8")
```

Los argumentos principales son:

- `filename`: ruta del archivo;
- `mode`: forma en que se utilizará;
- `encoding`: codificación empleada al convertir entre texto y bytes.

En programas nuevos conviene indicar explícitamente `encoding="utf-8"` para los archivos de texto. Así el comportamiento no depende de la configuración regional del sistema operativo.

```python
file = open("notes.txt", "r", encoding="utf-8")
```

Si se omite el modo, Python utiliza lectura de texto (`"r"`). Sin embargo, escribir el modo de forma explícita suele comunicar mejor la intención.

## Modos de apertura

| Modo | Uso | Si el archivo no existe | Si ya existe |
| --- | --- | --- | --- |
| `"r"` | Leer texto | Produce `FileNotFoundError` | Conserva el contenido. |
| `"w"` | Escribir texto | Lo crea | Borra el contenido anterior al abrir. |
| `"a"` | Añadir (anexar) texto al final | Lo crea | Conserva el contenido y escribe al final. |
| `"x"` | Crear y escribir | Lo crea | Produce `FileExistsError`. |
| `"r+"` | Leer y escribir | Produce `FileNotFoundError` | Conserva inicialmente el contenido. |

El sufijo `b` activa el modo binario:

```python
open("photo.jpg", "rb")
open("archive.bin", "wb")
```

En modo texto se leen y escriben objetos `str`. En modo binario se utilizan objetos `bytes`. No se especifica `encoding` en modo binario.

### Elegir entre `w` y `a`

El modo `w` no espera hasta la primera llamada a `write()` para borrar el contenido: el truncado ocurre al abrir el archivo.

```python
with open("report.txt", "w", encoding="utf-8") as file:
    file.write("New report\n")
```

El modo `a` conserva lo anterior y coloca cada nueva escritura al final:

```python
with open("activity.log", "a", encoding="utf-8") as file:
    file.write("Application started\n")
```

Antes de elegir `w`, hay que confirmar que reemplazar el contenido completo sea realmente la operación deseada.

## El administrador de contexto `with`

La forma recomendada de trabajar con archivos es utilizar `with`:

```python
with open("notes.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

Al entrar en el bloque, el archivo está abierto. Al salir, Python lo cierra automáticamente, incluso cuando una instrucción produce una excepción.

La variable `file` puede seguir existiendo después del bloque, pero el recurso ya está cerrado:

```python
print(file.closed)  # True
```

Intentar leer o escribir después del cierre produce `ValueError`.

### Por qué cerrar los archivos

Un archivo abierto utiliza recursos del sistema. Además, las escrituras pueden permanecer temporalmente en un búfer. No cerrar correctamente un archivo puede provocar:

- pérdida de datos todavía no enviados al disco;
- bloqueo innecesario del recurso;
- agotamiento del límite de archivos abiertos;
- resultados diferentes entre sistemas.

`with` expresa la duración exacta del recurso y evita depender de que el programa termine para liberarlo.

## Leer todo el contenido

`read()` sin argumentos devuelve desde la posición actual hasta el final:

```python
with open("message.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content, end="")
```

Se utiliza `end=""` porque `content` puede contener sus propios saltos de línea. Un `print(content)` normal agregaría otro salto al final.

Al llegar al final del archivo, otra llamada a `read()` devuelve una cadena vacía:

```python
with open("message.txt", "r", encoding="utf-8") as file:
    first_read = file.read()
    second_read = file.read()

print(second_read == "")  # True
```

Leer el archivo completo es apropiado cuando su tamaño es conocido y pequeño. Un archivo muy grande podría ocupar demasiada memoria.

### Leer una cantidad limitada

`read(size)` limita la cantidad solicitada:

```python
with open("message.txt", "r", encoding="utf-8") as file:
    fragment = file.read(20)
```

En modo texto, `size` se interpreta en caracteres. En modo binario se interpreta en bytes.

## Leer línea por línea

### Con `readline()`

`readline()` devuelve una línea y conserva su salto final cuando existe:

```python
with open("message.txt", "r", encoding="utf-8") as file:
    first_line = file.readline()
```

Una cadena vacía indica el final del archivo. Una línea vacía dentro del archivo se representa como `"\n"`, por lo que ambos casos pueden distinguirse.

### Iterando sobre el archivo

La forma más clara y eficiente para procesar un archivo línea por línea es iterar sobre el objeto:

```python
with open("records.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line, end="")
```

La iteración no necesita cargar todo el contenido en memoria. Es adecuada para archivos grandes y para procesos en los que cada línea puede tratarse de manera independiente.

### Con `readlines()`

`readlines()` devuelve una lista con todas las líneas:

```python
with open("records.txt", "r", encoding="utf-8") as file:
    lines = file.readlines()
```

Esta opción conserva normalmente los saltos de línea y carga la lista completa en memoria. Resulta cómoda cuando se necesita acceder varias veces a las líneas de un archivo pequeño.

## Escribir texto

`write()` recibe una cadena y devuelve la cantidad de caracteres escritos:

```python
text = "Hello, file!\n"  # "¡hola, archivo!"

with open("greeting.txt", "w", encoding="utf-8") as file:
    written = file.write(text)

print(written)
```

`write()` no agrega espacios ni saltos de línea automáticamente. Si se necesita terminar una línea, `\n` debe estar incluido en la cadena.

El valor retornado cuenta caracteres, no necesariamente bytes. Por ejemplo, algunos caracteres de Unicode ocupan más de un byte al codificarse como UTF-8.

```python
text = "á"
print(len(text))                 # 1 carácter
print(len(text.encode("utf-8")))  # 2 bytes
```

En modo texto, otros tipos deben convertirse primero a cadenas:

```python
quantity = 12

with open("quantity.txt", "w", encoding="utf-8") as file:
    file.write(str(quantity))
```

### Escribir varias cadenas

`writelines()` recibe un iterable de cadenas:

```python
lines = ["first\n", "second\n", "third\n"]

with open("items.txt", "w", encoding="utf-8") as file:
    file.writelines(lines)
```

Al igual que `write()`, `writelines()` no agrega separadores. Cada cadena debe contener el salto de línea necesario.

## Añadir contenido (anexar)

El modo `a` crea el archivo si no existe y conserva el contenido existente:

```python
entry = "New event\n"  # "nuevo evento"

with open("events.log", "a", encoding="utf-8") as file:
    added = file.write(entry)

print(added)
```

Ejecutar el mismo código varias veces agrega una entrada en cada ejecución. La separación correcta depende de que cada fragmento incluya el salto de línea apropiado.

El modo de anexado es común en registros, pero no garantiza por sí solo que varias escrituras formen una transacción indivisible en todos los sistemas y situaciones de concurrencia.

## Posición y cursor del archivo

Un objeto de archivo mantiene una posición que avanza al leer o escribir.

### Consultar la posición

`tell()` devuelve la posición actual:

```python
with open("data.txt", "rb") as file:
    print(file.tell())
    file.read(4)
    print(file.tell())
```

En modo binario la posición corresponde a una cantidad de bytes desde el comienzo. En modo texto, el valor puede ser una representación opaca y no debe interpretarse siempre como un número de caracteres.

### Cambiar la posición

`seek(offset, whence)` mueve el cursor:

```python
with open("data.txt", "rb") as file:
    file.seek(5)
    byte = file.read(1)
```

Los valores de referencia de `whence` son:

| Valor | Referencia |
| --- | --- |
| `0` | Comienzo del archivo. |
| `1` | Posición actual. |
| `2` | Final del archivo. |

En archivos de texto existen restricciones sobre los desplazamientos admitidos. Para movimientos exactos por bytes se debe utilizar modo binario. Volver al comienzo de un archivo de texto puede hacerse con `file.seek(0)`.

```python
with open("message.txt", "r", encoding="utf-8") as file:
    first = file.read()
    file.seek(0)
    second = file.read()

print(first == second)  # True
```

## Texto, bytes y codificación

Un archivo almacena bytes. La codificación define cómo se representan los caracteres mediante esos bytes.

En modo texto, Python realiza la conversión automáticamente:

```python
with open("names.txt", "r", encoding="utf-8") as file:
    text = file.read()  # str
```

En modo binario no hay conversión:

```python
with open("names.txt", "rb") as file:
    data = file.read()  # bytes
```

Usar una codificación incorrecta puede provocar `UnicodeDecodeError` al leer o producir texto ilegible. Declarar UTF-8 de forma explícita evita depender de la codificación predeterminada del entorno.

### Saltos de línea

Los sistemas operativos pueden almacenar finales de línea de forma diferente. En modo texto, Python normaliza normalmente esas diferencias al leer y adapta `\n` al escribir. En modo binario se reciben exactamente los bytes presentes.

No se deben procesar imágenes, ejecutables o archivos comprimidos como texto: la conversión de codificación y saltos de línea puede alterar sus bytes.

## Rutas de archivos

Una ruta relativa se interpreta desde el directorio de trabajo actual, no necesariamente desde el directorio donde se encuentra el script:

```python
with open("data/input.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

El directorio actual puede consultarse desde la terminal:

```bash
pwd
```

Y desde Python:

```python
import os

print(os.getcwd())
```

Para aplicaciones con rutas más complejas, `pathlib` proporciona objetos de ruta portables:

```python
from pathlib import Path

path = Path("data") / "input.txt"

with path.open("r", encoding="utf-8") as file:
    content = file.read()
```

Las rutas construidas a partir de entradas externas deben validarse antes de usarse para evitar accesos fuera de la ubicación permitida.

## Manejo de errores de entrada y salida

Algunas excepciones habituales son:

| Excepción | Situación |
| --- | --- |
| `FileNotFoundError` | La ruta de lectura no existe. |
| `PermissionError` | El proceso no tiene permisos suficientes. |
| `IsADirectoryError` | Se intentó abrir un directorio como archivo. |
| `UnicodeDecodeError` | Los bytes no pueden decodificarse con la codificación indicada. |
| `UnicodeEncodeError` | El texto no puede representarse con la codificación indicada. |
| `OSError` | Otro fallo relacionado con el sistema operativo. |

Solamente deben capturarse los errores para los que exista una respuesta útil:

```python
def load_optional_text(filename):
    try:
        with open(filename, "r", encoding="utf-8") as file:
            return file.read()
    except FileNotFoundError:
        return None
```

Capturar `OSError` puede ser apropiado en un límite de la aplicación, pero hacerlo demasiado cerca de cada operación puede ocultar diferencias importantes entre un archivo ausente, permisos insuficientes y una ruta inválida.

## Datos estructurados con JSON

Guardar directamente `str()` de una lista o un diccionario no siempre produce un formato apropiado para intercambiar datos. La biblioteca estándar incluye `json`:

```python
import json

settings = {
    "language": "es",
    "theme": "dark",
}

with open("settings.json", "w", encoding="utf-8") as file:
    json.dump(settings, file, ensure_ascii=False, indent=2)
```

Para recuperar los datos:

```python
with open("settings.json", "r", encoding="utf-8") as file:
    settings = json.load(file)
```

La serialización convierte objetos de Python compatibles en texto JSON. La deserialización reconstruye estructuras como diccionarios y listas a partir de ese texto.

JSON no admite directamente todos los objetos de Python. Además, un archivo JSON debe contener datos válidos antes de que `json.load()` pueda interpretarlo.

## Escritura segura y pérdida de datos

Abrir un archivo existente con `w` elimina su contenido antes de escribir el reemplazo. Si el programa falla después de abrirlo, el archivo puede quedar vacío o incompleto.

Para información importante, una estrategia más segura consiste en:

1. generar primero todo el contenido;
2. escribirlo en un archivo temporal ubicado en el mismo sistema de archivos;
3. cerrar y comprobar la escritura;
4. reemplazar el archivo final mediante una operación atómica cuando el sistema lo permita.

Esta estrategia reduce el riesgo de dejar un archivo parcialmente actualizado. Para ejercicios breves puede ser innecesaria, pero es importante en configuraciones, estados persistentes y datos de producción.

## Errores frecuentes

### Abrir con `w` cuando se quería añadir

`w` trunca el archivo. Para conservar el contenido y escribir al final se utiliza `a`.

### Olvidar el salto de línea

`write()` no agrega `\n`. Dos escrituras consecutivas pueden quedar unidas si las cadenas no incluyen el separador.

### Agregar un salto extra al imprimir

El texto leído puede terminar en `\n`. `print(content)` agrega otro. `print(content, end="")` conserva exactamente los saltos presentes.

### Confundir caracteres con bytes

El valor retornado por `write()` en modo texto cuenta caracteres. El tamaño del archivo en disco puede ser mayor cuando contiene caracteres multibyte.

### Leer dos veces sin mover el cursor

Después de `read()`, el cursor queda al final. Se utiliza `seek(0)` o se vuelve a abrir el archivo para leerlo otra vez.

### Usar el recurso fuera de `with`

Al salir del bloque el archivo está cerrado. Las operaciones posteriores fallan.

### Confiar en la codificación predeterminada

La configuración puede cambiar entre equipos. Para texto conocido se debe especificar `encoding="utf-8"`.

### Leer archivos grandes de una sola vez

`read()` y `readlines()` pueden consumir tanta memoria como el contenido completo. La iteración línea por línea mantiene un uso de memoria reducido.

### Suponer que una ruta relativa parte del script

La ruta se resuelve desde el directorio de trabajo. Ejecutar el mismo script desde otro lugar puede cambiar el archivo encontrado.

## Comprobaciones útiles

### Ejecutar un archivo

```bash
python3 example.py
```

### Comprobar la sintaxis

```bash
python3 -m py_compile example.py
```

### Revisar el estilo

```bash
pycodestyle example.py
```

### Inspeccionar contenido y tamaño

```bash
cat output.txt
wc -m output.txt
wc -c output.txt
```

`wc -m` cuenta caracteres según la configuración regional; `wc -c` cuenta bytes.

### Ver bytes y caracteres especiales

```bash
od -An -tx1c output.txt
```

Esto permite distinguir saltos de línea, bytes UTF-8 y espacios que no son evidentes en una salida normal.

### Casos que conviene comprobar

- archivo inexistente en lectura;
- archivo inexistente en escritura y anexado;
- archivo existente con contenido previo;
- cadena vacía;
- texto con caracteres Unicode;
- texto con y sin salto de línea final;
- lectura repetida después de llegar al final;
- archivo sin permisos suficientes;
- ruta que identifica un directorio;
- entrada suficientemente grande para justificar procesamiento por líneas.

## Referencias

- [Lectura y escritura de archivos — Tutorial de Python](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
- [Acciones de limpieza predefinidas — Tutorial de Python](https://docs.python.org/3/tutorial/errors.html#predefined-clean-up-actions)
- [`open()` — Funciones incorporadas](https://docs.python.org/3/library/functions.html#open)
- [Módulo `io`](https://docs.python.org/3/library/io.html)
- [Rutas orientadas a objetos con `pathlib`](https://docs.python.org/3/library/pathlib.html)
- [Codificador y decodificador JSON](https://docs.python.org/3/library/json.html)
- [Lectura y escritura de archivos — documentación de Python 3.8](https://docs.python.org/3.8/tutorial/inputoutput.html#reading-and-writing-files)
- [`open()` — documentación de Python 3.8](https://docs.python.org/3.8/library/functions.html#open)
- [Video: Learn to Program 8 — Reading / Writing Files](https://www.youtube.com/watch?v=EukxMIsNeqU)
