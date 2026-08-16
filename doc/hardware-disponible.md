# Inventario del hardware disponible



> Utilizo una laptop que  como estacion de trabajo y un computador con Ryzen 5-5600GT, 16GB RAM y SSD.  
>
> Mi plan es dedicar  el segundo computador para VMware ESXi... 

Quiero crear el laboratorio lo mejor parecido a un entorno de producción.



Con tu hardware actual **no puedes montar un laboratorio que simule un entorno de producción enterprise realista** (cluster con vCenter, HA, DRS, vMotion). Pero puedes construir una **base sólida** y complementarla con labs gratuitos en la nube. Te explico por qué y cómo sacarle el máximo provecho a lo que tienes.

------

## 1. La realidad de tus recursos



| Recurso           | Tienes                              | Necesitas para "producción realista"           | Veredicto      |
| :---------------- | :---------------------------------- | :--------------------------------------------- | :------------- |
| **RAM**           | 16 GB                               | 32-64 GB mínimo (vCenter solo pide ~14 GB)     | ❌ Insuficiente |
| **Hosts físicos** | 1 (Ryzen 5600GT)                    | 2-3 hosts para cluster, HA, vMotion            | ❌ Insuficiente |
| **NICs**          | 1 (probablemente Realtek integrada) | 2+ NICs Intel (management + vMotion + storage) | ⚠️ Riesgoso     |
| **Storage**       | 1 SSD                               | NAS compartido o múltiples discos para vSAN    | ⚠️ Limitado     |

**El cuello de botella es la RAM.** vCenter Server Appliance (VCSA) requiere ~12-14 GB solo para arrancar. Si lo pones en tu Ryzen con 16 GB, solo te quedan 2-4 GB para ESXi y las VMs de prueba. Es inviable.

------

## 2. El problema que casi seguro te encontrarás: la tarjeta de red

Tu placa base (probablemente B550, A520 o X570) casi con seguridad trae una **NIC Realtek** (RTL8111H, RTL8125BG, etc.).

**ESXi NO incluye drivers para Realtek.** Al bootear la ISO verás:



```plain
No network adapters were detected.
```

Esto es porque VMware solo certifica hardware enterprise (Intel, Broadcom, Mellanox, QLogic). Los Ryzen de escritorio y sus placas base **no están en la HCL**

.

### Soluciones para la NIC:



| Opción                                                    | Viabilidad                                                   | Complejidad      |
| :-------------------------------------------------------- | :----------------------------------------------------------- | :--------------- |
| **Verificar si tu placa tiene Intel I211/I225/I226**      | Si es Intel, ESXi 8 la detecta nativamente                   | Baja             |
| **Comprar una NIC PCIe Intel i210/i350** (~$20-40 usados) | 100% compatible con ESXi                                     | Baja             |
| **Crear ISO custom con driver community (net55-r8168)**   | **No funciona en ESXi 8.0** — el driver stack VMKlinux fue eliminado en ESXi 7.0+ | Alta / Imposible |
| **USB NIC con fling de VMware**                           | Funciona para instalar, pero inestable para lab serio        | Media            |

> **Mi recomendación:** Antes de comprar nada, abre tu PC y verifica qué chip de red tiene tu placa base. Si es Realtek, compra una **Intel i210-T1 PCIe x1** (~$25 usada en eBay/MercadoLibre). Es la inversión más inteligente para tu lab.

------

## 3. Arquitectura realista para TU setup

Dado que no puedes tener vCenter local por falta de RAM, diseñemos un lab en **dos capas**:

### Capa física: Tu PC Ryzen (ESXi standalone nativo)



```plain
┌─────────────────────────────────────────────┐
│  PC Físico: Ryzen 5 5600GT + 16GB RAM       │
│  ├─ ESXi 8.0U3e (nativo, bare-metal)        │
│  │   ├─ 1 vSwitch (Standard)                │
│  │   ├─ Port Group: VM Network (VLAN 0)     │
│  │   ├─ Port Group: Management (VMkernel)   │
│  │   ├─ Datastore VMFS en SSD local         │
│  │   │                                      │
│  │   ├─ VM 1: Ubuntu Server 22.04 (2GB)     │
│  │   ├─ VM 2: Windows Server 2022 (4GB)     │
│  │   ├─ VM 3: AlmaLinux 9 (2GB)             │
│  │   └─ VM 4: Router/Firewall virtual (1GB) │
│  │                                          │
│  └─ (Sin vCenter — gestión vía Host Client) │
└─────────────────────────────────────────────┘
```

**Qué practicas aquí:**

- Instalación y configuración de ESXi bare-metal
- vSwitch Standard, Port Groups, VMkernel
- Datastores VMFS locales
- Creación de VMs, snapshots, clones, templates
- VMware Tools en Linux y Windows
- Resource Pools (limitar CPU/RAM por VM)
- Almacenamiento: thin vs thick provisioning
- Networking básico: VLAN tagging, promiscuous mode

### Capa "enterprise": VMware Hands-On Labs (desde tu laptop)

Desde tu laptop, accedes gratis a labs reales de VMware en navegador donde practicas:

- vCenter Server, clusters, HA, DRS

- vMotion y Storage vMotion

- Distributed Switches

- vSAN

- vSphere Lifecycle Manager

  

> 
>
> Esto te da exposición real a producción **sin consumir recursos de tu PC**.



------

## 4. Si quieres forzar vCenter local (avanzado, justo de recursos)

Si insistes en tener vCenter en tu Ryzen, la única forma viable es **no crear VMs de prueba dentro de ESXi**, sino usar ESXi solo como plataforma para alojar el VCSA:



```plain
Ryzen 16GB:
├─ ESXi 8.0U3e (usa ~2GB)
├─ VCSA 8.0 (tiny deployment, 10GB RAM, 2 vCPU)
└─ 1 VM de prueba con 2GB RAM (máximo)
```

**Problema:** Solo puedes tener **una VM de prueba** corriendo. No es un lab, es una demo.

**Veredicto:** No lo recomiendo. Mejor aprender ESXi nativo a fondo y usar HOL para vCenter.

------

## 5. Cómo conectar tu laptop al lab (topología de red)

Para que se sienta "real", configura esto:



```plain
[Laptop] ──────(cable Ethernet cruzado o switch)────── [Ryzen ESXi]
   │                                                      │
   │  IP: 192.168.10.10/24                                │
   │  Accede al Host Client: https://192.168.10.5         │
   │                                                      │
   └─ Puede montar un share NFS desde la laptop           │
      para practicar datastores NFS (más realista que     │
      solo storage local)                                  │
```

**En tu laptop (Linux):**



```bash
# Instala nfs-kernel-server
sudo apt install nfs-kernel-server

# Crea un directorio para exportar
sudo mkdir -p /export/nfs-vmware
sudo chmod 777 /export/nfs-vmware

# Añade al /etc/exports:
# /export/nfs-vmware 192.168.10.0/24(rw,sync,no_subtree_check,no_root_squash)

sudo exportfs -a
```

Luego en ESXi: **Storage > New Datastore > Mount NFS datastore** apuntando a la IP de tu laptop.

------

## 6. Tu roadmap de upgrade (para llegar a "producción real")



| Fase       | Inversión | Qué ganas                                                    |
| :--------- | :-------- | :----------------------------------------------------------- |
| **Ahora**  | $0        | ESXi standalone nativo. Aprendes el hypervisor, redes, storage local. |
| **Fase 1** | ~$25-40   | **NIC Intel i210 PCIe** (si tu placa tiene Realtek). Resuelves compatibilidad de red. |
| **Fase 2** | ~$60-80   | **32GB RAM DDR4** (2x16GB). Ahora sí puedes correr vCenter + 3-4 VMs de prueba. |
| **Fase 3** | ~$150-200 | **Mini PC/NUC usado** (Intel i3/i5, 16GB) como segundo host ESXi. Ahora puedes formar un **cluster real** con vCenter y practicar HA/DRS/vMotion entre dos hosts físicos. |
| **Fase 4** | ~$200/año | **VMUG Advantage**. Licencias de vCenter + Enterprise Plus + vSAN para tu lab. |

Con Fase 2 (32GB RAM) ya puedes montar:



```plain
Ryzen (32GB):
├─ ESXi nativo
├─ VCSA (12GB)
├─ NAS virtual (TrueNAS) (4GB)  ← para practicar iSCSI/NFS
├─ VM Router (1GB)
├─ VM Linux 1 (2GB)
├─ VM Linux 2 (2GB)
├─ VM Windows (4GB)
└─ VM ESXi anidado (8GB)        ← segundo "host" para practicar vMotion
```

------

## Mi recomendación final para TU caso específico

1. **Hoy:** Descarga ESXi 8.0U3e Free. Intenta instalarlo nativo en el Ryzen. Si la NIC no funciona, compra una Intel i210-T1 PCIe.
2. **Aprende a fondo:** ESXi standalone — networking (vSwitch, VLANs, VMkernel), storage (VMFS, NFS desde laptop), VMs, templates, snapshots.
3. **Complementa:** Usa **VMware Hands-On Labs** desde la laptop para tocar vCenter, HA, DRS, vMotion en infraestructura real de VMware.
4. **Próxima inversión:** **RAM** (pasar a 32GB). Es el recurso que más te limita.
5. **Después:** Un mini PC barato como segundo host físico.

Con 16GB y un solo host, no es un datacenter. Pero es un **ESXi real bare-metal**, y eso ya te pone por encima de quienes solo han usado VirtualBox. Aprenderás el lenguaje, la interfaz, los conceptos de red y storage de VMware — y cuando tengas más hardware, solo será escalar.

------

¿Quieres que te guíe paso a paso para la **instalación de ESXi 8.0U3e en tu Ryzen**, o prefieres que primero te ayude a **verificar qué chip de red tiene tu placa base** antes de que compres nada?



### Verificación del hardware disponible

> Ciertamente, la NIC es Realtek. 
>
> Considerando las limitaciones del hardware y los requerimientos de ESXi, elegir la ruta por Workstation Pro parece ser la única opción.



Excelente decisión. Reconocer que el hardware tiene límites y pivotar hacia una arquitectura que **sí funcione** es exactamente el tipo de juicio de ingeniería que se espera en infraestructura. Vamos a diseñar tu lab como un profesional: maximizando lo que tienes y siendo honestos sobre lo que no se puede.

------

## 1. Por qué Workstation Pro sobre Windows es la jugada correcta aquí



| Problema del Ryzen nativo                          | Solución con Workstation                                     |
| :------------------------------------------------- | :----------------------------------------------------------- |
| NIC Realtek no soportada por ESXi                  | Windows maneja el hardware; ESXi ve una NIC virtual Intel (vmxnet3) |
| 16GB RAM insuficientes para ESXi + vCenter + VMs   | Windows gestiona la memoria; puedes "overcommit" y paginar   |
| Un solo host físico = imposible practicar clusters | Puedes correr **múltiples VMs de ESXi** en un solo PC        |
| Riesgo de dejar un PC inservible como desktop      | Conservas Windows para usarlo normalmente cuando no estudies |

> **Dato clave:** VMware Workstation Pro es **gratuito para uso personal** desde noviembre de 2024 (después de la adquisición por Broadcom). Puedes descargarlo legalmente sin pagar.

------

## 2. La verdad incómoda sobre 16GB de RAM

Con 16GB no puedes montar un datacenter completo localmente. Los números no mienten:



| Componente                          | RAM mínima requerida    | Notas                   |
| :---------------------------------- | :---------------------- | :---------------------- |
| Windows 11 (host)                   | ~3-4 GB                 | Innegociable            |
| VMware Workstation overhead         | ~1 GB                   | Gestión de las VMs      |
| **1 VM de ESXi anidado**            | **4-6 GB**              | Mínimo funcional        |
| **vCenter Server Appliance (VCSA)** | **12 GB** (modo "Tiny") | El cerebro del cluster  |
| VMs de prueba dentro de ESXi        | ~1 GB c/u               | Para que tengan sentido |

**Suma:** Si intentas correr ESXi + VCSA local, necesitas ~20GB. Con 16GB, **no cabe**.

### Mi estrategia como tutor: Lab Híbrido de Dos Velocidades

No intentaremos encajar todo en el Ryzen. Dividiremos tu aprendizaje en dos capas:



| Capa                                 | Plataforma                 | Qué practicas                                                |
| :----------------------------------- | :------------------------- | :----------------------------------------------------------- |
| **Capa Local (Ryzen + Workstation)** | 1 ESXi anidado             | Hypervisor bare-metal, vSwitch, Port Groups, VMkernel, datastores, VMs, snapshots, templates, VMware Tools, resource pools, almacenamiento |
| **Capa Enterprise (Navegador)**      | VMware Hands-On Labs (HOL) | vCenter, clusters, HA, DRS, vMotion, Storage vMotion, Distributed Switches, vSAN |

Así usas tu hardware real para lo que **sí** puede dar, y usas la nube gratuita de VMware para lo que requiere infraestructura masiva.

------

## 3. Arquitectura de tu lab local



```plain
┌─────────────────────────────────────────────────────────────┐
│           PC Físico: Ryzen 5 5600GT + 16GB + SSD            │
│                    Windows 11 (Host)                        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │        VMware Workstation Pro 17.6+ (Gratis)        │    │
│  │                                                     │    │
│  │  ┌───────────────────────────────────────────────┐  │    │
│  │  │  VM 1: ESXi 8.0U3e (Anidado / Nested)         │  │    │
│  │  │  ├─ RAM: 6 GB                                 │  │    │
│  │  │  ├─ vCPU: 2 (con nested virtualization ON)    │  │    │
│  │  │  ├─ Disco 1: 20 GB (instalación ESXi)         │  │    │
│  │  │  ├─ Disco 2: 120 GB (Datastore VMFS)          │  │    │
│  │  │  ├─ NIC 1: vmxnet3 (Management / VM Network)  │  │    │
│  │  │  └─ NIC 2: vmxnet3 (vMotion - para practicar) │  │    │
│  │  │                                               │  │    │
│  │  │  Dentro de ESXi correrás:                    │  │    │
│  │  │  ├─ VM-A: Alpine Linux o Debian (512 MB)    │  │    │
│  │  │  ├─ VM-B: Ubuntu Server (1 GB, sin GUI)      │  │    │
│  │  │  └─ VM-C: Windows Server Core (2 GB)         │  │    │
│  │  └───────────────────────────────────────────────┘  │    │
│  │                                                     │    │
│  │  (Reserva de RAM: ~9 GB para Windows + Workstation) │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**¿Por qué solo 1 ESXi y no 2?** Porque con 16GB, 2 ESXi anidados (4GB cada uno) + Windows = saturación total. Un solo ESXi con 6GB te deja aire para respirar y correr 3 VMs de prueba pequeñas dentro de él.

------

## 4. Paso a paso: De cero a tu primer ESXi anidado

### Paso 1: Preparar el BIOS/UEFI

1. Reinicia el Ryzen y entra al BIOS (usualmente `Del` o `F2`).
2. Busca **SVM Mode** (Secure Virtual Machine) o **AMD-V**. **Habilita.**
3. Busca **NX Mode** o **No-Execute Memory Protection**. **Habilita.**
4. Guarda y reinicia.

### Paso 2: Instalar VMware Workstation Pro

1. Descarga desde el portal de Broadcom (o busca el instalador gratuito).
2. Instala en Windows con opciones por defecto.
3. Al terminar, verifica en **Edit > Preferences** que la ruta de VMs apunte a tu SSD (no a un disco lento).

### Paso 3: Crear la VM para ESXi anidado

1. **File > New Virtual Machine > Custom (advanced).**
2. **Hardware compatibility:** Workstation 17.6.x.
3. **Guest OS:** VMware ESX > **VMware ESXi 8.x**.
4. **Processors:** 2 cores. Expande y marca **"Virtualize CPU performance counters"** y **"Virtualize IOMMU"**.
5. **Memory:** 6144 MB (6 GB).
6. **Network:**
   - Adapter 1: **Bridged** (se conecta directo a tu red doméstica). Esto le da IP real a ESXi.
   - Después de crear la VM, añade un **segundo Network Adapter** (también Bridged, para practicar vMotion aunque no hagamos cluster real).
7. **Disk 1:** 20 GB, SCSI, **Thin provisioned**.
8. **Disk 2:** 120 GB, SCSI, **Thin provisioned** (este será tu datastore VMFS).
9. Antes de encender: **Edit virtual machine settings > Options > Advanced > Check "Disable side channel mitigations"** (mejora rendimiento en labs).

### Paso 4: Instalar ESXi 8.0U3e

1. Monta la ISO de ESXi 8.0U3e en la unidad CD/DVD virtual.
2. Enciende la VM.
3. Sigue la instalación estándar:
   - Selecciona el disco de 20 GB.
   - Teclado: US Default.
   - Password para root (anótalo bien).
4. Al reiniciar, ESXi arrancará. Verás la pantalla amarilla/naranja con la IP.
5. Desde tu laptop (o desde el mismo Ryzen en Windows), abre navegador y ve a `https://<IP-del-ESXi>`.

### Paso 5: Configurar ESXi por primera vez

1. En la consola DCUI (pantalla amarilla de ESXi), presiona `F2` y loguéate como root.
2. **Configure Management Network > Network Adapters:** Verifica que vea ambas NICs (vmxnet3).
3. **IP Configuration:** Pon IP estática en tu rango (ej: `192.168.1.50/24`).
4. **DNS Configuration:** Pon tu gateway como DNS temporal.
5. **Test Management Network:** Verifica que haga ping a tu gateway.

### Paso 6: Configurar el datastore

1. En el Host Client web (`https://192.168.1.50`), ve a **Storage > New datastore**.
2. Crea un datastore VMFS en el disco de 120 GB.
3. Nombre sugerido: `Datastore_Lab_01`.

### Paso 7: Crear tu primera VM dentro de ESXi

1. **Virtual Machines > Create / Register VM.**

2. Guest OS: Linux > Ubuntu Server 22.04 (64-bit).

3. Asigna: 1 vCPU, 1 GB RAM, disco thin de 20 GB en el datastore.

4. Conecta la ISO de Ubuntu al CD/DVD virtual.

5. Instala. Luego instala **open-vm-tools**:

   

   ```bash
   sudo apt install open-vm-tools
   ```

   

------

## 5. Qué practicar localmente vs. en Hands-On Labs

### En tu ESXi anidado (Ryzen) — practica ESTO:



| Skill                          | Cómo lo practicas                                            | Relevancia para certificación |
| :----------------------------- | :----------------------------------------------------------- | :---------------------------- |
| **Navegación del Host Client** | La UI web de ESXi                                            | 20% del examen                |
| **vSwitch Standard**           | Crea un vSwitch, asigna NICs físicas                         | Básico obligatorio            |
| **Port Groups**                | Crea grupos para VLANs (ej: PG_Mgmt, PG_VMs, PG_vMotion)     | Crítico                       |
| **VMkernel NICs**              | Añade vmk1 para vMotion, vmk0 para management                | Crítico                       |
| **Datastores VMFS**            | Crea, extiende, renombra, revisa espacio                     | Básico                        |
| **VMs y hardware virtual**     | Cambia de e1000 a vmxnet3, añade discos, cambia RAM/CPU en caliente | Muy frecuente en examen       |
| **Snapshots**                  | Crea, consolida, elimina                                     | Trampa común en producción    |
| **Templates y clones**         | Convierte VM en template, despliega desde template           | Frecuente                     |
| **Resource Pools**             | Crea pools con límites de CPU/Memoria                        | DRS previo                    |
| **VMware Tools**               | Instala en Linux y Windows                                   | Siempre necesario             |
| **Permisos locales**           | Crea usuarios, asigna roles en ESXi                          | Seguridad básica              |
| **ESXCLI básico**              | SSH habilitado, comandos como `esxcli storage filesystem list` | Troubleshooting               |

### En VMware Hands-On Labs (navegador) — practica ESTO:



| Skill                         | Por qué no lo haces local                   | Relevancia     |
| :---------------------------- | :------------------------------------------ | :------------- |
| **vCenter Server**            | VCSA requiere 12GB RAM                      | 40% del examen |
| **Clusters y HA**             | Necesitas 2+ hosts + vCenter                | Crítico        |
| **DRS**                       | Requiere vCenter + cluster                  | Crítico        |
| **vMotion**                   | Requiere vCenter + shared storage + 2 hosts | Muy frecuente  |
| **Storage vMotion**           | Requiere vCenter                            | Frecuente      |
| **Distributed Switch (vDS)**  | Requiere vCenter                            | Avanzado       |
| **vSAN**                      | Requiere 3 hosts mínimo                     | Especialidad   |
| **vSphere Lifecycle Manager** | Requiere vCenter                            | Actual         |
| **Content Library**           | Requiere vCenter                            | Útil           |
| **PowerCLI / API**            | Requiere vCenter para escritura             | Automatización |

> **Acceso a HOL:** Ve a [hol.vmware.com](https://hol.vmware.com/), crea cuenta gratuita, y busca el lab **"HOL-2501-01-VCF - VMware vSphere - Getting Started"**. Es el estándar para VCP-DCV.

------

## 6. Optimizaciones para que no se trabe con 16GB



| Problema                  | Solución                                                     |
| :------------------------ | :----------------------------------------------------------- |
| Windows consume mucha RAM | Desactiva efectos visuales, apps en segundo plano, Xbox Game Bar |
| ESXi anidado lento        | En Workstation, asigna **"Fit all virtual machine memory into reserved host RAM"** en Edit > Preferences > Memory |
| VMs dentro de ESXi lentas | Usa **Alpine Linux** (consume ~100MB RAM) o **Debian sin GUI** en vez de Ubuntu Desktop |
| Disco lento               | Asegúrate de que tanto Workstation como las VMs estén en el **SSD** |
| ESXi no arranca anidado   | Verifica que en la VM de ESXi, en **Processors**, esté marcado **"Virtualize Intel VT-x/EPT or AMD-V/RVI"** |

------

## 7. Tu roadmap de upgrade (cuando tengas presupuesto)



| Prioridad             | Compra                                      | Impacto                                                    | Costo aprox. |
| :-------------------- | :------------------------------------------ | :--------------------------------------------------------- | :----------- |
| **1. RAM**            | 32GB DDR4 (2x16GB)                          | Puedes correr ESXi + VCSA local + 2-3 VMs de prueba        | ~$60-80      |
| **2. Segundo PC**     | Mini PC usado (Intel i3/i5, 16GB)           | Cluster real de 2 hosts físicos; practicas HA/vMotion real | ~$150-200    |
| **3. NIC Intel**      | i210-T1 PCIe x1                             | Si algún día vuelves a intentar ESXi nativo en el Ryzen    | ~$25-40      |
| **4. VMUG Advantage** | Licencias anuales vCenter + Enterprise Plus | Lab perpetuo con todas las features                        | ~$200/año    |

Con solo el upgrade a **32GB RAM**, ya podrías montar esto localmente:



```plain
Workstation:
├─ ESXi-01 (4GB)
├─ ESXi-02 (4GB)  ← segundo host anidado
├─ VCSA Tiny (10GB) ← vCenter
└─ NAS virtual (2GB) ← TrueNAS Core para NFS/iSCSI
```

Y ahí sí tendrías un **cluster funcional** en un solo PC.

------

## Resumen: tu plan de acción de hoy

1. **Ahora:** Habilita SVM en BIOS, instala Workstation Pro, descarga ISO de ESXi 8.0U3e.
2. **Hoy:** Crea la VM de ESXi anidado (6GB RAM, 2 discos, 2 NICs).
3. **Esta semana:** Instala ESXi, configura red, datastore, y crea 2-3 VMs Linux dentro.
4. **Paralelo:** Abre VMware HOL y completa el lab de vSphere Getting Started.
5. **Meta:** Toca ESXi localmente todos los días; toca vCenter en HOL 2-3 veces por semana.

Así, cuando llegues a una entrevista o al examen, habrás usado **el hypervisor real** (no un simulador) y habrás operado **vCenter en infraestructura real** (aunque sea en sesiones temporales). Esa combinación es más valiosa que tener un lab perfecto pero inalcanzable.

------

¿Quieres que te guíe paso a paso para la **configuración de red en ESXi** (vSwitch, Port Groups, VMkernel) una vez que tengas el ESXi anidado corriendo, o prefieres que primero resolvamos cómo **descargar Workstation Pro gratis** y la ISO correcta de ESXi?
