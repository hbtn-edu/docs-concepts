# Funciones y modularidad en Python

## Por qué dividir un programa

A medida que un programa crece, repetir instrucciones y concentrar toda la lógica en un único archivo dificulta entenderlo, probarlo y modificarlo. Python permite organizar el código en dos niveles principales:

- una **función** reúne instrucciones relacionadas bajo un nombre y puede recibir datos y producir un resultado;
- un **módulo** es un archivo de Python que agrupa funciones, variables y otras definiciones reutilizables.

Una buena división busca que cada función tenga una responsabilidad clara y que cada módulo reúna elementos relacionados. Esto reduce la duplicación y permite verificar pequeñas partes de forma independiente.

## Definir y llamar funciones

Una función se define con la palabra clave `def`, seguida de su nombre, sus parámetros entre paréntesis y dos puntos:

```python
def rectangle_area(width, height):
    area = width * height
    return area
```

La definición registra la función, pero no ejecuta todavía su cuerpo. Para utilizarla hay que llamarla:

```python
result = rectangle_area(6, 4)
print(result)  # 24
```

En este ejemplo:

- `rectangle_area` es el nombre de la función;
- `width` y `height` son **parámetros**: nombres locales declarados en la definición;
- `6` y `4` son **argumentos**: valores suministrados en la llamada;
- `result` recibe el valor producido por `return`.

El cuerpo debe estar indentado. La convención habitual es usar cuatro espacios por nivel y no mezclar espacios con tabulaciones.

### Elegir nombres claros

Los nombres de funciones y variables se escriben normalmente en `snake_case`:

```python
def calculate_average(total, item_count):
    return total / item_count
```

Conviene usar verbos o expresiones que indiquen la acción: `calculate_total`, `is_valid`, `format_name` o `load_config`. Nombres genéricos como `do_thing` o `data` comunican poca intención.

También es aconsejable no reutilizar nombres de funciones incorporadas como `str`, `list`, `sum` o `max`, porque el nuevo nombre oculta temporalmente el original:

```python
sum = 10

# sum([1, 2, 3]) ya no puede llamar a la función incorporada.
```

## `return` y `print()` cumplen funciones distintas

`return` entrega un valor al código que hizo la llamada. `print()` escribe una representación en la salida estándar.

| Operación | Efecto | Posibilidad de reutilizar el resultado |
| --- | --- | --- |
| `return value` | Finaliza la función y entrega `value`. | Sí, se puede guardar, comparar o pasar a otra función. |
| `print(value)` | Escribe texto en la terminal. | No; la llamada a `print()` devuelve `None`. |

```python
def double_value(number):
    return number * 2


def show_double(number):
    print(number * 2)
```

La primera función permite seguir calculando:

```python
total = double_value(7) + double_value(3)
print(total)  # 20
```

La segunda está pensada para producir un efecto visible. Intentar usar su llamada como número falla porque el resultado de `show_double()` es `None`.

### Retorno implícito de `None`

Si la ejecución llega al final de una función sin encontrar un `return`, Python devuelve `None` de forma implícita. Un `return` sin expresión produce el mismo resultado:

```python
def announce(message):
    print(message)


result = announce("Ready")
print(result is None)  # True
```

Una función que solamente imprime puede ser adecuada, pero debe quedar claro que no está calculando un valor reutilizable.

### Retornos anticipados

Un `return` termina inmediatamente la llamada actual. Esto permite tratar primero los casos especiales y reducir la anidación:

```python
def safe_divide(dividend, divisor):
    if divisor == 0:
        return None

    return dividend / divisor
```

Ninguna instrucción situada después de un `return` ejecutado puede alcanzarse.

## Parámetros y argumentos

Los argumentos posicionales se asocian con los parámetros según su orden:

```python
def describe_city(city, country):
    return f"{city}, {country}"


print(describe_city("Montevideo", "Uruguay"))
```

También pueden indicarse por nombre:

```python
print(describe_city(country="Uruguay", city="Montevideo"))
```

Los valores predeterminados hacen opcional un argumento:

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}"
```

Los parámetros sin valor predeterminado deben aparecer antes que los que sí lo tienen.

## Objetos, referencias y mutabilidad

Python pasa a las funciones referencias a objetos. Una función puede modificar un objeto mutable recibido, como una lista:

```python
def add_label(labels, label):
    labels.append(label)


categories = ["python"]
add_label(categories, "modules")
print(categories)  # ['python', 'modules']
```

En cambio, reasignar el parámetro solo cambia el nombre local:

```python
def replace_value(value):
    value = 100


number = 7
replace_value(number)
print(number)  # 7
```

Cuando una función no necesita modificar una colección recibida, se recomienda evitar esa modificación o devolver una colección nueva. Así se reducen los efectos secundarios inesperados.

### Evitar valores predeterminados mutables

Los valores predeterminados se evalúan una sola vez, al definir la función. Una lista o un diccionario utilizado como valor predeterminado puede conservar cambios entre llamadas:

```python
def append_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items
```

El patrón con `None` crea una colección nueva cuando el llamador no proporciona una.

## Alcance de los nombres

Los nombres creados dentro de una función pertenecen normalmente a su alcance local:

```python
def calculate_tax(amount):
    rate = 0.22
    return amount * rate
```

`rate` existe durante la llamada, pero no está disponible fuera de la función. Una función puede leer un nombre del módulo si no encuentra uno local, aunque depender de variables globales suele dificultar las pruebas:

```python
tax_rate = 0.22


def calculate_tax(amount):
    return amount * tax_rate
```

Cuando sea razonable, conviene expresar las dependencias mediante parámetros:

```python
def calculate_tax(amount, rate):
    return amount * rate
```

Asignar un nombre dentro de una función crea por defecto un nombre local. Las palabras `global` y `nonlocal` permiten modificar otros alcances, pero deben utilizarse con cuidado porque introducen efectos menos visibles.

## Escribir carácter por carácter

`print()` añade un salto de línea de forma predeterminada. El parámetro `end` permite cambiarlo:

```python
for character in "Python":
    print(character, end="")

print()
```

El `print()` final completa la línea. Otra opción es construir una cadena y escribirla una sola vez, pero imprimir directamente puede ser útil cuando se necesita procesar una secuencia paso a paso.

## Aritmética dentro de funciones

### Obtener un último dígito no negativo

En Python, el operador `%` sigue la regla del divisor. Por ejemplo, `-27 % 10` produce `3`, no `7`. Si se necesita el último dígito de la magnitud de un entero, una expresión clara es:

```python
last_digit = abs(number) % 10
```

`abs()` elimina el signo antes de calcular el resto. Una función puede imprimir un valor y también devolverlo cuando ambos efectos forman parte explícita de su contrato:

```python
def display_and_return_remainder(number, divisor):
    remainder = abs(number) % divisor
    print(remainder)
    return remainder
```

```python
value = display_and_return_remainder(-27, 10)
# Se imprime: 7
print(value)  # 7
```

Al probar una función con dos efectos, hay que verificar por separado el texto escrito y el valor retornado.

### Potencias mediante multiplicación repetida

Para un exponente entero no negativo, una potencia puede calcularse sin el operador `**`:

```python
def power_by_loop(base, exponent):
    result = 1

    for _ in range(exponent):
        result *= base

    return result
```

```python
print(power_by_loop(2, 3))  # 8
print(power_by_loop(5, 0))  # 1
```

El valor inicial es `1` porque es el elemento neutro de la multiplicación. Con un exponente igual a cero, el bucle no se ejecuta y el resultado permanece en `1`, incluso si la base es cero bajo esta convención de programación.

Si se desean admitir exponentes negativos, hace falta definir el comportamiento de manera explícita:

```python
def integer_power(base, exponent):
    if exponent >= 0:
        return power_by_loop(base, exponent)

    if base == 0:
        raise ZeroDivisionError("zero cannot have a negative exponent")

    return 1 / power_by_loop(base, -exponent)
```

```python
print(integer_power(2, 3))   # 8
print(integer_power(2, -2))  # 0.25
```

En este caso el resultado puede ser un número de punto flotante. Definir el dominio de entrada y los casos límite evita que distintas implementaciones tomen decisiones incompatibles.

## Módulos

Cada archivo con extensión `.py` puede actuar como módulo. Por ejemplo, un archivo `geometry.py` podría contener:

```python
PI = 3.14159


def circle_area(radius):
    return PI * radius * radius
```

Otro archivo ubicado en el mismo directorio puede importarlo:

```python
import geometry

area = geometry.circle_area(3)
print(area)  # 28.27431
```

El nombre `geometry` identifica el espacio de nombres del módulo. El punto permite acceder a los objetos que define y deja claro de dónde procede cada nombre.

### Formas de importación

```python
import geometry
from geometry import circle_area
import geometry as geo
from geometry import circle_area as area_of_circle
```

| Forma | Ejemplo de uso posterior | Característica |
| --- | --- | --- |
| `import geometry` | `geometry.circle_area(3)` | Conserva visible el origen del nombre. |
| `from geometry import circle_area` | `circle_area(3)` | Importa un nombre concreto. |
| `import geometry as geo` | `geo.circle_area(3)` | Usa un alias para el módulo. |
| `from geometry import circle_area as area_of_circle` | `area_of_circle(3)` | Usa un alias para un objeto. |

Las variables también se pueden importar:

```python
from geometry import PI

print(PI)
```

Se recomienda evitar `from module import *`: no deja claro qué nombres entran en el archivo y puede sobrescribir nombres existentes.

### Qué sucede durante una importación

La primera vez que un módulo se importa en un proceso, Python ejecuta sus sentencias de nivel superior para crear su espacio de nombres. Esto incluye asignaciones y definiciones con `def`, pero también cualquier llamada a `print()` u otra acción escrita fuera de una función.

Después de una importación exitosa, el módulo queda almacenado en `sys.modules` y las importaciones posteriores del mismo módulo en ese proceso suelen reutilizarlo sin ejecutar nuevamente todo el archivo.

Por esta razón, un módulo reutilizable debería evitar acciones inesperadas al importarse. Sus sentencias de nivel superior deberían dedicarse principalmente a definiciones, constantes e importaciones.

## `__name__` y el punto de entrada

Python asigna a cada módulo una variable especial llamada `__name__`:

- cuando el archivo se ejecuta como programa principal, `__name__` vale `"__main__"`;
- cuando se importa, `__name__` contiene el nombre del módulo.

Esto permite separar las definiciones reutilizables del comportamiento que solo debe ocurrir al ejecutar el archivo directamente:

```python
def celsius_to_fahrenheit(celsius):
    return celsius * 9 / 5 + 32


def main():
    temperature = celsius_to_fahrenheit(20)
    print(temperature)


if __name__ == "__main__":
    main()
```

Con `python3 temperatures.py` se llama a `main()`. En cambio, `import temperatures` crea las funciones sin producir esa salida.

Colocar la lógica de inicio dentro de `main()` aporta varias ventajas:

- mantiene el bloque de protección breve;
- permite probar o reutilizar la lógica sin ejecutar el programa completo;
- evita crear innecesariamente muchos nombres globales;
- permite devolver un código de salida si el programa lo necesita.

### `__main__` no es lo mismo que `__main__.py`

El valor especial `"__main__"` identifica el entorno donde comienza la ejecución de nivel superior. Un archivo llamado `__main__.py`, por otro lado, permite definir qué ocurre al ejecutar un paquete o directorio con `python3 -m package_name`. Ambos conceptos están relacionados, pero no son equivalentes.

## Cómo encuentra Python los módulos

Al importar un módulo, Python busca su nombre en varias ubicaciones, entre ellas:

1. los módulos incorporados;
2. el directorio del programa de entrada;
3. las rutas incluidas en la variable de entorno `PYTHONPATH`, si existe;
4. los directorios de instalación configurados para ese intérprete.

La lista efectiva puede inspeccionarse con:

```bash
python3 -c 'import sys; print(*sys.path, sep="\n")'
```

Para módulos propios sencillos, mantener el archivo importado en el mismo directorio que el programa evita configuraciones adicionales. El nombre usado en `import` no incluye la extensión `.py`:

```python
import geometry  # carga geometry.py
```

No conviene dar a un archivo propio el mismo nombre que un módulo estándar, por ejemplo `random.py` o `math.py`, porque podría ocultar el módulo que se pretendía importar.

## Dependencias entre módulos

Un diseño simple dirige las dependencias en una sola dirección. Por ejemplo, `application.py` puede usar definiciones de `geometry.py` sin que `geometry.py` necesite importar `application.py`. Cuando dos módulos se importan entre sí, se forma una importación circular. Como cada archivo puede observar al otro antes de que termine de inicializarse, algunos nombres todavía no existen y aparecen errores difíciles de interpretar.

Las importaciones circulares suelen resolverse extrayendo la lógica compartida a un tercer módulo o revisando qué capa debería depender de cuál.

## Organización y estilo

Una estructura legible para un módulo pequeño es:

1. línea *shebang*, cuando el archivo debe ejecutarse directamente;
2. documentación del módulo;
3. importaciones;
4. constantes;
5. funciones;
6. función `main()`;
7. bloque `if __name__ == "__main__":`.

```python
#!/usr/bin/env python3
"""Small utilities for temperature conversion."""  # "pequeñas utilidades para conversión de temperatura"

import math


ABSOLUTE_ZERO_C = -273.15


def rounded_fahrenheit(celsius):
    """Convert Celsius to Fahrenheit and round down."""  # "convierte Celsius a Fahrenheit y redondea hacia abajo"
    value = celsius * 9 / 5 + 32
    return math.floor(value)


def main():
    print(rounded_fahrenheit(20))


if __name__ == "__main__":
    main()
```

Algunas convenciones especialmente útiles son:

- escribir las importaciones al comienzo del archivo;
- colocar normalmente una importación por línea;
- separar las funciones de nivel superior con dos líneas en blanco;
- usar `snake_case` para funciones y variables;
- usar `UPPER_CASE` para constantes;
- limitar las líneas de código a 79 caracteres;
- escribir espacios alrededor de operadores binarios;
- eliminar importaciones y variables que no se utilizan.

Las docstrings describen el propósito de módulos y funciones. Son cadenas colocadas como primera sentencia de la definición y pueden consultarse con `help()`:

```python
def square(number):
    """Return the square of number."""  # "devuelve el cuadrado del número"
    return number * number
```

## Scripts ejecutables y portables

En sistemas tipo Unix, un script puede comenzar con:

```python
#!/usr/bin/env python3
```

Después de conceder permiso de ejecución, se puede ejecutar directamente:

```bash
chmod u+x temperatures.py
./temperatures.py
```

La línea *shebang* no reemplaza la necesidad de usar sintaxis compatible con la versión de Python disponible. También es importante que el archivo termine con un salto de línea.

## Comprobaciones útiles

### Ejecutar el archivo directamente

```bash
python3 temperatures.py
```

### Comprobar que la importación no produce salida inesperada

```bash
python3 -c 'import temperatures'
```

### Importar y utilizar una función concreta

```bash
python3 -c 'from temperatures import celsius_to_fahrenheit; print(celsius_to_fahrenheit(20))'
```

### Comprobar la sintaxis sin ejecutar el programa

```bash
python3 -m py_compile temperatures.py
```

### Revisar el estilo

```bash
pycodestyle temperatures.py
```

Si `pycodestyle` está instalado en un entorno virtual, se ejecutará la versión perteneciente a ese entorno.

### Probar valores representativos

Una función numérica debería probarse con casos normales, cero, valores negativos y límites relevantes. También conviene verificar el tipo del resultado:

```python
assert rectangle_area(3, 4) == 12
assert isinstance(rectangle_area(3, 4), int)
```

`assert` es cómodo para comprobaciones rápidas durante el desarrollo. Para suites permanentes se utilizan herramientas de pruebas como `unittest` o `pytest`.

## Errores frecuentes

### Imprimir cuando se necesita devolver

Una salida visible no equivale a un resultado reutilizable. Si otra parte del programa debe operar con el valor, la función debe retornarlo.

### Olvidar un `return` en alguna rama

Si una ruta de ejecución termina sin `return`, el resultado es `None`. Hay que revisar todas las ramas de una función que promete devolver un valor.

### Llamar a la función dentro de su propia definición por accidente

```python
def calculate_total(values):
    return calculate_total(values)
```

Esto produce recursión infinita hasta alcanzar el límite de recursión. Para procesar una colección con un bucle, debe actualizarse un acumulador en lugar de repetir la misma llamada sin un caso base.

### Ejecutar demostraciones durante una importación

Las llamadas colocadas directamente en el nivel superior se ejecutan al importar el archivo. El bloque de protección con `__name__` separa la interfaz reutilizable de la ejecución directa.

### Confundir el archivo con el nombre del módulo

Se importa `geometry`, no `geometry.py`. Los nombres de archivo con espacios o guiones tampoco son apropiados como identificadores de módulos.

### Esperar que una importación se recargue automáticamente

Una sesión interactiva conserva los módulos importados. Si el archivo cambia en disco, reiniciar la sesión es la opción más sencilla para garantizar que se use el código nuevo.

### Crear dependencias circulares

Si dos módulos se importan mutuamente, uno puede intentar acceder a nombres que el otro todavía no terminó de definir. Reorganizar las responsabilidades suele ser mejor que mover importaciones arbitrariamente dentro de funciones.

## Referencias

- [Definición de funciones — Tutorial de Python](https://docs.python.org/3/tutorial/controlflow.html#defining-functions)
- [Módulos — Tutorial de Python](https://docs.python.org/3/tutorial/modules.html)
- [`__main__`: entorno de ejecución de nivel superior](https://docs.python.org/3/library/__main__.html)
- [PEP 8 — Guía de estilo para código Python](https://peps.python.org/pep-0008/)
- [El sistema de importación de Python](https://docs.python.org/3/reference/import.html)
- [Definición de funciones — documentación de Python 3.8](https://docs.python.org/3.8/tutorial/controlflow.html#defining-functions)
- [Módulos — documentación de Python 3.8](https://docs.python.org/3.8/tutorial/modules.html)
- [`__main__` — documentación de Python 3.8](https://docs.python.org/3.8/library/__main__.html)
