# Python: clases y modelo de objetos

Hasta este punto, los datos de un programa se representaron con variables individuales o se agruparon mediante listas, tuplas o diccionarios. Esas estructuras alcanzan cuando los datos no necesitan comportamiento propio: una lista de números no hace nada por sí misma, una función externa decide qué operación aplicarle.

Una clase funciona como un plano: describe qué datos y qué comportamiento va a tener cada objeto construido a partir de ella. Así como el plano de una casa no es una casa, sino la descripción que permite construir varias casas con la misma estructura pero con datos propios (una dirección distinta, una cantidad de habitaciones distinta), una clase permite crear varios objetos que comparten estructura pero mantienen su propio estado.

En Python, esta idea se aplica incluso a lo que ya se usaba antes: los números, las cadenas, las listas y los diccionarios también son objetos, construidos a partir de clases que el propio lenguaje define.

## El modelo de objetos de Python

Todo objeto posee tres características fundamentales:

- **Identidad:** distingue al objeto de los demás durante su tiempo de vida.
- **Tipo:** determina qué operaciones admite.
- **Valor:** representa los datos contenidos por el objeto.

```python
message = "Hello"

print(type(message))
print(id(message))
print(message)
```

`type()` devuelve el tipo del objeto. `id()` devuelve un entero que representa su identidad durante su tiempo de vida.

En CPython, el valor de `id()` suele coincidir con una dirección de memoria, pero esto es un detalle de implementación. No debe utilizarse para realizar cálculos ni para deducir dónde se almacena un objeto. Python también puede reutilizar un identificador después de que el objeto original deje de existir.

### Identidad e igualdad

Los operadores `is` y `==` responden preguntas diferentes:

- `is` comprueba si dos nombres hacen referencia al mismo objeto.
- `==` comprueba si dos objetos tienen valores equivalentes.

```python
first = [1, 2]
second = [1, 2]
alias = first

print(first == second)  # True
print(first is second)  # False
print(first is alias)   # True
```

`is` se utiliza principalmente con objetos únicos como `None`:

```python
if result is None:
    print("Sin resultado")
```

No se debe usar `is` para comparar números o cadenas. Algunas implementaciones reutilizan determinados objetos inmutables como optimización, por lo que el resultado podría parecer correcto de forma accidental.

## Explorar objetos existentes

El intérprete interactivo permite examinar los objetos antes de definir clases propias.

```bash
python3
```

Herramientas útiles:

| Herramienta | Información que proporciona |
|---|---|
| `type(object)` | Tipo o clase del objeto |
| `id(object)` | Identidad del objeto |
| `dir(object)` | Nombres de atributos y métodos disponibles |
| `help(object)` | Documentación interactiva |
| `object.__dict__` | Atributos almacenados en el espacio de nombres, cuando existe |
| `isinstance(object, Class)` | Si el objeto fue creado a partir de una clase o de una subclase |

```python
text = "python"

print(type(text))
print(text.upper())
print(text.replace("p", "P"))
```

`dir()` es una herramienta de exploración, no una lista que deba copiarse o memorizarse. Muchos de los nombres rodeados por dobles guiones corresponden a métodos especiales utilizados por Python.

## Clases e instancias

Una **clase** describe una estructura y un comportamiento. Una **instancia** es un objeto concreto creado a partir de esa clase.

```python
class Counter:
    """Representa un contador con un valor entero."""

    def __init__(self, initial=0):
        self.value = initial

    def increment(self):
        """Incrementa en uno el valor almacenado."""
        self.value += 1
```

```python
first = Counter()
second = Counter(10)

first.increment()

print(first.value)   # 1
print(second.value)  # 10
```

`first` y `second` pertenecen a la misma clase, pero son objetos distintos y mantienen estados independientes.

### Crear una instancia

Una clase se instancia llamándola como si fuera una función:

```python
counter = Counter(5)
```

Durante esta operación Python crea la instancia y después llama a `__init__()` para inicializarla. En el uso cotidiano, `__init__()` suele llamarse constructor, aunque técnicamente inicializa un objeto que ya fue creado. La creación propiamente dicha corresponde a `__new__()`, que rara vez necesita redefinirse en clases sencillas.

## El parámetro `self`

El primer parámetro de un método de instancia se llama `self` por convención. Representa la instancia sobre la que se está trabajando.

```python
class Greeter:
    """Crea saludos personalizados."""

    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hola, {self.name}"
```

Al ejecutar:

```python
person = Greeter("Ada")
print(person.greet())
```

Python pasa `person` automáticamente como primer argumento. De forma conceptual, estas llamadas son equivalentes:

```python
person.greet()
Greeter.greet(person)
```

`self` no es una palabra reservada, pero cambiar su nombre dificulta la lectura y contradice la convención utilizada por la comunidad.

## Atributos de instancia y de clase

Los atributos de instancia contienen datos propios de cada objeto. Normalmente se crean mediante asignaciones a `self`:

```python
class User:
    """Representa una cuenta de usuario."""

    def __init__(self, username):
        self.username = username
```

Los atributos de clase pertenecen a la clase y se comparten entre sus instancias:

```python
class User:
    """Representa una cuenta de usuario."""

    category = "standard"

    def __init__(self, username):
        self.username = username
```

```python
first = User("ada")
second = User("grace")

print(first.category)
print(second.category)
```

### Evitar estado mutable compartido

Una lista o un diccionario definido como atributo de clase será compartido por todas las instancias:

```python
class Playlist:
    tracks = []  # Shared by every instance: usually a mistake
```

Si cada objeto necesita su propia colección, debe crearse dentro de `__init__()`:

```python
class Playlist:
    """Almacena una colección independiente de pistas."""

    def __init__(self):
        self.tracks = []
```

Esta diferencia evita que una modificación realizada mediante una instancia aparezca inesperadamente en otra.

## Métodos de instancia

Un método define un comportamiento relacionado con el estado del objeto.

```python
class Product:
    """Representa un producto con un precio y una cantidad."""

    def __init__(self, price, quantity):
        self.price = price
        self.quantity = quantity

    def total(self):
        """Devuelve el precio total."""
        return self.price * self.quantity
```

Es importante distinguir entre **devolver** e **imprimir**:

```python
def total(self):
    return self.price * self.quantity
```

Un valor devuelto puede guardarse, combinarse o probarse. Un método que llama a `print()` produce una salida en la terminal y, si no tiene un `return` explícito, devuelve `None`.

Como regla general, la lógica de cálculo debería devolver valores. La impresión se reserva para comportamientos cuyo propósito sea generar salida.

## Encapsulamiento en Python

El encapsulamiento reúne datos y operaciones relacionadas y protege las reglas que mantienen válido el estado de un objeto.

Python no impone atributos privados inaccesibles de la misma forma que algunos otros lenguajes. Utiliza convenciones y un mecanismo llamado **name mangling**.

| Nombre | Significado habitual |
|---|---|
| `name` | Parte pública de la interfaz |
| `_name` | Detalle interno; no debería usarse desde fuera sin necesidad |
| `__name` | Activa name mangling para reducir colisiones accidentales |
| `__name__` | Nombre especial definido por el modelo de datos de Python |

Cuando una clase asigna `self.__value`, Python transforma internamente el nombre para incluir la clase:

```python
class Sample:
    def __init__(self, value):
        self.__value = value
```

En una instancia de `Sample`, el atributo se almacena normalmente como `_Sample__value`. Esto no constituye una barrera de seguridad: el atributo todavía puede inspeccionarse o modificarse deliberadamente. El objetivo del mecanismo es evitar accesos accidentales y colisiones de nombres, especialmente durante la herencia.

No se deben inventar nombres como `__calculate__`. Los nombres con dobles guiones a ambos lados están reservados para operaciones especiales reconocidas por Python.

## Propiedades

Una propiedad permite ofrecer acceso mediante la sintaxis de un atributo y ejecutar lógica al leer o modificar su valor.

```python
class Ticket:
    """Representa un pedido de entradas."""

    def __init__(self, quantity=0):
        self.quantity = quantity

    @property
    def quantity(self):
        """Devuelve la cantidad de entradas."""
        return self.__quantity

    @quantity.setter
    def quantity(self, value):
        if type(value) is not int:
            raise TypeError("quantity must be an integer")
        if value < 0:
            raise ValueError("quantity must be >= 0")
        self.__quantity = value
```

```python
order = Ticket(2)
print(order.quantity)

order.quantity = 4
print(order.quantity)
```

La asignación `self.quantity = quantity` dentro de `__init__()` utiliza el setter. Así se centraliza la validación y se evitan reglas duplicadas.

### `@property` frente a getters y setters explícitos

En Python se prefieren normalmente las propiedades cuando el valor debe leerse y asignarse como un atributo:

```python
order.quantity
order.quantity = 3
```

Los métodos explícitos, como `get_quantity()` y `set_quantity()`, pueden ser apropiados si la operación es costosa, asíncrona o representa una acción más compleja que una lectura o asignación.

## Validar el estado

Una clase debería impedir la creación de estados inválidos. Las validaciones se pueden aplicar durante la inicialización y cada vez que un atributo cambia.

Es habitual distinguir:

- `TypeError`: el objeto recibido es de un tipo no admitido.
- `ValueError`: el tipo es correcto, pero el valor está fuera del dominio permitido.

```python
if type(value) is not int:
    raise TypeError("value must be an integer")

if value < 0:
    raise ValueError("value must be >= 0")
```

El orden importa. Primero se comprueba el tipo y después se realizan comparaciones o cálculos que dependen de ese tipo.

### El caso especial de `bool`

`bool` es una subclase de `int` en Python:

```python
print(isinstance(True, int))  # True
print(type(True) is int)     # False
```

Si una interfaz admite enteros pero debe rechazar booleanos, una comprobación estricta con `type(value) is int` resulta adecuada. Si se desean aceptar subclases de `int`, se utiliza `isinstance(value, int)`. La decisión depende del contrato de la clase.

### Mantener invariantes

Una **invariante** es una condición que siempre debe cumplirse mientras el objeto sea válido. Algunos ejemplos son:

- una cantidad no puede ser negativa;
- una fecha final no puede ser anterior a la inicial;
- una posición debe contener exactamente dos coordenadas válidas;
- un identificador no puede estar vacío.

Todas las rutas que modifican el estado deben preservar esas condiciones.

## Métodos especiales

Los métodos especiales permiten que los objetos definidos por el usuario participen en operaciones del lenguaje. Sus nombres comienzan y terminan con dobles guiones; por eso suelen llamarse *dunder methods*.

| Método | Operación relacionada |
|---|---|
| `__init__()` | Inicialización de una instancia |
| `__str__()` | `str(object)` y `print(object)` |
| `__repr__()` | `repr(object)` y representación en el intérprete |
| `__eq__()` | Comparación con `==` |
| `__len__()` | Llamada a `len(object)` |

Solo se deben implementar los métodos especiales que tengan sentido para el objeto modelado.

## `__str__()` y `__repr__()`

`__str__()` devuelve una representación legible para una persona. `__repr__()` devuelve una representación útil para desarrollo y depuración, idealmente inequívoca.

```python
class Book:
    """Representa un libro."""

    def __init__(self, title):
        self.title = title

    def __str__(self):
        return self.title

    def __repr__(self):
        return f"Book({self.title!r})"
```

```python
book = Book("Python Basics")

print(str(book))
print(repr(book))
print(book)
```

Ambos métodos deben **devolver una cadena**. No deben limitarse a imprimirla.

Si una clase define `__repr__()` pero no `__str__()`, Python puede utilizar `__repr__()` como representación informal. La operación inversa no ocurre de la misma manera, por lo que resulta útil definir un `__repr__()` claro incluso cuando también existe `__str__()`.

## Construir salidas de texto

Cuando un objeto debe producir varias líneas, suele ser más sencillo construir primero la cadena y después imprimirla o devolverla.

```python
symbol = "#"
width = 4
height = 3
offset = 2

rows = [" " * offset + symbol * width for _ in range(height)]
result = "\n".join(rows)

print(result)
```

Resultado:

```text
  ####
  ####
  ####
```

Conceptos útiles:

- `text * n` repite una cadena `n` veces;
- `"\n".join(lines)` une elementos insertando saltos de línea;
- `print("")` imprime una línea vacía;
- una representación devuelta por `__str__()` no debería incluir un salto final salvo que forme parte del formato definido.

Separar la construcción del texto de su impresión facilita reutilizar el resultado y probarlo.

## Diseñar una clase sencilla

Antes de escribir código conviene responder:

1. ¿Qué concepto representa la clase?
2. ¿Qué datos forman el estado de cada instancia?
3. ¿Qué condiciones debe cumplir ese estado?
4. ¿Qué operaciones pertenecen naturalmente al objeto?
5. ¿Qué parte de la interfaz será pública?
6. ¿Qué representación facilitará su lectura o depuración?

Una clase no debería limitarse a agrupar variables sin criterio. Los datos y métodos deben tener una relación clara y mantener un conjunto coherente de reglas.

### Responsabilidad única

Una clase es más fácil de comprender cuando tiene una responsabilidad principal. Si una clase valida usuarios, escribe archivos, consulta una API y genera informes al mismo tiempo, probablemente contenga responsabilidades que deberían separarse.

### Composición

Los objetos pueden contener otros objetos. Esta relación se conoce como composición:

```python
class Address:
    def __init__(self, city):
        self.city = city


class Customer:
    def __init__(self, name, address):
        self.name = name
        self.address = address
```

```python
address = Address("Montevideo")
customer = Customer("Ada", address)

print(customer.address.city)
```

La composición suele expresar una relación del tipo “tiene un”. La herencia, que expresa una relación del tipo “es un”, debe reservarse para casos en los que exista una relación conceptual y de comportamiento clara.

## Documentación y estilo

Las clases utilizan normalmente nombres en `PascalCase`, mientras que los atributos y métodos utilizan `snake_case`:

```python
class ShoppingCart:
    """Almacena los productos seleccionados por un cliente."""

    def calculate_total(self):
        """Devuelve el precio total de todos los productos."""
        return 0
```

Las docstrings describen el propósito observable del módulo, la clase o el método. No deberían repetir literalmente el código.

Para archivos ejecutables en sistemas Unix puede utilizarse:

```python
#!/usr/bin/env python3
```

El código debe terminar con una nueva línea y seguir las convenciones de estilo aplicables al entorno utilizado.

## Comprobar el comportamiento

Las pruebas deben observar la interfaz pública y las reglas de la clase:

- creación con valores válidos;
- valores predeterminados;
- independencia entre instancias;
- actualización de propiedades;
- rechazo de tipos incorrectos;
- rechazo de valores fuera del dominio;
- valores límite, como cero;
- retorno de métodos de cálculo;
- contenido exacto de las representaciones de texto.

Ejemplo con `assert`:

```python
order = Ticket(2)
assert order.quantity == 2

order.quantity = 0
assert order.quantity == 0
```

Para comprobar excepciones sin una biblioteca externa:

```python
try:
    Ticket(-1)
except ValueError as error:
    assert str(error) == "quantity must be >= 0"
else:
    raise AssertionError("ValueError was not raised")
```

También conviene crear dos instancias y modificar solo una para verificar que no comparten estado accidentalmente.

## Errores frecuentes

- Confundir una clase con una instancia.
- Omitir `self` en la definición o en el acceso a atributos.
- Crear atributos locales dentro de `__init__()` en lugar de atributos de instancia.
- Colocar listas o diccionarios mutables como atributos de clase sin querer compartirlos.
- Duplicar validaciones en vez de centralizarlas en una propiedad.
- Usar `is` cuando se quiere comparar valores.
- Considerar `__name` como una protección de seguridad.
- Aceptar accidentalmente `True` o `False` al validar enteros.
- Imprimir dentro de un método que debería devolver un resultado.
- Hacer que `__str__()` imprima en lugar de devolver una cadena.
- Devolver desde `__str__()` un objeto que no sea `str`.
- Modificar directamente un atributo interno y romper las invariantes.
- Depender de la estructura exacta de `__dict__` como parte de la interfaz pública.

## Compatibilidad entre versiones

Los fundamentos de clases, propiedades y métodos especiales descritos aquí son compatibles con Python 3.8 y versiones posteriores. La documentación general de `docs.python.org/3/` muestra la versión estable seleccionada por el sitio, que puede ser más reciente que el intérprete utilizado localmente.

Cuando un detalle pueda depender de la versión, se puede seleccionar la versión correspondiente desde el menú de la documentación o utilizar una URL específica, por ejemplo:

- [Tutorial de clases de Python 3.8](https://docs.python.org/3.8/tutorial/classes.html)
- [Modelo de datos de Python 3.8](https://docs.python.org/3.8/reference/datamodel.html)

## Recursos

### Documentación oficial

- [Tutorial de Python: clases](https://docs.python.org/3/tutorial/classes.html)
- [Modelo de datos de Python](https://docs.python.org/3/reference/datamodel.html)
- [Nombres de métodos especiales](https://docs.python.org/3/reference/datamodel.html#special-method-names)
- [Función incorporada `property()`](https://docs.python.org/3/library/functions.html#property)
- [Guía de estilo PEP 8](https://peps.python.org/pep-0008/)

### Lecturas complementarias

- [Programación orientada a objetos en Python — Real Python](https://realpython.com/python3-object-oriented-programming/)
- [Encapsulamiento en Python — GeeksforGeeks](https://www.geeksforgeeks.org/python/encapsulation-in-python/)
- [Clases y objetos en Python — GeeksforGeeks](https://www.geeksforgeeks.org/python/python-classes-and-objects/)

Para resolver dudas sobre el comportamiento del lenguaje, la documentación oficial de Python debe considerarse la referencia principal.
