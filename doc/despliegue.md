# Implementación de Lab  Red Hat + VMware

> Manos a la obra!  
>
> - Pues, ya tengo instalado Workstation Pro y descargada la imagen VMware-VMvisor-Installer-8.0U3e-24677879.x86_64.iso

Perfecto. Vamos a construir esto con precisión de ingeniería. No daremos pasos al tuntún; cada configuración tiene un propósito técnico que te servirá cuando estés en una entrevista o en producción.



## Primeros pasos

------

## FASE 0: Verificación de pre-vuelo (No omitas esto)

Antes de crear nada, confirma que tu hardware está listo. Si falla aquí, todo lo demás falla.

### 1. Verifica que la virtualización anidada está soportada

En tu Ryzen con Windows, abre **PowerShell como Administrador** y ejecuta:

```powershell
systeminfo.exe
```

Busca al final de la salida estas líneas:

```plain
Hyper-V Requirements:      VM Monitor Mode Extensions: Yes
                           Virtualization Enabled In Firmware: Yes
                           Second Level Address Translation: Yes
```

Si dice **"No"** en cualquiera, entra al BIOS del Ryzen y habilita:

- **SVM Mode** (Secure Virtual Machine) → **Enabled**
- **NX Mode** → **Enabled**

Guarda, reinicia, y verifica de nuevo.

### 2. Verifica espacio en disco

Asegúrate de que tu SSD tenga al menos **150 GB libres**. Usaremos thin provisioning, pero necesitas espacio para crecer.

------

## FASE 1: Crear la VM anfitriona para ESXi 8.0U3e

Abre **VMware Workstation Pro**. No uses "Typical"; usaremos configuración manual.

### Paso 1.1: Nueva VM Custom

1. **File > New Virtual Machine**
2. Selecciona **Custom (advanced)** → **Next**
3. **Hardware compatibility:** Selecciona **Workstation 17.6.x** (o la más alta disponible) → **Next**

### Paso 1.2: Selección de Guest OS

1. Selecciona **I will install the operating system later** → **Next**
2. **Guest Operating System:** Selecciona **VMware ESX**
3. **Version:** Selecciona **VMware ESXi 8.x** → **Next**

> **Nota del tutor:** Esto le dice a Workstation que optimice la VM para un hypervisor, no para un SO convencional.

### Paso 1.3: Ubicación y nombre

1. **Virtual machine name:** `ESXi-Lab-01`
2. **Location:** Elige una carpeta en tu SSD (ej: `D:\VMs\ESXi-Lab-01`) → **Next**

### Paso 1.4: Procesadores (CRÍTICO)

1. **Number of processors:** `1`
2. **Number of cores per processor:** `2` → **Next**

> No necesitas más de 2 cores para ESXi anidado. Cada vCPU que asignas es un core que dejas de tener para Windows.

### Paso 1.5: Memoria (CRÍTICO)

1. **Memory:** Asigna **6144 MB** (6 GB) → **Next**

> Con 16 GB totales, 6 GB para ESXi es el máximo razonable. Esto te deja ~9-10 GB para Windows y Workstation. Si ves que Windows se ahoga, baja esto a 5120 MB (5 GB) más adelante.

### Paso 1.6: Red

1. **Network connection:** Selecciona **Use bridged networking** → **Next**

> Bridged le da a ESXi una IP real en tu red doméstica, accesible desde tu laptop y desde el mismo Ryzen.

### Paso 1.7: Controlador I/O

1. Selecciona **LSI Logic** (default) → **Next**

### Paso 1.8: Disco virtual (Sistema ESXi)

1. Selecciona **Create a new virtual disk** → **Next**
2. **Maximum disk size:** `20.0` GB
3. Selecciona **Store virtual disk as a single file** → **Next**
4. **Disk file:** `ESXi-Lab-01.vmdk` → **Next**

### Paso 1.9: Finalizar creación básica

1. Click en **Finish**. No enciendas la VM todavía.

------

## FASE 2: Configuración avanzada de la VM (Lo que marca la diferencia)

Antes de encender, debemos ajustar hardware crítico. Selecciona la VM `ESXi-Lab-01` en la biblioteca de Workstation y click en **Edit virtual machine settings**.

### Paso 2.1: Habilitar Virtualización Anidada (OBLIGATORIO)

1. Ve a la pestaña **Options**
2. Selecciona **Advanced**
3. Marca **Disable side channel mitigations** → Esto mejora rendimiento en labs.
4. Ve a la pestaña **Hardware**
5. Selecciona **Processors**
6. Expande la sección **Virtualization engine**
7. Marca **Virtualize Intel VT-x/EPT or AMD-V/RVI** → **ESTO ES OBLIGATORIO.**
8. Marca **Virtualize CPU performance counters** → Opcional pero recomendado para monitoreo realista.

> Sin el paso 7, ESXi anidado no podrá arrancar VMs dentro de sí. Es el equivalente a habilitar SVM en BIOS, pero para la VM.

### Paso 2.2: Añadir el segundo disco (Datastore VMFS)

1. Click en **Add...** (abajo)
2. Selecciona **Hard Disk** → **Next**
3. **SCSI** (recommended) → **Next**
4. **Create a new virtual disk** → **Next**
5. **Maximum disk size:** `100.0` GB
6. Selecciona **Store virtual disk as a single file**
7. **Disk file:** `ESXi-Lab-01-Datastore.vmdk` → **Finish**

> Este segundo disco simulará tu datastore VMFS. Las VMs de AlmaLinux vivirán aquí.

### Paso 2.3: Añadir segundo adaptador de red

1. Click en **Add...**
2. Selecciona **Network Adapter** → **Finish**
3. Selecciona el nuevo **Network Adapter 2** en la lista de hardware
4. A la derecha, selecciona **Bridged** y marca **Connect at power on**
5. Click en **Advanced...** y cambia **MAC Address** a **Generate new MAC address** (para diferenciarla de la primera)

> Dos NICs en ESXi te permiten practicar vmkernel separation: una para management, otra para VMs (o vMotion, aunque no hagamos cluster real).

### Paso 2.4: Montar la ISO de ESXi

1. Selecciona **CD/DVD (SATA)** en la lista de hardware
2. A la derecha, selecciona **Use ISO image file:**
3. Browse y selecciona tu archivo: `VMware-VMvisor-Installer-8.0U3e-24677879.x86_64.iso`
4. Asegúrate de que **Connect at power on** esté marcado

### Paso 2.5: Remover hardware innecesario

1. Selecciona **Sound Card** → Click **Remove**
2. Selecciona **USB Controller** → Click **Remove** (a menos que quieras pasar USBs a ESXi, que no necesitas ahora)
3. Selecciona **Printer** → Click **Remove**

> Menos hardware virtual = menos overhead = más RAM y CPU para lo que importa.

### Paso 2.6: Guardar configuración

1. Click **OK**

Tu VM ahora debe verse así en hardware:

- Memory: 6 GB
- Processors: 2 cores, Virtualize AMD-V/RVI: ON
- Hard Disk (SCSI): 20 GB
- Hard Disk 2 (SCSI): 100 GB
- CD/DVD: ISO de ESXi 8.0U3e montada
- Network Adapter: Bridged (vmxnet3 implícito)
- Network Adapter 2: Bridged

------

## FASE 3: Instalación de ESXi 8.0U3e

Enciende la VM (`ESXi-Lab-01`). Verás la pantalla de carga de ESXi.

### Paso 3.1: Boot e instalación

1. La VM bootea desde la ISO. Verás el menú de VMware ESXi 8.0 Update 3.
2. Selecciona **ESXi-8.0U3e-standard Installer** (o la opción por defecto) y presiona **Enter**.
3. Espera la carga del kernel (pantalla amarilla/naranja con barra de progreso).

### Paso 3.2: Selección de disco

1. Cuando aparezca **"Select a Disk to Install or Upgrade"**, verás dos discos:
   - `Local ATA Disk (20.00 GB)` ← Este es el disco de sistema
   - `Local ATA Disk (107.37 GB)` ← Este es el datastore (NO lo toques ahora)
2. Selecciona el disco de **20.00 GB** (usa flechas arriba/abajo)
3. Presiona **Enter**

### Paso 3.3: Teclado

1. Selecciona **US Default** → **Enter**

### Paso 3.4: Password de root (CRÍTICO)

1. Ingresa un password fuerte para root. Debe tener:
   - Mínimo 7 caracteres
   - Mayúsculas, minúsculas, números y símbolos
   - Ejemplo válido: `Lab$2026!Esxi`
2. Confirma el password → **Enter**

> **Anótalo en un archivo de texto en tu laptop.** Si pierdes este password, reinstalar es más rápido que recuperar.

### Paso 3.5: Confirmación

1. Presiona **F11** para instalar
2. Espera la instalación (~3-5 minutos en SSD)

### Paso 3.6: Reinicio

1. Cuando termine, verás **"Installation Complete"**. Presiona **Enter** para reiniciar.
2. La VM ejectará la ISO automáticamente (o deberías verificar que el CD se desconectó). Si no, apaga la VM, ve a Settings > CD/DVD y desmarca "Connect at power on", luego enciende.

------

## FASE 4: Configuración de red de ESXi (DCUI)

Al arrancar, verás la **DCUI** (Direct Console User Interface) — la pantalla amarilla/naranja con la IP actual (probablemente por DHCP).

### Paso 4.1: IP estática (Recomendado para labs)

1. Presiona **F2** en la pantalla amarilla
2. Ingresa tu password de root
3. Selecciona **Configure Management Network** → **Enter**
4. Selecciona **Network Adapters** → **Enter**
5. Asegúrate de que **vmnic0** esté seleccionado (debe tener una `X`). Presiona **Enter** para volver.

> vmnic0 es tu primer vmxnet3. vmnic1 es el segundo, lo dejamos para más tarde.

1. Selecciona **IPv4 Configuration** → **Enter**
2. Selecciona **Set static IPv4 address and network configuration** (flechas abajo)
3. Ingresa:
   - **IPv4 Address:** `192.168.1.50` (ajusta a tu red; si tu router es 192.168.0.1, usa 192.168.0.50)
   - **Subnet Mask:** `255.255.255.0`
   - **Default Gateway:** `192.168.1.1` (tu router real)
4. Presiona **Enter** para guardar
5. Selecciona **DNS Configuration** → **Enter**
6. Selecciona **Use the following DNS server addresses and hostname**
7. **Primary DNS Server:** `8.8.8.8` (o la IP de tu router si hace DNS)
8. **Hostname:** `esxi-lab-01.local`
9. Presiona **Enter** para guardar
10. Presiona **Esc** para salir
11. Presiona **Y** para confirmar reinicio de management network

Espera 30 segundos. La pantalla amarilla debe mostrar ahora:

```plain
https://192.168.1.50
```

### Paso 4.2: Prueba de conectividad

Desde tu laptop (o desde Windows en el mismo Ryzen), abre un navegador y ve a:

```plain
https://192.168.1.50
```

Verás una advertencia de certificado (normal en ESXi). Click en **Advanced** > **Proceed to 192.168.1.50 (unsafe)**.

Si ves la pantalla de login del **VMware Host Client**, la instalación base es exitosa. Cierra el navegador por ahora.

------

## FASE 5: Post-instalación en el Host Client

### Paso 5.1: Licencia (Verificación)

1. En el Host Client, loguéate como `root` con tu password.
2. Ve a **Manage > Licensing**
3. Deberías ver:
   - **License:** `XXXXX-XXXXX-XXXXX-XXXXX` (la licencia Free embebida de 8.0U3e)
   - **Product:** `VMware ESXi 8.0`
   - **Expiration:** `Never`

Si dice **Evaluation Mode**, significa que usaste la ISO equivocada. Debe ser la que descargaste: `VMware-VMvisor-Installer-8.0U3e-24677879.x86_64.iso` con licencia embebida.

### Paso 5.2: Crear el Datastore VMFS

1. Ve a **Storage > New datastore**
2. Selecciona **Create new VMFS datastore** → **Next**
3. Selecciona el disco de **~100 GB** (debe aparecer como `Local ATA Disk` o similar)
4. **Name:** `Datastore_Lab_01`
5. **VMFS version:** `VMFS 6` (default)
6. **Partition configuration:** Usa todo el espacio disponible → **Next**
7. **Finish**

Ahora tienes un datastore donde vivirán tus VMs de AlmaLinux.

------

## FASE 6: Preparar AlmaLinux 9 para subir al datastore

Necesitas la ISO de AlmaLinux 9. Si no la tienes, descárgala:

- **AlmaLinux-9.4-x86_64-minimal.iso** (o la última 9.x) desde almalinux.org
- Elige la versión **minimal** (~2 GB) para ahorrar espacio y RAM.

### Paso 6.1: Subir la ISO al datastore ESXi

1. En el Host Client, ve a **Storage**
2. Click en tu datastore `Datastore_Lab_01`
3. Arriba, click en **Datastore browser**
4. Click en **Create directory** → Nombre: `ISOs` → **Create directory**
5. Entra a la carpeta `ISOs`
6. Click en **Upload** (icono de flecha hacia arriba)
7. Selecciona tu ISO de AlmaLinux desde tu disco local
8. Espera a que suba (puede tardar 2-3 minutos dependiendo de tu red)

> Esto simula exactamente lo que haces en producción: tener un repositorio de ISOs en un datastore compartido.

------

## FASE 7: Crear tu primera VM AlmaLinux dentro de ESXi

Ahora crearemos una VM con las especificaciones "enterprise" que mencionarás en entrevistas.

### Paso 7.1: Nueva VM

1. En Host Client, ve a **Virtual Machines > Create / Register VM**
2. **Select creation type:** `Create new virtual machine` → **Next**
3. **Name:** `alma-srv01`
4. **Guest OS family:** `Linux`
5. **Guest OS version:** `Other 5.x or later Linux (64-bit)` (AlmaLinux 9 no aparece específicamente; usa esta o `Red Hat Enterprise Linux 9 (64-bit)` si está disponible)
6. **Next**

### Paso 7.2: Hardware virtual "enterprise"

1. **Datastore:** Selecciona `Datastore_Lab_01`
2. **Customize settings:**
   - **CPU:** `1` vCPU
   - **Cores per socket:** `1`
   - **Memory:** `1024` MB (1 GB) → Para servidor sin GUI es suficiente
   - **Hard disk 1:** `20` GB, **Thin provisioned** → Marca esto explícitamente
   - **SCSI controller:** Cambia de `LSI Logic SAS` a **VMware Paravirtual** (PVSCSI) → Esto es lo que usan en producción para mejor I/O
   - **Network Adapter 1:** Cambia de `E1000e` a **vmxnet3** → Esto es crítico; en producción no usan E1000
   - **CD/DVD Drive 1:** `Datastore ISO file` → Navega a `Datastore_Lab_01/ISOs/AlmaLinux-9.x-minimal.iso`
   - Marca **Connect at power on** en el CD/DVD
3. **Next** → **Finish**

### Paso 7.3: Instalar AlmaLinux 9

1. Enciende la VM (`alma-srv01`) desde el Host Client
2. Click en la miniatura para abrir la consola web
3. Verás el boot de AlmaLinux. Selecciona **Install AlmaLinux 9.x**

Durante la instalación de AlmaLinux (instalador Anaconda): 13. **Idioma:** Español o English → **Continue** 14. **Installation Destination:** Selecciona el disco. Asegúrate de que **Custom** esté seleccionado para practicar LVM: - Click en **Custom** → **Done** - Click en **Click here to create them automatically** (para ver el layout LVM que propone) - O crea manualmente: - `/boot` → 1 GB → Standard Partition → xfs - `almalinux` (VG) → LVM: - `root` → 15 GB → xfs - `swap` → 2 GB → swap - `home` → Lo que queda → xfs - **Done** → **Accept Changes** 15. **Network & Host Name:** - Ethernet (`ens160` o similar, que es el vmxnet3) → Cambia de **OFF** a **ON** - Click **Configure**: - **General:** Marca **Automatically connect to this network when it is available** - **IPv4 Settings:** `Manual` - **Address:** `192.168.1.101` - **Netmask:** `255.255.255.0` - **Gateway:** `192.168.1.1` - **DNS servers:** `8.8.8.8` - **Save** → **Done** 16. **Software Selection:** `Minimal Install` (o `Server` si quieres más paquetes; para RHCSA, `Minimal` + instalación manual de paquetes es mejor práctica) 17. **Root Password:** Configura 18. **User Creation:** Crea un usuario no-root (para practicar `sudo`) 19. **Begin Installation**

Espera ~5-10 minutos. Reinicia cuando termine.

------

## FASE 8: Post-instalación AlmaLinux (la parte que te hará brillar)

Una vez que `alma-srv01` arranque, entra como root.

### Paso 8.1: Instalar VMware Tools (open-vm-tools)

```bash
dnf install -y open-vm-tools
systemctl enable --now vmtoolsd
```

Verifica que esté corriendo:

```bash
systemctl status vmtoolsd
```

En el Host Client de ESXi, ve a la VM `alma-srv01`. En **Summary**, ahora deberías ver:

- **IP Addresses:** `192.168.1.101`
- **DNS Name:** (puede tardar unos minutos en aparecer)

Esto demuestra que el hypervisor y la VM se comunican correctamente.

### Paso 8.2: Verificar hardware virtual óptimo

Dentro de AlmaLinux:

```bash
lspci | grep -i vmware
```

Deberías ver:

- `VMware PVSCSI` (controlador de disco)
- `VMXNET3` (controlador de red)

```bash
ethtool -i ens160
```

Debe decir `driver: vmxnet3`

Esto es exactamente lo que verificarás en una entrevista cuando pregunten: *"¿Cómo confirmas que una VM RHEL está optimizada para VMware?"*

### Paso 8.3: Snapshot de estado limpio

En el Host Client:

1. Apaga la VM `alma-srv01` (desde la consola, `poweroff`)
2. Ve a **Snapshots > Take snapshot**
3. **Name:** `00-fresh-install-lvm-configured`
4. **Description:** `AlmaLinux 9 minimal con LVM, red estática, open-vm-tools`
5. **Take snapshot**

Este snapshot es tu punto de retorno. Antes de cada ejercicio destructivo de RHCSA, volverás aquí.

------

## FASE 9: Verificación final del lab

Tu arquitectura ahora debe ser:

```plain
Ryzen 5 5600GT + 16GB + Windows 11
└── Workstation Pro
    └── VM: ESXi-Lab-01 (6GB RAM, 2 cores)
        ├── Datastore: Datastore_Lab_01 (100GB VMFS)
        │   └── ISOs/AlmaLinux-9.x.iso
        └── VM: alma-srv01 (1GB RAM, 1vCPU, vmxnet3, PVSCSI)
            ├── IP: 192.168.1.101
            ├── LVM: root, swap, home
            └── open-vm-tools: running
```

**Pruebas de integridad:**

1. Desde tu laptop, haz ping a `192.168.1.50` (ESXi) y `192.168.1.101` (AlmaLinux). Ambos deben responder.
2. Desde tu laptop, abre `https://192.168.1.50` y loguéate al Host Client.
3. Desde AlmaLinux, haz ping a `192.168.1.1` (tu router) y `8.8.8.8`.

Si las tres pruebas pasan, tu lab está operativo.

------

## FASE 10: Tu próximo paso (después de esta instalación)

Con el hardware que tienes, puedes mantener encendidas simultáneamente:

- ESXi anidado: 6 GB
- `alma-srv01`: 1 GB
- `alma-srv02` (clonar desde srv01): 1 GB
- Total: 8 GB reservados para virtualización

Eso te deja margen. Puedes crear `alma-srv02` como servidor cliente para practicar SSH, NFS, firewalld entre ellas.

------

## ¿Dónde te atoras?

Dime en qué fase estás ahora mismo, o si algún paso te lanzó un error. No avances a ciegas: si la VM de ESXi no bootea, si la red no responde, o si AlmaLinux no ve el disco, detente y reporta. Cada error aquí es un escenario de troubleshooting que también te servirá en producción.

¿Todo claro hasta aquí, o hay algún paso que necesites que profundice?
