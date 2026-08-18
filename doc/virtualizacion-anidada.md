# Acerca de la virtualización anidada

> Quiero crear una VM maestra con VMWare ESXi para manejar gestionar varias VMs y crear una plataforma para el Laboratorio VMware. 
>
> Es posible hacer esto en VMware Workstatio Pro?

Sí, es técnicamente posible crear una máquina virtual (VM) con VMware ESXi dentro de tu infraestructura actual para gestionar otras VM. Esta práctica se conoce como **virtualización anidada**.

Sin embargo, es crucial que conozcas los detalles importantes sobre su soporte y limitaciones antes de implementarlo.

### ⚙️ ¿Qué es la virtualización anidada y por qué hacerla?

La virtualización anidada consiste en ejecutar un hipervisor (como ESXi) dentro de una máquina virtual que se ejecuta en otro hipervisor (tu ESXi físico). Es decir, tu "ESXi anfitrión" físico hospedará una VM que, a su vez, será un "ESXi invitado" que hospedará sus propias VM.

- **¿Para qué sirve?** Es una herramienta excelente para **entornos de prueba, laboratorios, formación y demostraciones** de productos VMware. Permite a los administradores experimentar con configuraciones, simular entornos complejos (como un clúster de vSAN) o prepararse para certificaciones sin necesidad de hardware físico adicional.

### ❗ Puntos clave: Soporte y Limitaciones

1. **No tiene soporte oficial de Broadcom (antes VMware)**: Este es el punto más importante. Broadcom **no ofrece soporte técnico** para ejecutar ESXi como un sistema operativo invitado anidado en un entorno de producción. Es decir, si encuentras un problema, no podrás abrir un caso de soporte para resolverlo.
2. **Excepción**: La única excepción a esta regla es el **vSAN Witness Appliance** (un dispositivo testigo para clústeres vSAN de 2 nodos), que sí está soportado para ejecutarse como una VM anidada en un host ESXi físico.
3. **Rendimiento**: Al añadir una capa extra de virtualización, el rendimiento se verá afectado. El ESXi anidado compartirá los recursos de CPU, memoria y E/S con el hipervisor anfitrión, lo que introduce una sobrecarga.
4. **Es posible**: A pesar de no ser una configuración soportada, es perfectamente posible de realizar y es una práctica común en entornos no productivos.

### 💻 Requisitos técnicos para tu ESXi anidado

Para que la VM que alojará ESXi funcione correctamente, el host físico debe estar configurado para "exponer" la virtualización asistida por hardware a la VM invitada.

- **Configuración de la VM**: Durante la creación de la VM para el ESXi anidado, debes:
  - Marcar la opción **"Exponer virtualización asistida por hardware al sistema operativo invitado"** (o similar, como `Expose hardware assisted virtualization to the guest OS`).
  - Asignar al menos **2 vCPUs** y **4 GB de RAM** (aunque para un rendimiento aceptable y si planeas ejecutar vCenter, se recomiendan más recursos).
  - Usar los controladores recomendados para rendimiento: **VMXNET3** para la red y **SCSI Paravirtual** para el almacenamiento.
  - Configurar la VM con firmware **EFI**.
- **Versiones**: Existen requisitos de versiones para que funcione. Por ejemplo, para ESXi 8.0 anidado se necesita un host con ESXi 7.0 o posterior, y VMware Workstation 16.0 o posterior si se usara en un equipo de escritorio.

### 🚀 En resumen

Tu idea de crear una VM con ESXi para gestionar otras VM es factible y se usa mucho para **pruebas y aprendizaje**. Pero recuerda que **no es una configuración apta para producción ni tiene soporte oficial**. Si tu objetivo es un entorno de laboratorio para experimentar y formarte, es una excelente opción. Si lo que buscas es una solución de gestión robusta para producción, la vía correcta es usar **vCenter Server** sobre tus hosts ESXi físicos.

Si tu objetivo principal es la gestión centralizada de varios hosts ESXi, la herramienta diseñada para ello es **vCenter Server**, que se instala como una VM (o appliance) y no como otro hipervisor anidado.

