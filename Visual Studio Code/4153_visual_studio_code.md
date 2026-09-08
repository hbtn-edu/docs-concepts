# Visual Studio Code en un entorno remoto

## Introducción

Visual Studio Code (VS Code) es un editor de código que combina una interfaz ligera con herramientas para escribir, explorar y ejecutar proyectos. Incluye funciones como resaltado de sintaxis, búsqueda, terminal integrada, atajos de teclado y soporte para extensiones.

En esta modalidad, VS Code se abre en el navegador, pero trabaja conectado a un servidor remoto con Ubuntu 22.04. Los archivos, comandos y paquetes pertenecen a ese servidor temporal, no a la computadora local.

> **Importante:** este entorno no es igual al sitio público `vscode.dev`. La versión pública que funciona completamente dentro del navegador no incluye una terminal. Un sandbox con un servidor Linux remoto como entorno de ejecución sí permite utilizarla.

## El entorno temporal (sandbox)

El sandbox proporciona un entorno aislado para experimentar sin configurar VS Code directamente en la computadora personal.

- La interfaz de VS Code se utiliza desde el navegador.
- Los archivos se almacenan temporalmente en el servidor remoto.
- El entorno tiene una duración limitada.
- Cuando el sandbox caduca o se cierra, sus archivos, extensiones y configuraciones se eliminan.
- La opción **Add More Time** permite extender la sesión antes de que finalice.
- Al crear un sandbox nuevo, es necesario repetir la configuración y las instalaciones.

Por este motivo, cualquier archivo que deba conservarse debe descargarse o guardarse en un repositorio antes de que termine la sesión.

## Componentes principales de la interfaz

La interfaz de VS Code se divide en varias áreas:

| Área | Función |
| --- | --- |
| **Activity Bar** | Permite cambiar entre vistas como Explorer, Search, Source Control y Extensions. |
| **Side Bar** | Muestra la vista seleccionada en la Activity Bar. |
| **Explorer** | Permite abrir, crear, renombrar, mover y eliminar archivos y carpetas. |
| **Editor** | Es el área central donde se visualizan y modifican los archivos. |
| **Panel** | Contiene herramientas como la terminal integrada, problemas y resultados. |
| **Status Bar** | Muestra información sobre el archivo y el espacio de trabajo actuales. |

### Pantalla de bienvenida y tema de color

Al abrir VS Code por primera vez puede aparecer la pantalla **Get Started with VS Code for the Web**. Desde allí se puede elegir un tema como **Dark**, **Light** o **High Contrast**.

El tema puede modificarse nuevamente mediante la paleta de comandos:

1. Abre la paleta con `Ctrl + Shift + P` o `F1`.
2. Busca `Preferences: Color Theme`.
3. Selecciona el tema deseado.

La apariencia cambia, pero no afecta los archivos ni el funcionamiento del entorno.

## Carpetas y espacios de trabajo

VS Code organiza el trabajo alrededor de archivos y carpetas. Cuando se abre una carpeta, su contenido aparece en **Explorer** y esa carpeta pasa a ser la raíz del espacio de trabajo.

Para abrir una carpeta como espacio de trabajo:

1. Abre **Explorer** desde la Activity Bar.
2. Selecciona **Open Folder**.
3. Escribe o selecciona la ruta de la carpeta deseada.
4. Confirma la selección.

Después de abrirla, el Explorer permite recorrer su contenido y administrar sus archivos y subdirectorios.

También se puede abrir una carpeta con **File → Open Folder**. El atajo habitual en Windows y Linux es `Ctrl + K Ctrl + O`: primero se presiona `Ctrl + K` y después `Ctrl + O`.

> **Nota:** en muchos sandboxes basados en Linux, el directorio de inicio de sesión es `/root` (el directorio personal del usuario administrador del sistema). Abrirlo no significa que todos los archivos mostrados sean parte de tu propio trabajo: puede contener archivos de configuración u otros directorios propios del entorno.

## Terminal integrada

La terminal integrada es una shell completa dentro de VS Code. Permite ejecutar los mismos comandos que una terminal independiente sin abandonar el editor.

Se puede abrir de varias formas:

- Desde **Terminal → New Terminal**.
- Desde **View → Terminal**.
- Con `Ctrl` + tecla de acento grave (`` ` ``) para mostrar u ocultar el panel de terminal.
- Con `Ctrl + Shift` + tecla de acento grave (`` ` ``) para crear una terminal nueva.

Normalmente, una terminal nueva comienza en la carpeta abierta como espacio de trabajo. Se puede comprobar la ubicación con:

```bash
pwd
```

Para listar los archivos del directorio actual:

```bash
ls
```

Algunos usos comunes de la terminal durante el desarrollo son:

- Navegar por las carpetas del espacio de trabajo.
- Crear, copiar, mover o eliminar archivos.
- Instalar herramientas y dependencias.
- Ejecutar programas, scripts y pruebas.
- Usar Git para gestionar versiones del código.

## Instalar paquetes de Linux

El sandbox utiliza Ubuntu y permite instalar paquetes desde la terminal. Para actualizar la información disponible sobre los paquetes:

```bash
apt-get update
```

Este comando actualiza el índice local de paquetes; no actualiza automáticamente todos los programas instalados.

Para instalar `tree` sin solicitar una confirmación interactiva:

```bash
apt-get install -y tree
```

La opción `-y` responde afirmativamente a la confirmación de instalación. Una vez finalizado el proceso, se puede ejecutar:

```bash
tree
```

`tree` representa gráficamente la jerarquía de archivos y subdirectorios a partir de la ubicación actual.

> **Importante:** la instalación afecta solamente al servidor temporal. Si se crea un sandbox nuevo, será necesario instalar el paquete otra vez.

## Extensiones

Las extensiones agregan lenguajes, herramientas y comportamientos a VS Code. Se administran desde la vista **Extensions**, identificada mediante el icono de bloques en la Activity Bar. También puede abrirse con `Ctrl + Shift + X`.

Para instalar una extensión:

1. Abre la vista **Extensions**.
2. Escribe su nombre en la barra de búsqueda.
3. Selecciona el resultado correcto.
4. Revisa el nombre, el publicador y el identificador de la extensión.
5. Haz clic en **Install**.

### Identificar correctamente una extensión

Pueden existir varias extensiones con nombres similares. Cada una posee un identificador único con el formato:

```text
publisher.extension
```

Un ejemplo de identificador de extensión oficial es la extensión de Python publicada por Microsoft:

```text
ms-python.python
```

Antes de instalarla, se debe comprobar que corresponda a la extensión de Python publicada por Microsoft. El número de instalaciones y la valoración pueden aportar contexto, pero el publicador y el identificador son datos más precisos para distinguirla.

> **Seguridad:** una extensión puede tener los mismos permisos que VS Code. Instala únicamente extensiones necesarias y verifica su publicador antes de confiar en ellas.

### Extensión e intérprete no son lo mismo

La extensión de Python agrega a VS Code funciones de edición, navegación, ejecución y depuración relacionadas con Python. No instala por sí sola el intérprete que ejecuta los programas. Son componentes diferentes:

- **VS Code:** proporciona el editor.
- **Extensión de Python:** integra herramientas de Python con el editor.
- **Intérprete de Python:** ejecuta el código.

## Paleta de comandos

La **Command Palette** permite buscar y ejecutar acciones sin recorrer los menús. Se abre con:

```text
Ctrl + Shift + P
```

También puede abrirse con `F1`, una alternativa especialmente útil cuando el navegador intercepta el atajo principal.

Algunos comandos útiles disponibles en la paleta son:

- `Preferences: Color Theme`
- `View: Toggle Terminal`
- `Markdown: Open Preview`
- `Markdown: Open Preview to the Side`
- `Preferences: Open Keyboard Shortcuts`

## Atajos esenciales

Los siguientes atajos corresponden a Windows y Linux:

| Acción | Atajo | Observación en el navegador |
| --- | --- | --- |
| Crear un archivo nuevo | `Ctrl + N` | El navegador puede abrir una ventana nueva. Si ocurre, utiliza **File → New File**, la paleta de comandos o `Ctrl + Alt + N`. |
| Buscar dentro del archivo actual | `Ctrl + F` | Si la terminal tiene el foco, busca dentro de la salida de la terminal. |
| Mostrar u ocultar la barra lateral | `Ctrl + B` | Alterna la visibilidad de la Side Bar. |
| Abrir la paleta de comandos | `Ctrl + Shift + P` | Si el navegador lo intercepta, utiliza `F1`. |
| Abrir la vista previa de Markdown | `Ctrl + Shift + V` | Abre o alterna la vista previa del archivo `.md` activo. |
| Abrir Markdown al costado | `Ctrl + K V` | Abre una vista dividida con el editor y la vista previa. |
| Abrir extensiones | `Ctrl + Shift + X` | Muestra la vista Extensions. |
| Mostrar u ocultar la terminal | `Ctrl` + tecla de acento grave (`` ` ``) | Alterna el panel de terminal. |

VS Code permite consultar y modificar los atajos desde **File → Preferences → Keyboard Shortcuts** o con `Ctrl + K Ctrl + S`. Los atajos disponibles pueden variar según el sistema operativo, la distribución del teclado, las extensiones instaladas y las combinaciones reservadas por el navegador.

## Trabajar con Markdown

VS Code admite Markdown de forma integrada. Para probar la vista previa:

1. Crea un archivo con extensión `.md`, por ejemplo, `example.md`.
2. Escribe contenido Markdown:

```markdown
# Mi primer documento

Este texto contiene una palabra en **negrita**.

- Primer elemento
- Segundo elemento
```

3. Guarda el archivo.
4. Usa `Ctrl + Shift + V` para abrir la vista previa o `Ctrl + K V` para verla al costado.

La vista previa se actualiza a medida que se modifica el documento y permite comprobar cómo se renderizará el contenido.

## Recomendaciones prácticas

- Confirma cuál es la carpeta abierta como espacio de trabajo antes de ejecutar comandos que dependan de la ubicación actual.
- Observa el prompt y utiliza `pwd` cuando no sepas en qué directorio se encuentra la terminal.
- Lee la salida completa de `apt-get` para comprobar que la instalación haya finalizado correctamente.
- Verifica el publicador y el identificador antes de instalar una extensión.
- Usa `F1` si un atajo es interceptado por el navegador.
- Conserva fuera del sandbox cualquier trabajo importante antes de que expire la sesión.

## Fuentes

- [Documentación oficial de Visual Studio Code](https://code.visualstudio.com/docs)
- [Interfaz de usuario](https://code.visualstudio.com/docs/getstarted/userinterface)
- [Terminal integrada](https://code.visualstudio.com/docs/terminal/basics)
- [Marketplace de extensiones](https://code.visualstudio.com/docs/configure/extensions/extension-marketplace)
- [Atajos de teclado](https://code.visualstudio.com/docs/configure/keybindings)
- [Markdown en Visual Studio Code](https://code.visualstudio.com/docs/languages/markdown)
- [Visual Studio Code para la Web](https://code.visualstudio.com/docs/remote/vscode-web)
- [Inicio rápido de Python en VS Code](https://code.visualstudio.com/docs/python/python-quick-start)
