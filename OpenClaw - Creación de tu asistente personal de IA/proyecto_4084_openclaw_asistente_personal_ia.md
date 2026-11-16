# OpenClaw: un asistente personal de IA que se ejecuta en el propio equipo

## Por qué existe un asistente autoalojado

Un asistente de IA en la nube funciona bien mientras no importe dónde quedan las conversaciones, cuánto cuesta cada mes o qué ocurre si el proveedor cambia sus condiciones. Además, su alcance es el de la ventana de chat: puede explicar cómo ordenar una carpeta, pero no ordenarla. Un asistente **autoalojado** invierte ambas cosas: el programa que coordina todo se ejecuta en la máquina de la persona usuaria, guarda sus datos en archivos locales y puede actuar sobre ese mismo equipo (leer archivos, abrir un navegador, programar avisos).

Esa capacidad de actuar es la razón de ser de OpenClaw y, a la vez, su principal riesgo. Dos ideas conviene tener presentes desde el principio:

- **El asistente no es el modelo de lenguaje.** OpenClaw es la capa que recibe mensajes, recuerda contexto, decide qué herramientas usar y responde. La inteligencia la aporta un modelo externo (por ejemplo, Gemini o GPT) al que OpenClaw se conecta con una clave de API. Si el modelo falla, el asistente falla, aunque el programa esté en perfecto estado.
- **Todo mensaje entrante se trata como no confiable.** Como el asistente puede ejecutar acciones reales, cualquier persona que pueda escribirle puede intentar influir en lo que hace. Gran parte de la configuración de seguridad existe para decidir quién puede hablarle.

## Las piezas y cómo se relacionan

| Pieza | Qué es | Analogía |
| --- | --- | --- |
| **Gateway** (puerta de enlace) | Proceso que permanece en ejecución, escucha en `127.0.0.1:18789` y conecta todo lo demás. | La centralita de un edificio. |
| **Channels** (canales) | Plataformas de mensajería por las que se habla con el asistente: Telegram, Discord, WhatsApp, Slack, etc. | Los teléfonos desde los que se llama a la centralita. |
| **Nodes** (nodos) | Dispositivos que ejecutan acciones locales (el equipo, un móvil). | Los empleados que realizan el trabajo. |
| **Skills** (habilidades) | Instrucciones modulares que enseñan al asistente a hacer algo concreto. | Un manual de procedimiento. |
| **Proveedor de modelo** | El servicio de IA que genera las respuestas. | El «cerebro» contratado aparte. |
| **Workspace** (espacio de trabajo) | La carpeta donde el asistente guarda su identidad, sus notas y sus habilidades propias. | Su escritorio. |

El recorrido de un mensaje ilustra cómo encajan:

```text
Mensaje en Discord  →  Gateway  →  proveedor de modelo (razona y decide)
                          ↑                    │
                          │                    ↓
                    respuesta ← habilidades y herramientas (leer archivos, buscar en la web...)
```

La Gateway nunca sale a buscar mensajes por sí sola al mundo: por defecto escucha únicamente en la dirección local (`127.0.0.1`, también llamada *loopback*), de modo que solo programas del mismo equipo pueden conectarse a ella. Los canales funcionan porque la propia Gateway se conecta hacia afuera con el servicio de mensajería (por ejemplo, consultando a Telegram cada pocos segundos), no porque alguien entre desde Internet.

## Instalación: qué hace cada decisión

El asistente de configuración inicial (*onboarding*) escribe un archivo de configuración y deja la Gateway lista. Conviene entender las decisiones que toma:

- **Servicio de arranque automático.** La opción `--install-daemon` registra la Gateway para que se inicie sola. En Windows se instala como un elemento de inicio de sesión, así que arranca al iniciar sesión, no al encender el equipo. Sin ese servicio, la Gateway solo funciona mientras una terminal la mantenga abierta (`openclaw gateway run`).
- **Proveedor y modelo.** Sin un proveedor configurado el asistente no responde. El asistente de configuración permite probarlo con una llamada real (`AI access works`).
- **Búsqueda web.** La búsqueda no la hace el modelo: es una herramienta con su propio proveedor. Existen proveedores que exigen clave (Brave, Tavily...) y otros marcados como *key-free*, como DuckDuckGo, que no la necesitan.
- **Memoria semántica (*embeddings*).** Opcional. Convierte las notas en vectores numéricos que representan su significado, para buscar por sentido y no solo por palabras. No hace falta para un uso básico y consume cuota del proveedor.

Un detalle práctico: tras cambiar configuración importante (un canal nuevo, activar los *hooks*), la Gateway en ejecución puede seguir usando la configuración anterior hasta reiniciarse.

## El modelo y sus límites

Cada respuesta es una petición al proveedor de modelo, y esas peticiones tienen costes y límites:

- **Cuota gratuita.** Los planes gratuitos limitan la cantidad de texto de entrada por minuto. Un asistente envía mucho más texto del que se escribe a mano, porque cada petición incluye sus instrucciones, las habilidades disponibles y el historial (la primera conversación ya ronda las 20 000 unidades de texto, o *tokens*). Por eso un plan gratuito puede agotarse con pocos mensajes.
- **Error 429.** Es el código HTTP que significa «demasiadas peticiones». Cuando aparece, OpenClaw reintenta tras una espera; mientras tanto el chat parece colgado en un estado como `finishing context`. La solución es esperar, usar un modelo más ligero o un proveedor con más cuota.
- **Modelos distintos, límites distintos.** Una clave puede tener cuota en un modelo y no en otro. Conviene comprobar con una llamada directa y mínima antes de culpar a la configuración.

## Qué hay dentro del espacio de trabajo

El asistente guarda su «personalidad» y su memoria en **archivos de texto** dentro del *workspace* (por defecto `~/.openclaw/workspace`), que además es un repositorio Git. Eso permite leerlos, editarlos y versionarlos sin ninguna herramienta especial:

| Archivo | Para qué sirve |
| --- | --- |
| `AGENTS.md` | Convenciones de trabajo y «líneas rojas»: qué no hacer sin preguntar (compartir datos privados, borrar de forma irreversible). |
| `SOUL.md` | Personalidad y tono: cómo se comporta, no qué sabe hacer. |
| `IDENTITY.md` | Nombre, estilo y emoji del asistente. Se rellena durante el primer chat. |
| `USER.md` | Preferencias estables de la persona usuaria, escritas como directivas. |
| `skills/` | Habilidades propias, una carpeta por habilidad. |

Al ser texto plano, la «memoria» del asistente es auditable: basta abrir un archivo para ver qué recuerda y por qué actúa como actúa.

## Habilidades

Una **habilidad** es una carpeta con un archivo `SKILL.md`. No contiene código que ejecutar: contiene instrucciones en lenguaje natural que el modelo lee y sigue. El archivo tiene dos partes:

```markdown
---
name: quick-note
description: Guarda una nota rápida cuando el usuario escriba "nota:".
---
Cuando el mensaje empiece por "nota:", añade el texto al final de notes.txt.
```

- El bloque entre `---` (*frontmatter*) declara el `name` y la `description`.
- El cuerpo contiene las instrucciones.

Lo importante es **cómo se activa**: el modelo ve la lista de habilidades disponibles, con sus descripciones, y decide cuál aplicar según el mensaje recibido. Por eso la `description` funciona como disparador: una descripción vaga produce una habilidad que casi nunca se usa, y una demasiado amplia la hace activarse sin querer. También puede invocarse de forma explícita con `/nombre`.

Una habilidad aparece como `ready` (lista) o `needs setup` (falta algo que necesita, como un programa o una variable de entorno). Su origen indica de dónde viene: las **incluidas** (`openclaw-bundled`) vienen con el programa, y las del **workspace** (`openclaw-workspace`) son las propias o las descargadas de **ClawHub**, el registro público de habilidades.

Instalar una habilidad de terceros equivale a dejar que otra persona escriba instrucciones para el asistente. Por eso ClawHub muestra una **auditoría de seguridad** antes de instalar, pero sigue siendo prudente leer el `SKILL.md` y comprobar qué herramientas pide.

## Canales, emparejamiento y permisos

Cada canal necesita una credencial propia (normalmente un **token de bot**), que se obtiene en la plataforma y se entrega a OpenClaw. Las diferencias prácticas entre plataformas están sobre todo en esa fase:

- **Telegram:** el token se pide a un bot llamado @BotFather y el bot se puede escribir directamente, sin más pasos.
- **Discord:** el token se crea en un portal de desarrolladores, hay que activar un permiso para que el bot lea el contenido de los mensajes (*Message Content Intent*) y hay que **invitarlo a un servidor**, porque Discord solo permite escribir por mensaje privado a un bot con el que se comparte servidor.

Una vez conectado el canal, quién puede hablar con el asistente lo decide el **emparejamiento** (*pairing*):

1. Una persona desconocida escribe al bot.
2. El bot responde con un **código** de 8 caracteres que caduca en una hora.
3. Solo cuando el propietario lo aprueba desde su equipo (`openclaw pairing approve <canal> <código>`) el remitente queda en la lista de permitidos.

Este modelo evita que cualquiera que encuentre el bot pueda darle órdenes. La política se controla con `dmPolicy`, cuyo valor por defecto, `pairing`, es el recomendable. No existe un comando para «bloquear» a alguien: quien no se aprueba, simplemente no se atiende.

Aparte de «quién puede hablar» existe «quién puede administrar»: el **propietario de comandos** (*command owner*, `commands.ownerAllowFrom`) es la cuenta que puede actualizar OpenClaw, reiniciar la Gateway y cambiar la configuración desde el chat. Si esa lista está vacía, el **primer remitente aprobado** pasa a ser propietario automáticamente. Por eso el primer emparejamiento debe ser siempre el de la propia persona dueña del equipo.

## Automatización

Un flujo de automatización combina tres elementos: un **desencadenador**, un **proceso** (una instrucción para el asistente) y una **salida** (normalmente un mensaje a un canal).

### Tareas programadas (*cron*)

Una tarea programada se define con una **expresión cron** de cinco campos:

```text
┌ minuto (0-59)
│ ┌ hora (0-23)
│ │ ┌ día del mes (1-31)
│ │ │ ┌ mes (1-12)
│ │ │ │ ┌ día de la semana (0-6, domingo = 0)
│ │ │ │ │
0 8 * * *       →  todos los días a las 08:00
30 7 * * 1-5    →  de lunes a viernes a las 07:30
```

Un `*` significa «cualquier valor». La hora se interpreta en una **zona horaria**, que debe indicarse de forma explícita (por ejemplo, `America/Montevideo`) para que el aviso no llegue a una hora inesperada. Una tarea también necesita un nombre y un destino de entrega: sin destino, se ejecuta, pero el resultado no llega a ningún chat. Ejecutar la tarea manualmente permite comprobarla sin esperar al horario.

La Gateway ya incluye tareas propias de este tipo, como un latido periódico (*heartbeat*) que se ejecuta cada 30 minutos; aparecen al listar las tareas junto a las creadas por la persona usuaria.

### Webhooks (*hooks*)

Un **webhook** es una dirección HTTP que, al recibir una petición, hace que el asistente ejecute una instrucción. Sirve para que otros sistemas (un repositorio, una canalización de integración continua, un sensor) avisen al asistente. Como cualquiera que conozca la dirección podría dar órdenes, el acceso se protege con un **token** propio, distinto del de la Gateway, enviado en la cabecera `Authorization: Bearer <token>`.

Los códigos de respuesta indican el problema:

| Respuesta | Significado |
| --- | --- |
| `200` | Petición aceptada; la ejecución continúa en segundo plano (devuelve un `runId`). |
| `401` | Falta el token o es incorrecto. |
| `404` | La ruta no existe, o los *hooks* no están activados. |
| `400` | El cuerpo de la petición no es un JSON válido. |

Para entregar el resultado en un chat, la petición indica el canal y el destino. Un ejemplo de la forma del cuerpo:

```json
{ "message": "Resume mi día", "deliver": true, "channel": "discord", "to": "user:123456789" }
```

En Windows PowerShell conviene saber que `curl` es un alias de otro comando (se usa `curl.exe`) y que las comillas dobles del JSON deben escaparse con una barra invertida, porque PowerShell las elimina antes de enviar el texto.

## Voz

La interacción por voz se compone de dos transformaciones y una condición:

- **STT** (*speech to text*) convierte la voz en texto.
- **TTS** (*text to speech*) convierte la respuesta en audio.
- **Talk Mode** encadena ambas: escucha, procesa la petición y responde hablando.

La voz del asistente no sale de OpenClaw sino de un **proveedor de voz**, y requiere credenciales igual que el modelo. Que el TTS esté «activado» no basta: hace falta un proveedor configurado. Otras funciones de voz dependen de las aplicaciones para dispositivos: la escucha continua de una palabra de activación (*Voice Wake*) y el panel visual (*Live Canvas*) no están disponibles en una instalación solo de escritorio.

El navegador solo permite usar el micrófono en `localhost` o en conexiones HTTPS; la dirección local de la Gateway cumple esa condición. La interfaz web conviene abrirla con `openclaw dashboard`, que la entrega ya autenticada, en lugar de escribir la dirección a mano, porque sin token la página lo solicita.

## Seguridad

OpenClaw parte de un modelo de confianza claro: **una sola persona operadora de confianza**. No está diseñado para separar a varias personas con intereses distintos dentro de una misma Gateway. Si varias personas pueden escribir a un asistente con herramientas activadas, todas influyen en lo que hace.

Principios que se derivan de ello:

- **Mínimo privilegio.** Activar solo las habilidades y funciones que se usan.
- **Control de quién entra.** Mantener el emparejamiento y aprobar únicamente cuentas propias.
- **No exponer la Gateway sin autenticación.** Por defecto escucha solo en local; abrirla a otras redes sin credenciales equivale a ofrecer el control del equipo.
- **Proteger los secretos.** Tokens, claves de API y token de *hooks* se guardan en la configuración; ese archivo debe tratarse como una contraseña.
- **Revisar lo de terceros.** Las habilidades y los complementos descargados ejecutan o dirigen acciones en el equipo.

El comando `openclaw security audit` revisa la instalación y lista advertencias, cada una con su corrección. Algunas habituales y lo que significan:

| Advertencia | Qué señala |
| --- | --- |
| `gateway.trusted_proxies_missing` | Si se publica la interfaz tras un proxy inverso, hay que declararlo para que no se puedan falsificar comprobaciones de «cliente local». |
| `hooks.default_session_key_unset` | Las peticiones a los *hooks* crean una sesión nueva cada vez; conviene fijar una sesión conocida. |
| `hooks.allowed_agent_ids_unrestricted` | Quien conozca el token puede dirigir la petición a cualquier agente configurado. |
| `plugins.installs_unpinned_npm_specs` | Los complementos se instalaron sin fijar versión; una versión futura podría cambiar su comportamiento. |

Una advertencia no es un error: indica un compromiso que hay que decidir conscientemente.

## Cómo diagnosticar un problema

Cuatro herramientas cubren casi todos los casos:

- `openclaw status`: estado general (Gateway, agentes, canales).
- `openclaw gateway status`: si el servicio está en ejecución y en qué puerto escucha.
- `openclaw logs --follow`: registro en tiempo real, útil para ver qué ocurre mientras se prueba algo.
- `openclaw doctor`: diagnóstico y reparaciones guiadas.

Mensajes frecuentes y su causa real:

| Mensaje o síntoma | Causa más probable |
| --- | --- |
| `No route-compatible authentication source is configured` | No hay proveedor de modelo configurado, o se cambió después de crear una tarea. |
| `429` / `Rate limit exceeded` | Se superó la cuota del proveedor. |
| El canal «sale» repetidamente con `token unavailable` | La Gateway en ejecución no cargó el token nuevo; hay que reiniciarla. |
| `running scripts is disabled on this system` | La política de ejecución de PowerShell bloquea los scripts de `npm` y `openclaw`. |
| Un cambio de configuración «se guardó» pero no surte efecto | La Gateway en marcha no lo aplicó; hay que reiniciarla. |
| Una habilidad recién creada no aparece | La detección tarda unos segundos; repetir el comando. |

## Qué conviene poder explicar al terminar

Más allá de ejecutar los comandos, la comprensión se comprueba con preguntas como:

- ¿Por qué un asistente que funciona en el equipo sigue necesitando un servicio externo para pensar?
- ¿Qué diferencia hay entre un canal, un nodo y una habilidad?
- ¿Por qué la descripción de una habilidad importa más que su cuerpo para que se use?
- ¿Qué riesgo concreto introduce cada nueva capacidad (herramientas, habilidades de terceros, *hooks*) y qué control lo reduce?
- ¿Qué cambia cuando una tarea programada o un webhook pasan de «ejecutarse» a «entregar el resultado en un chat»?
