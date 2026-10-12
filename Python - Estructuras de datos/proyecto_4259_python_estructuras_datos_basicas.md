# Estructuras de datos básicas en Python

## Por qué existen varias estructuras de datos

Casi todo programa necesita agrupar y organizar varios valores: una lista de nombres, los datos de una persona, un conjunto de etiquetas sin repetidos. Python no ofrece una sola forma de hacer esto porque distintos problemas necesitan propiedades distintas: mantener un orden, permitir duplicados, garantizar que los elementos no se repitan o asociar cada valor con un nombre.

A estas formas de agrupar valores se las llama **colecciones**. Python incluye cuatro colecciones básicas, cada una pensada para un tipo de problema distinto. Elegir la más adecuada para cada situación hace que el código comunique mejor su propósito y evita operaciones innecesarias.

## Colecciones incorporadas

Las cuatro colecciones básicas se diferencian por cómo organizan sus elementos:

| Estructura | Orden | Elementos repetidos | Modificable | Acceso principal |
| --- | --- | --- | --- | --- |
| `list` | Conserva el orden | Sí | Sí | Posición |
| `tuple` | Conserva el orden | Sí | No | Posición |
| `set` | Sin orden de posición | No | Sí | Pertenencia |
| `dict` | Conserva el orden de inserción | Claves únicas | Sí | Clave |

La elección depende de la intención:

- una lista representa una secuencia que puede cambiar;
- una tupla agrupa una cantidad estable de valores;
- un conjunto modela elementos únicos y relaciones de pertenencia;
- un diccionario asocia claves con valores.

Elegir la estructura adecuada hace que el código comunique mejor su propósito y evita operaciones innecesarias.

## Listas

Una lista es una secuencia ordenada y mutable. Se escribe entre corchetes:

```python
languages = ["Python", "C", "JavaScript"]
empty_list = []
mixed_values = [42, "text", True]
```

Aunque Python permite mezclar tipos, una lista con elementos de significado similar suele ser más fácil de procesar.

### Índices

Los índices comienzan en cero:

```python
colors = ["red", "green", "blue"]

print(colors[0])  # red
print(colors[2])  # blue
```

Los índices negativos cuentan desde el final:

```python
print(colors[-1])  # blue
print(colors[-2])  # green
```

Intentar acceder fuera del rango produce `IndexError`:

```python
colors[10]
```

Cuando una interfaz debe aceptar solamente posiciones no negativas, la validación tiene que rechazar explícitamente los índices negativos, aunque Python normalmente los admita:

```python
def get_non_negative_item(values, index):
    if index < 0 or index >= len(values):
        return None

    return values[index]
```

```python
print(get_non_negative_item(["a", "b", "c"], 1))   # b
print(get_non_negative_item(["a", "b", "c"], -1))  # None
```

El valor `None` puede representar que no existe un elemento válido. Quien llame a la función debe conocer y comprobar ese contrato.

### Modificar elementos

La asignación por índice modifica la lista existente:

```python
scores = [12, 18, 15]
scores[1] = 20
print(scores)  # [12, 20, 15]
```

Una función que modifica una lista puede devolver la misma referencia para facilitar su uso, pero eso no significa que haya creado una lista nueva:

```python
def replace_if_valid(values, index, new_value):
    if 0 <= index < len(values):
        values[index] = new_value

    return values
```

```python
data = [1, 2, 3]
result = replace_if_valid(data, 1, 50)

print(data)            # [1, 50, 3]
print(result is data)  # True
```

Si la entrada debe permanecer intacta, primero se crea una copia y se modifica esa copia.

### Métodos frecuentes

```python
values.append(item)        # agrega al final
values.extend(other)       # agrega todos los elementos de otro iterable
values.insert(index, item) # inserta antes de una posición
values.remove(item)        # elimina la primera coincidencia
item = values.pop()        # elimina y devuelve el último elemento
values.clear()             # elimina todo
```

Los métodos que modifican una lista, como `append()`, `sort()` y `reverse()`, devuelven `None`:

```python
numbers = [3, 1, 2]
result = numbers.sort()

print(numbers)  # [1, 2, 3]
print(result)   # None
```

Para obtener una lista ordenada nueva se utiliza `sorted()`:

```python
original = [3, 1, 2]
ordered = sorted(original)

print(original)  # [3, 1, 2]
print(ordered)   # [1, 2, 3]
```

### Recorrer una lista

Cuando solamente importa el valor:

```python
for language in languages:
    print(language)
```

Cuando también se necesita la posición, `enumerate()` evita administrar manualmente un contador:

```python
for index, language in enumerate(languages):
    print(index, language)
```

Modificar el tamaño de una lista mientras se la recorre puede hacer que se omitan elementos. Suele ser más seguro construir una lista nueva.

### Formatear enteros

El especificador `d` representa un entero decimal:

```python
number = 42
print("{:d}".format(number))
print(f"{number:d}")
```

El formato hace explícito el tipo de salida esperado. Un objeto incompatible provoca una excepción en lugar de convertirse silenciosamente.

## Listas anidadas y matrices

Una matriz sencilla puede representarse como una lista de filas, donde cada fila es otra lista:

```python
matrix = [
    [2, 4, 6],
    [8, 10, 12],
]
```

Se necesitan dos niveles de recorrido:

```python
for row in matrix:
    for value in row:
        print(value, end=" ")
    print()

# Se imprime:
# 2 4 6
# 8 10 12
```

`print()` agrega un salto de línea. El argumento `end` controla qué se escribe después de cada valor.

Cuando se necesita evitar un espacio al final de cada fila, puede construirse primero la línea completa:

```python
for row in matrix:
    line = " ".join("{:d}".format(value) for value in row)
    print(line)

# Se imprime:
# 2 4 6
# 8 10 12
```

Una fila vacía produce una línea vacía. Una matriz vacía no produce ninguna fila.

### Evitar filas compartidas

Esta construcción repite referencias a la misma lista:

```python
matrix = [[0] * 3] * 2
matrix[0][1] = 7
print(matrix)  # [[0, 7, 0], [0, 7, 0]]
```

Una comprensión crea cada fila por separado:

```python
matrix = [[0 for _ in range(3)] for _ in range(2)]
matrix[0][1] = 7
print(matrix)  # [[0, 7, 0], [0, 0, 0]]
```

A diferencia del ejemplo anterior, aquí solo cambia la primera fila: cada una es una lista independiente.

## Mutabilidad y referencias

Las listas, los conjuntos y los diccionarios son mutables: su contenido puede cambiar sin crear otro objeto. Las tuplas son inmutables.

Dos nombres pueden referirse a la misma colección mutable:

```python
original = [10, 20, 30]
alias = original

alias[1] = 99
print(original)  # [10, 99, 30]
```

La asignación no copia la lista; solamente crea otra referencia. Para obtener una copia superficial pueden utilizarse:

```python
original_copy = original.copy()
another_copy = original[:]
```

Una copia superficial crea un contenedor nuevo, pero no duplica los objetos anidados:

```python
matrix = [[1, 2], [3, 4]]
matrix_copy = matrix.copy()

matrix_copy[0][0] = 100
print(matrix)  # [[100, 2], [3, 4]]
```

Antes de modificar una colección recibida por una función conviene decidir si su contrato exige cambiar el objeto original o producir uno nuevo.

## Tuplas

Una tupla es una secuencia ordenada e inmutable:

```python
point = (4, 7)
empty_tuple = ()
```

Los paréntesis ayudan a leer la expresión, pero la coma es lo que construye la tupla. Por eso una tupla de un elemento necesita una coma final:

```python
single_value = (5,)
not_a_tuple = (5)

print(type(single_value))  # <class 'tuple'>
print(type(not_a_tuple))   # <class 'int'>
```

Intentar reemplazar una posición produce `TypeError`:

```python
point[0] = 10
```

La inmutabilidad afecta a las posiciones de la tupla, no necesariamente a los objetos que contiene. Una tupla puede contener una lista y esa lista todavía puede modificarse.

### Empaquetado y desempaquetado

```python
record = "Ada", 36
name, age = record
```

El desempaquetado requiere la misma cantidad de nombres y valores, salvo que se utilice un nombre con `*` para recoger el resto:

```python
first, *middle, last = [10, 20, 30, 40]

print(first)   # 10
print(middle)  # [20, 30]
print(last)    # 40
```

Las funciones utilizan tuplas de manera natural para devolver varios valores:

```python
def minimum_and_maximum(values):
    return min(values), max(values)


lowest, highest = minimum_and_maximum([8, 3, 11])

print(lowest, highest)  # 3 11
```

### Normalizar tuplas de longitud variable

Cuando solamente importan las primeras posiciones y las ausentes tienen un valor predeterminado, se puede normalizar cada entrada antes de operar:

```python
def first_two(values):
    first = values[0] if len(values) > 0 else 0
    second = values[1] if len(values) > 1 else 0
    return first, second
```

```python
print(first_two([5]))        # (5, 0)
print(first_two([5, 9, 1]))  # (5, 9)
```

Los valores posteriores se ignoran. Separar la normalización del cálculo hace más visible el tratamiento de entradas cortas.

## Conjuntos

Un conjunto contiene elementos únicos y no proporciona posiciones:

```python
tags = {"python", "linux", "python"}
print(len(tags))  # 2
```

Un conjunto vacío se crea con `set()`. Las llaves vacías crean un diccionario:

```python
empty_set = set()
empty_dictionary = {}
```

Los elementos deben ser *hashable*, lo que normalmente implica que no cambian durante su uso como miembros. Los números, las cadenas y muchas tuplas son válidos; las listas, los conjuntos mutables y los diccionarios no lo son.

### Pertenencia

```python
if "python" in tags:
    print("Tag found")  # mensaje de ejemplo: "elemento encontrado"
```

Las comprobaciones de pertenencia son uno de los usos principales de los conjuntos, especialmente cuando hay muchas búsquedas.

### Operaciones de conjuntos

```python
backend = {"python", "sql", "linux"}
frontend = {"javascript", "html", "linux"}
```

| Operación | Operador | Resultado conceptual |
| --- | --- | --- |
| Unión | `backend \| frontend` | Elementos presentes en cualquiera de los dos. |
| Intersección | `backend & frontend` | Elementos presentes en ambos. |
| Diferencia | `backend - frontend` | Elementos del primero que no están en el segundo. |
| Diferencia simétrica | `backend ^ frontend` | Elementos presentes en uno solo de los dos. |

También existen métodos equivalentes:

```python
backend.union(frontend)
backend.intersection(frontend)
backend.difference(frontend)
backend.symmetric_difference(frontend)
```

Estas operaciones producen conjuntos nuevos. Las variantes como `update()` e `intersection_update()` modifican el conjunto existente.

### Orden de salida

Los conjuntos no definen un orden posicional. El orden observado al imprimirlos no debe utilizarse como parte de la lógica:

```python
common = backend & frontend
print(sorted(common))  # ['linux']
```

`sorted()` devuelve una lista ordenada y resulta útil para generar una salida determinista cuando los elementos son comparables.

### Eliminar duplicados

```python
names = ["Ana", "Luis", "Ana", "Marta"]
unique_names = set(names)
```

Esta conversión elimina duplicados, pero no debe utilizarse si se necesita conservar el orden original de la lista: un conjunto no recuerda cuál apareció primero. Más adelante, en la sección de diccionarios, se muestra una forma de eliminar duplicados que sí conserva el orden.

## Diccionarios

Un diccionario almacena asociaciones entre claves y valores:

```python
profile = {
    "name": "Ada",
    "language": "Python",
    "active": True,
}
```

Las claves son únicas y deben ser *hashable*. Los valores pueden ser de cualquier tipo y pueden repetirse.

### Consultar valores

El acceso con corchetes exige que la clave exista:

```python
language = profile["language"]
```

Una clave inexistente produce `KeyError`. Cuando la ausencia es esperable, `get()` permite indicar un valor predeterminado:

```python
city = profile.get("city")
country = profile.get("country", "Unknown")

print(city)     # None
print(country)  # Unknown
```

También puede comprobarse la pertenencia:

```python
if "name" in profile:
    print(profile["name"])
```

`in` comprueba claves, no valores.

### Agregar y actualizar

La misma sintaxis sirve para crear una asociación o reemplazar su valor:

```python
profile["city"] = "Montevideo"
profile["language"] = "C"
```

Si la clave ya existía, se sustituye solamente su valor. Si no existía, se agrega un nuevo par. Ambas operaciones modifican el diccionario original.

Para incorporar varias asociaciones:

```python
profile.update({"active": False, "level": 2})
```

### Recorrer un diccionario

```python
for key in profile:
    print(key)

for value in profile.values():
    print(value)

for key, value in profile.items():
    print(key, value)
```

En Python 3.8 los diccionarios conservan el orden de inserción. Esa propiedad no significa que sus claves estén ordenadas alfabética o numéricamente.

### Eliminar duplicados conservando el orden

Como un diccionario conserva el orden de inserción y no admite claves repetidas, puede usarse para eliminar duplicados de una lista sin perder cuál apareció primero:

```python
names = ["Ana", "Luis", "Ana", "Marta"]
unique_in_order = list(dict.fromkeys(names))
print(unique_in_order)  # ['Ana', 'Luis', 'Marta']
```

`dict.fromkeys(names)` crea un diccionario cuyas claves son los elementos de `names`; como las claves no se repiten, cada nombre queda una sola vez, en el orden en que apareció primero. A diferencia de convertir la lista en un `set` (visto en la sección de conjuntos), este método conserva ese orden.

### Encontrar la clave con el mayor valor

Puede recorrerse el diccionario conservando la mejor clave encontrada:

```python
def highest_rated(ratings):
    if not ratings:
        return None

    best_key = None
    best_value = None

    for key, value in ratings.items():
        if best_key is None or value > best_value:
            best_key = key
            best_value = value

    return best_key
```

```python
print(highest_rated({"a": 3, "b": 9, "c": 5}))  # b
print(highest_rated({}))                        # None
```

La comprobación inicial cubre tanto un diccionario vacío como `None`. No se debe iniciar `best_value` en cero, porque todos los valores podrían ser negativos.

Una forma idiomática equivalente para un diccionario no vacío es:

```python
best_key = max(ratings, key=ratings.get)
```

Si varios elementos comparten el máximo, hay que definir cómo resolver el empate. Sin una regla adicional, el recorrido conserva la primera clave máxima encontrada según el orden de inserción.

## Comprensiones

Las comprensiones crean colecciones nuevas a partir de iterables:

```python
squares = [number * number for number in range(6)]
even_squares = [number * number for number in range(6) if number % 2 == 0]

initials = {name[0] for name in ["Ada", "Alan", "Grace"]}
lengths = {name: len(name) for name in ["Ada", "Grace"]}

print(squares)       # [0, 1, 4, 9, 16, 25]
print(even_squares)  # [0, 4, 16]
print(initials)      # {'A', 'G'}
print(lengths)       # {'Ada': 3, 'Grace': 5}
```

Una comprensión breve puede ser más clara que un bucle con `append()`. Si contiene muchas condiciones o efectos secundarios, un bucle convencional suele ser más legible.

## Elegir entre modificar y devolver una copia

Una API debe dejar clara su política de mutación:

```python
def add_tag_in_place(tags, tag):
    tags.add(tag)
    return tags


def with_added_tag(tags, tag):
    new_tags = tags.copy()
    new_tags.add(tag)
    return new_tags
```

```python
original_tags = {"python"}

add_tag_in_place(original_tags, "linux")
print(original_tags)  # {'python', 'linux'}

more_tags = {"python"}
result_tags = with_added_tag(more_tags, "linux")
print(more_tags)    # {'python'}
print(result_tags)  # {'python', 'linux'}
```

La primera función cambia el argumento. La segunda preserva la entrada. Ningún enfoque es siempre superior; lo importante es que el nombre, la documentación y el comportamiento sean coherentes.

## Errores frecuentes

### Confundir `{}` con un conjunto vacío

`{}` crea un diccionario. Para un conjunto vacío se utiliza `set()`.

### Esperar un valor de un método que modifica una lista

`append()`, `sort()` y `reverse()` modifican la lista y devuelven `None`.

### Aceptar accidentalmente índices negativos

Python admite posiciones negativas. Si una función define índices al estilo de otros lenguajes, debe comprobar `index < 0` antes de acceder.

### Modificar una colección compartida

Asignar una lista a otro nombre no la copia. Los cambios serán visibles mediante ambas referencias.

### Suponer un orden para un conjunto

El orden de impresión de un conjunto no es un contrato. Para salidas comparables puede utilizarse `sorted()`.

### Usar una lista como clave o elemento de un conjunto

Las listas son mutables y no son *hashable*. Una tupla de valores inmutables puede utilizarse cuando se necesita una clave compuesta.

### Usar cero como máximo inicial

Una colección puede contener solamente valores negativos. Conviene inicializar desde un elemento real o mantener por separado si todavía no se encontró uno.

### Crear matrices con filas compartidas

Multiplicar una lista que contiene otra lista repite referencias. Una comprensión crea filas independientes.

### Usar una colección como valor predeterminado de un parámetro

Un valor predeterminado se evalúa una sola vez, al definir la función. Si ese valor es una lista, un conjunto o un diccionario, todas las llamadas que no proporcionen el argumento comparten el mismo objeto:

```python
def append_item(item, items=[]):
    items.append(item)
    return items
```

Si la función modifica esa colección, los cambios persisten entre llamadas distintas aunque cada llamada parezca independiente. El patrón seguro consiste en usar `None` como valor predeterminado y crear la colección dentro de la función cuando sea necesario.

### Cambiar el tamaño durante un recorrido

Agregar o eliminar elementos de una lista o diccionario durante la iteración puede producir resultados inesperados o errores. Es preferible recorrer una copia o construir una colección nueva.

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

### Comprobar mutación e identidad

```python
values = [1, 2, 3]
result = replace_if_valid(values, 1, 9)

assert values == [1, 9, 3]
assert result is values
```

### Probar casos límite

Las comprobaciones deberían incluir:

- colecciones vacías;
- un solo elemento;
- índices negativos y fuera de rango;
- valores repetidos;
- tuplas más cortas y más largas de lo esperado;
- diccionarios `None` o vacíos cuando estén admitidos;
- valores negativos al buscar máximos;
- entradas originales que deben conservarse sin cambios.

## Referencias

- [Estructuras de datos — Tutorial de Python](https://docs.python.org/3/tutorial/datastructures.html)
- [Conjuntos — Tutorial de Python](https://docs.python.org/3/tutorial/datastructures.html#sets)
- [Diccionarios — Tutorial de Python](https://docs.python.org/3/tutorial/datastructures.html#dictionaries)
- [Tipos integrados](https://docs.python.org/3/library/stdtypes.html)
- [Sintaxis de cadenas de formato](https://docs.python.org/3/library/string.html#format-string-syntax)
- [Estructuras de datos — documentación de Python 3.8](https://docs.python.org/3.8/tutorial/datastructures.html)
- [Tipos integrados — documentación de Python 3.8](https://docs.python.org/3.8/library/stdtypes.html)
