# Fundamentos de Docker y contenedores

## Por qué existen los contenedores

Una aplicación rara vez depende solo de su código. También necesita una versión concreta del lenguaje, bibliotecas, herramientas del sistema, variables de entorno y archivos de configuración. Cuando esas condiciones cambian entre computadoras, aparecen problemas como “funciona en mi máquina”, conflictos entre dependencias y procedimientos de instalación difíciles de repetir.

Un contenedor empaqueta la aplicación y gran parte de su entorno de ejecución en una unidad distribuible. Esto permite crear entornos más consistentes, aislados y fáciles de reemplazar.

Los beneficios principales son:

- **consistencia:** se utiliza la misma definición de imagen en desarrollo, pruebas y despliegue;
- **aislamiento:** los procesos, el sistema de archivos y la red pueden separarse del anfitrión y de otros contenedores;
- **portabilidad:** una imagen puede ejecutarse en sistemas que proporcionen una plataforma compatible;
- **reproducibilidad:** el entorno puede reconstruirse a partir de una definición versionada;
- **ligereza:** varios contenedores pueden compartir el kernel y las capas de imágenes, reduciendo el costo frente a ejecutar un sistema operativo invitado completo por carga de trabajo.

Estas propiedades no son absolutas. Una compilación que utiliza etiquetas mutables, paquetes sin versión o descargas externas puede producir resultados diferentes con el tiempo. La portabilidad también depende de la arquitectura del procesador, el tipo de kernel, los dispositivos disponibles y la configuración externa.

## Máquina física, máquina virtual y contenedor

| Característica | Máquina física | Máquina virtual | Contenedor |
| --- | --- | --- | --- |
| Unidad principal | Hardware completo | Sistema operativo invitado | Proceso o grupo de procesos aislados |
| Kernel | Propio | Cada VM ejecuta su propio kernel | Comparte el kernel del anfitrión o de una VM intermedia |
| Aislamiento | Separación física | Fuerte aislamiento mediante hipervisor | Aislamiento mediante funciones del kernel |
| Inicio | Arranque completo del sistema | Arranque del sistema invitado | Inicio del proceso principal |
| Consumo habitual | Alto y dedicado | Mayor que un contenedor | Menor, aunque depende de la carga |
| Contenido | Sistema operativo, servicios y aplicaciones | Sistema operativo invitado y aplicaciones | Aplicación, bibliotecas, herramientas y configuración necesarias |
| Uso frecuente | Servidor o estación completa | Sistemas operativos distintos y límites de aislamiento fuertes | Desarrollo reproducible, servicios y despliegues reemplazables |

Los contenedores y las máquinas virtuales no son tecnologías excluyentes. Docker Desktop ejecuta normalmente contenedores Linux dentro de una máquina virtual Linux en Windows y macOS. Los contenedores comparten el kernel de esa VM, no el kernel de Windows o macOS directamente.

## Arquitectura de Docker

Docker utiliza una arquitectura cliente-servidor:

- el cliente `docker` interpreta la orden y envía una solicitud mediante la API;
- el daemon `dockerd` administra imágenes, contenedores, redes y volúmenes;
- un registro almacena y distribuye imágenes;
- Docker Hub es el registro público utilizado de forma predeterminada, aunque pueden configurarse otros;
- Docker Desktop integra el cliente, el daemon y herramientas adicionales dentro de una aplicación para Windows, macOS y Linux.

El cliente y el daemon no tienen que estar en la misma computadora. Los **contextos** permiten seleccionar a qué daemon se conecta la CLI:

```bash
docker context ls
docker context show
```

El acceso a un daemon remoto o al socket local equivale a conceder capacidad para crear contenedores y modificar recursos administrados por ese daemon. Debe tratarse como acceso privilegiado.

## Objetos fundamentales

### Imagen

Una imagen es una plantilla de solo lectura que contiene un sistema de archivos por capas y metadatos de ejecución. Puede incluir código, bibliotecas, ejecutables, variables predeterminadas y el comando que debe iniciarse.

Una imagen no es un contenedor detenido. Es la base inmutable desde la que pueden crearse muchos contenedores independientes.

### Contenedor

Un contenedor es una instancia creada a partir de una imagen. Combina:

- las capas de solo lectura de la imagen;
- una capa escribible propia;
- una configuración concreta de red, almacenamiento, variables y límites;
- un proceso principal cuyo estado determina si el contenedor continúa ejecutándose.

Detener un contenedor no lo elimina. Su configuración y su capa escribible continúan disponibles hasta que se lo borra. Al eliminarlo, los datos que solo existían en esa capa se pierden.

### Registro, repositorio, etiqueta y digest

Una referencia de imagen puede tener esta forma:

```text
registry.example.com/team/application:1.4
```

- `registry.example.com` identifica el registro;
- `team/application` identifica el repositorio;
- `1.4` es una etiqueta.

Si no se escribe una etiqueta, Docker utiliza `latest`. El nombre no significa “versión más nueva”: es solamente una etiqueta convencional y puede apuntar a contenido diferente con el tiempo.

El digest identifica contenido concreto:

```text
alpine@sha256:...
```

Las etiquetas son cómodas y mutables. Los digests son inmutables y permiten seleccionar exactamente el mismo contenido.

## Instalación según el sistema operativo

Los requisitos cambian con las versiones del sistema y de Docker. Antes de instalar, se debe consultar la guía oficial correspondiente.

### Windows

Docker Desktop es el camino habitual para ejecutar contenedores Linux. El backend WSL 2 ofrece un kernel Linux y es la opción predeterminada para la mayoría de los equipos compatibles.

Conviene comprobar:

- que la virtualización esté habilitada en BIOS o UEFI;
- que WSL 2 esté instalado y actualizado;
- que Docker Desktop utilice el backend adecuado;
- que la integración esté habilitada para la distribución WSL donde se trabajará.

La CLI puede usarse desde PowerShell, Windows Terminal o una distribución WSL integrada. Al trabajar desde WSL, guardar el código dentro del sistema de archivos Linux suele ofrecer mejor rendimiento que acceder repetidamente a archivos ubicados en una unidad de Windows.

### macOS

Docker Desktop proporciona una máquina virtual Linux y las herramientas de Docker. Se debe elegir el instalador correspondiente a Apple silicon o Intel y verificar que la versión de macOS sea compatible.

### Linux

Puede instalarse Docker Engine directamente o Docker Desktop. Para trabajar solo con la CLI y contenedores locales, Docker Engine suele ser suficiente.

En instalaciones de Docker Engine, el socket pertenece normalmente a `root`. Agregar una cuenta al grupo `docker` evita escribir `sudo`, pero también le concede privilegios equivalentes a root sobre el host. Cuando ese nivel de acceso no es aceptable, debe evaluarse el modo *rootless*.

Docker Desktop tiene condiciones de licencia para determinados usos comerciales. La documentación de instalación vigente es la fuente correcta para comprobar requisitos y términos.

## Verificar una instalación

Estos comandos comprueban capas diferentes del entorno:

```bash
docker --version
docker version
docker info
docker run --rm hello-world
```

| Comando | Qué confirma |
| --- | --- |
| `docker --version` | La CLI está instalada y puede iniciarse. No demuestra que el daemon funcione. |
| `docker version` | Muestra cliente y servidor cuando la conexión con el daemon funciona. |
| `docker info` | Muestra configuración, almacenamiento, recursos y estado general del daemon. |
| `docker run --rm hello-world` | Comprueba descarga, creación, inicio y eliminación de un contenedor sencillo. |

Si aparece un mensaje similar a `Cannot connect to the Docker daemon`, se debe comprobar que Docker Desktop o el servicio de Docker esté iniciado, que el contexto activo sea el correcto y que la cuenta tenga permiso para acceder al socket.

## Descargar y examinar imágenes

`docker pull` obtiene una imagen desde el registro configurado:

```bash
docker pull hello-world
docker pull alpine:latest
```

Si las capas ya existen localmente, Docker puede reutilizarlas. Para listar e inspeccionar imágenes:

```bash
docker image ls
docker image inspect alpine:latest
docker image history alpine:latest
```

`docker images` es un alias histórico de `docker image ls`. La forma organizada por objeto facilita descubrir subcomandos relacionados:

```bash
docker image --help
docker container --help
docker volume --help
```

## Crear y ejecutar contenedores

La estructura general es:

```text
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

`docker run` realiza varias operaciones:

1. localiza la imagen y la descarga si la política de descarga lo requiere;
2. crea un contenedor nuevo;
3. prepara su capa escribible, red y montajes;
4. inicia el proceso principal;
5. conecta la terminal o deja el proceso en segundo plano según las opciones.

Un ejemplo interactivo:

```bash
docker run -it --name alpine-shell alpine:latest sh
```

Dentro del contenedor pueden examinarse el kernel visible y el sistema de archivos:

```bash
uname -a
ls /
```

`exit` termina la shell principal y detiene el contenedor. No lo borra.

### Opciones frecuentes de `docker run`

| Opción | Función |
| --- | --- |
| `--name NAME` | Asigna un nombre estable y legible. |
| `-i` | Mantiene abierta la entrada estándar. |
| `-t` | Asigna un pseudo-terminal. |
| `-d` | Ejecuta en segundo plano. |
| `--rm` | Elimina el contenedor al terminar y también sus volúmenes anónimos asociados. |
| `-e NAME=value` | Define una variable de entorno. |
| `-p HOST:CONTAINER` | Publica un puerto del contenedor en el host. |
| `--mount ...` | Añade un volumen, bind mount o almacenamiento temporal. |
| `--read-only` | Hace de solo lectura el sistema de archivos raíz del contenedor. |

`-it` combina `-i` y `-t`. Es apropiado para una shell interactiva, pero no es necesario para procesos que trabajan en segundo plano.

## Ciclo de vida

El proceso principal es central: cuando termina, el contenedor pasa al estado detenido aunque otros procesos secundarios hubieran sido iniciados.

| Operación | Resultado |
| --- | --- |
| `docker create IMAGE` | Crea un contenedor sin iniciarlo. |
| `docker start NAME` | Inicia un contenedor existente con su configuración y capa escribible anteriores. |
| `docker run IMAGE` | Crea e inicia un contenedor nuevo. |
| `docker stop NAME` | Solicita una detención ordenada y fuerza el cierre si vence el tiempo de espera. |
| `docker restart NAME` | Detiene e inicia el mismo contenedor. |
| `docker exec NAME COMMAND` | Inicia un proceso adicional dentro de un contenedor en ejecución. |
| `docker rm NAME` | Elimina el contenedor y su capa escribible. |

Para reutilizar y adjuntarse a una shell detenida:

```bash
docker start -ai alpine-shell
```

Para abrir otra shell en un contenedor que ya está ejecutándose:

```bash
docker exec -it alpine-shell sh
```

`docker exec` no funciona sobre un contenedor detenido y no reemplaza su proceso principal.

### Listar contenedores

```bash
docker container ls
docker container ls -a
```

La primera forma muestra los que están en ejecución. `-a` agrega los creados, detenidos y terminados. `docker ps` y `docker ps -a` son sus alias tradicionales.

## Inspección y diagnóstico

### Metadatos

```bash
docker inspect alpine-shell
```

`docker inspect` devuelve JSON con configuración, estado, red, montajes, variables y otros metadatos. Para extraer un campo concreto puede utilizarse `--format`:

```bash
docker inspect --format '{{.State.Status}}' alpine-shell
```

### Registros

```bash
docker logs alpine-shell
docker logs --follow --tail 50 alpine-shell
```

Los registros contienen lo que el proceso escribió en `stdout` y `stderr`, según el controlador de logging. No muestran automáticamente archivos de log internos que la aplicación escriba por separado.

### Recursos y procesos

```bash
docker stats
docker top alpine-shell
docker ps --size
```

`docker stats` muestra consumo en vivo. `docker top` lista procesos. `docker ps --size` estima el tamaño de la capa escribible, sin representar necesariamente todo el espacio consumido por registros, volúmenes y cachés.

### Cambios en el sistema de archivos

```bash
docker diff alpine-shell
```

La salida marca rutas agregadas, modificadas o eliminadas en la capa escribible respecto de la imagen.

## Imágenes, capas y copy-on-write

Una imagen está formada por capas de solo lectura. Varios contenedores pueden compartirlas sin duplicar todo su contenido. Cada contenedor añade una capa escribible delgada en la parte superior.

Cuando un proceso modifica un archivo procedente de una capa inferior, el controlador de almacenamiento lo copia a la capa superior antes de modificarlo. Esta estrategia se denomina **copy-on-write**.

Sus consecuencias principales son:

- crear un contenedor es rápido porque no se copia toda la imagen;
- varios contenedores comparten capas comunes;
- cada contenedor conserva sus propios cambios;
- modificar archivos grandes puede producir copias y sobrecosto;
- la capa escribible no es el lugar adecuado para datos importantes o intensivos en escritura.

Docker Engine moderno puede usar el almacén de imágenes de containerd y *snapshotters* en lugar de los controladores clásicos, como `overlay2`. El modelo de capas continúa siendo útil, pero los nombres y detalles internos mostrados por `docker info` pueden variar según la versión y la instalación.

## Dockerfile

Un `Dockerfile` describe cómo construir una imagen. Las instrucciones se procesan en orden y cada una recibe el estado producido por las anteriores.

Ejemplo mínimo:

```Dockerfile
# syntax=docker/dockerfile:1
FROM alpine:3.22

RUN apk add --no-cache curl
WORKDIR /app
COPY message.txt ./message.txt

CMD ["cat", "/app/message.txt"]
```

Para construir y ejecutar:

```bash
docker build -t local/message-app:1.0 .
docker run --rm local/message-app:1.0
```

El punto final de `docker build` es el **contexto de construcción**. Determina qué archivos puede leer el constructor. No es simplemente la ubicación del Dockerfile.

### Instrucciones frecuentes

| Instrucción | Propósito |
| --- | --- |
| `FROM` | Selecciona la imagen base e inicia una etapa de construcción. |
| `RUN` | Ejecuta una orden durante la construcción y conserva el resultado. |
| `WORKDIR` | Define el directorio para instrucciones posteriores y para el proceso predeterminado. |
| `COPY` | Copia archivos desde el contexto, otra etapa o una imagen. |
| `ADD` | Añade funciones como archivos remotos o extracción automática; se prefiere `COPY` cuando no se necesitan. |
| `ENV` | Define variables persistentes en la imagen y disponibles en ejecución. |
| `ARG` | Define valores de construcción; no es un mecanismo seguro para secretos. |
| `EXPOSE` | Documenta un puerto previsto; no lo publica en el host. |
| `USER` | Selecciona el usuario predeterminado para construcción y ejecución posteriores. |
| `CMD` | Define el comando o los argumentos predeterminados al iniciar el contenedor. |
| `ENTRYPOINT` | Define el ejecutable principal y permite combinarlo con argumentos de `CMD`. |

No todas las instrucciones producen necesariamente una capa de sistema de archivos. `RUN`, `COPY` y `ADD` modifican contenido; otras instrucciones pueden cambiar metadatos o la configuración de la imagen.

### `RUN` no es `CMD`

`RUN` se ejecuta al construir:

```Dockerfile
RUN apk add --no-cache curl
```

`CMD` se utiliza como valor predeterminado al iniciar un contenedor:

```Dockerfile
CMD ["cat", "/app/message.txt"]
```

Un comando escrito después del nombre de la imagen en `docker run` reemplaza normalmente a `CMD`:

```bash
docker run --rm local/message-app:1.0 ls -la /app
```

### Forma shell y forma exec

La forma exec utiliza un arreglo JSON:

```Dockerfile
CMD ["python", "app.py"]
```

Ejecuta directamente el programa y permite que reciba señales como proceso principal. La forma shell pasa por un intérprete:

```Dockerfile
CMD python app.py
```

La segunda forma admite expansiones propias de la shell, pero introduce un proceso adicional y puede manejar señales de manera menos directa. Para el proceso principal suele preferirse la forma exec.

### Contexto y `.dockerignore`

El contexto puede incluir archivos grandes, credenciales o contenido irrelevante. Un archivo `.dockerignore` reduce lo enviado al constructor y evita invalidaciones innecesarias:

```text
.git
.env
*.log
node_modules
```

Excluir un archivo del contexto significa que `COPY` no podrá utilizarlo. Los secretos no deben copiarse en la imagen ni pasarse mediante `ARG`; BuildKit ofrece montajes de secretos para esos casos.

## Caché de construcción

El constructor intenta reutilizar resultados anteriores. Para cada instrucción comprueba si existe una entrada compatible en la caché.

- Si una instrucción y sus dependencias no cambiaron, puede aparecer como `CACHED`.
- Cambiar el texto de un `RUN` invalida esa operación.
- `COPY` y `ADD` también consideran metadatos y contenido de los archivos involucrados.
- Al invalidarse una capa, las instrucciones posteriores deben volver a evaluarse o reconstruirse.
- Cambiar solamente la etiqueta indicada con `-t` no obliga a reconstruir las capas.

Una reconstrucción actual puede mostrar `CACHED`; versiones o interfaces antiguas podían mostrar `Using cache`. La evidencia importante es qué paso se reutilizó, no el texto exacto de una versión determinada.

### Ordenar para reutilizar mejor

Las operaciones costosas y que cambian poco deben ubicarse antes de copiar contenido que cambia con frecuencia. En una aplicación con dependencias, suele copiarse primero el manifiesto, instalarse las dependencias y luego copiarse el código.

```Dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

Si solo cambia el código, la instalación puede permanecer en caché.

Para construir sin reutilizar la caché:

```bash
docker build --no-cache -t local/message-app:1.0 .
```

`--no-cache` no garantiza por sí solo descargar una versión nueva de la imagen base. Cuando también se necesita consultar una base actualizada puede añadirse `--pull`.

## Almacenamiento del contenedor

### Capa escribible

Los archivos creados dentro del sistema de archivos normal del contenedor se guardan en su capa escribible. Sobreviven a `stop`, `start` y `restart`, porque esas operaciones mantienen el mismo contenedor. Desaparecen al ejecutar `docker rm`.

Esta capa es apropiada para datos temporales y pequeños, no para bases de datos, información del usuario ni datos que deban sobrevivir a la sustitución del contenedor.

### Tipos de montaje

| Tipo | Administración | Persistencia | Uso habitual |
| --- | --- | --- | --- |
| Volumen | Docker | Sobrevive a la eliminación del contenedor | Datos duraderos administrados por Docker |
| Bind mount | Usuario y sistema anfitrión | Depende de la ruta del host | Compartir código, configuración o archivos con el host |
| `tmpfs` | Memoria del host | Se pierde al detener el contenedor | Datos temporales sensibles o de alta velocidad |

Un montaje oculta temporalmente cualquier contenido que ya existiera en la ruta de destino del contenedor. Los archivos no fueron borrados de la imagen, pero quedan cubiertos mientras el montaje está activo.

## Volúmenes

Los volúmenes tienen un ciclo de vida independiente de los contenedores que los utilizan.

```bash
docker volume create app-data
docker volume ls
docker volume inspect app-data
```

Para montarlo explícitamente:

```bash
docker run -it --name volume-demo \
  --mount type=volume,source=app-data,target=/data \
  alpine:latest sh
```

Dentro del contenedor:

```bash
printf '%s\n' 'persistent data' > /data/example.txt
```

Después de salir y eliminar el contenedor, el volumen permanece:

```bash
docker rm volume-demo
docker volume ls
```

Otro contenedor puede montar `app-data` y leer el archivo. La forma abreviada equivalente es `-v app-data:/data`, pero `--mount` expresa cada componente con mayor claridad. Si un volumen nombrado todavía no existe, Docker puede crearlo al iniciar el contenedor; crearlo antes permite inspeccionarlo y distinguir esa decisión de la ejecución.

Los permisos se evalúan con los identificadores de usuario y grupo visibles dentro del contenedor. Si una aplicación no puede escribir en un volumen o bind mount, conviene comparar su UID/GID con la propiedad de los archivos, en vez de conceder permisos globales.

### Volúmenes nombrados y anónimos

Un volumen nombrado tiene un identificador elegido, como `app-data`. Uno anónimo recibe un nombre generado por Docker. Ambos pueden sobrevivir a la eliminación normal del contenedor.

`docker run --rm` elimina los volúmenes anónimos asociados cuando termina el contenedor, pero conserva los volúmenes nombrados.

## Bind mounts y `tmpfs`

Un bind mount conecta una ruta concreta del host con una ruta del contenedor. Es útil para editar código con herramientas del host y ejecutarlo dentro del contenedor.

```bash
docker run --rm \
  --mount type=bind,source="$PWD",target=/work,readonly \
  alpine:latest ls -la /work
```

La sintaxis de la ruta de origen cambia entre Bash, PowerShell y otros entornos. Además, un bind mount vincula la ejecución a la estructura de directorios del host, por lo que es menos portable que un volumen.

Un bind mount con escritura permite que procesos del contenedor modifiquen archivos del host. `readonly` o `ro` reduce ese riesgo cuando solo se necesita lectura.

`tmpfs` mantiene la información en memoria y no la escribe en el sistema de archivos persistente:

```bash
docker run --rm --tmpfs /run:rw,noexec,nosuid alpine:latest sh
```

Sus datos se pierden al detener el contenedor.

## Publicación de puertos y red

Un servicio puede escuchar dentro del contenedor sin ser accesible desde el host. `-p` crea la publicación:

```bash
docker run -d --name web -p 8080:80 nginx:alpine
```

El puerto `8080` pertenece al host y se dirige al `80` del contenedor. `EXPOSE 80` en un Dockerfile solo documenta la intención; no crea esta publicación.

Para limitar el acceso al propio equipo puede indicarse la dirección de escucha:

```bash
docker run -d --name web-local -p 127.0.0.1:8080:80 nginx:alpine
```

Publicar sin una dirección concreta puede exponer el servicio en todas las interfaces, según la configuración del host y su firewall.

## Uso de disco y limpieza

Antes de eliminar recursos, conviene observar qué existe y cuánto ocupa:

```bash
docker system df
docker system df -v
docker container ls -a
docker image ls
docker volume ls
```

Un contenedor detenido no es necesariamente descartable: puede contener datos no guardados en su capa escribible. Una imagen sin contenedores asociados también puede ser costosa de volver a descargar o construir.

### Limpieza por tipo de recurso

| Comando | Elimina |
| --- | --- |
| `docker container prune` | Todos los contenedores detenidos. |
| `docker image prune` | Imágenes colgantes. |
| `docker image prune -a` | Imágenes no referenciadas por ningún contenedor. |
| `docker network prune` | Redes personalizadas que ningún contenedor utiliza. |
| `docker builder prune` | Caché de construcción no utilizada según las opciones aplicadas. |
| `docker volume prune` | Volúmenes anónimos no referenciados por ningún contenedor. |
| `docker volume prune -a` | Volúmenes nombrados y anónimos no referenciados. |

Los comandos `prune` solicitan confirmación de forma predeterminada. Antes de aceptarla hay que leer el alcance indicado por la versión instalada.

### Limpieza general

```bash
docker system prune
```

Elimina contenedores detenidos, redes no utilizadas, imágenes colgantes y caché de construcción elegible. `-a` amplía la limpieza a todas las imágenes no utilizadas:

```bash
docker system prune -a
```

Los volúmenes no se incluyen de forma predeterminada. Esta variante agrega volúmenes anónimos no utilizados:

```bash
docker system prune --volumes
```

No equivale a eliminar todos los volúmenes nombrados. Para estos debe revisarse `docker volume prune -a` o eliminarse un volumen específico después de comprobar su contenido y sus consumidores.

La limpieza no cuenta con una papelera general. Los contenedores, imágenes y volúmenes eliminados deben reconstruirse, descargarse o restaurarse desde una copia externa.

## Seguridad y buenas prácticas

- No montes `/var/run/docker.sock` dentro de contenedores que no sean totalmente confiables.
- No almacenes contraseñas, tokens ni claves mediante `ARG`, `ENV`, `COPY` o capas de imagen.
- Usa imágenes oficiales o de procedencia verificada y revisa sus etiquetas y plataformas.
- Evita ejecutar como `root` cuando la aplicación no lo necesita; utiliza `USER` y permisos apropiados.
- Publica solo los puertos necesarios y limita la dirección de escucha cuando corresponda.
- Usa `.dockerignore` para excluir secretos, repositorios Git y dependencias locales innecesarias.
- Mantén fuera de la capa escribible cualquier información que deba persistir.
- Prefiere contenedores reemplazables a modificaciones manuales difíciles de reproducir.
- Inspecciona el alcance antes de ejecutar comandos `prune`, `rm -f` o eliminaciones de volúmenes.

## Flujo de diagnóstico

Cuando algo falla, resulta útil separar las capas del sistema:

1. `docker --version`: ¿existe el cliente?
2. `docker context show`: ¿se está usando el daemon esperado?
3. `docker version`: ¿cliente y servidor pueden comunicarse?
4. `docker info`: ¿qué almacenamiento, plataforma y recursos están activos?
5. `docker container ls -a`: ¿el contenedor existe y en qué estado está?
6. `docker logs NAME`: ¿qué informó el proceso principal?
7. `docker inspect NAME`: ¿qué comando, montajes, red y estado fueron configurados?
8. `docker stats` y `docker system df`: ¿faltan recursos o espacio?

No conviene comenzar eliminando todo. Conservar el contenedor detenido permite inspeccionar su estado, sus metadatos y su capa escribible antes de decidir si es seguro descartarlo.

## Referencia rápida

```bash
# Información general
docker version
docker info
docker context ls

# Imágenes
docker pull IMAGE[:TAG]
docker image ls
docker image inspect IMAGE
docker image history IMAGE

# Contenedores
docker run [OPTIONS] IMAGE [COMMAND]
docker container ls -a
docker start CONTAINER
docker stop CONTAINER
docker restart CONTAINER
docker exec -it CONTAINER sh
docker logs CONTAINER
docker inspect CONTAINER
docker stats
docker rm CONTAINER

# Construcción
docker build -t NAME:TAG CONTEXT

# Volúmenes
docker volume create NAME
docker volume ls
docker volume inspect NAME
docker volume rm NAME

# Espacio y limpieza
docker system df -v
docker container prune
docker image prune
docker builder prune
docker volume prune
docker system prune
```

## Referencias

- [What is Docker?](https://docs.docker.com/get-started/docker-overview/)
- [Get Docker](https://docs.docker.com/get-started/get-docker/)
- [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker storage](https://docs.docker.com/engine/storage/)
- [Docker volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker storage drivers](https://docs.docker.com/engine/storage/drivers/)
- [Containers and virtual machines](https://www.redhat.com/en/topics/containers/containers-vs-vms)
- [Docker build cache](https://docs.docker.com/build/cache/)
- [Build cache invalidation](https://docs.docker.com/build/cache/invalidation/)
- [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Prune unused Docker objects](https://docs.docker.com/engine/manage-resources/pruning/)
- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
