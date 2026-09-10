# Permisos en Linux

## Un sistema multiusuario

Linux fue diseñado para que varias personas y procesos utilicen el mismo sistema sin acceder indiscriminadamente a los datos de los demás. Para ello, cada proceso se ejecuta con una identidad y cada archivo o directorio conserva información de propiedad y acceso.

Los elementos principales de este modelo son:

| Elemento | Descripción |
| --- | --- |
| Usuario | Cuenta identificada internamente por un número llamado UID. |
| Grupo | Conjunto de usuarios identificado por un GID. |
| Propietario | Usuario asociado con un archivo o directorio. |
| Grupo propietario | Grupo asociado con ese objeto. |
| Modo | Bits que expresan los permisos del propietario, del grupo y de los demás usuarios. |
| Proceso | Programa en ejecución que actúa con determinados UID y GID. |

Los nombres — por ejemplo, el usuario `alice` y el grupo `developers`, que se usan como ejemplo a lo largo de este documento — son representaciones legibles. El kernel toma decisiones mediante identificadores numéricos.

## Identidad de usuarios y grupos

### UID, GID y grupos suplementarios

Cada usuario tiene:

- un UID;
- un grupo primario;
- cero o más grupos suplementarios.

El grupo primario se usa normalmente como grupo inicial de los archivos que crea el usuario. Los grupos suplementarios permiten compartir recursos con otros equipos sin cambiar la identidad principal.

El comando `id` reúne esta información:

```bash
id
```

Una salida posible es:

```text
uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo),1002(developers)
```

Opciones útiles:

| Comando | Información |
| --- | --- |
| `id -u` | UID efectivo. |
| `id -un` | Nombre del usuario efectivo. |
| `id -g` | GID efectivo. |
| `id -gn` | Nombre del grupo efectivo. |
| `id -G` | Todos los GID del proceso. |
| `id -Gn` | Todos los grupos por nombre. |
| `id -ru` | UID real. |
| `id username` | Identidad registrada para otro usuario. |

`whoami` imprime únicamente el nombre correspondiente al UID efectivo:

```bash
whoami
```

En la práctica equivale a:

```bash
id -un
```

`groups` muestra los grupos a los que pertenece el usuario actual o el usuario indicado:

```bash
groups
groups alice
```

La salida depende de las cuentas, los grupos y el servicio de identidad configurado en cada sistema. No debe esperarse la misma lista en dos equipos distintos.

### Identidad real y efectiva

Un proceso puede tener un identificador real y otro efectivo:

- El **UID real** identifica al usuario que inició el proceso.
- El **UID efectivo** es el que se utiliza normalmente para comprobar el acceso a archivos y otros recursos.

En procesos comunes ambos valores coinciden. Pueden diferir al usar mecanismos de elevación de privilegios o programas con el bit `setuid` (bit especial que se explica más adelante). De la misma forma existen GID real y efectivo.

Esta distinción explica por qué `$USER` no siempre es una prueba fiable de la identidad con la que actúa un proceso: es una variable de entorno y puede conservar un valor anterior. Para comprobar la identidad efectiva deben usarse `whoami` o `id`.

## Propiedad y selección de permisos

Cada archivo tiene un propietario, un grupo propietario y permisos para tres clases:

| Clase | Símbolo | A quién representa |
| --- | :---: | --- |
| Usuario propietario | `u` | Al UID propietario del archivo. |
| Grupo propietario | `g` | A procesos que pertenecen al GID asociado. |
| Otros | `o` | A quienes no coinciden con las clases anteriores. |
| Todas las clases | `a` | Abreviatura de `ugo` en modos simbólicos. |

Para una operación concreta, las clases no se suman. El sistema selecciona una:

1. Si el UID efectivo coincide con el propietario, comprueba los bits de `u`.
2. En caso contrario, si algún GID del proceso coincide con el grupo propietario, comprueba los bits de `g`.
3. Si tampoco coincide, comprueba los bits de `o`.

Esto tiene una consecuencia poco intuitiva: si el propietario no posee permiso de lectura, un permiso más amplio para `g` u `o` no se agrega para compensarlo. La coincidencia con el propietario ya determinó la clase aplicable.

## Interpretar `ls -l`

El formato largo de `ls` muestra tipo, permisos, propietario y grupo:

```bash
ls -l report.txt
```

```text
-rw-r----- 1 alice developers 2048 Sep 12 10:30 report.txt
```

Los campos más relevantes son:

| Fragmento | Significado |
| --- | --- |
| `-` | Tipo de objeto. |
| `rw-` | Permisos del propietario. |
| `r--` | Permisos del grupo. |
| `---` | Permisos de los demás usuarios. |
| `alice` | Propietario. |
| `developers` | Grupo propietario. |

El primer carácter describe el tipo:

| Carácter | Tipo |
| :---: | --- |
| `-` | Archivo regular. |
| `d` | Directorio. |
| `l` | Enlace simbólico. |
| `c` | Dispositivo de caracteres. |
| `b` | Dispositivo de bloques. |
| `p` | Tubería con nombre. |
| `s` | Socket. |

Para consultar el directorio como objeto, y no listar su contenido, se usa `-d`:

```bash
ls -ld shared_dir
```

`stat` permite obtener una vista precisa, incluida la representación numérica:

```bash
stat -c '%A %a %U %G %n' report.txt
```

```text
-rw-r----- 640 alice developers report.txt
```

Los especificadores usados fueron: `%A` (permisos en formato simbólico), `%a` (en formato numérico), `%U` (propietario), `%G` (grupo propietario) y `%n` (nombre del archivo).

## Significado de lectura, escritura y ejecución

Los permisos se representan con `r`, `w` y `x`. Su efecto cambia según el tipo de objeto.

### En archivos regulares

| Permiso | Valor | Efecto habitual |
| :---: | :---: | --- |
| `r` | 4 | Leer el contenido. |
| `w` | 2 | Modificar o truncar el contenido. |
| `x` | 1 | Solicitar su ejecución como programa. |

El permiso `x` no convierte cualquier archivo en un programa válido. Un script necesita un intérprete reconocible, normalmente indicado mediante un *shebang*, y un binario debe tener un formato que el sistema pueda cargar.

Poder escribir un archivo no implica necesariamente poder eliminarlo. Eliminar o renombrar una entrada depende principalmente de los permisos del directorio que la contiene.

### En directorios

| Permiso | Efecto habitual |
| :---: | --- |
| `r` | Leer la lista de nombres contenidos en el directorio. |
| `w` | Modificar sus entradas: crear, eliminar o renombrar nombres. |
| `x` | Atravesar el directorio y acceder a una entrada conocida. |

Las combinaciones importan:

- `r` sin `x` permite obtener nombres, pero impide acceder normalmente a sus metadatos o contenidos.
- `x` sin `r` permite acceder a un nombre conocido, pero no enumerar libremente el directorio.
- `w` suele necesitar `x` para crear, eliminar o renombrar entradas.
- Para llegar a un archivo se necesita `x` en cada directorio de la ruta.

Por ejemplo, aunque un archivo sea legible, faltará acceso si no se puede atravesar alguno de sus directorios padre.

## Notación octal

Cada permiso posee un valor:

```text
r = 4
w = 2
x = 1
```

Los valores presentes en una clase se suman:

| Dígito | Forma simbólica | Cálculo | Acceso |
| :---: | :---: | :---: | --- |
| `0` | `---` | 0 | Ninguno. |
| `1` | `--x` | 1 | Ejecución o acceso. |
| `2` | `-w-` | 2 | Escritura. |
| `3` | `-wx` | 2 + 1 | Escritura y ejecución. |
| `4` | `r--` | 4 | Lectura. |
| `5` | `r-x` | 4 + 1 | Lectura y ejecución. |
| `6` | `rw-` | 4 + 2 | Lectura y escritura. |
| `7` | `rwx` | 4 + 2 + 1 | Todos los permisos básicos. |

Un modo de tres dígitos sigue siempre el orden propietario, grupo y otros:

```text
640 = rw-r-----
755 = rwxr-xr-x
701 = rwx-----x
```

Algunos modos frecuentes son:

| Modo | Uso habitual |
| :---: | --- |
| `600` | Datos privados modificables por su propietario. |
| `640` | Datos privados que un grupo puede leer. |
| `644` | Archivo modificable por el propietario y legible por los demás. |
| `700` | Directorio o programa privado. |
| `750` | Directorio o programa accesible para un grupo. |
| `755` | Directorio atravesable o programa ejecutable por todos. |

El contexto determina si un modo es apropiado. `777` concede escritura y ejecución a cualquier usuario y rara vez es una solución segura.

## Cambiar permisos con `chmod`

La forma general es:

```text
chmod MODE FILE...
```

Solo el propietario del objeto o un proceso con privilegios suficientes puede cambiar su modo.

### Modo simbólico

Un modo simbólico combina una clase, un operador y permisos:

```text
[ugoa][+-=][rwxXst]
```

`X`, `s` y `t` son casos especiales que se explican más adelante (directorios y bits especiales); por ahora alcanza con `r`, `w` y `x`.

Los operadores son:

| Operador | Acción |
| :---: | --- |
| `+` | Agrega permisos y conserva los demás. |
| `-` | Quita permisos y conserva los demás. |
| `=` | Define exactamente los permisos indicados para esa clase. |

Ejemplos:

```bash
chmod u+x deploy.sh
chmod g-w notes.txt
chmod o-r private.log
chmod ug+rx handbook.pdf
chmod u=rw,g=r,o= report.txt
chmod a+x launcher.sh
```

Las operaciones relativas son útiles cuando deben conservarse los bits que no se mencionan. El operador `=` reemplaza el conjunto correspondiente, por lo que requiere más cuidado.

Pueden expresarse varias modificaciones separadas por comas:

```bash
chmod u+x,g+rx,o-rwx tool.sh
```

Si se omite la clase, `chmod` trata la operación aproximadamente como si se hubiera indicado `a`, pero respeta restricciones introducidas por la `umask` (se explica más adelante). Es preferible escribir la clase cuando se busca un resultado inequívoco.

### Modo numérico

El modo numérico establece el patrón indicado para las tres clases:

```bash
chmod 640 report.txt
chmod 755 deploy.sh
```

Una diferencia práctica:

- `chmod u+x file` agrega un bit sin alterar los demás.
- `chmod 744 file` reemplaza los permisos básicos con un valor exacto.

Antes y después de una modificación conviene verificar el estado:

```bash
stat -c '%A %a %U %G %n' deploy.sh
chmod u+x deploy.sh
stat -c '%A %a %U %G %n' deploy.sh
```

### Copiar el modo de otro archivo

GNU `chmod` puede utilizar un archivo como referencia:

```bash
chmod --reference=template.conf active.conf
```

El modo de `active.conf` pasa a coincidir con el de `template.conf`, sin necesidad de conocer previamente su número octal. Esto copia el modo, no el propietario ni el grupo.

### Directorios y `X`

La `X` mayúscula agrega ejecución únicamente cuando el objeto es un directorio o cuando algún bit de ejecución ya estaba presente:

```bash
chmod -R a+rX public_tree
```

Este patrón resulta útil para árboles con archivos y directorios: habilita el acceso a los directorios sin convertir automáticamente todos los archivos de datos en ejecutables.

No obstante, un archivo que ya tenga algún bit `x` puede recibir otros bits de ejecución con `X`. Si se necesita actuar exclusivamente sobre directorios, debe seleccionarse el tipo explícitamente:

```bash
find workspace -type d -exec chmod a+x {} +
```

Para los directorios visibles ubicados inmediatamente en la carpeta actual, el patrón `*/` puede ser suficiente:

```bash
chmod a+x */
```

Ese patrón no incluye nombres ocultos y no recorre niveles inferiores.

### Recursividad

`-R` modifica un árbol completo:

```bash
chmod -R u=rwX,g=rX,o= docs
```

Una operación recursiva incorrecta amplifica rápidamente un error. Antes de ejecutarla se deben revisar:

- la ruta de inicio;
- la clase y el operador;
- el tratamiento de archivos frente a directorios;
- la presencia de enlaces simbólicos;
- el alcance sobre nombres ocultos.

No conviene aplicar a todo un árbol un mismo modo numérico como `chmod -R 644 dir`: los directorios perderían `x` y dejarían de ser transitables.

### Nombres que comienzan con guion

`--` marca el final de las opciones:

```bash
chmod 600 -- -draft
```

También puede utilizarse una ruta explícita:

```bash
chmod 600 ./-draft
```

## Permisos iniciales y `umask`

Al crear un objeto, el programa solicita un modo inicial y la `umask` elimina determinados bits. El cálculo conceptual es una operación de bits:

```text
final_mode = requested_mode & ~umask
```

No debe tratarse como una resta decimal común.

Las solicitudes iniciales habituales son:

- archivos regulares: `666`, sin ejecución automática;
- directorios: `777`, porque necesitan `x` para ser transitables.

Con una `umask` de `022`, el resultado típico es:

```text
file:      666 & ~022 = 644
directory: 777 & ~022 = 755
```

La máscara actual se consulta con:

```bash
umask
umask -S
```

Puede cambiarse para la sesión actual:

```bash
umask 027
```

Con `027`, los archivos nuevos suelen quedar en `640` y los directorios en `750`.

### Crear archivos y directorios

`touch` crea un archivo vacío si no existe. Si existe, actualiza sus marcas de tiempo:

```bash
touch empty.txt
```

`mkdir -m` permite solicitar un modo concreto al crear un directorio:

```bash
mkdir -m 750 private_dir
```

Para comprobar el resultado:

```bash
stat -c '%A %a %n' private_dir
```

La creación y el cambio posterior de modo son operaciones distintas. `touch` no es una forma de vaciar de manera incondicional un archivo existente.

### Scripts ejecutables

Un script Bash sencillo identifica su intérprete en la primera línea:

```bash
#!/bin/bash
whoami
```

Después de guardar el archivo, el permiso de ejecución permite invocarlo directamente:

```bash
chmod u+x identity.sh
./identity.sh
```

La ruta `./` es necesaria cuando el directorio actual no forma parte de `PATH`. Ejecutar un script crea otro proceso; los cambios de identidad, variables o directorio realizados allí no modifican de forma permanente la shell que lo inició.

## Cambiar propietario y grupo

### `chown`

`chown` cambia el propietario, el grupo o ambos:

```bash
chown alice report.txt
chown alice:developers report.txt
chown :developers report.txt
```

Las formas significan:

| Forma | Resultado |
| --- | --- |
| `OWNER FILE` | Cambia el propietario. |
| `OWNER:GROUP FILE` | Cambia propietario y grupo. |
| `:GROUP FILE` | Cambia solamente el grupo. |

En Linux, cambiar el propietario normalmente requiere privilegios administrativos:

```bash
sudo chown alice report.txt
```

Esta restricción protege la trazabilidad y evita que un usuario transfiera archivos para eludir cuotas o prepare archivos con implicaciones de seguridad para otra cuenta. Un cambio de propietario también puede provocar que el sistema elimine bits especiales como `setuid` o `setgid`.

### Cambio condicional con `--from`

`--from` aplica el cambio solo si la propiedad actual coincide con la condición:

```bash
sudo chown --from=olduser newuser archive.dat
sudo chown --from=olduser:oldgroup newuser:newgroup archive.dat
```

Es una forma segura de evitar que una operación alcance objetos cuya propiedad cambió desde que se planificó el comando.

### `chgrp`

`chgrp` cambia el grupo propietario:

```bash
chgrp developers report.txt
```

El propietario puede asignar normalmente uno de los grupos a los que pertenece. Asignar grupos arbitrarios o modificar objetos ajenos requiere privilegios adicionales.

La misma operación puede expresarse con:

```bash
chown :developers report.txt
```

### Cambios recursivos

Para modificar todo un árbol:

```bash
sudo chown -R alice:developers shared_tree
```

Antes de usar `-R` se debe inspeccionar la ruta. Un comodín como `*` no incluye normalmente nombres cuyo primer carácter es `.`. Además, `chown -R ... directory` puede cambiar tanto el directorio inicial como su contenido, mientras que una expansión de nombres actúa solamente sobre los operandos que la shell produjo.

Las opciones `-H`, `-L` y `-P` controlan cómo se recorren enlaces simbólicos durante una operación recursiva. No deben seleccionarse sin leer la página de manual de la versión instalada.

### Verificación

Después de cambiar propiedad:

```bash
stat -c '%U %G %n' report.txt
ls -ld report.txt
```

`chmod` cambia permisos; `chown` y `chgrp` cambian propiedad. Ninguno reemplaza a los otros.

## Enlaces simbólicos

Un enlace simbólico almacena una ruta hacia otro objeto:

```bash
ln -s target.txt shortcut.txt
ls -l shortcut.txt
```

Una salida habitual es:

```text
lrwxrwxrwx 1 alice alice 10 Sep 12 11:00 shortcut.txt -> target.txt
```

En Linux, los permisos mostrados para el propio enlace suelen ser `rwxrwxrwx` y normalmente no se utilizan para autorizar el acceso. El acceso efectivo depende del destino y de los directorios de la ruta.

Hay que distinguir dos objetos:

- el enlace;
- el archivo o directorio al que apunta.

Por defecto, GNU `chown` actúa sobre el destino de un enlace indicado como operando. `-h` solicita cambiar la propiedad del enlace en sí:

```bash
sudo chown -h alice:developers shortcut.txt
```

Puede verificarse sin seguir el enlace con:

```bash
stat -c '%U %G %N' shortcut.txt
readlink shortcut.txt
```

GNU `chmod` no cambia los permisos del propio enlace simbólico en Linux; al recibir un enlace como operando normalmente actúa sobre su destino. La combinación de enlaces y opciones recursivas merece una comprobación previa en `man chmod` o `man chown`.

## Ejecutar con otra identidad

### `su`

`su` inicia una shell o ejecuta un comando con la identidad de otro usuario:

```bash
su alice
```

Sin un nombre, el usuario objetivo suele ser `root`:

```bash
su
```

El guion solicita un entorno de inicio de sesión, incluido el directorio personal y los archivos de configuración correspondientes:

```bash
su - alice
```

Para ejecutar un único comando:

```bash
su -c 'id' alice
```

Normalmente `su` autentica con la contraseña de la cuenta objetivo. Al terminar la shell creada se vuelve a la sesión anterior:

```bash
exit
```

### `sudo`

`sudo` permite a un usuario autorizado ejecutar un comando según una política. El usuario objetivo predeterminado suele ser `root`:

```bash
sudo chown alice report.txt
```

Para seleccionar otro usuario:

```bash
sudo -u alice id
```

Comandos útiles:

| Comando | Función |
| --- | --- |
| `sudo -l` | Lista las operaciones autorizadas por la política. |
| `sudo -u USER COMMAND` | Ejecuta un comando como otro usuario. |
| `sudo -i` | Abre una shell de inicio de sesión como el usuario objetivo. |
| `sudo -k` | Invalida las credenciales almacenadas para la sesión. |

`sudo` suele solicitar la contraseña del usuario que invoca el comando, no la de `root`, aunque la política puede configurarse de otro modo.

### Diferencias prácticas

| `su` | `sudo` |
| --- | --- |
| Cambia de identidad y suele abrir una shell. | Ejecuta una operación autorizada por una política. |
| Normalmente autentica al usuario objetivo. | Normalmente autentica al usuario que lo invoca. |
| Una shell abierta permite ejecutar varias órdenes. | Favorece elevar solo la orden necesaria. |

Debe preferirse el mínimo privilegio necesario. Una shell administrativa prolongada aumenta el impacto de errores de escritura, rutas y comodines.

`cd` es un builtin que modifica la shell actual. Ejecutar `sudo cd /path` no puede cambiar el directorio de la shell que ya está abierta; para trabajar con otro entorno se necesita una shell autorizada o ejecutar un comando concreto dentro de la ruta apropiada.

## Crear usuarios y grupos en Ubuntu

Estas operaciones modifican la configuración del sistema y requieren privilegios administrativos.

### Herramientas de alto nivel

En Ubuntu, `adduser` y `addgroup` son interfaces orientadas a la administración interactiva. Aplican valores predeterminados de Debian y Ubuntu y simplifican la creación del directorio personal y otros datos.

```bash
sudo adduser alice
sudo addgroup developers
sudo adduser alice developers
```

La tercera forma agrega un usuario existente a un grupo existente.

### Herramientas de bajo nivel

`useradd` y `groupadd` ofrecen control más directo y son habituales en automatización:

```bash
sudo groupadd developers
sudo useradd -m -s /bin/bash alice
```

Opciones comunes de `useradd`:

| Opción | Función |
| :---: | --- |
| `-m` | Crea el directorio personal. |
| `-s SHELL` | Define la shell de inicio de sesión. |
| `-g GROUP` | Define el grupo primario. |
| `-G LIST` | Define grupos suplementarios. |
| `-r` | Crea una cuenta de sistema. |

Para agregar una cuenta existente a un grupo suplementario también se usa:

```bash
sudo usermod -aG developers alice
```

`-a` es esencial junto con `-G`: sin ella, la lista de grupos suplementarios puede reemplazarse.

Las nuevas pertenencias a grupos no siempre aparecen en procesos que ya estaban en ejecución. El usuario suele necesitar cerrar la sesión y volver a iniciarla. `newgrp` puede iniciar una shell con otro grupo efectivo, pero no reemplaza la comprobación en una sesión nueva.

La información local de cuentas y grupos se almacena tradicionalmente en `/etc/passwd`, `/etc/shadow`, `/etc/group` y `/etc/gshadow`. Deben gestionarse con herramientas administrativas, no mediante edición improvisada.

## Bits especiales

Además de `rwx`, existen tres bits especiales. Se representan como un cuarto dígito situado a la izquierda:

| Bit | Valor | Uso principal |
| --- | :---: | --- |
| `setuid` | 4 | Un ejecutable actúa con el UID efectivo de su propietario. |
| `setgid` | 2 | Un ejecutable actúa con el GID efectivo de su grupo; en directorios, propaga el grupo. |
| `sticky` | 1 | En un directorio compartido, restringe quién puede eliminar o renombrar entradas. |

Ejemplos:

```bash
chmod 2775 shared_dir
chmod 1777 dropbox_dir
```

En un directorio con `setgid`, los nuevos objetos heredan normalmente el grupo del directorio, lo que facilita el trabajo colaborativo. En un directorio con *sticky bit*, aunque varios usuarios puedan escribir, cada uno no puede eliminar libremente archivos ajenos. `/tmp` suele utilizar el modo `1777`.

En la forma simbólica se usan `s` y `t`:

```bash
chmod g+s shared_dir
chmod +t dropbox_dir
```

`ls -l` puede mostrar `s`, `S`, `t` o `T` en posiciones de ejecución. Una mayúscula indica que el bit especial está activo pero el bit de ejecución correspondiente no lo está.

El bit `setuid` en scripts se ignora normalmente en Linux por razones de seguridad. Los bits especiales no deben aplicarse sin comprender el programa, el propietario y el entorno donde se ejecutará.

## ACL y otros controles

Los bits clásicos no siempre cuentan toda la historia. Una ACL puede conceder permisos a usuarios o grupos adicionales. `ls -l` suele mostrar un `+` al final del modo cuando existen entradas de ACL:

```text
-rw-r-----+ 1 alice developers 2048 Sep 12 10:30 report.txt
```

Para inspeccionarlas, si las herramientas están instaladas:

```bash
getfacl report.txt
```

También pueden influir:

- permisos de todos los directorios de la ruta;
- sistemas de archivos montados con `noexec` o `ro`;
- atributos extendidos o inmutables;
- políticas como AppArmor o SELinux;
- permisos de red en sistemas de archivos remotos.

Por eso, `chmod 777` no garantiza que una operación funcione y puede crear una exposición innecesaria.

## Diagnosticar `Permission denied`

Una revisión ordenada evita cambiar permisos al azar.

### Confirmar la identidad

```bash
id
whoami
```

### Inspeccionar el objeto

```bash
ls -ld path
stat -c '%A %a %U %G %n' path
```

### Inspeccionar toda la ruta

Si está disponible, `namei` muestra cada componente:

```bash
namei -l /srv/app/config/settings.ini
```

Debe existir permiso de búsqueda `x` en cada directorio padre.

### Distinguir la operación

Preguntas útiles:

- ¿Se intenta leer, modificar o ejecutar un archivo?
- ¿Se intenta crear, eliminar o renombrar una entrada de directorio?
- ¿El operando es un enlace o su destino?
- ¿El proceso coincide con el propietario, el grupo o la clase de otros?
- ¿Hay ACL, una montura de solo lectura o una política adicional?

### Revisar el formato ejecutable

```bash
file script.sh
head -n 1 script.sh
```

Un script puede tener `x` y aun así fallar por un intérprete inexistente, finales de línea incorrectos o una ruta de *shebang* inválida.

## Hábitos seguros

- Usar `stat` o `ls -ld` antes y después de una modificación.
- Preferir cambios simbólicos relativos cuando sea importante conservar permisos existentes.
- Usar modos numéricos cuando se necesite un estado final exacto.
- Evitar permisos globales de escritura salvo que exista una razón y controles adicionales.
- Revisar cuidadosamente rutas, comodines y nombres ocultos antes de usar `-R`.
- Utilizar `sudo` solo para la operación que realmente requiere privilegios.
- Añadir `--` antes de operandos que puedan comenzar con `-`.
- Consultar la documentación instalada, porque las opciones pueden variar entre versiones.

## Consultar la documentación

Las páginas de manual reflejan las herramientas instaladas en el sistema:

```bash
man chmod
man chown
man chgrp
man id
man groups
man whoami
man sudo
man su
man adduser
man useradd
man addgroup
man groupadd
```

También puede consultarse una ayuda breve:

```bash
chmod --help
chown --help
id --help
```

Dentro de `man`, `/pattern` busca texto, `n` repite la búsqueda y `q` sale.

## Recursos

### Introducción

- [Learning the Shell: Permissions](https://linuxcommand.org/lc3_lts0090.php)

### Documentación de GNU Coreutils

- [GNU Coreutils: `chmod`](https://www.gnu.org/software/coreutils/manual/html_node/chmod-invocation.html)
- [GNU Coreutils: `chown`](https://www.gnu.org/software/coreutils/manual/html_node/chown-invocation.html)
- [GNU Coreutils: `id`](https://www.gnu.org/software/coreutils/manual/html_node/id-invocation.html)
- [GNU Coreutils: `groups`](https://www.gnu.org/software/coreutils/manual/html_node/groups-invocation.html)
- [GNU Coreutils: `whoami`](https://www.gnu.org/software/coreutils/manual/html_node/whoami-invocation.html)

### Manuales de Ubuntu 22.04 LTS

- [Ubuntu Manpage: `chmod`](https://manpages.ubuntu.com/manpages/jammy/en/man1/chmod.1.html)
- [Ubuntu Manpage: `chown`](https://manpages.ubuntu.com/manpages/jammy/en/man1/chown.1.html)
- [Ubuntu Manpage: `sudo`](https://manpages.ubuntu.com/manpages/jammy/en/man8/sudo.8.html)
- [Ubuntu Manpage: `su`](https://manpages.ubuntu.com/manpages/jammy/en/man1/su.1.html)
- [Ubuntu Manpage: `adduser` y `addgroup`](https://manpages.ubuntu.com/manpages/jammy/en/man8/adduser.8.html)
- [Ubuntu Manpage: `useradd`](https://manpages.ubuntu.com/manpages/jammy/en/man8/useradd.8.html)
- [Ubuntu Manpage: `groupadd`](https://manpages.ubuntu.com/manpages/jammy/en/man8/groupadd.8.html)

## Conclusión

Los permisos de Linux combinan la identidad efectiva del proceso, la propiedad del objeto y un modo dividido entre propietario, grupo y otros. Comprender cómo se selecciona una clase y cómo cambia el significado de `rwx` entre archivos y directorios permite razonar sobre el acceso en lugar de probar números al azar.

`chmod` modifica el modo, `chown` y `chgrp` administran la propiedad, y `sudo` o `su` cambian la identidad con la que se ejecutan operaciones. La notación simbólica, la octal, `umask`, los enlaces y los bits especiales forman partes distintas de un mismo sistema de control de acceso.
