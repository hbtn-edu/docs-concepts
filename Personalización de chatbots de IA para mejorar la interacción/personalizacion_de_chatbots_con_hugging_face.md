# Personalización de chatbots de IA con Hugging Face

## Introducción

Personalizar un chatbot consiste en darle instrucciones claras sobre su función, tono, límites y forma de responder. No es necesario entrenar un modelo ni escribir código: puede comenzarse con un mensaje inicial bien diseñado y comprobar su comportamiento mediante varias conversaciones.

En HuggingChat, esta personalización puede utilizarse para crear asistentes informales destinados a organizar información, resumir textos, revisar código, preparar material de estudio o ayudar con una actividad concreta.

## Interfaz actual de HuggingChat

La interfaz de [HuggingChat](https://huggingface.co/chat/) cambia con frecuencia, pero normalmente incluye:

- **New Chat**: inicia una conversación sin el contexto anterior.
- **Models**: permite explorar y seleccionar modelos disponibles.
- **Settings**: reúne opciones generales de la cuenta y la interfaz.
- **Omni**: selecciona automáticamente un modelo según la solicitud.

La cantidad de modelos y sus nombres pueden cambiar. Algunas capacidades, como trabajar con imágenes o utilizar herramientas, dependen del modelo seleccionado y de la configuración disponible.

### Instrucción inicial y mensaje del sistema

Un texto escrito en el cuadro de chat siempre se envía como mensaje del usuario. Aunque se lo utilice para definir el comportamiento del chatbot, no se convierte técnicamente en un mensaje con rol `system`.

Por eso, en este contexto conviene llamarlo **instrucción inicial** o **prompt inicial**. Puede orientar el resto de la conversación, pero el modelo podría interpretarlo de manera imperfecta o dejar de seguir alguna regla.

Para probar una versión modificada, es recomendable utilizar **New Chat** y comenzar con un contexto limpio.

## Cómo diseñar la instrucción inicial

Una instrucción útil debería responder estas preguntas:

| Elemento | Pregunta |
| --- | --- |
| Función | ¿Qué debe hacer el chatbot? |
| Audiencia | ¿Para quién debe responder? |
| Tono | ¿Cómo debe expresarse? |
| Formato | ¿Cómo debe organizar la respuesta? |
| Límites | ¿Qué debe evitar o rechazar? |
| Incertidumbre | ¿Qué debe hacer si falta información? |

### Ejemplo

```text
A partir de ahora, actúa como un asistente para organizar notas de reuniones.

Tu función es transformar notas desordenadas en un resumen breve y claro.

Responde siempre con estas secciones:
1. Resumen
2. Decisiones
3. Acciones pendientes
4. Preguntas abiertas

Usa un tono profesional y directo.
No inventes fechas, responsables ni decisiones.
Si falta un dato, escribe "No especificado".
```

Las reglas deben ser concretas y compatibles entre sí. Expresiones como “sé útil” o “responde mejor” son demasiado vagas para producir un comportamiento consistente.

## Probar y mejorar el comportamiento

Una sola respuesta correcta no demuestra que la personalización funcione. Conviene utilizar varias entradas:

| Tipo de prueba | Qué permite observar |
| --- | --- |
| Normal | Si cumple su función principal |
| Incompleta | Si detecta información ausente |
| Ambigua | Si pregunta antes de asumir |
| Fuera de alcance | Si respeta sus límites |
| Caso límite | Si mantiene el formato y el tono |

Para comparar versiones:

1. Guardar la instrucción inicial.
2. Probarla con varias entradas.
3. Registrar los problemas observados.
4. Modificar una regla por vez.
5. Repetir las pruebas en una conversación nueva.

Si se busca reproducibilidad, conviene seleccionar manualmente un modelo y registrar su nombre. Con **Omni**, el modelo utilizado puede variar según la solicitud y la disponibilidad.

## Registro básico

Una ficha breve permite conservar y compartir la personalización:

```markdown
# Nombre del asistente

## Propósito
Qué hace y para quién está diseñado.

## Instrucción inicial
Texto completo utilizado al comenzar la conversación.

## Pruebas
- Entrada:
- Respuesta obtenida:
- Resultado esperado:
- Observaciones:

## Cambios realizados
Qué se modificó y por qué.

## Limitaciones
Situaciones en las que necesita verificación o ayuda humana.
```

Una captura de pantalla puede complementar el registro. Debería mostrar la instrucción inicial y una respuesta representativa, ocultando nombres, correos, conversaciones privadas y otros datos personales.

## Limitaciones y seguridad

- Un modelo puede ignorar instrucciones o producir información incorrecta.
- Una respuesta bien redactada no garantiza que sea verdadera.
- No deben compartirse contraseñas, claves, documentos confidenciales ni datos personales innecesarios.
- Las recomendaciones médicas, legales o financieras requieren verificación profesional.
- Las capacidades de carga de imágenes, herramientas y navegación pueden cambiar según el modelo y la interfaz.
- La información importante debe comprobarse con fuentes confiables.

## Recursos oficiales

- [HuggingChat](https://huggingface.co/chat/)
- [Documentación de Chat UI](https://huggingface.co/docs/chat-ui/index)
- [Catálogo de modelos](https://huggingface.co/models)
- [Documentación general de Hugging Face](https://huggingface.co/docs)

