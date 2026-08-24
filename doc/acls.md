#  ACLs (Access Control Lists)

Es una  excelente iniciativa profundizar en la seguridad de Linux. Como alguien que ya conoce las bases del sistema, ya sabes que los permisos tradicionales de Propietario, Grupo y Otros (`rwx`) se quedan cortos cuando un entorno escala o requiere restricciones de acceso a nivel empresarial.

En el ecosistema Red Hat (RHEL, Fedora, CentOS), la seguridad granular va mucho más allá de los bits tradicionales. Hoy nos enfocaremos en los dos pilares fundamentales que todo administrador moderno debe dominar: **Listas de Control de Acceso (ACLs)** y **SELinux (Security-Enhanced Linux)**.



## 1. ACLs (Access Control Lists): Rompiendo la regla del "Trío"

En un sistema de archivos estándar (ext4, XFS), los permisos se dividen estrictamente en tres categorías: `u` (user), `g` (group) y `o` (others). 

> ¿Qué pasa si necesitas que un usuario específico (por ejemplo, `desarrollador3`) tenga acceso de lectura/escritura a un directorio, sin dárselo a todo su grupo ni hacerle propietario? 

Ahí entran las **ACLs**.

Las ACLs permiten definir permisos específicos para usuarios o grupos individuales de forma granular sobre un archivo o directorio.

### Comandos Esenciales

- **Verificar ACLs actuales:** Utiliza el comando `getfacl`:
  
  ```bash
  getfacl archivo_o_directorio
  ```

  *Nota: Verás que los permisos tradicionales terminan en un signo `+`, indicando que existen ACLs activas.*

- **Establecer una ACL para un usuario:** Usa `setfacl` con la opción `-m` (modify):


  ```bash
  setfacl -m u:desarrollador3:rwX directorio/
  ```

  *(Tip: Usar `X` en mayúscula otorga permisos de ejecución `x` solo a directorios o archivos que ya tienen alguna marca de ejecución, evitando ejecutarlo en archivos planos por error).*

- **Establecer permisos por defecto (Inheritencia):** Para que los archivos nuevos creados dentro de un directorio hereden automáticamente estas reglas:

  
  ```bash
  setfacl -d -m u:desarrollador3:rwX directorio/
  ```

- **Eliminar ACLs:** Para borrar una regla específica o limpiar todas:


  ```bash
  setfacl -x u:desarrollador3 directorio/  # Borra una regla
  setfacl -b directorio/                 # Borra todas las ACLs
  ```

## 2. SELinux: Seguridad basada en Contextos (Mandatory Access Control)

Si vienes de sistemas donde la seguridad tradicional (`DAC`) confía en el usuario root, **SELinux** cambia las reglas del juego. SELinux implementa **MAC (Mandatory Access Control)**. Incluso si un proceso o usuario es `root`, si no tiene el contexto de seguridad permitido, SELinux le denegará el acceso.

Piénsalo como un guardia de seguridad con una lista estricta: no importa quién seas, si tu credencial (contexto) no coincide con la puerta a la que quieres entrar, no pasas.

### Anatomía de un Contexto de SELinux

Cada proceso y archivo en Red Hat tiene una etiqueta de contexto visible con la opción `-Z`:


```bash
ls -lZ /var/www/html/index.html
```

Verás algo como: `system_u:object_r:httpd_sys_content_t:s0`

Los componentes clave son:

1. **Usuario (`system_u`)**: Usuario de SELinux (distinto a los usuarios del sistema operativo).
2. **Rol (`object_r`)**: Define qué roles pueden acceder al objeto.
3. **Tipo (`httpd_sys_content_t`)**: **El más importante.** Define exactamente qué procesos pueden interactuar con este archivo. Por ejemplo, Apache (`httpd`) puede leer este tipo, pero un demonio de FTP podría tener prohibido tocarlo.
4. **Nivel (`s0`)**: Usado para políticas de seguridad multinivel (MLS/MCS).

### Operativa Diaria con SELinux

- **Consultar el estado de SELinux:**

  ```bash
  sestatus
  ```

  *(Modos comunes: `Enforcing` -bloquea y registra-, `Permissive` -solo advierte/registra-, `Disabled` -desactivado).*

- **Cambiar temporalmente de modo (sin reiniciar):**


  ```bash
  setenforce 0  # Cambia a modo Permissive
  setenforce 1  # Cambia a modo Enforcing
  ```

- **Restaurar contextos por defecto (La solución a muchos problemas):** Si mueves archivos de un directorio a otro y dejan de funcionar, a menudo es porque heredaron el contexto incorrecto. Restáuralo con:


  ```bash
  restorecon -Rv /var/www/html/
  ```

- **Modificar contextos de forma permanente:** Si creas una ruta personalizada para un servicio (ej. Nginx en `/miweb/`), debes decirle a SELinux qué tipo usar o clonar el contexto de la ruta original usando `semanage`:

  ```bash
  semanage fcontext -a -t httpd_sys_content_t "/miweb(/.*)?"
  restorecon -Rv /miweb/
  ```

- **Gestionar Booleans (Interruptores de seguridad):** Muchas veces no necesitas cambiar contextos, sino activar permisos predefinidos que vienen bloqueados por defecto (ej. permitir que Apache se conecte a una base de datos remota).

  - Ver estados: `getsebool -a | grep httpd`

  - Cambiar un booleano al vuelo y hacerlo persistente con `-P`:


    ```bash
    setsebool -P httpd_can_network_connect 1
    ```

## 3. Diagnóstico: ¿Qué hacer cuando algo falla?

Cuando implementas seguridad granular, la regla de oro ante un fallo de permisos es revisar dos frentes:

1. **¿Es un problema de permisos tradicionales o ACLs?** Revisa con `ls -la` y `getfacl`.

2. **¿Es SELinux bloqueando la acción?** Revisa los registros de auditoría en busca de denegaciones (*AVC Denied*):


   ```bash
   grep AVC /var/log/audit/audit.log
   ```

   O utiliza la herramienta de análisis automatizado:

   ```bash
   sealert -a /var/log/audit/audit.log
   ```

   *(Te dará una explicación exacta del problema y el comando sugerido para solucionarlo).*

¿Hay algún escenario práctico en tu infraestructura o entorno de pruebas donde quieras aplicar estas capas de seguridad y prefieras que lo analicemos paso a paso?

 > ### Cuál es la configuración por defecto para usuarios no root?

En los sistemas operativos de la familia Red Hat (RHEL, Fedora, Rocky Linux, AlmaLinux), la configuración por defecto para un usuario estándar (no `root`) abarca varios aspectos clave de seguridad y control de acceso.

Aquí tienes el desglose técnico de cómo se configuran por defecto:

## 1. Contexto de SELinux para Usuarios

A diferencia del usuario `root` (que opera bajo un contexto con privilegios administrativos amplios, normalmente `unconfined_u:unconfined_r:unconfined_t:s0`), un usuario estándar no privilegiado está confinado:

- **Usuario SELinux:** Por defecto se asigna al usuario del sistema operativo al mapeo de SELinux `user_u` (o `staff_u` si pertenece al grupo de administradores secundarios).
- **Rol y Dominio:** Operan bajo roles confinados (como `user_r` o `staff_r`) y dominios específicos (`user_t`). Esto significa que, aunque un usuario ejecute aplicaciones o intente realizar acciones maliciosas, las políticas de SELinux limitan estrictamente su capacidad para alterar archivos del sistema, leer información de otros usuarios o abrir puertos de red no autorizados, incluso si el usuario tiene permisos estándar en el sistema de archivos.

## 2. Privilegios y Capacidad de Elevación (`sudo`)

Por defecto, al crear un nuevo usuario mediante herramientas estándar como `useradd`:

- **Sin privilegios directos:** El usuario **no** pertenece al grupo `wheel` (el grupo encargado de otorgar privilegios administrativos a través de `sudo` en Red Hat).
- **Restricción de comandos:** No puede ejecutar comandos administrativos (`systemctl`, gestión de paquetes `dnf`, modificación de redes, etc.). Si intenta usar `sudo`, el sistema denegará la acción y por defecto registrará un intento de seguridad en `/var/log/secure`.

## 3. Permisos en el Sistema de Archivos y Umask

Cuando un usuario no root interactúa con archivos y directorios:

- **Máscara de creación (Umask):** El umask por defecto para usuarios estándar en Red Hat es **`0022`** (o en distribuciones más modernas basada en perfiles de shell, a veces implementado a través de *PAM* con un umask privado por usuario de `0077` en configuraciones más estrictas).
  - Esto significa que al crear un **archivo**, los permisos por defecto son `644` (`rw-r--r--`).
  - Al crear un **directorio**, los permisos por defecto son `755` (`rwxr-xr-x`).
- **Aislamiento del Home:** El directorio personal se crea bajo `/home/nombre_de_usuario/` con permisos estrictos de `700` (`rwx------`), asegurando que ningún otro usuario del sistema (fuera de `root`) pueda listar su contenido o acceder a sus archivos privados.

## 4. Recursos del Sistema (Límites con PAM / Systemd)

Los usuarios no root están sujetos a límites estrictos de recursos para evitar ataques de denegación de servicio (*DoS*) locales (como bombas de forks o consumo excesivo de memoria):

- **Controladores de recursos (cgroups):** Gestionados automáticamente por `systemd-logind`. Cada sesión de usuario tiene restricciones predeterminadas sobre el número máximo de procesos que puede ejecutar y la cantidad de memoria o CPU concurrente, configuradas en `/etc/security/limits.conf` o mediante perfiles dinámicos en `/etc/systemd/logind.conf`.



> ## En este contexto, existen preguntas asociadas en el examen RHCSA?



¡Exacto! El examen **RHCSA (EX200)** es 100% práctico y basado en desempeño. No es un examen de opción múltiple; se te presenta un entorno virtual real donde debes resolver problemas bajo presión de tiempo. La seguridad granular, específicamente **ACLs y SELinux**, es un área crítica que suele determinar si un estudiante aprueba o suspende.  

Aquí te detallo qué esperar en el examen respecto a estos temas:

### 1. ACLs (Listas de Control de Acceso)

El examen suele pedirte que resuelvas necesidades de colaboración donde los permisos estándar (`rwx` básico) fallan.

- **Tipo de pregunta:** "Configure el directorio `/data/proyectos` para que el grupo `desarrollo` tenga acceso de lectura y escritura, pero otros usuarios no tengan acceso alguno. Además, cualquier archivo nuevo creado dentro de ese directorio debe pertenecer automáticamente al grupo `desarrollo`".  
- **Lo que debes dominar:**
  - Uso fluido de `setfacl` para usuarios y grupos.  
  - Configuración de ACLs **por defecto** (`-d`) para herencia.
  - Verificación rápida con `getfacl`.  

### 2. SELinux (El punto donde más gente falla)

SELinux es infame en el examen porque una mala configuración puede dejar servicios (como Apache o Nginx) inoperativos, lo cual te resta puntos automáticamente en la evaluación.  

- **Tipo de pregunta:** "Se ha movido un sitio web a una ubicación personalizada `/webdata`. Configure SELinux para que el servicio web pueda leer el contenido, de modo que el cambio sobreviva a un relabel del sistema".
- **Lo que debes dominar:**  
  - **Contextos:** Diferencia entre `chcon` (temporal, no recomendado para el examen porque no sobrevive a reinicios) y `semanage fcontext` + `restorecon` (la forma correcta y permanente).  
  - **Booleans:** Activación de permisos específicos (ej. permitir que un servidor web haga conexiones de red hacia una base de datos) usando `setsebool -P`.  
  - **Diagnóstico:** Si un servicio no levanta, tu primer instinto debe ser `sealert` o revisar los logs de auditoría para ver si hay una denegación de SELinux.

### Consejos de "Tutor" para el RHCSA

1. **No busques el atajo:** En el examen, no intentes desactivar SELinux (`setenforce 0` o editar `/etc/selinux/config`) para "arreglar" un problema de permisos. Los evaluadores automáticos detectarán que no resolviste el problema de seguridad y perderás los puntos. **Debes aprender a vivir con SELinux en `Enforcing`**.
2. **La persistencia es clave:** Muchas tareas te pedirán que el cambio sea persistente. Si usas `setsebool` sin la bandera `-P`, el cambio se perderá al reiniciar y el sistema de evaluación automática te marcará el ejercicio como incorrecto.  
3. **Muscle Memory:** Como no tendrás acceso a Internet para buscar la sintaxis de `semanage`, practica hasta que puedas escribir los comandos de memoria. El examen es cronometrado y no puedes perder 10 minutos buscando cómo aplicar un contexto de SELinux en el manual (`man`).  

**Resumen de objetivos del examen:**

- **ACLs:** Uso de `setfacl` y `getfacl`.  
- **SELinux:** Gestión de modos (`Enforcing`/`Permissive`), contextos (`semanage fcontext`, `restorecon`) y booleanos (`getsebool`, `setsebool`).  

¿Te gustaría que practiquemos un ejercicio de laboratorio tipo examen? Puedo proponerte un escenario y tú intentas resolverlo.

Vamos a construir una base sólida con un par de ejercicios rápidos y guiados, enfocados puramente en la práctica manual en tu propia terminal (si tienes una máquina virtual o contenedor a la mano).

### Ejercicio 1: Entendiendo ACLs paso a paso

Imagina que en tu sistema creas un directorio compartido para un proyecto, pero quieres que un usuario específico tenga acceso total sin alterar los permisos tradicionales del grupo o de otros.

**1. Prepara el escenario:** Abre tu terminal y crea un directorio de prueba:

```bash
sudo mkdir -p /srv/proyectos/alpha
sudo groupadd equipo-alpha
```

**2. El reto:** Aplica una ACL para que un usuario (por ejemplo, `testuser`, o uno que ya tengas creado) tenga permisos de lectura, escritura y ejecución (`rwx`) sobre `/srv/proyectos/alpha`, y asegúrate de que **cualquier archivo nuevo que se cree allí dentro herede automáticamente esa regla**.

*Pista: Necesitarás usar `setfacl` dos veces: una para el directorio actual y otra agregando la bandera de herencia por defecto (`-d`).*

¿Te animas a escribir en tu terminal los comandos que usarías para este escenario o prefieres que lo revisemos juntos?

Para resolver este escenario paso a paso, aquí tienes exactamente cómo se ejecuta en la terminal.

Primero, supongamos que el usuario al que quieres darle acceso se llama `operador1` (puedes cambiarlo por cualquier usuario real de tu sistema).

### 1. Aplicar la ACL al directorio actual

Para otorgarle permisos de lectura, escritura y ejecución (`rwx`) de forma inmediata sobre la carpeta existente, usamos el modificador `-m` (modify) apuntando al usuario `u:`:

```bash
sudo setfacl -m u:operador1:rwx /srv/proyectos/alpha
```

### 2. Aplicar la ACL por defecto (Herencia)

Para asegurarnos de que **todos los archivos y subdirectorios nuevos** que se creen dentro de `/srv/proyectos/alpha` hereden automáticamente esta misma regla para `operador1`, agregamos la bandera de directorio por defecto (`-d`):


```bash
sudo setfacl -d -m u:operador1:rwx /srv/proyectos/alpha
```

### 3. ¿Cómo verificamos que quedó bien?

Utilizamos el comando `getfacl` que mencionamos antes:

```bash
getfacl /srv/proyectos/alpha
```

Al ejecutarlo, verás una salida similar a esta:

```plaintext
# file: /srv/proyectos/alpha
# owner: root
# group: root
user::rwx
user:operador1:rwx            # <--- Aquí está la ACL aplicada
group::r-x
mask::rwx
other::r-x
default:user::rwx
default:user:operador1:rwx    # <--- Aquí está la ACL hereditaria configurada
default:group::r-x
default:mask::rwx
default:other::r-x
```

¿Ves cómo aparecen las líneas de `user:operador1` tanto para el directorio actual como en el bloque `default:`? Eso garantiza que la seguridad granular está activa y lista para los archivos futuros.

¿Tiene sentido la lógica de separar el permiso inmediato (`-m`) del permiso de herencia (`-d -m`)? Si quieres, podemos probar con un ejercicio de SELinux para redondear el panorama.

---

Vamos entonces con el segundo pilar fundamental del que hablamos: **SELinux**.

Imagina este escenario clásico de examen RHCSA o de troubleshooting real en producción:

### El Escenario: El misterio del directorio web personalizado

Has decidido alojar una nueva aplicación web corporativa, pero en lugar de usar la ruta por defecto de Apache (`/var/www/html`), la has colocado en un directorio totalmente nuevo creado por ti: **`/srv/web/miapp`**.

1. Creas el directorio y un archivo `index.html`:

   ```bash
   sudo mkdir -p /srv/web/miapp
   echo "Hola Mundo Red Hat" | sudo tee /srv/web/miapp/index.html
   ```

2. Configuras tu servidor web (o simulas el acceso) y notas que, a pesar de que los permisos tradicionales de Linux (`chmod`/`chown`) están correctos y el usuario de Apache puede leer la carpeta, **SELinux está bloqueando el acceso** y la página arroja un error **403 Forbidden** (o el log de auditoría registra una denegación AVC).

### Tu Reto:

Basándote en lo que revisamos sobre cómo corregir contextos de SELinux de forma permanente:

1. ¿Qué comando usarías para averiguar **qué contexto actual** tiene asignado `/srv/web/miapp` en comparación con el que debería tener (por ejemplo, el tipo estándar de contenido web de Apache, que es `httpd_sys_content_t`)?
2. ¿Qué secuencia de dos comandos utilizarías para **cambiar el contexto de forma permanente** para que sobreviva a un reinicio o un `restorecon`, aplicando la regla a todo lo que esté dentro de `/srv/web/miapp`?

Aquí tienes la solución exacta paso a paso para resolver este reto de SELinux como un profesional en el examen RHCSA:

### 1. Para ver el contexto actual:

Usas el comando `ls` con la bandera `-Z`:

```bash
ls -ldZ /srv/web/miapp
```

*(Probablemente verás que tiene un contexto genérico como `default_t` o `usr_t`, el cual Apache tiene prohibido leer).*

### 2. Para cambiar el contexto de forma permanente (los dos comandos clave):

Para que la regla se guarde en la base de datos de SELinux de manera permanente y afecte tanto al directorio actual como a su contenido futuro, utilizas `semanage fcontext` combinado con `restorecon`:

```bash
# Paso A: Registrar la regla permanente en la política de SELinux
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/web/miapp(/.*)?"

# Paso B: Aplicar (restaurar) el contexto inmediatamente al directorio y su contenido
sudo restorecon -Rv /srv/web/miapp
```

**¿Por qué esta estructura?**

- El sufrijo `(/.*)?` en la ruta es vital: le indica a SELinux que la regla aplique tanto al directorio principal `/srv/web/miapp` como a **todos** los archivos y subdirectorios que estén o lleguen a estar dentro de él.
- `semanage` actualiza la base de datos (asegurando que sobreviva a un reinicio o a un `restorecon` futuro).
- `restorecon -Rv` (Recursivo y Verbose) aplica físicamente la etiqueta en el sistema de archivos en ese preciso instante.

¿Ves cómo se conectan las piezas? ¿Te queda alguna duda sobre este proceso o quieres que pongamos a prueba otro escenario (como los booleanos de SELinux)?

