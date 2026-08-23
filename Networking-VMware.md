# Arquitectura de Red en VMware ESXi (Breve Explicación)



El modelo de red de ESXi se basa en dos componentes principales virtuales que emulan el hardware físico:

1. **Virtual Switches (vSwitches):** Funcionan de manera similar a un switch físico de Capa 2. Conectan las máquinas virtuales a las redes físicas del servidor (a través de las tarjetas de red físicas o *vmnics*) y también permiten la comunicación interna entre VMs.
2. **Port Groups (Grupos de Puertos):** Son los "puertos" lógicos del vSwitch. Definen políticas de seguridad, VLANs y asignan etiquetas de red (como las VLAN IDs). Las máquinas virtuales no se conectan directamente al vSwitch, sino a un **Port Group**. Por defecto, el vSwitch principal trae el grupo llamado `VM Network`.



### 1) Reconfigurar / Recuperar la red de la VM `alma-srv01`

Dado que la VM estaba en la misma red que el hipervisor (`192.168.1.0/24`), lo más probable es que necesites asignarle un puerto de red adecuado en el ESXi y, si perdió la configuración IP, ajustarla dentro del sistema operativo (AlmaLinux).

#### Paso A: Verificación en el ESXi (vSphere Client / Host Client)

1. Inicia sesión en la interfaz web de tu ESXi.
2. Ve a **Virtual Machines** y selecciona `alma-srv01`.
3. Haz clic en **Edit** (Editar ajustes).
4. Busca el adaptador de red (**Network Adapter**). Asegúrate de que esté conectado (*Connected* / *Connect at power on*) y que el **Network Label** (Port Group) esté apuntando al vSwitch correcto (por lo general, `VM Network`).

#### Paso B: Configuración dentro de AlmaLinux (`alma-srv01`)

Si la interfaz perdió su configuración de red estática o DHCP, conéctate a la consola de la VM y usa `NetworkManager` (`nmtui`) o edita la conexión directamente:

```
# Interfaz interactiva recomendada para AlmaLinux
sudo nmtui
```

- Selecciona **Edit a connection**, elige tu interfaz (ej. `eth0` o `ens192`) y configura:
  - **IPv4 CONFIGURATION:** Manual
  - **Addresses:** `192.168.1.X/24` (la IP que tenía la VM)
  - **Gateway:** `192.168.1.1` (IP de tu pfSense/router)
  - **DNS Servers:** `192.168.1.1` (o el que uses en tu red)
- Sal de `nmtui` y reinicia la red:



```
sudo systemctl restart NetworkManager
```

### 2) Crear un nuevo segmento de red (`10.10.10.0/24`)

Para aislar o separar tráfico en un nuevo segmento sin necesidad de comprar switches físicos adicionales, puedes crear un Port Group dedicado etiquetado con una **VLAN ID** (si tu switch físico y router soportan 802.1Q) o usar un Port Group interno puro si el enrutamiento lo harás mediante una VM router (como pfSense virtualizado).

#### Opción A: Red interna / aislada (Enrutada por una VM tipo pfSense)

Si el tráfico de la red `10.10.10.0/24` será manejado por un firewall virtual dentro del mismo ESXi:

1. En el menú izquierdo de ESXi, ve a **Networking** -> pestaña **Port groups**.
2. Haz clic en **Add port group**.
3. Configura los siguientes parámetros:
   - **Name:** `VLAN-10-Red` (o un nombre descriptivo).
   - **vSwitch:** Selecciona tu vSwitch existente (ej. `vSwitch0`).
   - **VLAN ID:** `0` (si no usas etiquetas VLAN en el switch físico) o el número de VLAN correspondiente (ej. `10`).
4. Haz clic en **Add**.

*Nota posterior:* Para que la VM `alma-srv01` o cualquier otra pase a esta red, solo debes editar su adaptador de red en el ESXi y cambiar el *Port Group* al nuevo `VLAN-10-Red`. Luego, dentro del sistema operativo, configuras una IP del rango `10.10.10.0/24` apuntando como puerta de enlace (`gateway`) a la IP que tenga tu pfSense en esa misma red.

¿Necesitas ayuda adicional para configurar el enrutamiento de esta nueva subred en tu firewall o prefieres usar VLANs etiquetadas?



> ## Puedo  asignar el rango de IP a este nuevo segmento de red ?



Para asignar el rango de IP `10.10.10.0/24` a nivel de red, debes tener en cuenta que **un hipervisor ESXi por sí solo no asigna IPs de forma automática (no actúa como servidor DHCP para las VMs)** a menos que configures un servicio para ello.

La forma en que se asigna este rango depende de cómo quieras gestionarlo: **Estipulando IPs fijas** en cada máquina virtual o **configurando un servidor DHCP** (por ejemplo, en tu pfSense).

Aquí te explico los dos escenarios paso a paso:

### Opción 1: Asignación Estática (Manual en cada VM)

Si prefieres controlar las IPs manualmente en los servidores o contenedores que conectes al nuevo Port Group (`VLAN-10-Red`):

1. **Conéctate a la consola de la VM** (ej. `alma-srv01` si decides moverla a esta nueva red, o cualquier otra).

2. Abre el asistente de red con `nmtui`:

   ```
   sudo nmtui
   ```

3. Selecciona **Edit a connection** y elige la interfaz de red correspondiente.

4. Configura los parámetros del nuevo segmento:

   - **IPv4 CONFIGURATION:** `Manual` (<-- *Aquí defines el rango*)
   - **Addresses:** `10.10.10.X/24` (Donde `X` es la IP específica para esta máquina, por ejemplo `10.10.10.10`)
   - **Gateway:** `10.10.10.1` (La IP que tendrá tu firewall/router en esta nueva interfaz de red)
   - **DNS Servers:** Los que utilices habitualmente (ej. `1.1.1.1` o la IP de tu DNS local).

5. Guarda los cambios, sal de `nmtui` y reinicia el servicio de red:

   ```
   sudo systemctl restart NetworkManager
   ```

### Opción 2: Creando un servidor DHCP (Recomendado si tendrás muchas VMs)

Si prefieres que las máquinas obtengan su IP automáticamente dentro del rango `10.10.10.0/24`, debes configurar el servicio DHCP en el dispositivo que actúa como **Gateway (Puerta de Enlace)** de esa red (por ejemplo, tu firewall pfSense).

#### Pasos generales en tu pfSense (o router):

1. **Crear la interfaz virtual (SVI):** En tu pfSense, debes asignar una interfaz física o VLAN al nuevo Port Group de ESXi y asignarle la IP de la puerta de enlace, por ejemplo `10.10.10.1/24`.
2. **Habilitar el servicio DHCP Server:**
   - Ve a **Services** -> **DHCP Server** en el menú de pfSense.
   - Selecciona la pestaña correspondiente a la nueva interfaz de la red `10.10.10.0/24`.
   - Marca la casilla **Enable DHCP server on [Nombre de la interfaz]**.
   - Define el rango de IPs que entregará automáticamente (por ejemplo, desde `10.10.10.100` hasta `10.10.10.200`).
   - Configura la puerta de enlace (`10.10.10.1`) y los servidores DNS.
   - Guarda los cambios.

Una vez hecho esto, cualquier máquina virtual que conectes al Port Group `VLAN-10-Red` y configures con **DHCP (Automatic)** obtendrá una IP de ese rango de forma automática al encenderse o reiniciar su red.

