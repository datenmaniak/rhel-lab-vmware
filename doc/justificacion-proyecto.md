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



