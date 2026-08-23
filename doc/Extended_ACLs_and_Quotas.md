#  ACLs Extendidas y Quotas 

Vamos directo al grano con una clase rápida, técnica y al pie sobre **ACLs Extendidas (Access Control Lists)** y **Cuotas de Disco (Disk Quotas)** en entornos Red Hat (RHEL/CentOS/Fedora).

Como ya manejas las bases de Linux, nos saltaremos los permisos tradicionales (`rwx` para usuario, grupo y otros) y veremos cómo superar sus limitaciones.

## 1. ACLs Extendidas (Access Control Lists)

Por defecto, los permisos POSIX tradicionales son limitados: solo puedes asignarle un dueño, un grupo y "el resto". ¿Qué pasa si necesitas que un usuario específico (`desarrollador3`) tenga acceso de lectura y escritura a una carpeta, sin meterlo al grupo principal ni darle permisos a todo el mundo? Ahí entran las ACLs.

### Conceptos Clave

- **ACL de Propietario / Grupo / Otros:** Coinciden con los permisos tradicionales.
- **ACLs de Usuario y Grupo Específicos:** Permiten afinar permisos para usuarios o grupos puntuales más allá del propietario.
- **Máscara (Mask):** Define el **límite máximo** de permisos que pueden conceder las ACLs (excepto para el usuario dueño y los "otros"). Si la máscara no permite escritura, ninguna ACL de grupo/usuario podrá escribir, aunque su permiso individual diga lo contrario.
- **Heredadas (Default ACLs):** Se aplican a un directorio para que todos los archivos y carpetas que se creen dentro hereden automáticamente esas reglas.

### Comandos Esenciales

- **Ver ACLs:**

  ```bash
  getfacl /ruta/al/archivo_o_directorio
  ```
  
- **Configurar una ACL (Modificar/Añadir):**

  - Para un usuario: `setfacl -m u:nombre_usuario:rwx /ruta`
  - Para un grupo: `setfacl -m g:nombre_grupo:rx /ruta`

- **Configurar una ACL Heredada (Default) en un directorio:**

  ```
  setfacl -m d:u:nombre_usuario:rwx /ruta_directorio
  ```
  
- **Eliminar una ACL específica:**

  ```
  setfacl -x u:nombre_usuario /ruta
  ```
  
- **Eliminar todas las ACLs:**

  ```
  setfacl -b /ruta
  ```

## 2. Quotas de Disco (Disk Quotas)

Las cuotas permiten limitar el uso de almacenamiento por **usuario** o por **grupo** en un sistema de archivos específico. En RHEL moderno, esto se gestiona a nivel de kernel mediante el sistema de archivos (generalmente XFS, que viene por defecto).

### Conceptos Clave

- **Soft Limit (Límite blando):** Un umbral que el usuario puede superar temporalmente. Al superarlo, recibe advertencias y se activa un **periodo de gracia** (por defecto, 7 días en XFS). Si no reduce su consumo antes de que acabe el tiempo, el límite blando se convierte en duro.
- **Hard Limit (Límite duro):** El tope absoluto. El usuario **jamás** podrá rebasarlo. Si llega a este límite, el sistema le negará la creación de más archivos o escritura.
- **Inodos vs. Bloques:** Puedes limitar tanto el espacio en disco (bloques/megabytes) como la **cantidad de archivos** que puede crear el usuario (inodos), ideal para evitar que llenen la partición con millones de archivos pequeños.

### Comandos Esenciales

- **Ver el estado y uso de cuotas:**

  ```bash
  quotaon -v /punto/de/montaje
  repquota -avs
  ```

- **Editar la cuota de un usuario específico:**

```bash
edquota nombre_usuario
```

#### (Esto abrirá un editor de texto donde defines los límites de bloques e inodos para los soft y hard limits).*



- **Ver las cuotas de un usuario:**

  ```bash
  quota nombre_usuario
  ```



## Tarea 1: Habilitar Quotas a los usuarios en  `/data/apps`



```bash
# Chequear si esta habilitado la quota en la particion
 quotaon -v /data/apps
```

**Advertencia:**


```bash
quotaon: Mountpoint (or device) /data/apps not found or has no quota enabled.
```

Ese error es un clásico en Red Hat cuando intentas activar las cuotas antes de configurar el sistema de archivos. Te explico por qué pasa y cómo solucionarlo paso a paso.

El comando `quotaon` te avisa que `/data/apps` no tiene cuotas habilitadas porque **falta un paso fundamental**: indicarle al kernel que active el soporte para cuotas (inodos y/o bloques) en las opciones de montaje de ese punto de montaje, y luego inicializar los archivos de control.

### Pasos para solucionarlo y habilitar la cuota:

#### 1. Verifica cómo está montado `/data/apps`  (RECOMENDADO)

Ejecuta esto para ver el dispositivo y el sistema de archivos:

```bash
findmnt /data/apps
```

*(Generalmente en RHEL esto es un sistema de archivos `xfs`).*

#### 2. Asegúrate de que las opciones de cuota estén activas

Para XFS, las cuotas de usuario se habilitan añadiendo la opción `usrquota` (y/o `grpquota` para grupos) en el archivo `/etc/fstab`.

1. Abre el archivo:

   ```bash
   sudo nano /etc/fstab
   ```
   
2. Busca la línea correspondiente a `/data/apps` y añade `,usrquota` (o asegúrate de que esté presente en la columna de opciones). Debería verse más o menos así:

   ```bash
   /dev/mapper/vg0-apps  /data/apps  xfs  defaults,usrquota  0  0
   ```
   
3. Guarda y sal del editor.

#### 3. Remonta el sistema de archivos para aplicar los cambios

Para no tener que reiniciar la VM, vuelve a montar la partición aplicando las opciones del `fstab`:

```bash
sudo mount -o remount /data/apps
```

*(Si usas XFS, el sistema de archivos ya está listo para aceptar cuotas en caliente).*

#### 4. Inicializa y enciende las cuotas

Ahora sí, ejecuta el comando de activación:

```bash
sudo quotaon -v /data/apps
```

¡Y listo! Con esto ya deberías poder configurar las cuotas de tus usuarios con `edquota` o `setquota`. ¿Pudiste verificar el `/etc/fstab` o te arroja algún otro detalle al hacer el remount?



> ### Tengo dudas acerca de la particiones ext4. Puedo habilitar quota en este tipo de sistema de archivo?



¡Sí, totalmente! **EXT4 soporta cuotas de disco**, pero el mecanismo es un poco diferente al de XFS.

Mientras que en XFS las cuotas se activan directamente con opciones en el montaje (`usrquota`), en **EXT4** el sistema de archivos necesita inicializar archivos ocultos de metadatos de cuota dentro de la propia partición (`aquota.user` y `aquota.group`) antes de poder encenderlas.

### Cómo habilitar cuotas en EXT4 paso a paso:

#### 1. Edita el `/etc/fstab` con las opciones correctas

Para EXT4, las opciones de montaje típicas para cuotas de usuario y grupo son `usrjquota=aquota.user,grpjquota=aquota.group` especificando el formato `vfsv0`:



```bash
/dev/sdb1  /tu/punto/montaje  ext4  defaults,usrjquota=aquota.user,grpjquota=aquota.group,jqfmt=vfsv0  0  0
```

#### 2. Remonta el sistema de archivos

Aplica los cambios sin reiniciar:

```bash
sudo mount -o remount /tu/punto/montaje
```

#### 3. Crea los archivos de cuota (Paso clave en EXT4)

A diferencia de XFS (que los gestiona internamente el kernel), EXT4 requiere que generes los binarios de control con el comando `quotacheck`:

```bash
sudo quotacheck -cumv /tu/punto/montaje
```

- `-c`: Crea nuevos archivos de cuota.
- `-u`: Revisa cuotas de usuario.
- `-m`: No intentes desmontar el sistema de archivos si está en uso.
- `-v`: Modo verbose (detallado).

#### 4. Enciende las cuotas

Ahora sí, actívalas con normalidad:

```bash
sudo quotaon -v /tu/punto/montaje
```

¿Estás trabajando sobre EXT4 en tu práctica actual o era una duda general para comparar con XFS?



## Tarea 2:  Establecer para el usuario 'ana',  límite soft de 50 MB (bloques), límite hard de 80 MB (bloques).



¡Manos a la obra con la práctica! Para configurar los límites de bloques (espacio en disco) que te piden para la usuaria `ana`, tienes dos formas principales de hacerlo: de forma interactiva con un editor de texto o directamente con un comando de una sola línea (ideal si quieres automatizar o ir rápido).

Aquí tienes ambas opciones:

### Opción A: Usando el comando `setquota` (La más rápida y directa)

Con este comando estableces los límites exactos de bloques (espacio) e inodos (cantidad de archivos) de un solo golpe.

Como el sistema de cuotas suele trabajar en bloques de **1 Kilobyte (1024 bytes)**, para definir 50 MB y 80 MB hacemos el cálculo:

- **50 MB** = $50 \times 1024 \text{ KB} = 50432$ bloques (o redondeado a bloques de 1K: $50 \times 1024 \times 1024 / 1024 = 50688$ ... mejor aún: $50 \times 1024 = 50240$ no, $50 \times 1024 = 51200$ bloques).
- **80 MB** = $80 \times 1024 = 81920$ bloques.

Ejecuta el siguiente comando (ajustando el punto de montaje si no es `/data/apps`):

```bash
sudo setquota -u ana 51200 81920 0 0 /data/apps
```

*Explicación de los parámetros:*

- `-u ana`: Aplica al usuario ana.
- `51200`: Soft limit en bloques (50 MB).
- `81920`: Hard limit en bloques (80 MB).
- Los dos ceros (`0 0`) siguientes son para los límites de *inodos* (archivos), indicando que no se limitará la cantidad de archivos, solo el espacio.
- `/data/apps`: El sistema de archivos afectado.

### Opción B: Usando el editor interactivo `edquota`

Si prefieres verificar visualmente las tablas:

1. Ejecuta el comando para la usuaria:

   ```bash
   sudo edquota -u ana
   ```

2. Se abrirá un archivo en tu editor predeterminado (como `vi` o `nano`) que se verá más o menos así:

   ```bash
   Disk quotas for user ana (uid 1001):
     Filesystem           blocks       soft       hard     inodes     soft     hard
     /dev/mapper/...      1200       0          0          15         0        0
   ```

3. Modifica los ceros (o los valores actuales) en la columna **soft** y **hard** correspondientes a los `blocks` colocando los valores en bloques (puedes usar los mismos de arriba: `51200` y `81920`).

4. Guarda y sal del editor (en nano con `Ctrl+O`, `Enter`, `Ctrl+X`).

### ¿Cómo verificar que quedó bien configurado?

Puedes auditar la cuota de `ana` en cualquier momento ejecutando:

```bash
sudo quota -v ana
```

O bien revisar un resumen general de todas las cuotas activas con:

```bash
sudo repquota -avs
```

¿Pudiste aplicarlo sin problemas o te arroja algún mensaje al configurar los límites?



> ## Existe alguna manera más simple de habilitar quota en particiones ext4?



¡Sí, totalmente! Las versiones modernas de Linux (y por ende Red Hat) soportan **inodos de cuota internos nativos** en EXT4. Esto significa que ya no necesitas lidiar con archivos externos como `aquota.user` ni correr el tedioso comando `quotacheck`.  

El método moderno y limpio para EXT4 se asemeja mucho al comportamiento de XFS:

### El método moderno (Con inodos internos en EXT4)

## (RECOMENDADO)



Si la partición es nueva o la puedes desmontar un momento, puedes activar la característica nativa de cuotas directamente en el sistema de archivos:  

1. **Desmonta el sistema de archivos:**

   ```bash
   sudo umount /ruta/punto/montaje
   ```

2. **Habilita la característica de cuotas internas con `tune2fs`:**

   ```bash
   sudo tune2fs -O quota /ruta/dispositivo
   ```

3. **Modifica tu `/etc/fstab` de forma limpia** (sin necesidad de apuntar a archivos `aquota.user`, solo usando las opciones estándar):  

   ```bash
   /dev/tu_dispositivo  /ruta/punto/montaje  ext4  defaults,usrquota,grpquota  0  2
   ```

4. **Vuelve a montar y enciende las cuotas:**

   ```bash
   sudo mount /ruta/punto/montaje
   sudo quotaon -v /ruta/punto/montaje
   ```

Con esto, el propio kernel gestionará el registro de espacio mediante inodos ocultos internos, ahorrándote la creación manual de los archivos de cuota tradicionales.  



