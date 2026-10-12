# Manejo de excepciones en Python

## Por qué manejar errores

Un programa interactúa con datos que no siempre tienen la forma esperada: un archivo puede no existir, una entrada del usuario puede no ser un número, una división puede tener un divisor igual a cero. Si cada uno de estos fallos detiene la ejecución sin control, el programa resulta frágil y poco confiable.

Python distingue entre dos cosas relacionadas pero distintas:

- un **error** es cualquier situación que impide que el programa continúe de la forma prevista;
- una **excepción** es el mecanismo que usa Python para representar un error durante la ejecución, de modo que pueda detectarse y tratarse con código específico.

Manejar excepciones permite decidir, en cada situación, si el programa debe recuperarse, mostrar un mensaje claro o detenerse de forma controlada, en lugar de terminar abruptamente con un error sin explicación.

## Errores de sintaxis y excepciones

Python distingue dos grandes categorías de errores:

- un **error de sintaxis** impide que el intérprete comprenda el código;
- una **excepción** ocurre durante la ejecución de código sintácticamente válido.

Por ejemplo, olvidar los dos puntos genera un `SyntaxError` antes de ejecutar el programa:

```python
if temperature > 20
    print("Warm")
```

En cambio, dividir entre cero es una operación válida desde el punto de vista sintáctico, pero falla al ejecutarse:

```python
result = 10 / 0
```

Python representa esta situación con una excepción `ZeroDivisionError`. Si ninguna parte del programa la maneja, la ejecución finaliza y aparece un *traceback*.

## Cómo leer un *traceback*

Un *traceback* muestra el recorrido de llamadas que condujo a una excepción. Conviene leerlo desde la última línea hacia arriba:

```text
Traceback (most recent call last):
  File "report.py", line 8, in <module>
    print(calculate_ratio(10, 0))
  File "report.py", line 4, in calculate_ratio
    return total / count
ZeroDivisionError: division by zero
```

La última línea proporciona:

- el tipo de excepción: `ZeroDivisionError`;
- el mensaje: `division by zero`.

Las líneas anteriores indican los archivos, números de línea y llamadas involucradas. El punto donde Python detecta el error no siempre coincide con la causa conceptual, por lo que también hay que revisar los valores recibidos y las decisiones previas del programa.

## Excepciones incorporadas frecuentes

Python incluye una jerarquía de excepciones para describir diferentes fallos:

| Excepción | Situación habitual |
| --- | --- |
| `TypeError` | Una operación recibe un tipo que no admite. |
| `ValueError` | El tipo es aceptable, pero el valor no lo es. |
| `IndexError` | Se intenta acceder a una posición inexistente de una secuencia. |
| `KeyError` | Una clave no existe en un diccionario. |
| `ZeroDivisionError` | El divisor es cero. |
| `NameError` | Se usa un nombre que no está definido. |
| `AttributeError` | Un objeto no posee el atributo solicitado. |
| `FileNotFoundError` | No se encuentra un archivo requerido. |
| `OSError` | Una operación del sistema operativo falla. |

Algunos ejemplos:

```python
int("seven")            # ValueError
"total" + 4            # TypeError
[10, 20][5]             # IndexError
{"name": "Ada"}["age"]  # KeyError
```

Conocer el tipo concreto permite responder únicamente al fallo esperado, sin ocultar errores no relacionados.

## La estructura `try` y `except`

El bloque `try` contiene la operación que puede fallar. El bloque `except` define qué hacer si aparece una excepción determinada:

```python
try:
    age = int(user_input)
except ValueError:
    print("The value must be an integer")  # "el valor debe ser un entero"
```

El flujo es el siguiente:

1. Python comienza a ejecutar el bloque `try`.
2. Si no ocurre ninguna excepción, omite el `except`.
3. Si ocurre una excepción, deja de ejecutar el resto del `try`.
4. Busca el primer `except` compatible con el tipo producido.
5. Si encuentra uno, ejecuta ese bloque y continúa después de la estructura.
6. Si no encuentra uno, la excepción se propaga al nivel que realizó la llamada.

El bloque protegido debe ser tan pequeño como sea razonable. Así resulta claro qué operación puede provocar la excepción:

```python
try:
    quantity = int(raw_quantity)
except ValueError:
    quantity = 0

total = quantity * unit_price
```

Si la multiplicación estuviera innecesariamente dentro del `try`, un error suyo podría confundirse con un problema de conversión.

## Capturar excepciones específicas

Se recomienda indicar siempre las excepciones que el programa sabe manejar:

```python
try:
    result = numerator / denominator
except ZeroDivisionError:
    result = None
```

Un `except:` sin tipo captura incluso señales especiales como `KeyboardInterrupt` y `SystemExit`. Esto puede impedir que el usuario interrumpa el programa o que el proceso finalice correctamente.

`except Exception:` es menos amplio que un `except:` vacío, pero todavía puede esconder errores de programación. Debe reservarse para límites bien definidos donde exista una estrategia real, como registrar el fallo y volver a lanzarlo.

### Varias excepciones con la misma respuesta

Si diferentes tipos requieren exactamente el mismo tratamiento, se agrupan en una tupla:

```python
try:
    result = dividend / divisor
except (TypeError, ZeroDivisionError):
    result = None
```

Los paréntesis son necesarios. Esta sintaxis no equivale a escribir varios argumentos separados después de `except`.

### Respuestas distintas para errores distintos

También pueden declararse varios manejadores:

```python
try:
    position = int(raw_position)
    value = values[position]
except ValueError:
    print("Position must be an integer")  # "la posición debe ser un entero"
except IndexError:
    print("Position is outside the list")  # "la posición está fuera de la lista"
```

Como máximo se ejecuta un `except`: el primero compatible. Las excepciones derivadas deben colocarse antes que sus clases base para que el manejador específico sea alcanzable.

## Obtener información de una excepción

La forma `except ExceptionType as error` conserva la instancia de la excepción:

```python
try:
    number = int(raw_value)
except ValueError as error:
    print(f"Invalid number: {error}")  # "número inválido: <detalle del error>"
```

El objeto contiene información útil, pero los mensajes internos no siempre son adecuados para usuarios finales. En una interfaz conviene mostrar un mensaje claro; en registros de diagnóstico puede conservarse el detalle técnico.

No se deben publicar datos sensibles al imprimir excepciones. Rutas, consultas, identificadores y valores de configuración pueden formar parte del mensaje.

## `else`: código para el caso exitoso

Un bloque `else` se ejecuta solamente si el `try` termina sin excepciones:

```python
try:
    quantity = int(raw_quantity)
except ValueError:
    print("Invalid quantity")  # "cantidad inválida"
else:
    print(f"Quantity accepted: {quantity}")  # "cantidad aceptada: <valor>"
```

`else` permite dejar dentro de `try` únicamente la operación que se desea proteger. Si el código del caso exitoso produce una excepción, esa excepción no será capturada por los manejadores anteriores de la misma estructura.

La forma completa respeta este orden:

```python
try:
    risky_operation()
except ExpectedError:
    recover()
else:
    continue_after_success()
finally:
    cleanup()
```

## `finally`: acciones que siempre deben ocurrir

El bloque `finally` se ejecuta al finalizar la estructura, haya ocurrido o no una excepción:

```python
result = None

try:
    result = total / count
except (TypeError, ZeroDivisionError):
    pass
finally:
    print(f"Result: {result}")  # "resultado: <valor>"
```

Inicializar `result` antes del `try` garantiza que el nombre exista incluso si la división falla antes de asignarle un nuevo valor.

`finally` también se ejecuta cuando:

- la excepción no fue capturada y está por propagarse;
- el bloque ejecuta un `return`;
- un bucle ejecuta `break` o `continue`.

Por ello es apropiado para liberar recursos o restaurar un estado. No conviene escribir `return`, `break` o `continue` dentro de `finally`, porque puede reemplazar un resultado o silenciar una excepción pendiente.

### Context managers para recursos

Cuando un objeto ofrece un administrador de contexto, `with` expresa la limpieza con mayor claridad que un `finally` manual:

```python
with open("notes.txt", encoding="utf-8") as file:
    content = file.read()
```

El archivo se cierra al salir del bloque, incluso si la lectura falla. El mismo patrón se utiliza con bloqueos, conexiones y otros recursos que implementan el protocolo de contexto.

## Imprimir datos de forma segura

Al procesar una secuencia pueden presentarse dos problemas diferentes:

- la posición solicitada no existe, lo que produce `IndexError`;
- el valor existe, pero no admite el formato u operación esperados.

Es importante decidir por separado cuál debe recuperarse y cuál debe propagarse.

### Recorrer posiciones sin consultar la longitud

Un contador y `IndexError` permiten intentar una cantidad limitada de accesos sin usar `len()`:

```python
def show_available_items(items, amount):
    printed = 0

    try:
        while printed < amount:
            print(items[printed], end="")
            printed += 1
    except IndexError:
        pass

    print()
    return printed
```

```python
total = show_available_items(["a", "b", "c"], 5)
# Se imprime: abc
print(total)  # 3
```

El contador se incrementa solamente después de imprimir. De este modo representa la cantidad real procesada si la secuencia termina antes de lo solicitado.

Este patrón presupone que `amount` es un entero no negativo. Una interfaz de uso general debería validar o documentar ese contrato.

### Utilizar el formato como operación validable

El especificador `:d` solicita una representación decimal entera:

```python
print("{:d}".format(42))
```

Un valor incompatible puede provocar `ValueError` o `TypeError`, según el objeto:

```python
def print_decimal(value):
    try:
        print("{:d}".format(value))
    except (TypeError, ValueError):
        return False

    return True
```

Aquí no hace falta consultar el tipo por adelantado. Se intenta la operación requerida y se manejan únicamente sus fallos esperados.

### Continuar después de un elemento incompatible

Si una colección contiene valores heterogéneos, el `try` debe estar dentro del bucle para que un elemento inválido no detenga el procesamiento de los siguientes:

```python
def show_decimal_values(values, amount):
    printed = 0

    for index in range(amount):
        try:
            print("{:d}".format(values[index]), end="")
        except (TypeError, ValueError):
            continue

        printed += 1

    print()
    return printed
```

```python
total = show_decimal_values([10, "x", 20, 30], 4)
# Se imprime: 102030
print(total)  # 3
```

Este ejemplo ignora incompatibilidades de formato, pero no captura `IndexError`. Por tanto, pedir una posición inexistente sigue siendo un error visible. Capturar cada excepción por separado permite que el comportamiento comunique decisiones precisas.

## EAFP y LBYL

En Python es frecuente el enfoque **EAFP** (*Easier to Ask Forgiveness than Permission*): intentar una operación y manejar la excepción esperada.

```python
try:
    value = settings["theme"]
except KeyError:
    value = "light"
```

El enfoque **LBYL** (*Look Before You Leap*) comprueba primero:

```python
if "theme" in settings:
    value = settings["theme"]
else:
    value = "light"
```

Ninguno es siempre superior. EAFP suele ser claro cuando la operación define naturalmente el éxito, evita repetir búsquedas y reduce problemas entre la comprobación y el uso. LBYL es apropiado cuando la validación es barata, expresa mejor la intención o fallar tendría efectos que no pueden deshacerse.

Las excepciones no deben utilizarse para ocultar entradas inválidas sin una política clara.

## Lanzar excepciones con `raise`

Una función debe lanzar una excepción cuando no puede cumplir su contrato y tampoco sabe cómo recuperarse:

```python
def calculate_discount(price, percentage):
    if percentage < 0 or percentage > 100:
        raise ValueError("percentage must be between 0 and 100")  # "el porcentaje debe estar entre 0 y 100"

    return price * percentage / 100
```

```python
print(calculate_discount(200, 10))  # 20.0
```

`raise` puede recibir una instancia con un mensaje:

```python
raise NameError("required name is not available")  # "el nombre requerido no está disponible"
```

También puede recibir la clase directamente:

```python
raise TypeError
```

El mensaje debería explicar qué condición se incumplió y, cuando sea útil, cómo corregirla. No hace falta describir el funcionamiento interno de la función.

### Elegir el tipo correcto

Algunas convenciones habituales son:

- `TypeError` cuando una operación no admite el tipo recibido;
- `ValueError` cuando el tipo es válido, pero el valor queda fuera del dominio permitido;
- `RuntimeError` cuando el fallo no encaja en una categoría más específica;
- una excepción propia cuando el dominio necesita distinguir ese problema de los errores generales.

Las excepciones propias suelen heredar de `Exception` y sus nombres terminan en `Error`:

```python
class InsufficientBalanceError(Exception):
    """Raised when an account cannot cover an operation."""  # "se lanza cuando una cuenta no puede cubrir una operación"
```

## Propagación y relanzamiento

Si una función no captura una excepción, esta se propaga a quien hizo la llamada. Eso no es necesariamente un defecto: una capa superior puede tener más contexto para decidir si reintenta, muestra un mensaje o finaliza.

Dentro de un `except`, `raise` sin argumentos vuelve a lanzar la misma excepción y conserva su *traceback*:

```python
try:
    save_record(record)
except OSError:
    log_failure(record)
    raise
```

No debe reemplazarse por `raise error`, porque esa forma puede alterar la información del punto original del fallo.

### Encadenar excepciones

Una capa puede traducir una excepción técnica a otra que represente mejor su dominio:

```python
try:
    value = int(raw_value)
except ValueError as error:
    raise RuntimeError("configuration contains an invalid number") from error  # "la configuración contiene un número inválido"
```

`from error` conserva explícitamente la causa. El *traceback* muestra tanto el error original como la excepción nueva.

## Excepciones y valores de retorno

Devolver un valor especial y lanzar una excepción representan contratos distintos:

- devolver `None` puede indicar que la ausencia es un resultado previsto;
- lanzar una excepción indica que la función no pudo completar la operación normal.

```python
def find_user(users, username):
    for user in users:
        if user["username"] == username:
            return user

    return None
```

```python
users = [{"username": "ada"}, {"username": "alan"}]

print(find_user(users, "ada"))     # {'username': 'ada'}
print(find_user(users, "grace"))   # None
```

Si `None` también pudiera ser un resultado válido, el contrato sería ambiguo y una excepción específica podría ser más clara.

No conviene mezclar silenciosamente ambos enfoques. Quien llama a la función debe saber si tiene que comprobar un valor o capturar una excepción.

## Errores frecuentes

### Capturar demasiado

```python
try:
    process_data()
except:
    pass
```

Este patrón oculta cualquier fallo y elimina información necesaria para depurar. Hay que capturar tipos específicos y realizar una acción intencional.

### Envolver demasiadas instrucciones en `try`

Cuanto mayor sea el bloque, más difícil resulta identificar qué operación se pretendía proteger. Las acciones del caso exitoso pueden ir en `else`.

### Continuar con un estado incompleto

Después de una excepción, algunas asignaciones quizá no ocurrieron. Los nombres necesarios en `except` o `finally` deben inicializarse antes del `try`.

### Usar excepciones como sustituto de toda validación

Las excepciones son apropiadas para fallos de operaciones. Las reglas simples de dominio, como un porcentaje permitido, suelen expresarse mejor con una condición y un `raise` claro.

### Silenciar una excepción sin motivo

`pass` puede ser correcto cuando ignorar exactamente esa excepción forma parte del contrato. Si no existe una razón concreta, el fallo debería propagarse o registrarse.

### Retornar desde `finally`

Un `return` en `finally` reemplaza otros retornos y puede impedir la propagación de una excepción. La limpieza debe realizarse sin cambiar el flujo pendiente.

### Confundir `TypeError` y `ValueError`

`int([])` falla porque la lista no es un tipo aceptado; `int("hello")` recibe una cadena, pero su contenido no representa un entero. El primer caso produce `TypeError` y el segundo `ValueError`.

### Usar `assert` para validar entradas externas

Las aserciones sirven para comprobar invariantes internas durante el desarrollo y pueden desactivarse al ejecutar Python con optimizaciones. Para entradas de usuarios o contratos públicos deben utilizarse condiciones y excepciones explícitas.

## Comprobaciones útiles

### Ejecutar el archivo

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

### Probar casos representativos

Una función que maneja errores debería probar al menos:

- una entrada válida;
- cada excepción que promete manejar;
- una excepción distinta que debe propagarse;
- valores límite como cero o una colección vacía;
- el valor retornado y cualquier salida producida.

```python
assert print_decimal(12) is True
assert print_decimal("twelve") is False
```

Para comprobar que una excepción se propaga sin una biblioteca adicional puede utilizarse una pequeña estructura de control:

```python
try:
    calculate_discount(100, 150)
except ValueError:
    print("Expected ValueError received")  # "se recibió el ValueError esperado"
else:
    print("Expected ValueError was not raised")  # "no se lanzó el ValueError esperado"
```

## Referencias

- [Errores y excepciones — Tutorial de Python](https://docs.python.org/3/tutorial/errors.html)
- [Excepciones incorporadas](https://docs.python.org/3/library/exceptions.html)
- [Sentencia `try` — Referencia del lenguaje](https://docs.python.org/3/reference/compound_stmts.html#the-try-statement)
- [Sintaxis de cadenas de formato](https://docs.python.org/3/library/string.html#format-string-syntax)
- [Errores y excepciones — documentación de Python 3.8](https://docs.python.org/3.8/tutorial/errors.html)
- [Excepciones incorporadas — documentación de Python 3.8](https://docs.python.org/3.8/library/exceptions.html)
- [Sentencia `try` — documentación de Python 3.8](https://docs.python.org/3.8/reference/compound_stmts.html#the-try-statement)
