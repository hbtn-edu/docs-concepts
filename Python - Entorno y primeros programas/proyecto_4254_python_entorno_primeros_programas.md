# Entorno de Python y primeros programas

## ¿Qué es programar en Python?

Hasta ahora, usar una computadora significó ejecutar programas ya construidos por otra persona: una herramienta de línea de comandos, una aplicación con interfaz gráfica, un asistente de IA. Programar es escribir, uno mismo, las instrucciones que la computadora va a ejecutar.

Esas instrucciones se escriben siguiendo reglas precisas de escritura y significado — la **sintaxis** y la **semántica** del lenguaje — y se guardan como texto en uno o más archivos, lo que se conoce como **código fuente**. Una computadora no entiende ese texto directamente: necesita otro programa que lo traduzca a algo ejecutable. Según cómo lo haga, un lenguaje de programación se describe como **compilado** (todo el código se traduce de antemano a un programa ejecutable) o **interpretado** (otro programa, el intérprete, lee y ejecuta el código fuente en el momento de correrlo).

Python pertenece al segundo grupo. Es uno de los lenguajes de programación más usados actualmente, en parte porque su sintaxis prioriza la legibilidad y porque existe un ecosistema muy amplio de bibliotecas ya escritas — desde análisis de datos hasta inteligencia artificial — listas para instalarse con las herramientas que se explican en este documento (`pip`, entornos virtuales).

Las secciones siguientes distinguen con precisión tres términos que se usan todo el tiempo al trabajar con Python: el lenguaje, el intérprete y el entorno.

## Python como lenguaje e intérprete

Python es un lenguaje interpretado: el intérprete lee el código fuente y lo ejecuta directamente. Para trabajar con Python es importante distinguir tres elementos:

- **el lenguaje**, que define la sintaxis y el comportamiento del código;
- **el intérprete**, que ejecuta ese código;
- **el entorno**, que determina qué versión del intérprete y qué paquetes están disponibles.

En sistemas GNU/Linux, el ejecutable de Python 3 suele llamarse `python3`:

```bash
python3 --version
command -v python3
```

El primer comando muestra la versión. El segundo indica qué ejecutable resolverá la shell. Estos datos son útiles cuando hay varias instalaciones de Python o un entorno virtual activo.

## Formas de ejecutar código

### Intérprete interactivo

Al ejecutar `python3` sin indicar un archivo se inicia el modo interactivo, también llamado REPL por *Read-Eval-Print Loop*:

```text
$ python3
Python 3.x.x (...)
>>> 8 * 7
56
>>> message = "Hello"
>>> print(message)
Hello
>>> exit()
```

El prompt principal es `>>>`. Cuando una construcción necesita más líneas, aparece el prompt secundario `...`:

```text
>>> if 10 > 3:
...     print("The condition is true")
...
The condition is true
```

Una línea vacía termina el bloque compuesto en una sesión interactiva. Este modo es apropiado para experimentar, consultar valores y comprobar ideas pequeñas. El historial de una sesión no reemplaza un archivo fuente versionado.

### Archivo de script

Cuando se pasa un archivo, Python lo ejecuta como un programa:

```bash
python3 hello.py
```

En un script no se muestra automáticamente el valor de cada expresión. La salida debe producirse de forma explícita, normalmente con `print()`.

### Otras formas útiles

```bash
python3 -c 'print(6 ** 2)'
python3 -m module_name
```

| Forma | Uso |
| --- | --- |
| `python3 -c 'CODE'` | Ejecuta una instrucción breve recibida como texto. |
| `python3 -m MODULE` | Localiza un módulo con ese intérprete y lo ejecuta como programa. |

La opción `-m` es especialmente útil con herramientas instaladas como módulos, porque deja claro qué intérprete las ejecuta.

## Expresiones y sentencias

Una **expresión** se evalúa y produce un valor. Algunos ejemplos son:

```python
4 + 5
length * width
temperature >= 20
type("Python")
```

Una **sentencia** realiza una acción. La asignación, por ejemplo, asocia un nombre con un objeto:

```python
language = "Python"
```

La asignación no imprime el valor. En el intérprete interactivo, una expresión escrita por sí sola sí muestra una representación del resultado:

```text
>>> total = 12 + 8
>>> total
20
>>> type(total)
<class 'int'>
>>> total > 10
True
```

En cambio, un archivo con una expresión aislada no genera salida visible:

```python
total = 12 + 8
total
```

Para comunicar el resultado desde un script se utiliza `print(total)`.

## Representación automática y `print()`

El REPL y `print()` no cumplen exactamente la misma función:

- el REPL muestra normalmente una representación orientada al desarrollo, equivalente a `repr(value)`;
- `print()` convierte sus argumentos con `str()` y escribe texto en la salida estándar.

La diferencia se aprecia con caracteres especiales:

```text
>>> text = "first\nsecond"
>>> text
'first\nsecond'
>>> print(text)
first
second
```

## Tipos básicos y comparaciones

Python es de tipado dinámico: los nombres se asocian con objetos y cada objeto tiene un tipo. Algunos tipos frecuentes son:

| Tipo | Ejemplo | Descripción |
| --- | --- | --- |
| `int` | `42` | Número entero. |
| `float` | `3.5` | Número de punto flotante. |
| `str` | `"Python"` | Cadena de texto. |
| `bool` | `True`, `False` | Valor lógico. |
| `NoneType` | `None` | Ausencia intencional de un valor. |

`type()` permite consultar el tipo de un objeto:

```python
value = 3.5
print(type(value))
```

Los operadores de comparación producen valores booleanos:

```python
age >= 18
name == "Ada"
temperature != 0
minimum < value <= maximum
```

Los literales booleanos se escriben exactamente `True` y `False`. No son cadenas; `"True"` es texto y se comporta de forma diferente.

## Estructura de un script ejecutable

Un archivo sencillo puede comenzar así:

```python
#!/usr/bin/env python3

language = "Python"
print(f"Language: {language}")
```

La primera línea es el **shebang**. Cuando el sistema ejecuta el archivo directamente, `/usr/bin/env` busca `python3` en el `PATH` vigente:

```bash
chmod u+x hello.py
./hello.py
```

Sin permiso de ejecución, todavía puede iniciarse pasando el archivo al intérprete:

```bash
python3 hello.py
```

En ese caso la shell ejecuta `python3`; el permiso ejecutable del archivo fuente no es necesario.

### Detalles que evitan problemas

- El shebang debe ser la primera línea, sin espacios ni líneas anteriores.
- Los sistemas Unix esperan finales de línea LF. Un archivo con CRLF puede provocar un error como `bad interpreter` debido al carácter `\r` oculto.
- Los archivos de texto deben terminar con un salto de línea.
- Python 3 interpreta el código fuente como UTF-8 de forma predeterminada.
- El nombre del archivo no debería coincidir con el de un módulo estándar o paquete utilizado, por ejemplo `sys.py` o `venv.py`.

Se puede inspeccionar el formato del archivo con:

```bash
file hello.py
head -n 1 hello.py
```

## Salida clara y determinista

`print()` acepta varios argumentos y permite controlar el separador y el final:

```python
print("one", "two", "three")
print("2026", "10", "05", sep="-")
print("loading", end="...")
print("done")
```

Por defecto, los argumentos se separan con un espacio y se añade `\n` al final. Cambiar `sep` o `end` modifica ese comportamiento.

Una salida es determinista cuando los mismos datos de entrada producen el mismo texto. Para obtenerla conviene controlar:

- el orden y la cantidad de líneas;
- mayúsculas, minúsculas, espacios y puntuación;
- el formato de números;
- la ausencia de mensajes adicionales;
- el salto de línea final.

### Interpolación con f-strings

Las f-strings permiten insertar expresiones dentro de texto:

```python
name = "Ada"
score = 95
print(f"Student: {name}")
print(f"Score: {score}")
```

También vas a encontrar `str.format()` en código existente, con el mismo propósito.

### Formato de números decimales

El especificador `.2f` presenta un número con exactamente dos cifras después del punto decimal:

```python
ratio = 22 / 7
print(f"Approximation: {ratio:.2f}")
```

El formato afecta la representación textual, no reemplaza el valor numérico almacenado en `ratio`. Los valores `float` usan representación binaria y no pueden representar exactamente todos los decimales; para una salida breve, el formato explícito evita depender de la representación predeterminada.

Un ejemplo completo con valores calculados podría ser:

```python
#!/usr/bin/env python3

sensor_name = "Thermostat"
samples = 8
average_reading = 71 / 8
is_calibrated = samples >= 5

print(f"Sensor: {sensor_name}")
print(f"Samples: {samples}")
print(f"Average: {average_reading:.2f}")
print(f"Calibrated: {is_calibrated}")
```

Aquí el decimal proviene de una operación y el booleano de una comparación. Ambos se convierten a texto únicamente al construir la salida.

### Comprobar la salida exacta

Para hacer visibles los finales de línea y ciertos espacios:

```bash
./hello.py | cat -e
```

Para comparar el resultado con un archivo esperado:

```bash
./hello.py > actual.txt
diff -u expected.txt actual.txt
```

Una diferencia invisible a simple vista, como un espacio final o una línea adicional, aparecerá en la comparación.

## Paquetes y `pip`

La biblioteca estándar se distribuye con Python. Otros paquetes se obtienen de índices como Python Package Index (PyPI) y se instalan normalmente con `pip`.

La forma recomendada de invocarlo es a través del intérprete elegido:

```bash
python3 -m pip --version
python3 -m pip install package_name
```

`python3 -m pip` significa “ejecutar el módulo `pip` con este `python3`”. Es más explícito que el comando independiente `pip`, que podría pertenecer a otra instalación.

La salida de `--version` muestra tanto la versión de `pip` como la ruta donde está instalado y la versión de Python asociada:

```text
pip X.Y from .../site-packages/pip (python 3.x)
```

### Operaciones frecuentes

```bash
python3 -m pip install package_name
python3 -m pip install --upgrade package_name
python3 -m pip show package_name
python3 -m pip list
python3 -m pip uninstall package_name
```

Antes de instalar conviene comprobar el entorno activo y la procedencia de `pip`:

```bash
python3 -c 'import sys; print(sys.executable)'
python3 -m pip --version
```

### Especificar versiones

Instalar siempre “la versión más reciente” hace que el resultado cambie con el tiempo. Para fijar una versión exacta:

```bash
python3 -m pip install 'example==1.4.2'
```

Las comillas evitan que la shell interprete caracteres como `<` y `>` como operadores de redirección si se usan otras restricciones (por ejemplo, un rango de versiones). Fijar una versión compartida ayuda a que todas las personas apliquen las mismas reglas.

### Instalaciones globales y entornos administrados

Instalar paquetes en el Python del sistema puede crear conflictos con programas administrados por el sistema operativo. Algunas distribuciones modernas marcan ese intérprete como `EXTERNALLY-MANAGED` y `pip` rechaza la instalación fuera de un entorno virtual.

Cuando se instala con privilegios de administrador (por ejemplo, como `root`), `pip` puede mostrar una advertencia similar a esta:

```text
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead.
```

La advertencia no impide la instalación, pero señala el mismo riesgo de fondo: modificar paquetes compartidos por el sistema operativo en lugar de aislarlos en un entorno virtual propio.

No se recomienda resolver este problema con `sudo pip` ni forzando una modificación del entorno del sistema. Las alternativas habituales son:

- crear un entorno virtual para las dependencias de un proyecto;
- usar el gestor de paquetes del sistema cuando se necesita un paquete proporcionado por la distribución;
- usar una herramienta como `pipx` para aplicaciones de línea de comandos que deban estar disponibles de manera general.

## Entornos virtuales con `venv`

Un entorno virtual es un directorio asociado con un intérprete base, pero con su propia ubicación de paquetes instalados. Permite que dos proyectos utilicen versiones diferentes de una dependencia sin modificar el Python del sistema ni interferir entre sí.

No es una máquina virtual ni un contenedor: no virtualiza el sistema operativo, el kernel, la red ni todos los comandos disponibles en la shell.

### Crear un entorno

Desde el directorio del proyecto:

```bash
python3 -m venv .venv
```

`.venv` es una convención frecuente, no un nombre obligatorio. En algunas distribuciones de Debian o Ubuntu puede ser necesario instalar primero el componente `python3-venv` con el gestor de paquetes del sistema.

### Activar y desactivar

La activación modifica variables de la shell, principalmente anteponiendo el directorio de ejecutables del entorno a `PATH`.

| Shell o sistema | Comando de activación |
| --- | --- |
| Bash o Zsh en GNU/Linux/macOS | `source .venv/bin/activate` |
| Fish | `source .venv/bin/activate.fish` |
| PowerShell en Windows | `.venv\Scripts\Activate.ps1` |
| `cmd.exe` en Windows | `.venv\Scripts\activate.bat` |

Después de activarlo:

```bash
which python
python --version
python -m pip --version
```

La ruta de `python` y la de `pip` deberían apuntar dentro de `.venv`. Dentro del entorno activado, `python` y `python3` resuelven al mismo intérprete; ambos nombres son válidos. Para salir:

```bash
deactivate
```

Cerrar la terminal también descarta la activación. El entorno no se elimina; puede activarse otra vez.

### Comprobar el aislamiento correctamente

La activación no oculta todos los ejecutables instalados fuera del entorno; solamente cambia el orden de búsqueda en `PATH`. Si una herramienta no existe dentro del entorno, la shell todavía podría encontrar una instalación global.

Para comprobar si un paquete está instalado en un entorno concreto, se debe consultar con el intérprete de ese entorno:

```bash
python3 -m venv env_a
python3 -m venv env_b

env_a/bin/python -m pip install 'six==1.16.0'
env_a/bin/python -m pip show six
env_b/bin/python -m pip show six
```

Otra comprobación directa consiste en intentar importar el paquete:

```bash
env_a/bin/python -c 'import six; print(six.__version__)'
env_b/bin/python -c 'import six'
```

La primera orden usa el paquete de `env_a`; la segunda falla si `env_b` no lo tiene instalado. Así se comprueba el aislamiento del paquete sin confundirlo con la resolución de comandos de la shell.

### Entornos desechables y reproducibles

Un entorno virtual no debería versionarse ni trasladarse a otra ruta o computadora: contiene rutas absolutas y depende de la instalación base con la que fue creado. Debe poder eliminarse y recrearse con `python3 -m venv` sin pérdida real, ya que lo que importa conservar son las herramientas instaladas en él, no el directorio en sí.

## Estilo de código con `pycodestyle`

PEP 8 reúne convenciones de estilo para código Python. `pycodestyle` es una herramienta que comprueba una parte de esas convenciones y reporta infracciones. No ejecuta el programa para comprobar su lógica y no reformatea el archivo automáticamente.

### Instalación y versión

Dentro de un entorno virtual:

```bash
python -m pip install pycodestyle
python -m pycodestyle --version
```

Cuando un equipo o un proceso automatizado requiere un comportamiento concreto, se debe fijar la versión:

```bash
python -m pip install 'pycodestyle==2.7.0'
```

Para verificar tanto el paquete como su ubicación:

```bash
python -m pip show pycodestyle
```

### Analizar archivos

```bash
python -m pycodestyle script.py
python -m pycodestyle src/
```

Si no encuentra problemas, el comando normalmente no imprime nada y termina con código de salida `0`. Si encuentra infracciones, cada mensaje indica archivo, línea, columna, código y descripción:

```text
script.py:4:10: E225 missing whitespace around operator
```

Las familias principales incluyen:

| Prefijo | Área habitual |
| --- | --- |
| `E1`, `W1` | Sangría. |
| `E2`, `W2` | Espacios y caracteres en blanco. |
| `E3`, `W3` | Líneas en blanco. |
| `E4` | Posición y forma de importaciones. |
| `E5`, `W5` | Longitud y cortes de línea. |
| `E7` | Construcción de sentencias. |
| `E9` | Errores de sintaxis o de entrada/salida. |

Para ver la línea exacta de cada infracción junto con el mensaje:

```bash
python -m pycodestyle --show-source script.py
```

### Estilo y corrección son dimensiones distintas

Un archivo puede pasar `pycodestyle` y producir un resultado incorrecto. También puede funcionar correctamente y contener problemas de estilo. Por eso se comprueban por separado:

```bash
python3 script.py
python3 -m pycodestyle script.py
```

## Diagnóstico de problemas frecuentes

### `python3: command not found`

El intérprete no está instalado o no se encuentra en `PATH`. Se debe instalar mediante el mecanismo correspondiente al sistema y volver a comprobar con `command -v python3`.

### `No module named pip`

Ese intérprete no dispone de `pip`. En GNU/Linux puede ser necesario instalar `python3-pip`; en otras instalaciones oficiales puede utilizarse `ensurepip` si está disponible. La solución debe corresponder al origen de la instalación de Python.

### `No module named venv` o fallo al crear el entorno

En Debian y Ubuntu suele faltar el paquete del sistema `python3-venv`. También puede ocurrir que la instalación de Python sea mínima.

### `ModuleNotFoundError` después de instalar

Es frecuente que el paquete se haya instalado con un `pip` y el programa se ejecute con otro Python. Conviene comparar:

```bash
python -c 'import sys; print(sys.executable)'
python -m pip --version
python -m pip show package_name
```

### Una herramienta existe fuera de un entorno pero no dentro

Los paquetes instalados globalmente y los instalados en un entorno virtual pertenecen a ubicaciones distintas. Debe instalarse la herramienta en el entorno que la necesita y ejecutarse con ese mismo intérprete.

### `externally-managed-environment`

El sistema protege su instalación de Python. La solución normal es crear un entorno virtual y efectuar allí la instalación. Forzar una instalación global puede dañar dependencias administradas por el sistema operativo.

### `Permission denied` al ejecutar un script

Se puede conceder permiso de ejecución:

```bash
chmod u+x script.py
```

También se puede ejecutar con `python3 script.py`, siempre que el archivo tenga permiso de lectura.

### `bad interpreter`

Se debe revisar el shebang, el formato de finales de línea y la existencia del intérprete. Los entornos virtuales trasladados de ubicación también pueden conservar rutas inválidas; es preferible recrearlos.

## Hábitos recomendados

- Comprobar la versión y la ruta del intérprete antes de instalar herramientas.
- Usar `python -m pip` para vincular explícitamente `pip` con el Python activo.
- Crear un entorno virtual por proyecto cuando existan dependencias externas.
- Evitar `sudo pip` y no modificar a la fuerza el Python administrado por el sistema.
- Fijar las versiones que deban ser reproducibles, incluidas las herramientas de estilo.
- No versionar los entornos virtuales: deben poder eliminarse y recrearse sin pérdida real.
- Tratar mayúsculas, espacios, puntuación y saltos de línea como parte de una salida textual.
- Ejecutar por separado el programa y las comprobaciones de estilo.
- Leer el mensaje completo de un error antes de aplicar cambios.

## Referencias

- [Uso del intérprete de Python](https://docs.python.org/3/tutorial/interpreter.html)
- [Uso del intérprete en Python 3.8](https://docs.python.org/3.8/tutorial/interpreter.html)
- [`venv`: creación de entornos virtuales](https://docs.python.org/3/library/venv.html)
- [`venv` en Python 3.8](https://docs.python.org/3.8/library/venv.html)
- [Guía de usuario de `pip`](https://pip.pypa.io/en/stable/user_guide/)
- [Instalar paquetes con `pip` y `venv`](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/)
- [Entornos administrados externamente](https://packaging.python.org/en/latest/specifications/externally-managed-environments/)
- [PEP 8: guía de estilo para código Python](https://peps.python.org/pep-0008/)
- [Documentación de `pycodestyle`](https://pycodestyle.pycqa.org/en/latest/)
