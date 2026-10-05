# Flujo de control en Python

## Cómo decide un programa qué ejecutar

Python ejecuta normalmente las sentencias de arriba hacia abajo. Las estructuras de control permiten alterar ese recorrido para:

- ejecutar un bloque solamente cuando se cumple una condición;
- elegir entre varias alternativas;
- repetir sentencias mientras una condición sea verdadera;
- recorrer los elementos de una secuencia o de otro objeto iterable;
- omitir una iteración o finalizar un bucle antes de tiempo.

En Python, la sangría forma parte de la sintaxis. Las sentencias que pertenecen al mismo bloque deben tener la misma indentación:

```python
temperature = 24

if temperature >= 20:
    print("Warm day")
    print("Light clothing is enough")

print("Forecast completed")
```

Las dos primeras llamadas a `print()` pertenecen al `if`. La última queda fuera porque regresó al nivel de indentación anterior.

Se recomienda utilizar cuatro espacios por nivel y no mezclar tabulaciones con espacios.

## Condiciones con `if`, `elif` y `else`

La forma general de una decisión es:

```python
if condition_a:
    statement_a
elif condition_b:
    statement_b
else:
    default_statement
```

Python evalúa las condiciones en orden:

1. Si la condición del `if` es verdadera, ejecuta ese bloque y omite las demás ramas.
2. Si es falsa, prueba cada `elif` de arriba hacia abajo.
3. Si ninguna condición es verdadera, ejecuta el `else`, cuando existe.

Solo se ejecuta una rama de una misma cadena `if`–`elif`–`else`:

```python
balance = -8

if balance > 0:
    print("Credit")
elif balance == 0:
    print("Balanced")
else:
    print("Debt")
```

El orden importa. Las condiciones más específicas deben aparecer antes que otras que también podrían incluirlas:

```python
score = 95

if score >= 90:
    print("Excellent")
elif score >= 60:
    print("Approved")
else:
    print("Not approved")
```

Si se comprobara primero `score >= 60`, el valor `95` entraría en esa rama y nunca alcanzaría la condición más específica.

### Varios `if` independientes

Dos sentencias `if` separadas pueden ejecutar ambos bloques:

```python
number = 12

if number > 0:
    print("Positive")

if number % 2 == 0:
    print("Even")
```

Se utiliza una cadena con `elif` cuando las alternativas son excluyentes. Se utilizan varios `if` cuando más de una condición puede cumplirse y cada comprobación debe ser independiente.

## Operadores de comparación

Las comparaciones producen `True` o `False`:

| Operador | Significado | Ejemplo |
| --- | --- | --- |
| `==` | Igual en valor | `age == 18` |
| `!=` | Diferente en valor | `status != "closed"` |
| `<` | Menor que | `temperature < 0` |
| `<=` | Menor o igual que | `attempts <= 3` |
| `>` | Mayor que | `total > 100` |
| `>=` | Mayor o igual que | `score >= 60` |

La asignación y la igualdad son operaciones distintas:

```python
count = 5       # asigna un valor
count == 5      # compara y produce True
```

Usar `=` dentro de una condición produce un error de sintaxis; para comparar se utiliza `==`.

### Comparaciones encadenadas

Python admite la notación matemática:

```python
minimum = 10
value = 15
maximum = 20

if minimum <= value <= maximum:
    print("Inside the interval")
```

La condición equivale conceptualmente a:

```python
minimum <= value and value <= maximum
```

La expresión intermedia se evalúa una sola vez en la comparación encadenada.

### Igualdad, identidad y pertenencia

`==` compara valores. `is` comprueba si dos referencias señalan exactamente el mismo objeto. Para valores como números y cadenas se utiliza normalmente `==`:

```python
name == "Ada"
```

La identidad se reserva principalmente para objetos únicos como `None`:

```python
result is None
result is not None
```

Los operadores `in` y `not in` comprueban pertenencia:

```python
letter in "aeiou"
command not in ("start", "stop")
```

En una cadena, `in` busca una subcadena. En un diccionario, comprueba sus claves.

## Valores verdaderos y falsos

Una condición no tiene que contener explícitamente `True` o `False`. Python interpreta determinados valores como falsos:

- `False` y `None`;
- los ceros numéricos, como `0` y `0.0`;
- cadenas y colecciones vacías, como `""`, `[]`, `()` y `{}`.

La mayoría de los demás valores se consideran verdaderos:

```python
message = "Ready"

if message:
    print("There is a message")
```

Esto suele ser más claro que escribir `if message != "":`.

## Operadores booleanos

Python proporciona `and`, `or` y `not`:

```python
age = 20
has_ticket = True

if age >= 18 and has_ticket:
    print("Access granted")

if age < 18 or not has_ticket:
    print("Access denied")
```

| Operador | Resultado lógico |
| --- | --- |
| `a and b` | Verdadero cuando ambos operandos son verdaderos. |
| `a or b` | Verdadero cuando al menos uno es verdadero. |
| `not a` | Invierte la interpretación booleana de `a`. |

### Cortocircuito

`and` y `or` evalúan solamente lo necesario:

```python
divisor = 0

if divisor != 0 and 100 / divisor > 5:
    print("Large quotient")
```

Como la primera condición es falsa, Python no calcula `100 / divisor` y evita una división por cero.

Además, `and` y `or` devuelven uno de sus operandos, no necesariamente un `bool`:

```python
username = ""
display_name = username or "Anonymous"
print(display_name)
```

`not` sí produce siempre `True` o `False`.

### Precedencia

Entre estos operadores, el orden de precedencia es:

1. comparaciones;
2. `not`;
3. `and`;
4. `or`.

Aunque Python pueda interpretar una expresión sin paréntesis, estos ayudan a comunicar la intención:

```python
if is_admin or (is_active and has_permission):
    print("Allowed")
```

## Repetición con `while`

Un bucle `while` repite su bloque mientras la condición sea verdadera:

```python
count = 1

while count <= 4:
    print(count)
    count += 1
```

Cada repetición se denomina **iteración**. La variable que controla la condición debe cambiar; de lo contrario, el bucle puede no terminar:

```python
count = 1

while count <= 4:
    print(count)
    # Falta actualizar count: la condición seguirá siendo verdadera.
```

Antes de escribir un `while`, conviene identificar:

- el estado inicial;
- la condición de continuación;
- el cambio realizado en cada iteración;
- el momento en que la condición será falsa.

## Repetición con `for`

Un bucle `for` toma, en orden, cada elemento de un iterable:

```python
for letter in "code":
    print(letter)
```

La variable `letter` recibe un carácter distinto en cada iteración. Python también puede recorrer listas, tuplas, diccionarios, archivos y muchos otros objetos.

Cuando se necesita una progresión de enteros se utiliza `range()`.

## Cómo funciona `range()`

Las formas principales son:

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

El límite `stop` nunca se incluye:

| Expresión | Valores producidos |
| --- | --- |
| `range(4)` | `0, 1, 2, 3` |
| `range(2, 6)` | `2, 3, 4, 5` |
| `range(1, 8, 2)` | `1, 3, 5, 7` |
| `range(5, 0, -1)` | `5, 4, 3, 2, 1` |

Ejemplo:

```python
for value in range(3, 7):
    print(value)
```

El resultado contiene `3`, `4`, `5` y `6`, pero no `7`.

`range()` produce un objeto iterable de forma eficiente; no construye de antemano una lista con todos los valores. Para observar una progresión durante una prueba puede convertirse temporalmente:

```python
print(list(range(2, 9, 2)))
```

### Elegir los límites

Para recorrer los enteros desde `first` hasta `last`, ambos inclusive, el límite final debe ser `last + 1` cuando el paso es positivo:

```python
for value in range(first, last + 1):
    print(value)
```

Los errores de uno en uno aparecen cuando se confunde “cantidad de valores” con “último valor”. `range(10)` produce diez enteros, pero su último valor es `9`.

## `break`, `continue` y `pass`

### Finalizar con `break`

`break` termina inmediatamente el bucle más interno que lo contiene:

```python
for value in range(1, 20):
    if value % 7 == 0:
        print(value)
        break
```

### Omitir con `continue`

`continue` abandona el resto de la iteración actual y comienza la siguiente:

```python
for letter in "developer":
    if letter in "aeiou":
        continue
    print(letter, end="")

print()
```

### Reservar un bloque con `pass`

`pass` no realiza ninguna acción. Se utiliza cuando la sintaxis requiere un bloque pero todavía no se desea implementar comportamiento:

```python
if maintenance_mode:
    pass
else:
    print("Service available")
```

No se debe utilizar `pass` como sustituto accidental de una actualización necesaria en un `while`.

## Aritmética modular y último dígito

El operador `%` devuelve el resto asociado con la división entera:

```python
17 % 10   # 7
24 % 2    # 0
25 % 2    # 1
```

Por eso resulta útil para detectar múltiplos, separar ciclos y obtener dígitos.

### Comportamiento con números negativos

En Python, el resultado de `x % y` tiene el mismo signo que el divisor `y`, o es cero. Con un divisor positivo:

```python
print(-18 % 10)   # 2
```

Esto es correcto según la definición de Python, pero puede resultar inesperado si se busca conservar el signo del número original. Una estrategia explícita consiste en calcular primero la magnitud y después aplicar el signo:

```python
value = -347
magnitude = abs(value) % 10
signed_digit = -magnitude if value < 0 else magnitude

print(signed_digit)
```

El caso cero funciona sin tratamiento especial porque `-0` y `0` representan el mismo entero.

La relación entre división entera y resto es:

```text
x == (x // y) * y + (x % y)
```

## Expresiones condicionales

Una expresión condicional elige entre dos valores:

```python
label = "even" if number % 2 == 0 else "odd"
```

Su forma general es:

```text
value_if_true if condition else value_if_false
```

Es conveniente para una decisión breve que produce un valor. Cuando existen varias acciones o condiciones complejas, una sentencia `if` normal suele ser más legible.

## Formato de enteros

Las f-strings permiten controlar cómo se representa un entero:

```python
value = 31

print(f"Decimal: {value:d}")
print(f"Hexadecimal: {value:x}")
print(f"Hexadecimal with prefix: {value:#x}")
print(f"Padded: {value:04d}")
```

Resultado:

```text
Decimal: 31
Hexadecimal: 1f
Hexadecimal with prefix: 0x1f
Padded: 0031
```

| Especificador | Efecto |
| --- | --- |
| `d` | Entero en base decimal. |
| `x` | Entero hexadecimal con letras minúsculas. |
| `X` | Entero hexadecimal con letras mayúsculas. |
| `#x` | Hexadecimal con prefijo `0x`. |
| `02d` | Decimal de ancho mínimo 2, rellenado con ceros. |
| `04x` | Hexadecimal de ancho mínimo 4, rellenado con ceros. |

El ancho es un mínimo, no un máximo: `{123:02d}` produce `123`, no recorta el valor.

También puede utilizarse `.format()`:

```python
print("{0:d} = 0x{0:x}".format(value))
```

## Controlar espacios, separadores y salto final

`print()` termina normalmente con `\n`. El parámetro `end` permite reemplazarlo:

```python
print("A", end="")
print("B")
```

El resultado es una sola línea:

```text
AB
```

Para separar valores sin añadir un delimitador después del último, puede elegirse el final según la posición:

```python
for value in range(1, 6):
    ending = ", " if value < 5 else "\n"
    print(value, end=ending)
```

Resultado:

```text
1, 2, 3, 4, 5
```

Otra posibilidad, cuando está permitido construir cadenas, es reunir primero los fragmentos y utilizar `str.join()`:

```python
parts = []

for value in range(1, 6):
    parts.append(str(value))

print(", ".join(parts))
```

Una única sentencia `print()` dentro de un bucle puede ejecutarse muchas veces. La cantidad de sentencias escritas y la cantidad de llamadas realizadas durante la ejecución no son la misma cosa.

## Caracteres, códigos y recorrido de texto

Una cadena es iterable, por lo que puede recorrerse directamente:

```python
for letter in "python":
    print(letter)
```

`ord()` devuelve el punto de código Unicode de un carácter y `chr()` realiza la operación inversa:

```python
print(ord("a"))   # 97
print(chr(97))    # a
```

Esto permite generar secuencias de caracteres:

```python
for code in range(ord("m"), ord("q") + 1):
    print(chr(code), end="")

print()
```

Para omitir caracteres concretos se puede combinar la pertenencia con `continue`:

```python
for letter in "framework":
    if letter in "aeiou":
        continue
    print(letter, end="")

print()
```

## Bucles anidados y combinaciones únicas

Un bucle puede contener otro. Por cada valor del bucle exterior, el interior realiza todo su recorrido:

```python
for row in range(2):
    for column in range(3):
        print(row, column)
```

Esto produce seis pares: dos valores posibles para `row` multiplicados por tres para `column`.

Cuando se quieren pares diferentes sin obtener también el orden inverso, el segundo recorrido puede comenzar después del primer valor:

```python
for left in range(5):
    for right in range(left + 1, 5):
        print(left, right)
```

La relación `left < right` garantiza simultáneamente que:

- los dos valores sean diferentes;
- cada par aparezca una sola vez;
- el valor menor aparezca primero;
- la salida mantenga un orden predecible.

En bucles anidados, `break` y `continue` afectan solamente al bucle más interno que los contiene.

## Recorrer con índice y valor

Cuando se necesita la posición además del elemento, `enumerate()` suele ser más claro que combinar `range()` con `len()`:

```python
languages = ["Python", "C", "JavaScript"]

for index, language in enumerate(languages):
    print(index, language)
```

También puede establecerse otro índice inicial:

```python
for position, language in enumerate(languages, start=1):
    print(position, language)
```

Para una progresión numérica independiente de una colección, `range()` continúa siendo la opción adecuada.

## Salida determinista

Cuando un programa debe generar un formato exacto, cada carácter es significativo:

- mayúsculas y minúsculas;
- espacios antes o después de un valor;
- comas y otros separadores;
- ceros a la izquierda;
- prefijos como `0x`;
- líneas adicionales;
- salto de línea final.

Se puede hacer visibles los finales de línea con:

```bash
./script.py | cat -e
```

Y comparar dos salidas:

```bash
./script.py > actual.txt
diff -u expected.txt actual.txt
```

Evitar mensajes de depuración en la salida normal es tan importante como calcular correctamente los valores.

## Casos límite y pruebas manuales

Una estructura de control debe probarse en los puntos donde cambia de comportamiento.

Para una clasificación por signo conviene comprobar:

- un valor positivo;
- cero;
- un valor negativo.

Para intervalos y bucles conviene comprobar:

- el primer valor incluido;
- el último valor incluido;
- el límite que debe quedar fuera;
- un intervalo vacío;
- un paso negativo, cuando corresponda.

Para una salida separada por comas conviene comprobar:

- que no exista una coma adicional al final;
- que cada coma tenga exactamente el espacio esperado;
- que la salida termine con una nueva línea.

Para bucles anidados conviene calcular primero la cantidad esperada de iteraciones. Si el recorrido exterior tiene `m` valores y el interior siempre tiene `n`, se esperan `m * n` iteraciones. Cuando el inicio del bucle interior depende del exterior, la cantidad disminuye en cada vuelta.

## Errores frecuentes

### Olvidar los dos puntos

```python
if value > 0:      # Los dos puntos son obligatorios.
    print(value)
```

### Usar `=` en vez de `==`

```python
if value == 0:
    print("Zero")
```

### Sangrar una rama de forma incorrecta

Todos los bloques de una misma cadena deben comenzar en el mismo nivel:

```python
if value > 0:
    print("Positive")
elif value == 0:
    print("Zero")
else:
    print("Negative")
```

### Incluir por error el límite final

`range(start, stop)` llega como máximo hasta `stop - 1` cuando el paso es `1`.

### Crear un `while` infinito

Debe existir un cambio que acerque el estado a la condición de salida.

### Suponer que `% 10` conserva el signo

Con divisor positivo, el resto de Python es siempre positivo o cero. Si el signo forma parte del resultado deseado, debe manejarse explícitamente.

### Confundir `is` con `==`

`==` compara valores; `is` compara identidad. Para comparar números o texto se utiliza `==`.

### Perder el salto de línea final

Cuando se usa `end=""` repetidamente, puede ser necesario producir `"\n"` en la última iteración o añadir una llamada final a `print()`.

## Compatibilidad entre versiones

Las estructuras `if`, `while`, `for`, `range()`, `break` y `continue` funcionan en Python 3.8. La sentencia `match` que aparece en versiones actuales del tutorial fue incorporada en Python 3.10 y no puede utilizarse con Python 3.8.

Cuando un entorno exige una versión concreta, conviene consultar la edición correspondiente de la documentación y comprobar el intérprete activo:

```bash
python3 --version
```

## Hábitos recomendados

- Elegir `if`–`elif`–`else` para alternativas excluyentes y varios `if` para comprobaciones independientes.
- Colocar primero las condiciones más específicas.
- Utilizar comparaciones encadenadas para intervalos claros.
- Aprovechar el cortocircuito para proteger operaciones dependientes.
- Preferir `for` al recorrer un iterable y `while` cuando la repetición depende de una condición cambiante.
- Recordar que el límite final de `range()` queda excluido.
- Comprobar explícitamente el comportamiento de `%` cuando intervienen valores negativos.
- Expresar reglas de unicidad mediante los límites de los bucles, en lugar de generar duplicados para filtrarlos después.
- Formatear números con especificadores explícitos cuando la representación sea parte del resultado.
- Probar los límites de cada rama y revisar la salida carácter por carácter.

## Referencias

- [Herramientas de control de flujo](https://docs.python.org/3/tutorial/controlflow.html)
- [Herramientas de control de flujo en Python 3.8](https://docs.python.org/3.8/tutorial/controlflow.html)
- [Comparaciones y operaciones booleanas](https://docs.python.org/3/reference/expressions.html#comparisons)
- [Expresiones en Python 3.8](https://docs.python.org/3.8/reference/expressions.html)
- [Comprobación de valores verdaderos y falsos](https://docs.python.org/3/library/stdtypes.html#truth-value-testing)
- [Formato de cadenas](https://docs.python.org/3/library/string.html#format-specification-mini-language)
- [Funciones incorporadas](https://docs.python.org/3/library/functions.html)
- [PEP 8: guía de estilo para código Python](https://peps.python.org/pep-0008/)
