# Troubleshooting network at Linux



## Entorno: WorkStation Pro en Linux

La VM anfitrión es VMware ESXi 8.0 se ejecuta utilizando virtualización anidada,  corriendo sobre Linux (Omarchy Quatro, por cierto!  😎)

```text
The virtual machine's operating system has attempted to enable promiscuous mode on adapter 'Ethernet0'. This is not allowed for security reasons.
Please go to the Web page "http://vmware.com/info?id=161" for help enabling promiscuous mode in the virtual machine.
```



> The error you're seeing is a security feature of VMware products. It prevents the virtual machine's operating system from putting its virtual network adapter into promiscuous mode, which would allow it to see all network traffic on the virtual switch, not just its own .



Este error ocurre porque la máquina virtual está intentando poner su interfaz en modo promiscuo para gestionar tráfico de red que no está destinado exclusivamente a su propia dirección MAC. En un host Linux, VMware Workstation o Player bloquea esta acción por defecto para cualquier usuario que no sea `root`.

Esto es un comportamiento esperado y necesario si dentro de esa VM estás corriendo servicios que manejan subredes, bridges o múltiples MACs (como un firewall pfSense, contenedores con Podman/Docker, o clústeres locales de k3s/Kind).

Para solucionarlo sin tener que ejecutar el hipervisor como `root`, debes otorgar permisos de lectura y escritura a la interfaz virtual (`/dev/vmnetX`) directamente en tu host Linux.

## Solución Inmediata

1. **Identifica el tipo de red virtual:**

   - **Bridged (Puente):** `/dev/vmnet0`
   - **Host-only (Solo anfitrión):** `/dev/vmnet1`
   - **NAT:** `/dev/vmnet8`

2. **Aplica los permisos:** Abre tu emulador de terminal en el host Linux y ejecuta el siguiente comando (asumiendo que la VM usa red Bridged en `vmnet0`):

   

   **Crear el script de asignación:**  Requiere privilegios sudo.

   Crea un archivo en `~/vmnet-promisc.sh` con el siguiente contenido para ajustar los adaptadores que utilices (ej. vmnet1 y vmnet8):
   

   ```bash
   #!/bin/sh
   
   sudo chgrp vmware /dev/vmnet0 /dev/vmnet8 /dev/vmnet1
   sudo chmod g+rw /dev/vmnet0 /dev/vmnet8 /dev/vmnet1
   
   # Ejecuta antes de invocar VMWare WorkStation/Player
   ./vmnet-promisc.sh
   ```


3. **Inicia la VM:** Arranca nuevamente la máquina desde VMware. La alerta desaparecerá y el tráfico de red interno comenzará a enrutarse correctamente.


The solution depends on the VMware product you are using. There are two main scenarios:

### 🖥️ Scenario 1: VMware Workstation or Fusion on a Linux Host

This is the most common cause **when running VMware Workstation** or Fusion **on a Linux system**, especially if you are **using nested virtualization** (e.g., running ESXi inside Workstation) or packet-capturing tools inside a VM .

**Root Cause:** The `/dev/vmnet*` device files that control the virtual network interfaces have restrictive permissions that only allow the `root` user to enable promiscuous mode. The user running the VMware software does not have the necessary permissions .

**Solution:** You can grant permission by changing the permissions on the appropriate device files.  **(RECOMENDADO)**

1. **Identify the correct device:**

   - If your VM uses **Bridged** networking, the device is `/dev/vmnet0` .
   - If your VM uses **NAT** networking, the device is `/dev/vmnet8` .

2. **Grant read/write permissions:** Open a terminal on your **Linux host** (not inside the VM) and run a command to give the necessary permissions. You can either:

   - **Give all users permission (quick but less secure):** This makes the device writable for any user.

     ```bash
   sudo chmod a+rw /dev/vmnet0
     ```
     

     *(Replace `/dev/vmnet0` with `/dev/vmnet8` if you are using NAT)* .

   - **Use a group for more granular control:** This is the more secure, recommended approach .

     

     ```bash
     # Create a new group (e.g., vmware)
     sudo groupadd vmware 
     # Add your username to the group
     sudo usermod -aG vmware $USER
     # Change the group ownership of the device
     sudo chgrp vmware /dev/vmnet0 /dev/vmnet1 /dev/vmnet8
     # Give the group read/write permissions
     sudo chmod g+rw /dev/vmnet0 /dev/vmnet1 /dev/vmnet8
     ```


### Configuración Persistente

Dado que el sistema de archivos `/dev/` es volátil, estos permisos se perderán cada vez que reinicies el equipo físico/anfitrión o servidor. La forma más robusta de mantener este ajuste es automatizarlo a través de `systemd` para que se aplique justo después de que los servicios de red de VMware levanten.

**1.Crear el script de asignación:**  Requiere privilegios sudo.

Crea un archivo en `/usr/local/bin/vmnet-promisc.sh` con el siguiente contenido para ajustar los adaptadores que utilices (ej. vmnet0 y vmnet8):

```bash
#!/bin/bash
chmod a+rw /dev/vmnet0 /dev/vmnet8 /dev/vmnet1
```

Haz que el script sea ejecutable:

```bash
sudo chmod +x /usr/local/bin/vmnet-promisc.sh
```

**2.Crear el servicio de systemd:**

Crea un nuevo archivo de servicio en `/etc/systemd/system/vmnet-promisc.service`:


```toml
[Unit]
Description=Habilitar modo promiscuo para vmnets de VMware
After=vmware.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/vmnet-promisc.sh

[Install]
WantedBy=multi-user.target
```


**3.Habilitar el servicio:**

Activa el servicio para que se ejecute en el próximo inicio del sistema:


```bash
sudo systemctl enable vmnet-promisc.service
sudo systemctl start vmnet-promisc.service
```



1. **Verify the change:** You can check the new permissions with `ls -l /dev/vmnet0 /dev/vmnet1 /dev/vmnet8` .

2. **Restart the VM or VMware service:** For the change to take effect, you may need to restart the virtual machine or the **VMware Workstation service** .


### ☁️ Scenario 2: VMware vSphere (ESXi)

If your VM is running on a VMware ESXi host, the security policy is controlled at the virtual switch or port group level in the vSphere Client.

**Root Cause:** The security policy for the virtual switch or port group has the **Promiscuous Mode** option set to **Reject** .

**Solution:** You must change the security policy to **Accept** for the port group or switch your VM is connected to .

1. **Open the vSphere Client** and navigate to your host.
2. Go to the **Configure** tab, expand **Networking**, and select **Virtual Switches** .
3. Select the virtual switch or the specific port group used by your VM.
4. Click **Edit settings**.
5. Navigate to the **Security** section .
6. Change the **Promiscuous Mode** drop-down from **Reject** to **Accept** .
7. Click **OK** to save the changes.

**Important Considerations for vSphere:**

- Enabling promiscuous mode is considered insecure and should be done with caution .
- Be aware that enabling this on a switch used for NFS storage traffic can cause network congestion, VM freezes, and datastore connectivity issues. It is highly recommended to use a **dedicated vSwitch** for these types of activities .

