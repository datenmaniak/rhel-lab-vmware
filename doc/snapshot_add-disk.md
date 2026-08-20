# Configuración de Snapshot y Almacenamiento en VMware ESXi 8.0

## 1. Crear un Snapshot de la VM `alma-srv-01`

Un *snapshot* (instantánea) guarda el estado actual y el almacenamiento de la máquina virtual para que puedas restaurarla a este punto limpio tras realizar las prácticas del examen RHCSA.

### Paso a paso:

1. **Acceso a la interfaz web (vSphere Host Client):**
   - Abre tu navegador web en Windows 11 e ingresa la dirección IP de tu servidor ESXi (ejemplo: `[https://192.168.](https://192.168.)x.x`).
   - Inicia sesión con el usuario `root` y su contraseña.
2. **Navegar a la Máquina Virtual:**
   - En el menú lateral izquierdo, haz clic en **Virtual Machines** (Máquinas virtuales).
   - Selecciona o ingresa a la VM `alma-srv-01`.
3. **Tomar la instantánea:**
   - Haz clic derecho sobre la VM `alma-srv-01` (o utiliza el menú desplegable de acciones superiores) y selecciona **Snapshots** > **Take snapshot** (Tomar instantánea).
   - **Nombre:** Asigna un nombre descriptivo, por ejemplo: `Pre-EX200-EstadoLimpio`.
   - **Descripción (Opcional):** "Estado base funcional previo a los ejercicios del examen RHCSA".
   - **Opciones de memoria:**
     - Si la VM está **encendida**, puedes desmarcar la opción *Include virtual machine's memory* (Incluir la memoria de la máquina virtual) si deseas un proceso más rápido y un tamaño de snapshot menor, aunque apagar la VM antes de tomar el snapshot es la opción más recomendada.
   - Haz clic en **Take snapshot**.
4. **Verificación:**
   - Revisa el panel inferior de **Recent Tasks** (Tareas recientes) para asegurarte de que la tarea finalice correctamente al 100%.

## 2. Agregar un disco virtual de 20 GB a la VM `alma-srv-01`

Para simular escenarios del RHCSA (como creación de particiones, volúmenes LVM, VDO o Stratis), la VM requerirá un disco secundario limpio de 20 GB.

### Paso a paso:

1. **Editar la configuración de la VM:**

   - En el menú de **Virtual Machines**, haz clic derecho en `alma-srv-01` y selecciona **Edit settings** (Editar configuración).

2. **Añadir el nuevo disco:**

   - En la parte superior de la ventana de configuración, haz clic en el botón **Add hard disk** (Agregar disco duro) y selecciona **New hard disk** (Nuevo disco duro).

3. **Configurar el disco:**

   - Se desplegará una nueva fila llamada **New Hard Disk**.
   - Cambia el tamaño a **20 GB**.
   - Expande las opciones del nuevo disco haciendo clic en la flecha adjunta para definir el aprovisionamiento:
     - **Thin Provision:** Ocupará espacio en el Datastore a medida que se vaya escribiendo datos. Es la opción recomendada para entornos de pruebas en laboratorio sobre VMware Workstation.

4. **Guardar los cambios:**

   - Haz clic en **Save** (Guardar).

5. **Verificación dentro del Sistema Operativo (AlmaLinux):**

   - Accede a la consola de `alma-srv-01` o mediante conexión SSH.

   - Ejecuta el comando:

     ```
     lsblk
     ```

   - Verás un nuevo dispositivo de almacenamiento (habitualmente `/dev/sdb` o `/dev/nvme0n2`, dependiendo del controlador virtual asignado), listo para ser administrado en tus prácticas del EX200.

## Recomendaciones

- **Estado de la VM antes de la instantánea:** Para garantizar consistencia completa del sistema de archivos, es preferible **apagar la VM (`power off`)** antes de tomar el snapshot inicial.
- **Uso prudente de los Snapshots:** Los snapshots en VMware están diseñados para ser temporales. Mantener snapshots abiertos durante mucho tiempo o realizar continuas escrituras sobre ellos puede comprometer el rendimiento del disco e incrementar el uso de espacio en el Datastore.
- **Orden de las especificaciones:** Si agregas el disco de 20 GB *antes* de tomar el snapshot, tu punto de restauración ya incluirá el segundo disco limpio adjunto, ahorrándote tener que volver a vincularlo en caso de revertir el estado.

## Alternativas

1. **Añadir almacenamiento a nivel de VMware Workstation (Anidamiento/Nested):**
   - Si el Datastore local de ESXi se queda sin capacidad, puedes apagar la VM de ESXi en VMware Workstation Pro, editar la configuración de la VM de ESXi y agregarle un segundo disco virtual desde la interfaz de Workstation. Posteriormente, dentro de ESXi, creas o extiendes un **Datastore VMFS** con ese nuevo disco.
2. **Uso de discos independientes en la VM:**
   - En las opciones avanzadas del disco de 20 GB dentro de ESXi, puedes marcar el modo de disco como **Independent - Persistent**. Esto evita que el disco sea afectado cuando reviertas un snapshot de la VM, lo cual es útil si deseas conservar datos o volúmenes de prueba de manera independiente a los estados del sistema operativo principal.