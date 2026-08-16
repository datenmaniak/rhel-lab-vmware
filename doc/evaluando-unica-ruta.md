# Evaluando la única ruta de despliegue



> Antes de romper cosa:
>
> 1) Es necesario crear un anidado con ESXi 8.0U3i ?
> 2) Es posible trabajar con VMs y redes solamente utilizando  VMware Workstation Pro ?



Voy a ser directo contigo, como debe ser un tutor cuando las restricciones son reales.

------

## 1. ¿Es necesario crear un anidado con ESXi 8.0U3?

**No. No es obligatorio.** Pero hay una diferencia enorme entre *poder* hacer algo y *necesitar* hacerlo para tu objetivo.

Primero, aclaremos lo de la versión: probablemente te refieras a **ESXi 8.0U3e** o **8.0U3g** (las últimas disponibles para descarga pública). Si existe un *update* posterior (como una hipotética "i"), para efectos de tu lab no cambia nada: cualquier ESXi 8.0U3 es funcionalmente idéntico para aprender.

Ahora, la pregunta real: **¿necesitas ESXi anidado?**



| Tu objetivo                                                  | ¿Necesitas ESXi anidado? | Por qué                                                 |
| :----------------------------------------------------------- | :----------------------- | :------------------------------------------------------ |
| Aprender virtualización general (crear VMs, redes, snapshots) | **No**                   | Workstation Pro solo lo hace todo                       |
| Aprender Linux/Windows Server en VMs                         | **No**                   | Workstation Pro es perfecto                             |
| Prepararte para **certificación VCP-DCV**                    | **Parcialmente**         | El examen es 85% vCenter + ESXi. Workstation no aparece |
| Conseguir trabajo operando VMware en producción              | **Sí, idealmente**       | En producción no usan Workstation; usan ESXi + vCenter  |

### La verdad con 16 GB de RAM

Anidar ESXi dentro de Workstation **sí funciona**, pero con 16 GB te quedarás en un ESXi standalone con 3-4 VMs pequeñas y **sin vCenter**. Es como querer aprender a manejar un camión usando solo una moto: aprendes a balancearte, pero no a cambiar 18 marchas.

**Conclusión:** No es *necesario* si tu prioridad es aprender virtualización de escritorio. **Sí es necesario** si quieres decir en una entrevista: *"He administrado ESXi"*. Para eso, sin embargo, necesitarás complementar con los **VMware Hands-On Labs** porque tu hardware local no puede correr vCenter.

------

## 2. ¿Es posible trabajar con VMs, laboratorio y redes solo con Workstation Pro?

**Sí. Pero entiende el techo.**

Workstation Pro es un hypervisor tipo 2 muy poderoso. Puedes hacer mucho más de lo que la gente cree:

### ✅ Lo que SÍ puedes hacer con Workstation Pro solo



| Tarea                      | Cómo se hace                                               | Nivel        |
| :------------------------- | :--------------------------------------------------------- | :----------- |
| **Múltiples VMs**          | Crear tantas como tu RAM permita                           | Básico       |
| **Redes NAT**              | VMnet8, traducción de IPs, aislamiento                     | Intermedio   |
| **Redes Bridged**          | VMnet0, las VMs son ciudadanas de tu red física            | Intermedio   |
| **Redes Host-only**        | VMnet1, aislamiento total, sin internet                    | Intermedio   |
| **LAN Segments**           | Redes privadas entre VMs sin adaptador virtual predefinido | **Avanzado** |
| **VLANs 802.1Q**           | Configurar tagging en adaptadores vmxnet3                  | **Avanzado** |
| **Snapshots en cadena**    | Snapshot sobre snapshot, revertir, consolidar              | Intermedio   |
| **Clones**                 | Full clone y linked clone                                  | Intermedio   |
| **Equipos (Teams)**        | Encender/apagar grupos de VMs en orden                     | Intermedio   |
| **Almacenamiento virtual** | Discos IDE, SCSI, SATA, NVMe, thin/thick                   | Intermedio   |

Con LAN Segments y VLANs, puedes construir topologías de red complejas: DMZ, redes internas, segmentos de management. Eso sí es útil y transferible.

### ❌ Lo que NO puedes hacer con Workstation Pro (y es el 85% del examen VCP)



| Feature de producción                            | Por qué no existe en Workstation                | Relevancia laboral                  |
| :----------------------------------------------- | :---------------------------------------------- | :---------------------------------- |
| **ESXi Host Client**                             | Workstation no es ESXi                          | Es la interfaz que tocas diario     |
| **vCenter Server**                               | No hay servicio de gestión centralizada         | El cerebro del datacenter           |
| **Clusters de hosts**                            | Solo hay un host: tu PC                         | Base de la alta disponibilidad      |
| **HA (High Availability)**                       | No hay múltiples hosts físicos                  | Si un servidor muere, ¿qué pasa?    |
| **DRS**                                          | No hay múltiples hosts para balancear           | Distribución automática de carga    |
| **vMotion**                                      | No hay infraestructura de migración en caliente | Operación crítica en mantenimientos |
| **Storage vMotion**                              | No hay datastores compartidos                   | Mover VMs sin apagarlas             |
| **vSphere Distributed Switch**                   | Workstation usa bridges del SO anfitrión        | Redes enterprise centralizadas      |
| **vSAN**                                         | No hay múltiples discos en múltiples hosts      | Storage definido por software       |
| **ESXCLI / SSH al hypervisor**                   | No hay hypervisor ESXi                          | Troubleshooting de producción       |
| **VMware Tools en modo paravirtualizado óptimo** | Sí instala tools, pero el contexto es diferente | Optimización de rendimiento         |
| **Resource Pools con límites estrictos**         | Workstation gestiona recursos del SO host       | Control de cuotas en multi-tenant   |

------

## 3. La comparativa honesta: Workstation Solo vs. ESXi Anidado vs. HOL



```plain
┌──────────────────────────────────────────────────────────────────────┐
│                    RUTAS DE APRENDIZAJE CON TU HARDWARE              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  A) WORKSTATION PRO SOLO                                             │
│     ├─ Pros: Fluido, todo cabe en 16GB, redes avanzadas posibles     │
│     ├─ Contras: No tocas ESXi, no tocas vCenter                      │
│     └─ Ideal para: Virtualización general, redes, sistemas operativos│
│                                                                      │
│  B) WORKSTATION + 1 ESXi ANIDADO (6GB)                               │
│     ├─ Pros: Tocas el Host Client real, ESXCLI, vmkernel, datastores │
│     ├─ Contras: Sin vCenter (no cabe), sin HA/DRS/vMotion            │
│     └─ Ideal para: Familiarizarte con ESXi, pero NO con vSphere      │
│                                                                      │
│  C) VMWARE HANDS-ON LABS (Navegador, gratis)                         │
│     ├─ Pros: vCenter real, clusters, HA, DRS, vMotion, vSAN          │
│     ├─ Contras: Sesiones temporales (horas), no guardas configs      │
│     └─ Ideal para: To-do el stack enterprise que no cabe en tu PC    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

------

## 4. Mi recomendación como tutor para TU caso específico

Tienes:

- Laptop (estación de trabajo)
- Ryzen 5 5600GT + 16GB RAM + Realtek (PC dedicado al lab)
- Decidiste usar Workstation Pro sobre Windows en el Ryzen

Con esa configuración, te propongo una **estrategia híbrida realista**:

### Fase 1: Dominar Workstation Pro (Semanas 1-2)

Usa solo Workstation en el Ryzen. Construye esto:



```plain
Workstation:
├─ VM 1: pfSense / OPNsense (firewall/router) ──→ 1GB RAM, 2 NICs
├─ VM 2: Ubuntu Server (servidor web/DNS) ────────→ 1GB RAM
├─ VM 3: Windows Server 2022 (DC/DNS) ────────────→ 2GB RAM
├─ VM 4: Debian (base de datos) ──────────────────→ 1GB RAM
├─ VM 5: Kali Linux / cliente de pruebas ─────────→ 1GB RAM
└─ Redes: LAN Segment "Internal", Bridged "WAN", Host-only "Mgmt"
```

**Qué aprendes:** Topologías de red, routing inter-VLAN, NAT, servicios, snapshots, clones. Todo esto es infraestructura real y te sirve en cualquier trabajo de IT, no solo VMware.

### Fase 2: Tocar ESXi real (Semanas 3-4)

Crea **una sola VM de ESXi 8.0U3e anidada** (6GB RAM) dentro de Workstation. No intentes vCenter local.



```plain
Workstation:
└─ VM: ESXi 8.0U3e (6GB RAM, 2 discos, 2 NICs vmxnet3)
    ├─ VM interna: Alpine Linux (128MB) ──→ practicar VMware Tools
    ├─ VM interna: Ubuntu Server (512MB) ──→ practicar discos, snapshots
    └─ VM interna: Windows Server Core (1GB) ──→ practicar drivers
```

**Qué aprendes:** El Host Client, crear datastores VMFS, vSwitch Standard, Port Groups, VMkernel NICs, ESXCLI por SSH, hardware virtual vmxnet3 vs e1000. Es el lenguaje de producción.

### Fase 3: Enterprise en la nube (Paralelo, desde tu laptop)

2-3 veces por semana, entra a **hol.vmware.com** y toma el lab:

> **"HOL-2501-01-VCF - VMware vSphere - Getting Started"**

**Qué aprendes:** vCenter, crear clusters, habilitar HA/DRS, ejecutar vMotion, configurar Distributed Switches, usar Content Library. Es la otra mitad del mundo que tu PC no puede correr.

------

## 5. Veredicto final a tus dos preguntas



| Pregunta                                    | Respuesta directa                                            |
| :------------------------------------------ | :----------------------------------------------------------- |
| **¿Es necesario el ESXi anidado?**          | **No es obligatorio**, pero si quieres trabajar con VMware en producción, **debes tocar ESXi alguna vez**. Con 16GB, una sola VM de ESXi anidado es viable; vCenter no lo es. |
| **¿Puedo hacer todo con Workstation solo?** | **Sí, para virtualización de escritorio y redes**. **No, para prepararte como administrador de vSphere en producción.** Workstation es el simulador de manejo; ESXi+vCenter es el camión real. |

------

## ¿Qué decides?

Te doy tres opciones concretas. Elige una:

1. **"Quiero empezar con Workstation solo y topologías de red"** → Te diseño el diagrama de red (LAN Segments, VLANs, pfSense) para tu lab de 16GB.
2. **"Quiero montar el ESXi anidado ya"** → Te doy los pasos exactos de configuración en Workstation (flags de CPU, discos, NICs vmxnet3).
3. **"Quiero saber cómo acceder a los Hands-On Labs"** → Te explico el registro y qué labs específicos tomar para el VCP-DCV.

¿Cuál resuena más con tu objetivo inmediato?

