# Escenarios de  virtualización



> Consideraciones: 
>
> 1. Una persona que sabe de Linux pero no tiene mucha experiencia  trabajando en producción con virtualización utilizando VMware. 
> 2. Se necesita un entorno simulado para ensayar y reforzar conocimientos de administración de sistemas basados en Red Hat Linux Enterprise para optar a exámenes de certificaciones. 
> 3. Se dispone de un Hardware con limitaciones.



Vamos a construir tu mapa mental de virtualización Enterprise desde la perspectiva de alguien que ya domina Linux. No vamos a perder tiempo explicando qué es una VM; vamos directo a lo que importa cuando llegas a un datacenter real.

------

## 1. El cambio de paradigma: de tu laptop al datacenter

Sabes cómo funciona KVM/QEMU en tu máquina local. En producción con VMware, el concepto es similar pero la **arquitectura de control** cambia radicalmente:



| Tu entorno Linux                           | VMware Enterprise                                      |
| :----------------------------------------- | :----------------------------------------------------- |
| `libvirt` + `virt-manager`                 | **vCenter Server** (el cerebro)                        |
| QEMU/KVM como hypervisor                   | **ESXi** (el hypervisor bare-metal)                    |
| Archivos de config en `/etc/libvirt/qemu/` | Base de datos distribuida (vCenter)                    |
| Storage local o NFS simple                 | **VMFS**, **vSAN**, **datastores** compartidos         |
| Redes con bridges Linux (`virbr0`)         | **vSwitch**, **Port Groups**, **Distributed Switches** |

**Concepto clave:** En producción no tocas ESXi directamente para operar. Todo pasa por vCenter. Tocar un host ESXi individual es como hacer `ssh` a un nodo Kubernetes en vez de usar `kubectl` — funciona, pero es un anti-patrón.

------

## 2. ESXi: el hypervisor que necesitas entender

ESXi es un microkernel (basado en VMkernel, no es Linux). Esto es importante:

- **No hay shell bash nativo** (aunque puedes habilitar SSH para troubleshooting).
- Los drivers son específicos de VMware (VIBs — vSphere Installation Bundles).
- El filesystem es **VMFS** para datastores locales/compartidos.

### Comandos ESXi útiles (cuando el SSH está habilitado para emergencias):

```bash
# Ver datastores
esxcli storage filesystem list

# Ver VMs en ejecución
esxcli vm process list

# Ver información de red
esxcli network ip interface ipv4 get
```

> **Regla de oro en producción:** Si estás en SSH de ESXi, algo salió mal o estás haciendo algo que debería hacerse desde vCenter.

------

## 3. vCenter: tu "systemd" a escala datacenter

vCenter es la pieza que transforma un grupo de servidores físicos en una **nube de recursos computacionales**. Piensa en él como el `systemd` o el `kubernetes` de tu infraestructura virtual.

### Conceptos que debes dominar:

| Concepto                                 | Analogía Linux                | Qué hace en producción                                       |
| :--------------------------------------- | :---------------------------- | :----------------------------------------------------------- |
| **Cluster**                              | Un grupo de nodos             | Agrupa hosts ESXi; habilita HA y DRS                         |
| **HA (High Availability)**               | Pacemaker/Corosync            | Si un host físico muere, las VMs se reinician automáticamente en otro host |
| **DRS (Distributed Resource Scheduler)** | Un scheduler inteligente      | Mueve VMs en caliente (vMotion) entre hosts para balancear carga |
| **vMotion**                              | Live migration de KVM         | Mueve una VM de un host a otro **sin downtime**              |
| **Storage vMotion**                      | —                             | Mueve los discos de una VM entre datastores sin downtime     |
| **Snapshot**                             | LVM snapshot (pero peligroso) | Congela estado de VM. **Nunca dejes snapshots por días en producción.** |

------

## 4. Redes en VMware: sal del mundo de `brctl`

En Linux usas bridges y `iptables`. En VMware las redes son un universo aparte:

### Componentes:

- **vSwitch (Standard)**: Creado en cada host ESXi. Equivalente a un bridge Linux.
- **Port Group**: Subdivisión del vSwitch con una VLAN específica. Es donde conectas las VMs.
- **Distributed Switch (vDS)**: El vSwitch centralizado gestionado desde vCenter. En producción grande, es el estándar.

### Analogía mental:



```plain
vDS (Distributed Switch)  =  Un switch físico gestionado centralmente
Port Group                =  Una VLAN en ese switch
VMKernel NIC (vmk)        =  La interfaz de management del hypervisor
```

**vmkernel ports (vmk):** Son interfaces especiales del hypervisor (no de las VMs) para:

- Management (vCenter se comunica aquí)
- vMotion (migración en caliente)
- Storage (iSCSI, NFS)
- Fault Tolerance (FT)

------

## 5. Almacenamiento: donde la mayoría de los problemas empiezan

En producción, el storage es el cuello de botella #1.

### Tipos de datastores que verás:



| Tipo     | Qué es                          | Caso de uso                                 |
| :------- | :------------------------------ | :------------------------------------------ |
| **VMFS** | Filesystem de bloques de VMware | LUNs de SAN (FC, iSCSI, SAS)                |
| **NFS**  | Export NFS montado en ESXi      | NAS, backups, entornos de desarrollo        |
| **vSAN** | Storage definido por software   | Convergido; usa discos locales de los hosts |

### Concepto crítico: **Multipathing**

Cuando un host ESXi tiene múltiples paths a un storage (por redundancia), VMware usa **PSP** (Path Selection Policies):

- **MRU** (Most Recently Used): Failover simple
- **Fixed**: Siempre usa el path preferido
- **Round Robin**: Balanceo entre paths (el más común en producción)

> **Consejo práctico:** Si ves latencia de storage, lo primero que revisas es si el multipathing está bien configurado.

------

## 6. Las VMs en producción: lo que cambia desde tu VM de escritorio

### VMware Tools

Es el equivalente a los `guest agents` de QEMU. **Siempre deben estar instalados y actualizados.** Proporcionan:

- Drivers paravirtualizados (vmxnet3, pvscsi)
- Heartbeat para HA
- Sincronización de tiempo
- Shutdown graceful

En Linux, se instalan como un paquete RPM/DEB (`open-vm-tools` en repos oficiales).



### Hardware virtual recomendado para Linux en producción:

| Dispositivo     | Recomendación                      | Por qué                                                  |
| :-------------- | :--------------------------------- | :------------------------------------------------------- |
| NIC             | **vmxnet3**                        | Paravirtualizado, mejor rendimiento que e1000            |
| SCSI Controller | **PVSCSI**                         | Menor overhead, mejor para alta I/O                      |
| Disco           | **Thin provisioned** con monitoreo | Ahorra espacio, pero vigila que no se llene el datastore |

------

## 7. Operaciones diarias: el mindset de producción

### ✅ Do's:

- **Templates:** Crea templates de VMs (Golden Images) con tus configs base. Nunca instales desde ISO en producción.
- **Resource Pools:** Agrupa VMs por proyecto/equipo y limita recursos (CPU/Memoria) para evitar que una VM desborde a las demás.
- **Alarmas y eventos:** Configura alertas en vCenter (uso de datastore > 80%, snapshots viejos, etc.).
- **Maintenance Mode:** Antes de cualquier trabajo en un host ESXi, ponlo en Maintenance Mode. vCenter migrará las VMs automáticamente (si hay DRS/HA).

### ❌ Don'ts:

- **Snapshots como backup:** Un snapshot no es backup. Es un punto de restauración temporal. En producción, los snapshots degradan rendimiento y consumen espacio rápidamente.
- **Overcommit sin control:** VMware permite overcommit de RAM (dar más memoria a VMs de la que tienes física). Funciona gracias a **memory ballooning** y **swap**, pero en producción esto mata el rendimiento si te pasas.
- **Ignorar los logs:** `vmkernel.log`, `hostd.log`, `vpxa.log` — aprende dónde están.

------

## 8. Troubleshooting básico (el "sysadmin" de VMware)

### Problema: VM no arranca

1. ¿Hay recursos disponibles en el host? (CPU/RAM reservada)
2. ¿El datastore tiene espacio?
3. ¿Hay un snapshot bloqueando?

### Problema: Red lenta en VM Linux

1. ¿Usa `vmxnet3` o el viejo `e1000`?
2. ¿El Port Group tiene la VLAN correcta?
3. ¿Hay packet drops en el vSwitch? (`esxtop` para ver en tiempo real)

### Problema: Storage lento

1. Revisa latencia en vCenter (Performance > Advanced > Datastore)
2. Revisa multipathing (`esxcli storage nmp device list`)
3. Revisa queues de disco en la VM

------

## 9. Tu roadmap de aprendizaje sugerido



| Semana | Enfoque                                                      |
| :----- | :----------------------------------------------------------- |
| 1      | Arquitectura vCenter + ESXi. Navega la UI de vSphere Client. Crea una VM desde template. |
| 2      | Redes: Crea Port Groups, entiende VLANs en vSwitch, configura un Distributed Switch de prueba. |
| 3      | Storage: Monta un datastore NFS, crea uno iSCSI, juega con Storage vMotion. |
| 4      | Features enterprise: HA, DRS, vMotion. Simula un fallo de host físico. |
| 5      | Linux en VMware: Instala open-vm-tools, optimiza con vmxnet3/pvscsi, prueba clones y templates. |
| 6      | Monitoreo y troubleshooting: `esxtop`, alarmas en vCenter, análisis de logs. |

------

## ¿Por dónde quieres empezar?

Te propongo dos caminos:

1. **Arquitectura y conceptos:** Profundizamos en cómo se diseña un cluster de producción desde cero (networking, storage, HA/DRS).
2. **Manos a la obra:** Simulamos escenarios operativos (deploy de VM Linux, troubleshooting de red, migración con vMotion).



### Esta información a donde nos conduce



> ¿Cuál  interesa más? ¿O hay algún concepto específico de los que mencioné que quieras que desarrolle primero?



> Interrogantes que se plantean un aspirante durante la formación:
>
> 1. Cómo se llama la aplicación o sistema que se debe instalar para comenzar a virtualizar.
> 2.  Comparativa con VMware workstation Pro.
> 3.  Puedo aprender con Workstation lo principal y tener el conocimiento adecuado para trabajar en producción con VMWare?



¡Excelentes preguntas! Son exactamente las que debe hacer alguien que viene de Linux y quiere entender el ecosistema VMware sin perder tiempo. Vamos una por una.

------

## 1. ¿Qué instalar para comenzar a virtualizar con VMware?

Depende del **contexto**. VMware tiene productos para distintos niveles:



| Escenario                                     | Producto                              | ¿Qué es?                                        |
| :-------------------------------------------- | :------------------------------------ | :---------------------------------------------- |
| **Tu laptop / escritorio**                    | **VMware Workstation Pro** (o Player) | Hypervisor tipo 2 (corre sobre Windows/Linux)   |
| **Servidor físico dedicado**                  | **VMware ESXi**                       | Hypervisor tipo 1 bare-metal (el de producción) |
| **Gestión centralizada de muchos servidores** | **VMware vCenter Server** (VCSA)      | La consola web que controla múltiples ESXi      |

### Para empezar a aprender en casa:

Instala **VMware Workstation Pro** en tu Linux/Windows. Es el equivalente a VirtualBox pero profesional.

### Para simular producción en casa:

Puedes instalar **ESXi** dentro de una VM de Workstation (nested virtualization). Así practicas con el hypervisor real de producción sin hardware dedicado.

> **Nota:** ESXi es gratuito con una licencia gratuita (VMware vSphere Hypervisor), pero tiene limitaciones (sin vMotion, sin HA, sin backups integrados). Para aprender, es suficiente.

------

## 2. Comparativa: Workstation Pro vs. ESXi + vCenter (Producción)



| Aspecto                             | **VMware Workstation Pro**            | **ESXi + vCenter (Producción)**                     |
| :---------------------------------- | :------------------------------------ | :-------------------------------------------------- |
| **Tipo de hypervisor**              | Tipo 2 (corre sobre SO anfitrión)     | Tipo 1 (bare-metal, directo al hardware)            |
| **Rendimiento**                     | Bueno, pero con overhead del SO host  | Máximo rendimiento, control total del hardware      |
| **Gestión**                         | Interfaz gráfica local en tu máquina  | vSphere Client (web) o vCenter (centralizado)       |
| **Alta disponibilidad (HA)**        | ❌ No existe                           | ✅ Si un host muere, las VMs migran solas            |
| **Migración en caliente (vMotion)** | ❌ No existe                           | ✅ Mueve VMs sin apagarlas entre hosts               |
| **Balanceo automático (DRS)**       | ❌ No existe                           | ✅ Distribuye VMs automáticamente según carga        |
| **Redes avanzadas**                 | Bridges NAT básicos                   | vSwitches, Port Groups, VLANs, Distributed Switches |
| **Storage compartido**              | Discos locales o carpetas compartidas | VMFS, iSCSI, FC, NFS, vSAN                          |
| **Snapshots**                       | ✅ Sí (pero no dejes crecer)           | ✅ Sí (misma advertencia)                            |
| **Licenciamiento**                  | Pago por usuario (~$200)              | Por socket de CPU (caro en producción)              |
| **Uso típico**                      | Desarrollo, testing, labs personales  | Datacenters, cloud privado, enterprise              |

### Analogía útil:

- **Workstation** = Tu `virt-manager` o VirtualBox local. Perfecto para jugar.
- **ESXi + vCenter** = Tu cluster de Kubernetes, pero para VMs. Es infraestructura diseñada para no fallar nunca.

------

## 3. ¿Puedo aprender con Workstation y luego saltar a producción?

**Sí, pero con reservas importantes.** Te explico qué sí te aporta Workstation y qué te va a faltar:

### ✅ Lo que Workstation te enseña bien:



| Concepto                           | ¿Aplica en producción?                           |
| :--------------------------------- | :----------------------------------------------- |
| Crear/configurar VMs               | ✅ Igual                                          |
| Instalar VMware Tools              | ✅ Igual                                          |
| Asignar recursos (CPU, RAM, disco) | ✅ Igual                                          |
| Concepto de snapshots              | ✅ Igual (pero con más cuidado en producción)     |
| Redes NAT/Bridged/Host-only        | ⚠️ Parcial — en producción usas vSwitches y VLANs |
| Discos thin/thick provisioned      | ✅ Igual                                          |
| Templates y clones                 | ✅ Igual (aunque en vCenter es más potente)       |

### ❌ Lo que Workstation NO te enseña (y es crítico en producción):



| Concepto                                      | Por qué importa en producción                                |
| :-------------------------------------------- | :----------------------------------------------------------- |
| **vCenter Server**                            | Es el cerebro. Sin él no hay HA, DRS, vMotion.               |
| **Clusters de ESXi**                          | Nunca trabajas con un solo host en producción.               |
| **vMotion / Storage vMotion**                 | Migración en caliente es el pan de cada día.                 |
| **HA (High Availability)**                    | Si un host físico se muere a las 3 AM, ¿qué pasa?            |
| **DRS**                                       | Balanceo automático de carga entre hosts.                    |
| **Redes distribuidas (vDS)**                  | En producción grande no configuras switches host por host.   |
| **Almacenamiento compartido (iSCSI/FC/vSAN)** | Las VMs viven en storage compartido, no en discos locales.   |
| **Resource Pools**                            | Control de cuotas y prioridades entre equipos/proyectos.     |
| **Permisos y RBAC**                           | En producción, no todos son admin.                           |
| **Backups (VADP / Veeam)**                    | En producción necesitas backups de VMs en caliente.          |
| **Licenciamiento y soporte**                  | vSphere tiene niveles: Standard, Enterprise, Enterprise Plus. |

### Mi recomendación honesta como tutor:

> **Workstation te da los fundamentos de "cómo funciona una VM". ESXi + vCenter te enseñan "cómo se opera infraestructura virtual a escala".**

#### Ruta de aprendizaje que sugiero:



| Fase                        | Herramienta                                       | Objetivo                                                     |
| :-------------------------- | :------------------------------------------------ | :----------------------------------------------------------- |
| **1. Fundamentos**          | Workstation Pro                                   | Entender VMs, snapshots, redes básicas, VMware Tools.        |
| **2. Hypervisor real**      | ESXi en Workstation (nested)                      | Familiarizarte con la interfaz de ESXi, datastores, vmkernel. |
| **3. Gestión centralizada** | vCenter Server Appliance (VCSA) en Workstation    | Aprender clusters, HA, DRS, vMotion, Distributed Switches.   |
| **4. Producción**           | Laboratorio con 2-3 hosts ESXi físicos o en cloud | Simular escenarios reales: fallo de host, migraciones, storage. |

#### Recursos gratuitos para practicar la parte "enterprise":

- **VMware Hands-On Labs (HOL):** Labs gratuitos en navegador con infraestructura real de VMware. Puedes tocar vCenter, HA, vMotion, etc. sin instalar nada.
- **Nested ESXi:** Descarga la ISO de ESXi gratis y correla dentro de Workstation. Así practicas con el hypervisor real.
- **vCenter Server Appliance (VCSA):** También puedes desplegarlo virtualizado para aprender.

------

## Resumen ejecutivo



| Pregunta                                         | Respuesta corta                                              |
| :----------------------------------------------- | :----------------------------------------------------------- |
| ¿Qué instalo para empezar?                       | **VMware Workstation Pro** para tu laptop.                   |
| ¿Workstation vs. Producción?                     | Workstation = hypervisor de escritorio. Producción = ESXi + vCenter + clusters + storage compartido. |
| ¿Puedo aprender con Workstation para producción? | **Sí, para los fundamentos. No, para la operación enterprise.** Necesitarás tocar ESXi + vCenter antes de presentarte a una entrevista o a un entorno real. |

------

## ¿Qué sigue?

Te propongo que elijas tu camino:

1. **"Quiero empezar ya con Workstation"** → Te guío paso a paso a instalar Workstation en Linux, crear tu primera VM, y luego hacer nested ESXi.
2. **"Quiero saltar directo a lo enterprise"** → Te explico cómo montar un lab con ESXi + vCenter anidados en Workstation, o cómo usar los VMware Hands-On Labs gratuitos.
3. **"Quiero entender la arquitectura de producción primero"** → Dibujamos juntos cómo se ve un cluster real (redes, storage, hosts, vCenter).

¿Cuál te resuena más?



