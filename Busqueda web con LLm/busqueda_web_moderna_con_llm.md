# Búsqueda web moderna con modelos de lenguaje

Los modelos de lenguaje pueden ayudar a localizar, organizar y explicar información, pero no todas las herramientas trabajan de la misma manera. Algunas consultan la web en tiempo real, otras responden principalmente con conocimiento aprendido durante su entrenamiento y otras convierten información existente en representaciones visuales.

Usarlas bien requiere distinguir cuatro procesos:

1. **Búsqueda:** localizar páginas, documentos y datos relevantes.
2. **Recuperación:** seleccionar fragmentos de esas fuentes para aportar contexto.
3. **Síntesis:** redactar una respuesta a partir del contexto disponible.
4. **Verificación:** comprobar que cada afirmación importante está respaldada por una fuente adecuada.

Una respuesta fluida no garantiza que sea correcta. La calidad del resultado depende tanto de la herramienta como de la consulta, las fuentes recuperadas y la verificación posterior.

## Modelos de lenguaje y acceso a la web

Un modelo puede responder de dos maneras principales:

- **Sin búsqueda web:** utiliza patrones y conocimientos adquiridos durante su entrenamiento. Puede explicar conceptos generales, pero desconoce acontecimientos posteriores a su fecha límite de conocimiento y puede recordar datos de forma imprecisa.
- **Con búsqueda web:** consulta información externa y usa los resultados como contexto. Esto mejora la actualidad, aunque todavía puede seleccionar fuentes débiles, interpretar mal un texto o asociar una cita con una afirmación que la fuente no respalda.

La fecha límite de conocimiento y la fecha de la búsqueda son conceptos diferentes. Un modelo antiguo puede responder sobre un hecho reciente si dispone de búsqueda web, mientras que un modelo nuevo puede producir una respuesta desactualizada si la búsqueda está desactivada.

No conviene determinar la fecha límite de conocimiento preguntándosela únicamente al propio modelo. La respuesta puede ser aproximada o corresponder a otra versión. Es preferible consultar la documentación o la ficha oficial del modelo y comprobar su comportamiento con información fechada.

## Un flujo de investigación confiable

### 1. Definir la pregunta

Una consulta precisa reduce resultados irrelevantes. Conviene indicar:

- el tema y el alcance;
- el país o la región;
- el período de interés;
- el nivel de profundidad;
- el tipo de fuentes esperado;
- el formato de la respuesta.

Ejemplo:

```text
Explica los principales cambios recientes en [tema] para [región].
Limita la investigación al período comprendido entre [fecha] y [fecha].
Prioriza fuentes oficiales y documentos primarios.
Separa hechos confirmados, interpretaciones e información todavía incierta.
Incluye una fuente junto a cada afirmación importante.
```

### 2. Dividir el tema en subpreguntas

Las preguntas amplias producen resúmenes superficiales. Es más útil separar el tema en dimensiones como:

- antecedentes;
- situación actual;
- actores involucrados;
- datos cuantitativos;
- posiciones contrapuestas;
- consecuencias;
- incertidumbres y aspectos pendientes.

### 3. Buscar primero fuentes primarias

Cuando estén disponibles, se deben priorizar:

- leyes, reglamentos y sitios gubernamentales;
- artículos científicos y repositorios académicos;
- documentación técnica oficial;
- informes, comunicados y bases de datos de la organización responsable;
- declaraciones o registros originales.

Los medios periodísticos y los análisis de terceros son útiles para obtener contexto, pero no deberían sustituir al documento original cuando este existe.

### 4. Verificar antes de sintetizar

Para cada afirmación relevante hay que comprobar:

- que el enlace funciona;
- que la fuente identifica autor u organización;
- que la fecha de publicación es adecuada;
- que el texto realmente respalda la afirmación;
- que el dato no está citado fuera de contexto;
- que una fuente independiente confirma la información cuando el asunto es controvertido o importante.

### 5. Registrar el proceso

Un registro breve hace que la investigación sea reproducible:

| Campo | Qué registrar |
|---|---|
| Fecha y hora | Momento de la consulta |
| Herramienta | Servicio utilizado |
| Modelo | Nombre y versión visibles |
| Modo | Búsqueda web, razonamiento profundo u otra opción |
| Consulta | Texto exacto enviado |
| Fuentes | Enlaces principales recuperados |
| Hallazgos | Información confirmada |
| Incertidumbres | Datos contradictorios o no verificados |

## Cómo evaluar una respuesta con citas

Una cita no convierte automáticamente una respuesta en verdadera. La evaluación debe considerar al menos tres niveles:

1. **Existencia:** la fuente existe y es accesible.
2. **Correspondencia:** la fuente respalda la afirmación concreta que acompaña.
3. **Calidad:** la fuente es suficientemente confiable y actual para esa afirmación.

Una respuesta puede tener muchas citas y aun así ser débil si todas conducen a resúmenes secundarios, páginas antiguas o textos que solo mencionan el tema de manera superficial.

### Matriz de afirmaciones y fuentes

Para investigaciones importantes resulta útil construir una tabla de control:

| Afirmación | Fuente principal | Fuente de contraste | Estado |
|---|---|---|---|
| Dato o conclusión verificable | Enlace directo | Enlace independiente | Confirmada, dudosa o pendiente |

Esta matriz ayuda a detectar afirmaciones sin respaldo, fuentes repetidas y desacuerdos que requieren más investigación.

## Perplexity

[Perplexity](https://www.perplexity.ai/) combina búsqueda web con respuestas conversacionales. Suele mostrar citas numeradas y enlaces a las páginas utilizadas, lo que facilita pasar del resumen a la fuente original.

### Uso recomendado

- Comenzar con una pregunta concreta y revisar todas las citas importantes.
- Abrir las fuentes originales en lugar de confiar solo en el fragmento mostrado.
- Usar preguntas de seguimiento para profundizar, corregir el alcance o buscar posiciones opuestas.
- Indicar explícitamente el período de interés cuando la actualidad sea importante.
- Guardar investigaciones relacionadas en **Projects** —nombre que puede variar según el idioma o la versión de la interfaz— para conservar conversaciones, archivos e instrucciones comunes.

Los modos avanzados de búsqueda pueden realizar más consultas, analizar más fuentes o permitir la elección del modelo. Su disponibilidad y sus límites dependen del plan vigente. Para una comparación justa debe registrarse el modo utilizado.

### Preguntas de seguimiento útiles

```text
¿Qué afirmaciones de la respuesta dependen de una sola fuente?
```

```text
Busca una fuente primaria para cada dato cuantitativo.
```

```text
Presenta evidencia que contradiga o limite la conclusión anterior.
```

```text
¿Qué información cambió durante los últimos 60 días?
```

## DeepSeek

[DeepSeek Chat](https://chat.deepseek.com/) permite mantener conversaciones, analizar problemas y, cuando la interfaz lo ofrece, activar funciones como búsqueda web y razonamiento profundo.

Estas funciones cumplen propósitos distintos:

- **Web Search** aporta información recuperada de internet.
- **DeepThink** dedica más procesamiento al razonamiento de la respuesta.

Un razonamiento más extenso no garantiza datos más recientes ni fuentes más confiables. Para cuestiones actuales se debe comprobar que la búsqueda esté habilitada y revisar las fuentes. Para planificación, lógica o descomposición de problemas, el modo de razonamiento puede ser útil incluso sin acceso a la web.

Cuando se comparan respuestas, conviene anotar por separado si estaban activadas la búsqueda y el razonamiento. Cambiar ambas opciones al mismo tiempo impide saber cuál produjo la diferencia.

## OpenRouter

[OpenRouter](https://openrouter.ai/) ofrece una interfaz común para utilizar modelos de distintos proveedores mediante chat o API. Resulta útil para observar cómo cambia una respuesta al mantener constante la consulta y modificar el modelo.

La plataforma dispone de modelos gratuitos y de pago, sujetos a límites y disponibilidad. También ofrece acceso web para modelos compatibles mediante variantes en línea, complementos o herramientas de búsqueda. La forma exacta de activarlo puede cambiar, por lo que conviene revisar la documentación vigente y confirmar que el acceso web esté activo antes de evaluar actualidad o citas.

### Comparaciones controladas

Para comparar modelos de manera justa se debe mantener constante:

- la consulta completa;
- el idioma;
- el contexto adjunto;
- el acceso a la web;
- los parámetros visibles;
- el formato solicitado.

También hay que registrar el identificador exacto del modelo. Los enrutadores automáticos pueden seleccionar modelos diferentes entre solicitudes y son útiles para experimentar, pero no para una comparación estrictamente reproducible.

### Criterios de comparación

| Criterio | Pregunta de control |
|---|---|
| Exactitud | ¿Los hechos coinciden con las fuentes? |
| Actualidad | ¿Incluye información dentro del período solicitado? |
| Cobertura | ¿Responde todas las partes de la consulta? |
| Trazabilidad | ¿Es posible localizar el origen de cada afirmación importante? |
| Incertidumbre | ¿Distingue hechos, inferencias y dudas? |
| Claridad | ¿La estructura permite comprender y revisar la respuesta? |
| Reproducibilidad | ¿Otra persona podría repetir la consulta con la misma configuración? |
| Eficiencia | ¿El tiempo, los límites y el costo son adecuados para el uso previsto? |

## ConceptMap.AI

[ConceptMap.AI](https://www.conceptmap.ai/) transforma documentos o texto en una estructura visual jerárquica. Puede ser útil para explorar un tema, detectar categorías y presentar relaciones generales.

La herramienta describe sus resultados principalmente como **mind maps** o mapas mentales. Un mapa mental organiza ideas alrededor de un tema central; un mapa conceptual formal, en cambio, suele expresar relaciones mediante proposiciones y palabras de enlace. Esta diferencia importa cuando se necesita representar causalidad, dependencia o una relación lógica precisa.

### Buenas prácticas

- Proporcionar notas ya verificadas en lugar de usar el mapa como fuente de información.
- Revisar los nombres de los nodos y eliminar categorías ambiguas.
- Reorganizar ramas que mezclen niveles de abstracción diferentes.
- Añadir relaciones cruzadas cuando la estructura jerárquica sea insuficiente.
- Conservar las fuentes fuera del mapa si la herramienta no mantiene los enlaces.
- Revisar cualquier enlace público antes de compartirlo.

Las condiciones de acceso, el número de mapas gratuitos y las opciones de exportación pueden cambiar. La página oficial debe consultarse antes de depender de una función específica.

## Napkin AI

[Napkin AI](https://www.napkin.ai/es/) convierte texto en diagramas, mapas mentales, gráficos y otras piezas visuales editables. Es especialmente útil para comunicar información que ya fue investigada y organizada.

Un flujo efectivo consiste en:

1. redactar una síntesis breve y estructurada;
2. pegar o importar el texto;
3. generar varias alternativas visuales;
4. elegir la que represente mejor la relación entre las ideas;
5. corregir etiquetas, jerarquía y énfasis;
6. exportar en el formato adecuado.

Las opciones de exportación pueden incluir PNG, PDF, SVG o presentaciones, según el plan y la configuración. Las visualizaciones deben revisarse igual que el texto: una flecha puede sugerir causalidad, una posición puede insinuar importancia y un tamaño puede aparentar una proporción que los datos no demuestran.

## Elegir la representación visual correcta

| Representación | Uso principal | Riesgo frecuente |
|---|---|---|
| Mapa mental | Explorar y agrupar ideas | Confundir proximidad con una relación demostrada |
| Mapa conceptual | Expresar relaciones mediante conceptos y enlaces | Usar conectores vagos o no verificables |
| Diagrama de flujo | Mostrar decisiones o una secuencia | Omitir excepciones y ciclos |
| Línea de tiempo | Ordenar acontecimientos | Confundir fecha de publicación con fecha del hecho |
| Gráfico de datos | Comparar magnitudes o tendencias | Usar escalas o unidades engañosas |
| Infografía | Comunicar una síntesis | Simplificar demasiado o perder las fuentes |

La visualización es una capa de comunicación, no una prueba. Los datos y las fuentes deben poder consultarse por separado.

## Evaluar el impacto de la fecha límite de conocimiento

Para observar la diferencia entre memoria del modelo y búsqueda web se puede aplicar el siguiente método:

1. Elegir un acontecimiento reciente y bien documentado.
2. Definir de antemano los hechos que se verificarán mediante fuentes oficiales.
3. Formular exactamente la misma consulta sin acceso web.
4. Repetirla con acceso web habilitado.
5. Registrar modelo, fecha, modo y fuentes.
6. Clasificar cada afirmación como correcta, incompleta, desactualizada o no comprobable.
7. Comparar no solo la respuesta, sino también la calidad y la fecha de las fuentes.

Una buena prueba utiliza información suficientemente reciente para revelar diferencias, pero con fuentes primarias disponibles. Los acontecimientos en desarrollo requieren cautela adicional porque incluso las fuentes confiables pueden corregirse con el tiempo.

## Técnicas de búsqueda complementarias

Los buscadores tradicionales siguen siendo útiles para descubrir y verificar fuentes. Algunos operadores habituales son:

| Operador | Ejemplo | Finalidad |
|---|---|---|
| Comillas | `"exact phrase"` | Buscar una frase exacta |
| Sitio | `site:who.int topic` | Limitar resultados a un dominio |
| Tipo de archivo | `filetype:pdf report` | Buscar documentos en un formato específico |
| Exclusión | `topic -unrelated` | Excluir un término |
| Alternativas | `term1 OR term2` | Buscar cualquiera de dos expresiones |

No todos los motores interpretan los operadores de la misma manera. Si una búsqueda asistida por IA no encuentra una fuente concreta, una búsqueda manual por título, autor o dominio puede localizarla.

## Plantillas de consulta

### Investigación actual

```text
Investiga [tema] en [región] durante el período [fechas].
Prioriza fuentes oficiales, académicas y documentos primarios.
Para cada conclusión incluye la fuente y su fecha de publicación.
Distingue entre hechos confirmados, estimaciones e interpretaciones.
Señala desacuerdos entre fuentes y evita completar vacíos con suposiciones.
```

### Contraste de una respuesta

```text
Audita la respuesta anterior.
Enumera sus afirmaciones verificables y vincula cada una con la fuente que realmente la respalda.
Marca las afirmaciones sin respaldo, las fuentes desactualizadas y cualquier salto lógico.
```

### Síntesis para una visualización

```text
Convierte estas notas verificadas en una estructura jerárquica breve.
No agregues datos nuevos.
Separa causas, consecuencias, actores y evidencia.
Conserva una lista final de fuentes para revisión.
```

### Comparación de modelos

```text
Analiza [pregunta] y responde con esta estructura:
1. conclusión breve;
2. evidencia;
3. supuestos;
4. incertidumbres;
5. fuentes primarias.
No inventes enlaces ni referencias. Indica expresamente cuando no puedas verificar un dato.
```

## Errores frecuentes

- Confiar en la seguridad del tono en lugar de comprobar la evidencia.
- Contar citas sin revisar si respaldan las afirmaciones asociadas.
- Usar una única fuente para un asunto controvertido.
- Confundir la fecha de una página con la fecha del acontecimiento descrito.
- Mezclar resultados obtenidos con y sin búsqueda web.
- Comparar modelos con consultas o configuraciones diferentes.
- Tratar el razonamiento generado como si fuera una fuente.
- Utilizar una visualización automática sin revisar las relaciones que comunica.
- Copiar un resumen sin conservar los enlaces originales.
- Suponer que los límites, modelos o funciones gratuitas permanecen sin cambios.

## Privacidad y seguridad

Antes de enviar texto, archivos o enlaces a un servicio externo se debe considerar si contienen:

- datos personales;
- credenciales, claves o tokens;
- información académica identificable;
- documentos internos o confidenciales;
- propiedad intelectual de terceros;
- datos sujetos a acuerdos o regulaciones.

La opción más segura es eliminar o anonimizar la información sensible y trabajar solo con el fragmento mínimo necesario. También conviene revisar las políticas de privacidad, retención y uso de datos, así como la visibilidad de conversaciones, archivos y enlaces compartidos.

## Lista de control de calidad

Antes de utilizar o publicar una investigación asistida por IA, comprobar que:

- la pregunta y el período están claramente definidos;
- se registraron la herramienta, el modelo y el modo utilizados;
- las afirmaciones importantes tienen fuentes directas;
- cada cita fue abierta y leída;
- se priorizaron fuentes primarias cuando estaban disponibles;
- se contrastaron los datos importantes;
- las fechas corresponden al hecho investigado;
- las incertidumbres están señaladas;
- la visualización no introduce relaciones inexistentes;
- no se compartió información sensible;
- los enlaces siguen funcionando.

## Recursos oficiales

- [Perplexity](https://www.perplexity.ai/)
- [Centro de ayuda de Perplexity](https://www.perplexity.ai/help-center/)
- [ConceptMap.AI](https://www.conceptmap.ai/)
- [Napkin AI](https://www.napkin.ai/es/)
- [DeepSeek Chat](https://chat.deepseek.com/)
- [Documentación de DeepSeek](https://api-docs.deepseek.com/)
- [OpenRouter](https://openrouter.ai/)
- [Documentación de OpenRouter](https://openrouter.ai/docs/)
- [Búsqueda web en OpenRouter](https://openrouter.ai/docs/features/web-search)

Las interfaces, los modelos, los límites de uso y los nombres de las funciones cambian con frecuencia. La documentación oficial vigente debe ser la referencia para los detalles operativos.
