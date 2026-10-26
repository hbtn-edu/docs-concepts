# Python: herencia y polimorfismo

La programación orientada a objetos permite relacionar clases para reutilizar comportamiento y representar variaciones de un mismo concepto. En Python, la **herencia** crea una clase especializada a partir de otra, mientras que el **polimorfismo** permite trabajar con objetos diferentes mediante una interfaz común.

La herencia puede reducir duplicación, pero también crea una relación fuerte entre las clases. Antes de utilizarla conviene comprobar que la subclase representa realmente una versión especializada de la clase base.

## Terminología fundamental

| Término | Significado |
|---|---|
| Clase base | Clase de la que se hereda comportamiento |
| Clase derivada | Clase que hereda de otra clase |
| Clase padre | Sinónimo frecuente de clase base |
| Clase hija | Sinónimo frecuente de clase derivada |
| Superclase | Clase situada por encima en una jerarquía |
| Subclase | Clase situada por debajo en una jerarquía |
| Sobrescritura | Nueva implementación de un método heredado |
| Polimorfismo | Uso de una misma operación con objetos de distintos tipos |

Una relación de herencia suele expresar que la clase derivada **es un tipo de** la clase base:

```text
Vehicle
   └── Car
          └── ElectricCar
```

Un `Car` es un `Vehicle`, y un `ElectricCar` es tanto un `Car` como un `Vehicle`.

## Definir una subclase

La clase base se escribe entre paréntesis después del nombre de la clase derivada:

```python
class Vehicle:
    """Representa un vehículo genérico."""

    def move(self):
        return "El vehículo se está moviendo"


class Bicycle(Vehicle):
    """Representa una bicicleta."""

    pass
```

```python
bicycle = Bicycle()
print(bicycle.move())
```

Aunque `Bicycle` no define `move()`, la instancia puede utilizar el método heredado de `Vehicle`.

### Búsqueda de atributos

Cuando se evalúa `object.attribute`, Python busca el nombre siguiendo la jerarquía:

1. en la instancia;
2. en la clase de la instancia;
3. en sus clases base según el orden de resolución de métodos (se explica más adelante, en la sección dedicada);
4. continúa hasta llegar a `object`.

La búsqueda se detiene en la primera coincidencia. Por eso una definición en la subclase puede reemplazar el comportamiento visible de la clase base.

## Sobrescribir métodos

Una subclase sobrescribe un método cuando define otro con el mismo nombre:

```python
class Vehicle:
    """Representa un vehículo genérico."""

    def move(self):
        return "El vehículo se está moviendo"


class Bicycle(Vehicle):
    """Representa una bicicleta."""

    def move(self):
        return "La bicicleta está rodando"
```

```python
vehicle = Vehicle()
bicycle = Bicycle()

print(vehicle.move())
print(bicycle.move())
```

La versión elegida depende del tipo real del objeto. No es necesario escribir una condición que compruebe manualmente cada clase.

La firma del método sobrescrito debería conservar un contrato compatible con el método original. Si una subclase exige argumentos inesperados o devuelve un resultado incompatible, puede dejar de ser utilizable en los lugares donde se espera la clase base.

## Extender comportamiento con `super()`

Sobrescribir no siempre significa descartar completamente la implementación heredada. `super()` permite delegar parte del trabajo a la siguiente clase del orden de resolución.

```python
class Employee:
    """Representa un empleado."""

    def __init__(self, name):
        self.name = name

    def describe(self):
        return f"Empleado: {self.name}"


class Manager(Employee):
    """Representa un empleado que gestiona un equipo."""

    def __init__(self, name, team_size):
        super().__init__(name)
        self.team_size = team_size

    def describe(self):
        base_description = super().describe()
        return f"{base_description}, team size: {self.team_size}"
```

`super().__init__(name)` inicializa la parte de la instancia definida por `Employee`. Después, `Manager` agrega su propio estado.

Es preferible utilizar `super()` en lugar de llamar directamente a `Employee.__init__(self, name)`. `super()` respeta el orden de resolución de métodos (MRO, explicado más adelante) y funciona mejor si la jerarquía cambia o incorpora herencia múltiple.

## Constructores y herencia

Si una subclase no define `__init__()`, utiliza el inicializador heredado:

```python
class User:
    def __init__(self, username):
        self.username = username


class Moderator(User):
    pass
```

```python
moderator = Moderator("ada")
print(moderator.username)
```

Si la subclase define su propio `__init__()`, el inicializador de la clase base no se ejecuta automáticamente. Debe invocarse cuando sea necesario:

```python
class Moderator(User):
    def __init__(self, username, section):
        super().__init__(username)
        self.section = section
```

No llamar al inicializador de la clase base puede dejar atributos esperados sin crear.

## Reutilizar validaciones

Una clase base puede centralizar reglas que varias subclases necesitan:

```python
class ValidatedRecord:
    """Provee funciones auxiliares de validación reutilizables."""

    def validate_positive_integer(self, name, value):
        if type(value) is not int:
            raise TypeError(f"{name} must be an integer")
        if value <= 0:
            raise ValueError(f"{name} must be greater than 0")
```

```python
class InventoryItem(ValidatedRecord):
    """Representa un artículo almacenado en inventario."""

    def __init__(self, code, quantity):
        self.validate_positive_integer("quantity", quantity)
        self.code = code
        self.quantity = quantity
```

La validación se define una sola vez y queda disponible para las clases derivadas. Los mensajes de error pueden incorporar el nombre recibido para que el método sea reutilizable con diferentes atributos.

### El caso de los booleanos

En Python, `bool` hereda de `int`:

```python
print(isinstance(True, int))   # True
print(issubclass(bool, int))   # True
```

Cuando una regla debe admitir enteros pero rechazar `True` y `False`, puede utilizarse una comparación estricta:

```python
if type(value) is not int:
    raise TypeError("value must be an integer")
```

`isinstance(value, int)` es adecuado cuando también se desean aceptar subclases de `int`.

## Métodos que establecen una interfaz

Una clase base puede declarar un comportamiento que las subclases deben proporcionar:

```python
class Exporter:
    """Define a common export interface."""

    def export(self, data):
        raise NotImplementedError("Subclasses must implement export()")
```

Algunos ejercicios pueden exigir en cambio una excepción genérica `Exception`, u otro tipo o mensaje específico: ese contrato manda por sobre la recomendación general anterior, así que siempre hay que seguir lo que pida el enunciado concreto.

```python
class TextExporter(Exporter):
    """Exporta valores como texto."""

    def export(self, data):
        return str(data)
```

`NotImplementedError` comunica que una operación debe implementarse en una subclase. Si una interfaz externa exige otro tipo de excepción o un mensaje exacto, ese contrato debe respetarse.

Para sistemas más grandes, el módulo estándar `abc` permite definir clases base abstractas y marcar métodos con `@abstractmethod`. Una clase abstracta impide crear instancias mientras existan métodos abstractos sin implementar.

```python
from abc import ABC, abstractmethod


class Exporter(ABC):
    @abstractmethod
    def export(self, data):
        """Devuelve una representación exportada de los datos."""
```

Esta herramienta formal resulta útil para bibliotecas y jerarquías extensas, pero no es necesaria para todas las clases base.

## Polimorfismo

El polimorfismo permite enviar la misma operación a objetos diferentes y obtener el comportamiento correspondiente a cada objeto.

```python
class TextReport:
    def render(self):
        return "Informe de texto plano"


class HtmlReport:
    def render(self):
        return "<p>Informe en HTML</p>"
```

```python
reports = [TextReport(), HtmlReport(), TextReport()]

for report in reports:
    print(report.render())
```

El bucle no necesita preguntar si el objeto es `TextReport` o `HtmlReport`. Solo depende de que cada objeto proporcione `render()`.

### Polimorfismo mediante herencia

Las subclases pueden compartir una clase base y sobrescribir un método común:

```python
class Notification:
    def send(self, message):
        raise NotImplementedError


class EmailNotification(Notification):
    def send(self, message):
        return f"Correo: {message}"


class ConsoleNotification(Notification):
    def send(self, message):
        return f"Consola: {message}"
```

### Duck typing

Python también admite polimorfismo sin una clase base común. Si un objeto ofrece el comportamiento requerido, puede utilizarse:

```python
def publish(renderer):
    return renderer.render()
```

Esta idea suele resumirse como *duck typing*: interesa lo que el objeto puede hacer, no únicamente el nombre de su clase.

No es necesario anteponer una comprobación `isinstance()` a cada operación. A menudo es preferible utilizar la interfaz esperada y manejar los errores en el límite donde ingresan datos externos.

## `isinstance()`

`isinstance(object, classinfo)` determina si un objeto es instancia de una clase o de alguna de sus subclases.

```python
class Document:
    pass


class PdfDocument(Document):
    pass
```

```python
document = PdfDocument()

print(isinstance(document, PdfDocument))  # True
print(isinstance(document, Document))     # True
print(isinstance(document, object))       # True
```

También puede comprobarse un conjunto de clases mediante una tupla:

```python
print(isinstance(document, (Document, str)))
```

La tupla es compatible con Python 3.8 y versiones posteriores. Las uniones escritas como `TypeA | TypeB` en `classinfo` requieren versiones más recientes.

### `isinstance()` frente a `type()`

```python
type(document) is Document
isinstance(document, Document)
```

La primera expresión solo es verdadera cuando el tipo exacto es `Document`. La segunda también acepta cualquier subclase. Para código preparado para extensión, normalmente se prefiere `isinstance()`.

Una comparación estricta con `type()` sigue siendo útil cuando las subclases deben rechazarse deliberadamente, como en ciertas validaciones numéricas.

## `issubclass()`

`issubclass(Class, classinfo)` comprueba relaciones entre clases, no entre instancias.

```python
print(issubclass(PdfDocument, Document))  # True
print(issubclass(PdfDocument, object))    # True
print(issubclass(Document, Document))     # True
```

Una clase se considera subclase de sí misma. La relación puede ser directa o indirecta:

```python
class SignedPdf(PdfDocument):
    pass


print(issubclass(SignedPdf, PdfDocument))
print(issubclass(SignedPdf, Document))
```

También se admite una tupla de clases:

```python
print(issubclass(SignedPdf, (Document, str)))
```

Pasar una instancia como primer argumento produce `TypeError`:

```python
issubclass(document, Document)  # Incorrect: document is an instance
```

Regla práctica:

- `isinstance()` recibe un objeto como primer argumento.
- `issubclass()` recibe una clase como primer argumento.

## Herencia transitiva

La herencia se aplica a toda la cadena:

```python
class Account:
    pass


class StaffAccount(Account):
    pass


class AdminAccount(StaffAccount):
    pass
```

```python
admin = AdminAccount()

print(isinstance(admin, AdminAccount))
print(isinstance(admin, StaffAccount))
print(isinstance(admin, Account))

print(issubclass(AdminAccount, Account))
```

Esto permite que una función diseñada para trabajar con `Account` reciba también un `StaffAccount` o un `AdminAccount`, siempre que las subclases mantengan el contrato esperado.

## Sustitución segura

Una subclase debería poder utilizarse donde se espera la clase base sin romper el comportamiento del programa. Esta idea se conoce como **principio de sustitución de Liskov**.

Una subclase problemática puede:

- rechazar valores aceptados por la clase base;
- exigir argumentos adicionales incompatibles;
- devolver un tipo completamente diferente;
- eliminar un comportamiento que los usuarios de la clase base esperan;
- lanzar excepciones inesperadas para operaciones normales.

La herencia no debe elegirse únicamente porque dos clases comparten algunas líneas de código. La relación conceptual y el contrato público también deben ser compatibles.

## Métodos especiales heredados

Los métodos especiales, como `__str__()`, también se heredan y pueden sobrescribirse:

```python
class Media:
    def __init__(self, title):
        self.title = title

    def __str__(self):
        return f"Contenido: {self.title}"


class Podcast(Media):
    def __str__(self):
        return f"Podcast: {self.title}"
```

```python
media = Media("General content")
podcast = Podcast("Python Weekly")

print(media)
print(podcast)
```

`__str__()` debe devolver un objeto `str`. No debe limitarse a imprimir el texto.

Si una subclase necesita parte de la representación heredada, puede extenderla:

```python
class Podcast(Media):
    def __init__(self, title, episode):
        super().__init__(title)
        self.episode = episode

    def __str__(self):
        base = super().__str__()
        return f"{base}, episode {self.episode}"
```

## Atributos internos y herencia

Los nombres con un guion bajo inicial, como `_value`, indican por convención que el atributo es interno. Las subclases pueden utilizarlo cuando la clase base lo diseña como parte protegida de su implementación.

Los nombres con dos guiones bajos iniciales activan *name mangling*:

```python
class Parent:
    def __init__(self):
        self.__value = 10


class Child(Parent):
    def set_child_value(self):
        self.__value = 20
```

Estos dos atributos no tienen el mismo nombre interno:

- `Parent` almacena `_Parent__value`.
- `Child` almacena `_Child__value`.

El doble guion bajo ayuda a evitar colisiones accidentales, pero no ofrece privacidad absoluta. También puede dificultar que una subclase reutilice el estado de la clase base. Cuando una subclase necesita acceso controlado, suele ser preferible proporcionar una propiedad, un método protegido o una interfaz pública explícita.

## Importar una clase base

La forma habitual de importar una clase es:

```python
from base_module import BaseClass


class DerivedClass(BaseClass):
    pass
```

Python también ofrece `__import__()`, una función de importación dinámica:

```python
BaseClass = __import__("base_module").BaseClass
```

`__import__()` devuelve el módulo y luego se accede a la clase como atributo. En aplicaciones de uso general suelen preferirse las sentencias `import` y `from ... import ...` porque son más claras y fáciles de analizar, pero algunos entornos de evaluación exigen explícitamente `__import__()` para cargar los módulos de forma dinámica. Conviene seguir siempre la convención que indique el enunciado del ejercicio.

## Orden de resolución de métodos

El **Method Resolution Order** o MRO determina dónde busca Python los métodos y atributos heredados.

```python
print(AdminAccount.__mro__)
```

También puede utilizarse:

```python
print(AdminAccount.mro())
```

En una jerarquía simple, Python recorre la clase actual y luego sus bases hasta llegar a `object`. En herencia múltiple utiliza una linearización que mantiene un orden coherente y permite que `super()` coopere entre varias clases.

## Herencia múltiple

Python permite heredar de varias clases:

```python
class Printable:
    def print_data(self):
        return "Imprimiendo datos"


class Serializable:
    def serialize(self):
        return "Datos serializados"


class Report(Printable, Serializable):
    pass
```

La herencia múltiple puede ser útil con clases pequeñas que aportan capacidades bien delimitadas, conocidas como *mixins*. También puede generar ambigüedad si varias clases base definen los mismos métodos o no utilizan `super()` de manera cooperativa.

Antes de aplicarla conviene inspeccionar el MRO y mantener las responsabilidades de cada clase claramente separadas.

## Probar jerarquías de clases

Las comprobaciones deberían cubrir tanto el comportamiento como las relaciones entre tipos:

```python
manager = Manager("Ada", 4)

assert isinstance(manager, Manager)
assert isinstance(manager, Employee)
assert issubclass(Manager, Employee)
assert manager.name == "Ada"
assert manager.team_size == 4
```

También conviene verificar:

- que la subclase herede los métodos esperados;
- que los métodos sobrescritos devuelvan el resultado correcto;
- que `super()` inicialice el estado heredado;
- que las validaciones se mantengan en las subclases;
- que los tipos incorrectos produzcan la excepción esperada;
- que una colección de objetos diferentes responda a la interfaz común;
- que las representaciones de texto sean exactas;
- que las subclases puedan sustituir a la clase base sin romper el código cliente.

## Errores frecuentes

- Utilizar herencia solo para evitar copiar unas pocas líneas.
- Modelar con “es un” una relación que realmente significa “tiene un”.
- Sobrescribir `__init__()` y olvidar inicializar la clase base.
- Llamar directamente a la clase padre cuando `super()` expresa mejor la cooperación.
- Cambiar de forma incompatible la firma o el retorno de un método sobrescrito.
- Comprobar el tipo exacto con `type()` cuando deberían admitirse subclases.
- Pasar una instancia a `issubclass()`.
- Suponer que `isinstance()` solo reconoce la clase exacta.
- Olvidar que una clase es subclase de sí misma.
- Aceptar accidentalmente booleanos al validar enteros.
- Acceder desde una subclase a un nombre con doble guion bajo como si no hubiese *name mangling*.
- Hacer que `__str__()` imprima en lugar de devolver una cadena.
- Crear jerarquías profundas que vuelven difícil determinar dónde se define cada comportamiento.
- Utilizar herencia múltiple sin revisar el MRO.

## Compatibilidad entre versiones

Los ejemplos principales son compatibles con Python 3.8 y versiones posteriores. Para admitir varios tipos con `isinstance()` o `issubclass()`, una tupla ofrece compatibilidad amplia:

```python
isinstance(value, (int, float))
```

Las uniones escritas como `int | float` requieren Python 3.10 o posterior y no deben utilizarse si el código debe ejecutarse en Python 3.8.

La URL general de la documentación puede mostrar una versión más reciente. Cuando un comportamiento pueda depender de la versión, conviene consultar la documentación específica del intérprete utilizado.

## Recursos

### Documentación oficial

- [Tutorial de Python: clases](https://docs.python.org/3/tutorial/classes.html)
- [Tutorial de Python: herencia](https://docs.python.org/3/tutorial/classes.html#inheritance)
- [`isinstance()`](https://docs.python.org/3/library/functions.html#isinstance)
- [`issubclass()`](https://docs.python.org/3/library/functions.html#issubclass)
- [`super()`](https://docs.python.org/3/library/functions.html#super)
- [Clases de Python 3.8](https://docs.python.org/3.8/tutorial/classes.html)
- [Funciones incorporadas de Python 3.8](https://docs.python.org/3.8/library/functions.html)

### Lecturas complementarias

- [Herencia en Python — W3Schools](https://www.w3schools.com/python/python_inheritance.asp)

La documentación oficial de Python debe utilizarse como referencia principal para determinar el comportamiento exacto del lenguaje.
