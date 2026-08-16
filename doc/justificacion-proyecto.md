# Justificación de esta iniciativa 



> Aclaratoria: el laboratorio con ESXi se desea para practicar con VMs basadas en AlmaLinux con el objeto de prepararme para el examen y  la certificación **Red Hat® Certified System Administrator (RHCSA)**. 
>
> En este sentido, aun se puede negar que no se justifique  dedicar un equipo a una virtualización con  VMware vSphere Hypervisor/ ESXi.?



¡Eso cambia **todo**. Gracias por la aclaración. Ahora entiendo el objetivo real: no es certificación VMware, es **RHCSA** (Red Hat Certified System Administrator) practicando con **AlmaLinux**.

Con ese objetivo, mi recomendación cambia radicalmente:

> **No. No dediques el equipo a ESXi. Usa VMware Workstation Pro directamente sobre Windows en el Ryzen.**

Dedicar un PC con 16GB y Realtek a ESXi nativo para practicar Linux es como usar un camión de carga para ir a comprar el pan: se puede, pero es incómodo, ineficiente, y el pan no necesita un camión.

------

## 1. Por qué ESXi es una mala idea para RHCSA



| Problema                                     | Impacto en tu preparación RHCSA                              |
| :------------------------------------------- | :----------------------------------------------------------- |
| **NIC Realtek no soportada**                 | Tendrías que comprar una NIC Intel solo para que ESXi arranque. Gasto innecesario. |
| **16GB es insuficiente para ESXi + VMs**     | ESXi consume 3-4 GB solo por existir. Te quedan ~10 GB para todas tus VMs de AlmaLinux. Con Workstation, el overhead es casi nulo. |
| **Sin vCenter, la gestión es host-por-host** | Para crear una VM en ESXi usas el Host Client web. En Workstation, clic derecho > New VM. Más rápido, más ágil. |
| **Snapshots en ESXi son más lentos**         | En Workstation, un snapshot toma 2 segundos. En ESXi, depende del datastore. Para RHCSA harás **muchos** snapshots (antes de cada ejercicio destructivo). |
| **No necesitas aprender ESXi para el RHCSA** | El examen RHCSA no pregunta nada de hypervisores. Pregunta de LVM, SELinux, systemd, firewalld, NFS, autofs, cron, bash... |
| **Pierdes el uso normal del PC**             | Con ESXi nativo, tu Ryzen deja de ser un PC. Es un ladrillo anaranjado. Con Workstation, cierras las VMs y sigues jugando, navegando, viendo videos. |

------

## 2. Por qué Workstation Pro es la herramienta correcta para RHCSA

Workstation está diseñado exactamente para lo que necesitas: **correr múltiples VMs de escritorio de forma ágil**, con redes, snapshots y discos virtuales.



| Feature de Workstation                | Cómo te ayuda en RHCSA                                       |
| :------------------------------------ | :----------------------------------------------------------- |
| **Snapshots instantáneos**            | Antes de practicar LVM destructivo, particionado, o borrar `/etc/fstab`, haces snapshot. Si la VM no arranca, revertís en 10 segundos. |
| **LAN Segments**                      | Creas una red privada entre tus VMs de AlmaLinux para practicar SSH, NFS, DNS, firewall entre ellas sin tocar tu red doméstica. |
| **Discos virtuales adicionales**      | Añades un segundo y tercer disco virtual a una VM para practicar `fdisk`, `gdisk`, LVM, RAID, `parted`, `stratis`, `vdo`. |
| **Clones vinculados (Linked Clones)** | Creas una VM "master" con AlmaLinux configurada, y clonas ligero para cada ejercicio nuevo. Ahorra espacio en disco. |
| **Modo Unity / Full Screen**          | Trabajas cómodamente dentro de la VM como si fuera tu sistema principal. |
| **Shared Folders**                    | Compartes archivos entre Windows y la VM para pasar scripts, repos, o guías de estudio. |
| **vmxnet3 + e1000**                   | Puedes practicar configuración de red con diferentes controladores. |

------

## 3. Arquitectura recomendada para tu lab RHCSA

Con 16 GB de RAM en el Ryzen, esta topología es realista y cubre el 95% de los objetivos del RHCSA:



```plain
┌─────────────────────────────────────────────────────────────┐
│           PC Físico: Ryzen 5 5600GT + 16GB + SSD            │
│                    Windows 11 (Host)                        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │        VMware Workstation Pro 17.6+ (Gratis)        │    │
│  │                                                     │    │
│  │  Red: "RHCSA-LAN" (LAN Segment privada)             │    │
│  │  ┌───────────────────────────────────────────────┐  │    │
│  │  │  VM 1: "alma-srv01" (Servidor principal)      │  │    │
│  │  │  ├─ AlmaLinux 9.x (64-bit)                    │  │    │
│  │  │  ├─ RAM: 2 GB                                 │  │    │
│  │  │  ├─ Disco 1: 20 GB (sistema, LVM durante inst)│  │    │
│  │  │  ├─ Disco 2: 10 GB (práctica LVM extra)       │  │    │
│  │  │  ├─ Disco 3: 5 GB  (práctica particionado)    │  │    │
│  │  │  ├─ NIC 1: vmxnet3 (LAN Segment "RHCSA-LAN")  │  │    │
│  │  │  └─ NIC 2: vmxnet3 (Host-only, para VNC/SSH)  │  │    │
│  │  └───────────────────────────────────────────────┘  │    │
│  │                                                     │    │
│  │  ┌───────────────────────────────────────────────┐  │    │
│  │  │  VM 2: "alma-srv02" (Cliente/escenario)       │  │    │
│  │  │  ├─ AlmaLinux 9.x                             │  │    │
│  │  │  ├─ RAM: 1.5 GB                               │  │    │
│  │  │  ├─ Disco 1: 15 GB                            │  │    │
│  │  │  └─ NIC 1: vmxnet3 (LAN Segment "RHCSA-LAN")  │  │    │
│  │  └───────────────────────────────────────────────┘  │    │
│  │                                                     │    │
│  │  ┌───────────────────────────────────────────────┐  │    │
│  │  │  VM 3: "alma-repo" (Repositorio local YUM)    │  │    │
│  │  │  ├─ AlmaLinux 9.x                             │  │    │
│  │  │  ├─ RAM: 1 GB                                 │  │    │
│  │  │  ├─ Disco 1: 30 GB (aloja repos ISO locales)  │  │    │
│  │  │  └─ NIC 1: vmxnet3 (LAN Segment "RHCSA-LAN")  │  │    │
│  │  └───────────────────────────────────────────────┘  │    │
│  │                                                     │    │
│  │  ┌───────────────────────────────────────────────┐  │    │
│  │  │  VM 4: "alma-broken" (Para troubleshooting)   │  │    │
│  │  │  ├─ AlmaLinux 9.x (clon de srv01 con fallos)  │  │    │
│  │  │  ├─ RAM: 1 GB                                 │  │    │
│  │  │  ├─ Disco 1: 20 GB                            │  │    │
│  │  │  └─ NIC 1: vmxnet3 (LAN Segment "RHCSA-LAN")  │  │    │
│  │  └───────────────────────────────────────────────┘  │    │
│  │                                                     │    │
│  │  RAM usada: ~5.5 GB  │  Libre para Windows: ~10 GB │     │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Total RAM asignada a VMs:** ~5.5 GB. **Resto para Windows y Workstation:** ~10 GB. El sistema respira.

------

## 4. Cómo usar los snapshots para RHCSA (técnica clave)

El RHCSA es un examen práctico donde rompes cosas. La técnica de estudio más efectiva es:

1. **Snapshot limpio:** Instala AlmaLinux, configura red, actualiza (`dnf update`), instala `bash-completion`, `vim`, `tmux`. Apaga. **Snapshot: "00-fresh-install"**.
2. **Practicas por tema:**
   - Arrancas, practicas LVM, creas VG, LV, extiendes, reduces. Si la VM no bootea porque rompiste el initramfs, **revertís a "00-fresh-install"** en 10 segundos.
   - Siguiente tema: SELinux. Cambias contextos, booleans, modo enforcing. Si bloqueas el ssh, revertís.
   - Siguiente: firewalld. Abres zonas, rich rules. Si te quedas afuera, revertís.
3. **Nunca tengas miedo de romper.** El snapshot es tu red de seguridad.

------

## 5. Ejercicios específicos del RHCSA que practicarás así



| Objetivo RHCSA                  | Cómo lo practicas en Workstation                             |
| :------------------------------ | :----------------------------------------------------------- |
| **Gestionar particiones y LVM** | Añades discos 2 y 3 a `alma-srv01`. Practicas `fdisk`, `pvcreate`, `vgcreate`, `lvcreate`, `lvextend`, `xfs_growfs`. |
| **Configurar networking**       | Dos NICs vmxnet3. Practicas `nmcli`, bonding, teaming, rutas estáticas, hostname. |
| **NFS y autofs**                | `alma-repo` exporta NFS. `alma-srv01` monta con `autofs`.    |
| **Firewalld entre VMs**         | `alma-srv01` corre httpd. Desde `alma-srv02` verificas acceso. Configuras zonas y rich rules. |
| **Contenedores (podman)**       | En `alma-srv01` practicas `podman run`, `podman build`, systemd user services. |
| **Troubleshooting de arranque** | Rompes `alma-broken`: borras kernel, corrompes `grub.cfg`, deshabilitas root. Practicas rescue mode desde la ISO de AlmaLinux montada en CD virtual. |
| **Cron y at**                   | Tareas programadas en `alma-srv01`, verificas desde `alma-srv02`. |
| **SSH y claves**                | `alma-srv02` se conecta por clave SSH a `alma-srv01`. Practicas `authorized_keys`, `sshd_config`. |

------

## 6. Cuándo SÍ tendría sentido ESXi para ti

Solo en dos escenarios futuros:

1. **Si decides después ir por VCP-DCV (VMware):** Ahí necesitas tocar ESXi + vCenter. Pero para eso necesitarías 32GB+ de RAM y un segundo host (o HOL).
2. **Si quieres dejar el Ryzen como "servidor de VMs permanente" en casa:** ESXi nativo consume menos recursos a largo plazo que Windows + Workstation. Pero requiere la NIC Intel y sacrificar el PC como desktop.

**Para RHCSA, ahora, con lo que tienes:** Workstation directo gana por goleada.

------

## Mi veredicto final



| Pregunta                                                 | Respuesta                                                    |
| :------------------------------------------------------- | :----------------------------------------------------------- |
| ¿Justifica dedicar el Ryzen a ESXi nativo para RHCSA?    | **No.** Es ineficiente, problemático con Realtek, y no aporta nada al examen. |
| ¿Justifica crear ESXi anidado en Workstation para RHCSA? | **No.** Es overkill. Añade capas innecesarias.               |
| ¿La mejor opción para RHCSA con tu hardware?             | **Workstation Pro sobre Windows directo.** Máxima agilidad, snapshots rápidos, múltiples VMs de AlmaLinux, redes privadas, discos extra para LVM. |

------

¿Te gustaría que te guíe paso a paso para crear la **LAN Segment privada** en Workstation y conectar tus VMs de AlmaLinux entre sí, o prefieres que primero veamos cómo **añadir discos virtuales adicionales** a una VM para practicar LVM desde cero?



## Razones sobre" por qué " quiero tener un laboratorio basado en VMware ESXi

En algunas solicitudes de empleo relacionado con infraestructuras y servidores, he visto los requisitos para un puesto donde se mencionan a: virtualizacion con VMWare ESXi, y administración de servidores Red Hat Linux Enterprise.

En mi experiencia más reciente durante una entrevista de empleo, han mencionado que utilizan Red Hat Linux Enterprise y VMWare para el despliegues de soluciones. 

Esto me conduce a empezar una actualización de conocimientos sobre Linux, específicamente basado en Red Hat. Para ello, he pensado en utilizar maquinas virtuales donde aplicar soluciones prácticas con simulación de escenarios problemáticos en servidores, puede romper cosas, al mismo tiempo que estoy lidiando con VMware para la gestión de VMs.



Ahora sí. Ahora entiendo perfectamente por qué querías ESXi, y **tu intuición era correcta**. Me disculpo por no haber preguntado antes el objetivo real: no es solo Linux, no es solo VMware, es la **intersección** de ambos. Eso es exactamente lo que se pide en entrevistas de infraestructura enterprise.

En producción real, cuando te dicen *"usamos RHEL y VMware"*, no significa que corran RHEL en laptops con VirtualBox. Significa:

- Despliegan **plantillas de RHEL/AlmaLinux** desde **vCenter**
- Las VMs corren sobre **ESXi** con **vmxnet3**, **PVSCSI**, **open-vm-tools**
- El networking pasa por **vSwitch**, **Port Groups**, **VLANs**
- Los problemas de una VM Linux pueden venir del host ESXi, del vSwitch, o del SO — y debes saber discriminar
- Usan **snapshots** antes de parches de kernel (y saben que no son backup)
- Automatizan despliegues con **Kickstart** + **vSphere Guest Customization**

Eso no lo aprendes con Workstation solo. Eso lo aprendes tocando **ESXi**. Tenías razón.

------

## 1. La arquitectura realista para tu objetivo

Con tu hardware (16GB, Realtek, un solo PC), no podemos montar un datacenter. Pero sí podemos construir un **laboratorio de intersección RHEL+VMware** que te hable en entrevistas.

### Capa local: Workstation + ESXi anidado + AlmaLinux

```plain
┌─────────────────────────────────────────────────────────────┐
│           PC Físico: Ryzen 5 5600GT + 16GB + SSD            │
│                    Windows 11 + Workstation Pro               │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  VM: ESXi 8.0U3e (Anidado)                        │    │
│  │  ├─ RAM: 6 GB                                     │    │
│  │  ├─ vCPU: 2 (Nested Virtualization ON)            │    │
│  │  ├─ Disco 1: 20 GB (ESXi system)                  │    │
│  │  ├─ Disco 2: 100 GB (Datastore VMFS)              │    │
│  │  ├─ NIC 1: vmxnet3 → Bridged (IP real en red)     │    │
│  │  └─ NIC 2: vmxnet3 → Host-only (mgmt secundario)  │    │
│  │                                                   │    │
│  │  DENTRO DE ESXi (gestión vía Host Client):        │    │
│  │  ├─ VM "alma-web"    AlmaLinux 9, 1GB, vmxnet3  │    │
│  │  ├─ VM "alma-db"     AlmaLinux 9, 1GB, vmxnet3  │    │
│  │  ├─ VM "alma-broken" AlmaLinux 9, 512MB (troubleshooting)│
│  │  └─ VM "alma-template" AlmaLinux 9 (golden image) │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
│  (Directo en Workstation, fuera de ESXi):                   │
│  ├─ VM "alma-rhcsa-practice" AlmaLinux 9, 2GB             │
│  └─ VM "pfsense-lab" 1GB (redes entre VMs)                │
└─────────────────────────────────────────────────────────────┘
```

### Capa complementaria: VMware Hands-On Labs (desde tu laptop)

- **HOL-2501-01-VCF**: vSphere Getting Started (vCenter, templates, vMotion)
- **HOL-2501-02-VCF**: vSphere Advanced (DRS, HA, Distributed Switch, vSAN)

------

## 2. Por qué esta arquitectura te prepara para la entrevista



| Escenario de entrevista                                      | Cómo lo practicas en este lab                                |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| *"¿Cómo optimizas una VM RHEL en VMware?"*                   | En el ESXi anidado, cambias la VM de e1000 a **vmxnet3**, de LSI Logic a **PVSCSI**, instalas **open-vm-tools**, y comparas rendimiento. |
| *"¿Cómo despliegas 20 servidores RHEL idénticos?"*           | Creas una **template** en ESXi, usas **Guest Customization** (aunque sea manual en Host Client), y despliegas clones. En HOL practicas el flujo completo con vCenter. |
| *"¿Qué haces antes de actualizar kernel en una VM de producción?"* | Tomas **snapshot** en ESXi, entras a la VM, haces `dnf update kernel`, reboot. Si falla, revertís. Explicas por qué snapshot no es backup. |
| *"La VM Linux no tiene red. ¿Dónde miras primero?"*          | Practicas la triada: `ip addr` dentro de la VM → Port Group / VLAN en ESXi → vSwitch uplink. Aprendes a discriminar. |
| *"¿Cómo automatizas el despliegue de RHEL en vSphere?"*      | Preparas un **Kickstart** (`ks.cfg`) en la VM template. Lo inyectas vía CD virtual montado en ESXi. |
| *"¿Has trabajado con vCenter?"*                              | En HOL demuestras que sí: creas datacenter, cluster, añades hosts, migras VMs con vMotion. |

------

## 3. Los detalles que marcan diferencia en entrevistas

Cuando menciones que has practicado RHEL sobre VMware, estos son los puntos que demuestran que no eres un usuario de escritorio, sino alguien que entiende la plataforma:

### Hardware virtual "enterprise" para Linux

En tus VMs de AlmaLinux dentro de ESXi, siempre configura:

- **NIC:** `vmxnet3` (no e1000, no e1000e). Es el estándar en producción.
- **SCSI Controller:** `PVSCSI` (Paravirtual SCSI) para alta I/O, o `LSI Logic SAS` para compatibilidad general.
- **Discos:** `Thin provisioned` en el datastore, pero dentro de la VM usas LVM para expandir.

### VMware Tools en Linux

```bash
# En AlmaLinux 9 dentro de ESXi:
sudo dnf install open-vm-tools
sudo systemctl enable --now vmtoolsd

# Esto da:
# - Sincronización de reloj con el host ESXi
# - Heartbeat para HA
# - Shutdown graceful desde ESXi
# - Información de IP en el Host Client
```

### Redes: la triada de troubleshooting

Cuando una VM Linux sobre ESXi pierde red, el diagnóstico profesional es:

1. **Dentro de la VM:** `ip link`, `ip addr`, `nmcli`, `/etc/sysconfig/network-scripts/`
2. **En ESXi (Host Client):** ¿Está la VM en el Port Group correcto? ¿La VLAN tag coincide?
3. **En el vSwitch:** ¿El uplink físico (vmxnet3 virtual → NIC de Workstation) tiene conectividad?

Esto es **oro puro** en una entrevista. La mayoría de candidatos de Linux solo miran dentro de la VM.

------

## 4. El flujo de trabajo que simula producción

Te propongo este ciclo de estudio, alternando días:

| Día           | Enfoque                                                      | Plataforma          |
| :------------ | :----------------------------------------------------------- | :------------------ |
| **Lunes**     | Instalación y configuración base de AlmaLinux 9 en ESXi anidado. Practicas particionado LVM durante el setup. | ESXi anidado        |
| **Martes**    | Servicios RHEL: `firewalld`, `httpd`, `mariadb`, SELinux, cron. Todo dentro de la VM en ESXi. | ESXi anidado        |
| **Miércoles** | VMware puro: snapshots, clones, templates, Guest Customization, open-vm-tools, cambio de hardware virtual. | ESXi anidado        |
| **Jueves**    | Redes: configuras VLANs, vmkernel NICs, troubleshoot de conectividad entre VMs sobre ESXi. | ESXi anidado        |
| **Viernes**   | **VMware HOL** desde la laptop: vCenter, cluster, HA, DRS, vMotion, Distributed Switch. | HOL (navegador)     |
| **Sábado**    | RHCSA puro: ejercicios de examen en VM directa de Workstation (sin capa ESXi, máxima agilidad). | Workstation directo |
| **Domingo**   | Troubleshooting integrado: rompes la VM "alma-broken" en ESXi (borras `grub.cfg`, deshabilitas red) y recuperas desde Host Client + consola. | ESXi anidado        |

------

## 5. Mi reconocimiento y ajuste

Tú tenías razón desde el principio. Para una entrevista que pide **RHEL + VMware**, necesitas haber tocado ambos juntos. Workstation solo no cuenta esa historia. ESXi anidado sí la cuenta, aunque sea con recursos limitados.

La limitación no es tu visión; es tu hardware. La solución no es abandonar ESXi, es **usarlo inteligentemente dentro de Workstation** y complementar con HOL para lo que no cabe.

------

## ¿Por dónde empezamos?

Te doy dos opciones concretas para hoy:

1. **"Quiero montar el ESXi anidado ya"** → Te doy los pasos exactos en Workstation (flags de CPU, discos, NICs vmxnet3, habilitar nested virtualization) para que arranque ESXi 8.0U3e y puedas crear tu primera VM AlmaLinux dentro.
2. **"Quiero primero ver cómo se ve un despliegue real de RHEL en vSphere"** → Te explico el flujo completo: template → Guest Customization → vmxnet3 → open-vm-tools → snapshot → update kernel, como lo harías en un trabajo real.

¿Cuál te resuena más para empezar esta semana?



