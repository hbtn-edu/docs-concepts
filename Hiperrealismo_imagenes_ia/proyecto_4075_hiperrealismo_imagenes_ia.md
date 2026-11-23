# Hiperrealismo en la generación de imágenes con IA

## Introducción

La generación de imágenes con inteligencia artificial permite crear retratos, objetos y escenas completas a partir de descripciones escritas. El **hiperrealismo** busca que el resultado presente características asociadas con una fotografía: iluminación creíble, materiales coherentes, profundidad, anatomía correcta y detalles naturales.

Una imagen visualmente convincente no es una fotografía ni demuestra que el hecho representado haya ocurrido. Siempre debe conservarse la distinción entre contenido real y contenido sintético.

## La familia FLUX

FLUX es una familia de modelos de generación y edición de imágenes desarrollada por Black Forest Labs. Sus variantes responden a necesidades diferentes.

| Variante | Uso principal | Característica destacada |
| --- | --- | --- |
| FLUX.1 Schnell | Probar ideas rápidamente | Genera con pocos pasos de inferencia |
| FLUX.1 Dev | Crear imágenes más elaboradas | Buen seguimiento del prompt y composiciones detalladas |
| FLUX con LoRA | Aplicar un estilo o especialización | Adapta el modelo base mediante pesos adicionales |
| FLUX.1 Kontext Dev | Editar una imagen existente | Conserva referencias mientras aplica cambios escritos |

### FLUX.1 Schnell

**Schnell** significa “rápido” en alemán. Está optimizado para obtener resultados con aproximadamente uno a cuatro pasos de inferencia, por lo que resulta útil para explorar composiciones, colores y encuadres.

Más pasos no siempre producen una mejora. En la implementación oficial, Schnell utiliza `guidance_scale=0` y pocos pasos debido a su proceso de destilación.

### FLUX.1 Dev

FLUX.1 Dev prioriza la calidad y el seguimiento de instrucciones. Resulta apropiado para escenas con varios elementos, iluminación elaborada y composiciones que requieren mayor control.

Normalmente necesita más tiempo y recursos que Schnell. Sus pesos se distribuyen bajo la licencia propia de FLUX.1 Dev, mientras que Schnell utiliza Apache 2.0. Antes de utilizar un modelo fuera de una práctica personal, se deben revisar sus condiciones vigentes.

### LoRA

Una **LoRA** es una adaptación pequeña que modifica el comportamiento de un modelo base sin reemplazarlo por completo. Puede especializar la generación en un estilo, personaje, producto o apariencia visual.

Una LoRA de realismo puede favorecer texturas de piel, iluminación o rasgos fotográficos, pero no garantiza por sí sola un resultado realista. El modelo base, el prompt, los parámetros y la imagen seleccionada siguen siendo importantes.

Los Spaces comunitarios pueden cambiar sus LoRA disponibles, sus nombres y sus controles sin previo aviso.

### FLUX.1 Kontext Dev

Kontext permite editar una imagen mediante instrucciones escritas. Puede modificar iluminación, fondo, ropa, color o estilo intentando conservar la identidad y la composición.

Funciona mejor cuando la instrucción diferencia claramente:

- qué debe cambiar;
- qué debe conservarse;
- cómo debe verse el resultado final.

## Hugging Face Spaces

[Hugging Face Spaces](https://huggingface.co/spaces) permite ejecutar aplicaciones de aprendizaje automático desde el navegador. Algunos Spaces utilizan recursos compartidos, por lo que pueden aparecer colas, tiempos de espera o límites de uso.

La interfaz y los controles disponibles pueden variar entre Spaces. No todos muestran los mismos parámetros ni utilizan exactamente la misma configuración interna.

### Qué hacer si una herramienta no está disponible

Un Space puede estar ocupado, alcanzar su límite de uso diario o quedar temporalmente fuera de servicio. Conviene tener identificada al menos una alternativa capaz de ejecutar un modelo de la familia FLUX, aunque sea con una interfaz distinta o con menos funciones disponibles — por ejemplo, Pollinations.ai o Mage.space, mencionadas en la consigna del proyecto, u otro servicio equivalente.

Las condiciones de uso gratuito de este tipo de servicios (necesidad de cuenta, créditos iniciales, límites diarios) cambian con frecuencia, y lo que hoy es gratuito o "sin límite" puede dejar de serlo. Antes de depender de una alternativa para completar una entrega, conviene confirmar en el sitio del servicio cuáles son sus condiciones vigentes en ese momento, en lugar de asumir que se mantienen iguales.

Además, un servicio gratuito o con créditos limitados no siempre genera la imagen con el modelo solicitado: puede sustituirlo por otro modelo más económico sin avisar. Conviene revisar el resultado (o su metadata, cuando esté disponible) para confirmar qué modelo se usó realmente, en lugar de asumir que fue FLUX solo porque la herramienta lo ofrece entre sus opciones.

## Cómo construir un prompt fotorrealista

Un prompt efectivo describe la escena con elementos concretos y compatibles entre sí.

### Estructura recomendada

```text
[Sujeto principal], [acción o expresión], [entorno],
[tipo y dirección de la iluminación], [encuadre],
[cámara o lente], [profundidad de campo],
[paleta de colores], [texturas y detalles visibles].
```

Ejemplo:

```text
A realistic editorial portrait of a young architect standing beside a concrete wall,
calm expression, soft window light from the left, chest-up composition,
85mm lens, shallow depth of field, neutral color palette,
natural skin texture, subtle fabric details, softly blurred background.
```

### Elementos útiles

#### Sujeto

Describe edad aproximada, vestimenta, postura, expresión y actividad sin acumular detalles contradictorios.

#### Entorno

Indica dónde ocurre la escena y qué elementos deben aparecer alrededor del sujeto.

#### Iluminación

La luz influye directamente en la sensación de realismo. Algunas opciones frecuentes son:

- soft natural light;
- window light;
- golden hour lighting;
- overcast daylight;
- studio lighting;
- soft shadows;
- rim light.

#### Composición

Puede especificarse el plano y la ubicación del sujeto:

- close-up;
- chest-up portrait;
- full-body shot;
- wide shot;
- centered composition;
- rule of thirds;
- eye-level view;
- low-angle view.

#### Lenguaje fotográfico

Las referencias fotográficas ayudan a comunicar una intención visual:

- `35mm lens`: campo más amplio y sensación ambiental;
- `50mm lens`: perspectiva natural y versátil;
- `85mm lens`: retratos con fondo comprimido;
- `shallow depth of field`: sujeto enfocado y fondo desenfocado;
- `sharp focus`: detalles claramente definidos;
- `subtle film grain`: textura ligera de película.

Mencionar una cámara o una lente orienta la estética, pero no simula físicamente una cámara real con total precisión.

## Parámetros principales

### Resolución y relación de aspecto

El ancho y el alto determinan la forma de la imagen.

| Relación | Uso frecuente |
| --- | --- |
| `1:1` | Retratos, productos y publicaciones cuadradas |
| `4:5` | Retratos verticales y redes sociales |
| `3:2` | Apariencia fotográfica tradicional |
| `16:9` | Paisajes y escenas panorámicas |
| `9:16` | Contenido vertical para dispositivos móviles |

La composición debe pensarse para la relación elegida. Cambiar únicamente las dimensiones puede recortar elementos o alterar su posición.

### Pasos de inferencia

Los pasos indican cuántas iteraciones realiza el proceso de generación. El valor adecuado depende del modelo:

- Schnell está diseñado para muy pocos pasos;
- Dev suele beneficiarse de una cantidad mayor;
- aumentar indefinidamente los pasos consume más recursos sin garantizar una mejora visible.

### Guidance scale

Este valor controla cuánto intenta el modelo ajustarse a la descripción. Un valor demasiado alto puede producir colores, bordes o texturas artificiales. Un valor bajo puede dar más libertad al modelo.

La escala no debe compararse de forma directa entre todas las variantes. Schnell, por ejemplo, está diseñado para trabajar sin guidance convencional en su implementación oficial.

### Seed

La **seed** inicia la generación desde un estado reproducible. Mantenerla permite comparar cambios pequeños en el prompt o los parámetros. Cambiarla produce otra variación.

Algunos Spaces no muestran ni permiten fijar este valor.

## Flujo de texto a imagen

Un proceso eficiente puede dividirse en dos etapas:

1. Utilizar Schnell para probar el sujeto, la composición y la paleta.
2. Trasladar el prompt más prometedor a Dev o a una LoRA apropiada para refinar el resultado.

Conviene modificar una variable por vez. Si se cambian simultáneamente el modelo, la seed, la iluminación, el encuadre y los pasos, será difícil identificar qué produjo la diferencia.

## Edición de imagen con Kontext

Una instrucción de edición debe ser breve y concreta.

```text
Change the background to a modern library with warm afternoon light.
Keep the same person, facial features, pose, clothing, camera angle, and composition.
Maintain realistic skin texture and natural shadows.
```

### Buenas prácticas

- utilizar una imagen clara y con resolución suficiente;
- solicitar pocos cambios por generación;
- nombrar los elementos que deben permanecer intactos;
- comparar siempre la entrada con el resultado;
- realizar ediciones sucesivas pequeñas en lugar de una transformación ambigua;
- guardar cada versión para poder retroceder.

Una edición puede introducir deriva visual: cambios no solicitados en el rostro, la ropa, la postura o los objetos. Si ocurre, conviene volver a la imagen anterior y formular una instrucción más limitada.

## Cómo reconocer un resultado convincente

El realismo puede revisarse observando:

- anatomía de manos, ojos, dientes y articulaciones;
- dirección coherente de la luz y las sombras;
- perspectiva y escala de los objetos;
- reflejos compatibles con los materiales;
- textura natural de piel, cabello y tela;
- continuidad de líneas y bordes;
- texto legible, si la escena contiene palabras;
- fondo coherente y sin objetos fusionados.

Los errores suelen aparecer en elementos pequeños, repeticiones, accesorios, reflejos y partes parcialmente ocultas.

## Coherencia visual en una colección

Para que varias imágenes parezcan parte de la misma serie, se pueden mantener constantes:

- tema y audiencia;
- modelo o LoRA;
- relación de aspecto;
- paleta de colores;
- temperatura y dirección de la luz;
- lente y encuadre;
- nivel de contraste;
- vocabulario principal del prompt.

Una base reutilizable evita comenzar desde cero:

```text
Visual style: realistic editorial photography, soft natural light,
neutral warm palette, 50mm lens, subtle film grain, natural textures.

Scene: [descripción específica de cada imagen].
```

## Registro del proceso

Guardar los datos de cada generación ayuda a reproducir resultados y mejorar los prompts.

```markdown
## Nombre de la imagen

- Modelo o Space:
- LoRA seleccionada:
- Prompt:
- Imagen de referencia, si corresponde:
- Resolución y relación de aspecto:
- Pasos:
- Guidance scale:
- Seed:
- Resultado útil:
- Aspectos para mejorar:
- Cambios realizados en la siguiente versión:
```

También es útil conservar los archivos con nombres consistentes:

```text
coleccion-01-retrato-v01.png
coleccion-01-retrato-v02.png
coleccion-02-escena-v01.png
```

## Seguridad, privacidad y uso responsable

- No subir fotografías privadas sin autorización.
- No utilizar el rostro de otra persona para crear contenido engañoso o perjudicial.
- Revisar las condiciones de uso y visibilidad del Space antes de cargar imágenes.
- No asumir que las imágenes o prompts permanecen privados.
- Evitar logotipos, personajes o estilos protegidos cuando el uso pueda infringir derechos.
- Identificar el contenido como generado o editado con IA cuando pueda confundirse con un hecho real.
- Revisar sesgos y estereotipos presentes en personas, profesiones y contextos representados.
- Comprobar la licencia del modelo y de cada LoRA antes de un uso comercial.

## Recursos

- [FLUX.1 Schnell en Hugging Face Spaces](https://huggingface.co/spaces/black-forest-labs/FLUX.1-schnell)
- [FLUX.1 Dev en Hugging Face Spaces](https://huggingface.co/spaces/black-forest-labs/FLUX.1-dev)
- [FLUX LoRA DLC](https://huggingface.co/spaces/prithivMLmods/FLUX-LoRA-DLC)
- [FLUX.1 Kontext Dev en Hugging Face Spaces](https://huggingface.co/spaces/black-forest-labs/FLUX.1-Kontext-Dev)
- [Ficha de FLUX.1 Schnell](https://huggingface.co/black-forest-labs/FLUX.1-schnell)
- [Ficha de FLUX.1 Dev](https://huggingface.co/black-forest-labs/FLUX.1-dev)
- [Ficha de FLUX.1 Kontext Dev](https://huggingface.co/black-forest-labs/FLUX.1-Kontext-dev)
- [Documentación de FLUX en Diffusers](https://huggingface.co/docs/diffusers/main/api/pipelines/flux)
- [Uso de LoRA con Diffusers](https://huggingface.co/docs/diffusers/main/tutorials/using_peft_for_inference)
