# Gestión del conocimiento con notas inteligentes e IA local

## Por qué un sistema de notas necesita estructura

Guardar información en archivos sueltos, sin ningún criterio de organización ni conexión entre ellos, funciona mientras hay pocas notas. En cuanto la cantidad crece, se vuelve imposible recordar dónde quedó algo o si ya se había escrito sobre ese tema antes. La información termina duplicada, desactualizada o simplemente perdida.

Un sistema de gestión del conocimiento resuelve esto combinando tres piezas: una forma consistente de **capturar** información, un criterio simple para **organizarla**, y un mecanismo para **conectarla** y **recuperarla** cuando hace falta. Cuando además se integra un modelo de lenguaje (LLM) que puede leer y buscar dentro de esas notas, el sistema empieza a comportarse como un "segundo cerebro": un repositorio externo que complementa la memoria, en vez de depender solo de ella.

## Obsidian y el concepto de bóveda

Obsidian es una aplicación de notas que guarda cada nota como un archivo de texto plano en formato Markdown, dentro de una carpeta común llamada **bóveda**. No hay una base de datos propietaria detrás: la bóveda es, literalmente, una carpeta del sistema de archivos con archivos `.md` adentro, lo que permite editarlos con cualquier otro programa, respaldarlos o sincronizarlos sin depender de un formato cerrado.

Dentro de una bóveda, las notas se pueden agrupar en subcarpetas igual que cualquier otro archivo, y además conectarse entre sí mediante enlaces internos (se explica más abajo), algo que una carpeta común no ofrece.

## Notas atómicas

Una **nota atómica** contiene una sola idea o concepto, en vez de mezclar varios temas en un documento largo. El objetivo es que cada nota pueda reutilizarse y conectarse de forma independiente: si una nota mezcla diez ideas distintas, solo puede enlazarse "en bloque", y pierde precisión a la hora de conectarla con otras notas relacionadas.

Una nota atómica bien escrita suele incluir:

- Un encabezado (`#`, `##`, `###`) que identifica el tema.
- Contenido breve y centrado en esa única idea.
- Etiquetas (`#tag`) que la clasifican temáticamente.
- Metadatos en la parte superior del archivo, mediante *frontmatter* YAML.

## Metadatos con *frontmatter* YAML

El *frontmatter* es un bloque de metadatos ubicado al principio del archivo, delimitado por tres guiones (`---`), que Obsidian interpreta como datos estructurados en vez de texto de la nota:

```yaml
---
date: 2026-02-16
tags: [ai, notes]
status: draft
---
```

Estos campos no son texto libre: `date` espera una fecha, `tags` una lista de valores entre corchetes, y `status` un valor simple. Guardar esta información como *frontmatter* (en vez de escribirla como texto suelto dentro de la nota) permite filtrar, ordenar y consultar las notas por esos campos más adelante, con el propio buscador de Obsidian o con complementos que leen metadatos.

Una convención de nombre de archivo consistente complementa al *frontmatter*: por ejemplo, anteponer la fecha al título (`2026-02-16_Project_Kickoff`) ayuda a identificar de un vistazo cuándo se creó una nota, sin tener que abrirla. Las notas puramente estructurales —como las que se explican en las próximas secciones— suelen seguir en cambio una convención descriptiva (`Projects Index.md`) en vez de llevar fecha, porque su función no es registrar un evento puntual sino servir de punto de navegación permanente.

## El método PARA para organizar

PARA es un marco de cuatro categorías, ideado por Tiago Forte, para decidir en qué carpeta va cada nota según su **grado de actividad**, no según su tema:

| Categoría | Qué contiene | Ejemplo |
| --- | --- | --- |
| **P**royectos | Trabajo activo, con un objetivo y una fecha de cierre definidos. | "Crear un sitio web para mi portafolio" |
| **A**reas | Responsabilidades continuas, sin fecha de finalización. | "Salud", "Desarrollo profesional" |
| **R**ecursos | Temas de interés o material de referencia, sin una acción pendiente asociada. | "Herramientas de IA", "Patrones de diseño" |
| **A**rchivos | Elementos completados o inactivos, procedentes de las otras tres categorías. | Un proyecto ya cerrado |

La ventaja de organizar por actividad en vez de por tema es que evita tener que inventar una taxonomía nueva cada vez que aparece una nota que no encaja claramente en ninguna categoría existente: alcanza con preguntarse "¿esto es algo que estoy haciendo ahora, algo que mantengo, algo que consulto, o algo que ya terminé?". Ante la duda, es preferible colocar la nota en Recursos y moverla más adelante antes que perder tiempo decidiendo la ubicación perfecta.

## Enlaces internos y el grafo de conocimiento

Un enlace interno conecta dos notas mediante la sintaxis `[[nombre de la nota]]`. A diferencia de una carpeta (que solo permite que una nota esté en un único lugar), los enlaces permiten que una nota se relacione con tantas otras como haga falta, sin duplicar contenido.

Dos herramientas ayudan a aprovechar esos enlaces:

- **Vista de grafo** (*Graph View* en la interfaz de Obsidian): una visualización donde cada nota es un punto y cada enlace una línea que los conecta. Las notas muy enlazadas aparecen agrupadas en el centro; las notas aisladas, sin ningún enlace, quedan en los bordes — una forma rápida de detectar contenido que quedó desconectado del resto.
- **Backlinks**: un panel que, abierto desde cualquier nota, muestra todas las demás notas que enlazan hacia ella. Es la dirección inversa de un enlace `[[...]]`: permite descubrir que una nota fue mencionada desde otro lugar, incluso si no se recuerda haberlo hecho.

## Notas maestras o MOC

Un **MOC** (*Map of Content*, mapa de contenido) es una nota que no agrega información nueva, sino que actúa como punto de navegación para un tema que abarca varias notas relacionadas. Enlaza cada nota relevante, con un breve resumen de qué contiene cada una, organizado en secciones lógicas.

La diferencia clave con una nota común es su función: un MOC no debe intentar resumir todo el contenido de las notas que enlaza (para eso ya existen las notas mismas), sino ayudar a encontrar rápidamente cuál de ellas consultar. Cuantas más notas acumula un tema, más útil se vuelve tener un MOC que funcione como "portada" de ese tema.

## El método CODE para el trabajo diario

Mientras que PARA responde a "¿en qué carpeta va esto?", **CODE** responde a "¿qué hago con la información a medida que la recibo?". Es un flujo de cuatro pasos:

1. **Capturar**: guardar cualquier información que resulte interesante o útil en el momento, sin preocuparse todavía por el formato ni por dónde va a terminar guardada. El objetivo en esta etapa es no perder la idea, no organizarla.
2. **Organizar**: mover lo capturado a la carpeta PARA correspondiente, con etiquetas y enlaces hacia notas relacionadas.
3. **Destilar**: reducir una nota extensa a sus puntos esenciales, resaltando lo más importante. Esta etapa se apoya en la técnica de **resumen progresivo**: cada vez que se revisa una nota, se destaca (con negrita, por ejemplo) una capa más concentrada de lo esencial, de modo que una lectura rápida futura alcance con leer solo lo resaltado.
4. **Expresar**: convertir el conocimiento ya destilado en algo utilizable — una decisión, un informe, un esquema — en vez de dejarlo acumulado sin ningún destino.

## Modelos de lenguaje locales con Ollama

Ollama es una herramienta que permite descargar y ejecutar modelos de lenguaje (LLM) directamente en el propio equipo, sin enviar las consultas a un servicio externo. Esto tiene dos consecuencias prácticas: no hace falta conexión a internet para usarlo una vez descargado el modelo, y el contenido de las consultas —en este caso, el contenido de las notas— no sale de la máquina.

```bash
ollama pull mistral
ollama run mistral
```

`ollama pull` descarga un modelo (Mistral, en este ejemplo) a disco. `ollama run` lo carga en memoria y abre una sesión de chat con él; ejecutado con un texto como argumento (`ollama run mistral "pregunta"`), responde esa consulta puntual y termina.

Los modelos de lenguaje no son la única pieza que puede ejecutarse localmente con Ollama: también existen **modelos de *embeddings*** (por ejemplo, `nomic-embed-text`), que no generan texto sino que convierten un fragmento de texto en un vector numérico que representa su significado. Estos modelos son los que permiten la búsqueda semántica que se explica a continuación, y se descargan de la misma forma (`ollama pull nomic-embed-text`).

## Complementos de IA para Obsidian

Un complemento conecta Obsidian con Ollama para poder chatear con el contenido de la bóveda. El más completo, gratuito y con licencia MIT es **Local LLM Helper**: se conecta a Ollama a través de `http://localhost:11434` (la dirección donde Ollama escucha por defecto en la misma máquina) y ofrece tres funciones dentro de Obsidian:

- **Chat directo**: hacer una pregunta o pedir un resumen sobre el texto seleccionado.
- **Búsqueda semántica**: encontrar notas relacionadas por significado, no solo por coincidencia exacta de palabras.
- **RAG** (*Retrieval-Augmented Generation*, generación aumentada por recuperación): el modelo de chat responde apoyándose en el contenido real de la bóveda, en vez de responder solo con lo que "sabe" de su entrenamiento.

### Por qué RAG necesita un modelo de *embeddings*

El chat básico funciona únicamente con el modelo de chat (Mistral, por ejemplo). Pero para que el complemento pueda "buscar" las notas más relevantes antes de responder —que es lo que distingue a RAG de un chat común— necesita comparar el significado de la pregunta contra el significado de cada nota. Esa comparación es exactamente lo que hace el modelo de *embeddings*: por eso hace falta descargarlo aparte (`ollama pull nomic-embed-text`), seleccionarlo en la configuración del complemento, y ejecutar una vez un comando de indexado para que la bóveda quede procesada. Sin ese paso, el chat común funciona pero la búsqueda semántica y el modo RAG no tienen con qué comparar.

### Alternativas

- **Obsidian Copilot**: complemento con plan gratuito que también permite usar Ollama como proveedor de modelo, configurando la propia clave/modelo (*Bring Your Own Key*) en vez de depender de un plan pago.
- **Smart Connections**: ofrece un gráfico visual de notas relacionadas usando su propio modelo de *embeddings* integrado, sin necesidad de configurar nada. Su función de chat con Ollama, en cambio, forma parte de un complemento separado de pago — por eso suele usarse en combinación con otro complemento (como Local LLM Helper) para el chat, y reservarse solo para el gráfico visual.

## Mantenimiento del sistema

Un sistema de notas se degrada si nadie lo revisa: aparecen notas sin ningún enlace (huérfanas), información desactualizada, o duplicados. Una revisión breve pero periódica —recorrer una carpeta PARA por vez, buscar notas huérfanas, actualizar los MOC a medida que se agregan notas nuevas— sostiene el sistema con mucho menos esfuerzo que una limpieza extensa ocasional. El propio chat con IA en modo RAG puede ayudar en esta revisión, por ejemplo respondiendo qué notas de un tema podrían estar desactualizadas o cuáles todavía no fueron conectadas al resto.

## Referencias

- [Obsidian](https://obsidian.md) — aplicación de notas.
- [Ollama](https://ollama.com) — ejecución de modelos de lenguaje locales.
- [Complemento Local LLM Helper](https://github.com/manimohans/obsidian-local-llm-helper) — chat, RAG y búsqueda semántica con Ollama dentro de Obsidian.
- [Complemento Obsidian Copilot](https://github.com/logancyang/obsidian-copilot) — alternativa de chat con soporte para modelos locales.
- [Complemento Smart Connections](https://github.com/brianpetro/obsidian-smart-connections) — gráfico visual de notas relacionadas mediante *embeddings*.
- [Método PARA, de Tiago Forte](https://fortelabs.com/blog/para/)
- [Resumen progresivo](https://fortelabs.com/blog/progressive-summarization-a-practical-technique-for-designing-discoverable-notes/)
- [Building a Second Brain: guía introductoria](https://fortelabs.com/blog/basboverview/) — presenta el método CODE y el concepto de segundo cerebro.
