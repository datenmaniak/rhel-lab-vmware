# Requerimientos de Hardware



> Puedo  dedicar mi computador para crear un laboratorio ideal para aprender y lograr la formación necesaria para presentar exámenes y obtener una certificación.? 



Para un laboratorio de certificación en tu computador dedicado, la respuesta es clara:

> **ESXi 8.0U3e** (la última versión gratuita disponible) + **vCenter Server Appliance 8.0U3e** en modo evaluación de 60 días.

Aquí te explico por qué esta es la combinación ganadora y cómo montarla.

------

## 1. Por qué NO la versión 9.x



| Razón                        | Explicación                                                  |
| :--------------------------- | :----------------------------------------------------------- |
| **Sin versión gratuita**     | ESXi 9.x no tiene licencia Free. Solo existe dentro de VVF/VCF con licenciamiento enterprise |
| **El examen se basa en 8.x** | El VCP-DCV actual (examen 2V0-21.23) cubre **vSphere 8.x** en su blueprint |
| **Cambio de modelo**         | En 9.x desaparecieron las ediciones Standard y Enterprise Plus. El examen aún no ha migrado a ese modelo |

------

## 2. Por qué SÍ la 8.0U3e

- Es la **última versión con licencia gratuita** perpetua.
- El examen VCP-DCV testea: HA, DRS, vMotion, Storage vMotion, vSphere Distributed Switch, vSAN, vSphere Lifecycle Manager, Content Library, etc. — todo eso existe en **vSphere 8.x**.
- Tienes **soporte oficial hasta octubre de 2027**, así que no estás aprendiendo tecnología muerta.

------

## 3. Las 3 formas de licenciar tu lab

### Opción A: ESXi 8.0U3e Free (gratis, perpetuo)

- Baja la ISO con licencia embebida del portal de Broadcom.
- **Problema:** No tiene vCenter, no tiene HA, no tiene DRS, no tiene vMotion, no tiene APIs de backup. Es un hypervisor aislado.
- **Veredicto:** Útil para aprender ESXi básico, **inútil para certificación VCP-DCV** (el examen es 90% vCenter).

### Opción B: Modo Evaluación de 60 días (gratis, todo desbloqueado)

- Instalas ESXi 8.0U3 **sin licencia** (la ISO de evaluación).
- Despliegas el **vCenter Server Appliance (VCSA)**.
- Tienes **Enterprise Plus completo** por 60 días: vCenter, HA, DRS, vMotion, Storage vMotion, Distributed Switches, Content Library, etc.
- **Veredicto:** Perfecto para un sprint de estudio intensivo. Puedes reinstalar cada 60 días si es necesario.

### Opción C: VMUG Advantage (~$200/año) — **LA MEJOR**

- Membresía anual que te da licencias **vSphere Enterprise Plus + vCenter Standard + vSAN + NSX** válidas para lab personal.
- Es el estándar de oro para quienes se preparan para VCP.
- Además te da **50% de descuento** en el examen de certificación.
- **Veredicto:** Si vas en serio por la certificación, esta es la inversión correcta.

------

## 4. Arquitectura recomendada para tu lab

Asumiendo que tu computador tiene **mínimo 32 GB de RAM** (ideal 64 GB) y un SSD grande:



```plain
┌─────────────────────────────────────────────────────────┐
│              TU COMPUTADOR FÍSICO                       │
│         (VMware Workstation Pro / Linux + KVM)          │
├─────────────────────────────────────────────────────────┤
│  VM 1: ESXi 8.0U3e (anidado)      │  VM 2: ESXi 8.0U3e  │
│  ├─ 12 GB RAM                     │  ├─ 12 GB RAM       │
│  ├─ 4 vCPU                        │  ├─ 4 vCPU          │
│  ├─ 2 discos ( datastore local )  │  ├─ 2 discos        │
│  └─ 2 NICs (management + vMotion) │  └─ 2 NICs          │
├─────────────────────────────────────────────────────────┤
│  VM 3: vCenter Server Appliance (VCSA)                  │
│  ├─ 14 GB RAM                                           │
│  ├─ 4 vCPU                                              │
│  └─ Almacenamiento: 50 GB + 250 GB                      │
├─────────────────────────────────────────────────────────┤
│  VM 4: NAS virtual (TrueNAS / OpenMediaVault)           │
│  └─ Para practicar NFS/iSCSI como datastore compartido  │
└─────────────────────────────────────────────────────────┘
```

**¿Por qué 2 hosts ESXi?** Para poder practicar:

- **vMotion** (migrar VMs de un host a otro en caliente)
- **HA** (simular fallo de un host y ver cómo las VMs se reinician en el otro)
- **DRS** (balanceo automático de carga entre hosts)
- **Storage vMotion** (mover discos entre datastores)

------

## 5. Ruta de montaje paso a paso



| Paso  | Qué haces                                                    | Tiempo estimado |
| :---- | :----------------------------------------------------------- | :-------------- |
| **1** | Instala **VMware Workstation Pro** (gratis desde 2024) en tu Linux/Windows | 30 min          |
| **2** | Descarga **ESXi 8.0U3e** (ISO de evaluación) y **VCSA 8.0U3e** | 1-2 h           |
| **3** | Crea 2 VMs anidadas con ESXi. Habilita "Virtualize Intel VT-x/EPT" en Workstation | 1 h             |
| **4** | Despliega el VCSA apuntando a uno de los ESXi                | 1-2 h           |
| **5** | Crea un **cluster** en vCenter, añade ambos hosts            | 30 min          |
| **6** | Configura **vSwitch**, **Port Groups**, **VMkernel** (management, vMotion) | 1 h             |
| **7** | Monta un NAS virtual (TrueNAS) y exporta un datastore NFS    | 1 h             |
| **8** | Habilita **HA**, **DRS**, **vMotion** en el cluster          | 30 min          |
| **9** | Crea VMs de prueba y practica migraciones, snapshots, templates | Continuo        |

------

## 6. Temas del examen que DEBES practicar en tu lab

Según el blueprint oficial del VCP-DCV 2024 (examen 2V0-21.23), estos son los objetivos que requieren manos en el teclado:



| Dominio            | Skills a practicar en tu lab                                 |
| :----------------- | :----------------------------------------------------------- |
| **Redes**          | vSphere Distributed Switch, Port Groups, VMkernel, NIOC      |
| **Almacenamiento** | Datastores NFS/VMFS, Storage Policies, Storage vMotion       |
| **Clusters**       | HA (Admission Control, Proactive HA), DRS (rules, EVC)       |
| **vCenter**        | SSO, VCSA deployment, High Availability de vCenter, backups  |
| **VMs**            | Templates, Content Library, OVF, snapshots, encryption, vMotion |
| **Seguridad**      | vTPM, Trust Authority, certificados, RBAC                    |
| **Lifecycle**      | vSphere Lifecycle Manager, Quick Boot, host profiles         |

------

## Mi recomendación final



| Tu situación                                  | Qué hacer                                                    |
| :-------------------------------------------- | :----------------------------------------------------------- |
| **Presupuesto cero, quiero aprender primero** | ESXi 8.0U3e Free + reinstala evaluación de vCenter cada 60 días |
| **Voy en serio por la certificación VCP-DCV** | **VMUG Advantage** ($200/año) + lab con 2 ESXi + vCenter + vSAN |
| **Quiero el lab más realista posible**        | Añade un tercer host ESXi anidado y practica con vSAN        |

> **Nota importante:** El examen VCP-DCV 2024 (basado en vSphere 8.x) se retiró oficialmente el 15 de diciembre de 2025 
>
> . Sin embargo, los materiales de estudio actuales y la comunidad aún se centran en vSphere 8.x, y es altamente probable que el nuevo examen (si ya existe o está por salir) mantenga la misma base arquitectónica. Aprender vSphere 8.x te prepara para cualquier versión reciente.

------

¿Quieres que te guíe paso a paso para montar el **Lab anidado con 2 ESXi + vCenter en Workstation**, o prefieres que primero explique cómo funciona el **modo evaluación de 60 días** y cómo reinstalarlo cuando venza?



> 
>
> Entonces, la versión ESXi 8.0U3e es un hypervisor, y el computador que se elija estará exclusivamente dedicado solo para  ESXi?
>
> 



Sí, **ESXi 8.0U3e es un hypervisor**, pero de un tipo que quizás no esperas si vienes del mundo de Linux. Y la respuesta a tu segunda pregunta es: **sí, si lo instalas de forma nativa, el equipo queda 100% dedicado a ESXi.** No queda espacio para nada más.

------

## 1. ESXi es un hypervisor Tipo 1 (bare-metal)

En el ecosistema Linux que conoces, probablemente has usado:



| Tecnología             | Tipo                                              | Cómo funciona                                               |
| :--------------------- | :------------------------------------------------ | :---------------------------------------------------------- |
| **KVM/QEMU**           | Tipo 1 (bare-metal, pero dentro del kernel Linux) | Corre como módulo del kernel de tu Ubuntu/Debian            |
| **VirtualBox**         | Tipo 2                                            | Es una aplicación que instalas **sobre** tu Linux o Windows |
| **VMware Workstation** | Tipo 2                                            | Igual, una app que corre sobre tu sistema operativo         |

**ESXi es diferente:** Es un **hypervisor tipo 1 puro y autónomo**. No es una aplicación que instalas sobre Linux ni sobre Windows. Es un sistema operativo mínimo y especializado (basado en el VMkernel, no en Linux) que se instala **directamente en el disco duro del servidor**, tomando control absoluto del hardware.

### Analogía clara:

- **VirtualBox / Workstation** = Como instalar Firefox. Corre *dentro* de tu sistema operativo.
- **ESXi** = Como instalar Debian o Fedora. **Reemplaza** a tu sistema operativo. Enciendes el PC y lo primero que ves es ESXi, no Linux.

------

## 2. Si lo instalas nativo, tu PC queda dedicado exclusivamente a ESXi

Cuando instalas la ISO de ESXi 8.0U3e en un equipo físico:



```plain
Disco del PC:
┌─────────────────────────────────────┐
│  ESXi 8.0U3e (VMkernel + sistema)   │  ← ÚNICO sistema operativo
│  ├─ Datastore VMFS (tus VMs)        │
│  └─ Configuración del hypervisor    │
└─────────────────────────────────────┘
```

**Lo que pierdes:**

- No puedes arrancar Ubuntu/Windows/Fedora nativo en ese mismo disco.
- No es un programa que abres y cierras. Es el sistema operativo del servidor.
- Todo el hardware (CPU, RAM, red, almacenamiento) es gestionado por ESXi.

**Lo que ganas:**

- Máximo rendimiento. Sin overhead de otro sistema operativo debajo.
- El equipo se convierte en un servidor de virtualización real.

------

## 3. Alternativa para tu lab: ESXi "anidado" (nested virtualization)

Si solo tienes **una computadora** y no quieres dedicarla exclusivamente a ESXi (porque también la usas para trabajo, navegar, etc.), la solución estándar para estudiantes es el **nested virtualization**:



```plain
Tu PC con Linux/Windows:
┌───────────────────────────────────────────────┐
│  Sistema operativo anfitrión (Ubuntu, etc.)   │
│  											    │
│  ├─ VMware Workstation Pro (o KVM/VirtualBox) │
│  │   										    │
│  │   ├─ VM 1: ESXi 8.0U3e (anidado)           │
│  │   │   ├─ VM de prueba (Windows/Linux)      │
│  │   │   └─ VM de prueba 2                    │
│  │   │									    │
│  │   ├─ VM 2: ESXi 8.0U3e (anidado)           │
│  │   │   └─ (segundo host para cluster)       │
│  │   │									    │
│  │   └─ VM 3: vCenter Server Appliance        │
│  │                                            │
│  └─ Tus aplicaciones normales (navegador,     │
│      editor, etc.)                            │
└───────────────────────────────────────────────┘
```

### ¿Cómo se logra esto?

ESXi corre **como una máquina virtual** dentro de VMware Workstation (o KVM con configuración especial). ESXi no sabe que está virtualizado; cree que está corriendo en hardware real. Y dentro de ESXi, puedes crear otras VMs.

### Requisitos de tu PC para esto:



| Recurso            | Mínimo recomendado                                   | Ideal                                            |
| :----------------- | :--------------------------------------------------- | :----------------------------------------------- |
| **RAM**            | 32 GB                                                | 64 GB o más                                      |
| **CPU**            | Intel/AMD con VT-x y **EPT/RVI** habilitados en BIOS | 8+ cores                                         |
| **Almacenamiento** | SSD de 500 GB                                        | NVMe de 1 TB                                     |
| **Red**            | 1 NIC                                                | 2 NICs (mejor para separar management y vMotion) |

> **Importante:** Debes habilitar **"Virtualize Intel VT-x/EPT or AMD-V/RVI"** en las opciones de la VM de Workstation. Sin esto, ESXi anidado no puede crear VMs dentro de sí mismo.

------

## 4. Comparativa: Nativo vs. Anidado para tu certificación



| Aspecto                                        | ESXi Nativo (dedicado)                   | ESXi Anidado (en Workstation)               |
| :--------------------------------------------- | :--------------------------------------- | :------------------------------------------ |
| **Rendimiento**                                | Máximo                                   | Bueno, con overhead del SO anfitrión        |
| **Necesitas PC dedicado**                      | Sí, exclusivo para ESXi                  | No, usas tu PC normal                       |
| **Puedes practicar clusters (HA/DRS/vMotion)** | Sí, si tienes 2+ PCs físicos             | Sí, con múltiples VMs de ESXi en un solo PC |
| **Costo**                                      | 1 PC dedicado + quizás otro para vCenter | Solo tu PC actual                           |
| **Complejidad**                                | Baja                                     | Media (configurar nested)                   |
| **Ideal para**                                 | Homelab permanente en casa               | Estudio y certificación VCP-DCV             |

------

## 5. Mi recomendación para tu caso

Como dices que quieres **prepararte para un examen de certificación** y asumo que no tienes 2-3 servidores físicos ni quieres dejar inservible tu laptop:

> **Usa ESXi 8.0U3e anidado dentro de VMware Workstation Pro en tu PC.**

Así puedes:

- Tener **2 o 3 hosts ESXi virtuales** en un solo PC.
- Desplegar **vCenter Server Appliance** y gestionar el cluster.
- Practicar **vMotion, HA, DRS, Storage vMotion** — todo lo que pide el examen.
- Cuando termines de estudiar, cierras Workstation y sigues usando tu PC normalmente.

Si en el futuro adquieres un servidor físico viejo (un Dell PowerEdge o HP ProLiant de eBay), ahí sí instalarás ESXi nativo. Pero para aprender y certificarte, el enfoque anidado es el estándar de la industria.

------

## Resumen



| Pregunta                                    | Respuesta                                                    |
| :------------------------------------------ | :----------------------------------------------------------- |
| ¿ESXi 8.0U3e es un hypervisor?              | **Sí**, tipo 1 bare-metal. No es una aplicación.             |
| ¿Queda el equipo dedicado solo a ESXi?      | **Sí, si lo instalas nativo.** Reemplaza a Windows/Linux.    |
| ¿Puedo usar mi PC normal para estudiar?     | **Sí**, corriendo ESXi anidado dentro de VMware Workstation. |
| ¿Qué necesito para el lab de certificación? | **Workstation Pro + 2 VMs de ESXi 8.0U3e + 1 VM de vCenter.** |

------

¿Te gustaría que te explique paso a paso cómo **habilitar la virtualización anidada** en VMware Workstation para correr ESXi dentro de una VM, o prefieres que primero veamos cómo **instalar ESXi nativo** si tienes un equipo viejo dedicado para eso?

