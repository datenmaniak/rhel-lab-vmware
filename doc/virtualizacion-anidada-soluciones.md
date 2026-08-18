# Error de inicio de ESXi 

### El siguiente mensaje se produce al encender la maquina virtual por primera vez.

#### Escenario:  `Workstation Pro corriendo en Windows 11`

**Por  que se produce esta advertencia ?** 

```text
 Virtualized AMD-V/RVI is not supported on this platform.

Continue without virtualized AMD-V/RVI?
```



Ese mensaje aparece porque **VMware Workstation Pro intenta activar la virtualización anidada** (*Nested Virtualization* / AMD-V virtualizado dentro de la VM), pero Windows 11 no le permite el acceso directo a la extensión de hardware del procesador AMD.

Esto sucede casi siempre porque **Hyper-V o alguna función de seguridad de Windows basada en virtualización (VBS/HVCI) está activa**. Cuando estas características de Windows están encendidas, el hipervisor de Microsoft (*Hyper-V*) toma el control exclusivo de AMD-V en la capa base y se coloca por encima de VMware Workstation.

### ¿Qué significa la advertencia?

- **Si haces clic en "Continuar":** La máquina virtual arrancará normalmente en la mayoría de los casos. Sin embargo, no podrás ejecutar dentro de ella hipervisores secundarios o contenedores que requieran virtualización por hardware (por ejemplo, WSL2, Docker Desktop, KVM o VirtualBox dentro de la VM).
- **Si necesitas la virtualización anidada:** Deberás desactivar las funciones de Windows que bloquean el acceso directo a AMD-V o ajustar la configuración de VMware.

### Soluciones principales

#### Opción A: Desactivar la virtualización anidada en la VM (si no la necesitas)

Si solo quieres usar la máquina virtual normalmente y evitar que el mensaje aparezca cada vez:

1. Apaga la máquina virtual.
2. Abre **VM Settings** (Ajustes de la VM) > pestaña **Hardware** > **Processors**.
3. Desmarca la casilla **"Virtualize Intel VT-x/EPT or AMD-V/RVI"**.
4. Haz clic en **OK** y enciende la VM.

#### Opción B: Liberar AMD-V desactivando Hyper-V y VBS en Windows 11 (si necesitas virtualización anidada)

Si necesitas hacer pruebas con Docker, KVM o WSL2 *dentro* de la máquina virtual, debes desactivar las características de Windows que toman el control del procesador:

1. **Desactivar aislamiento de aislamiento de núcleo / integridad de memoria:**

   - Ve a **Inicio > Configuración > Privacidad y seguridad > Seguridad de Windows > Seguridad del dispositivo**.
   - Haz clic en **Detalles de Aislamiento del núcleo**.
   - Desactiva **Integridad de la memoria** (HVCI) y reinicia el equipo.

2. **Desactivar características de Hyper-V:**

   - Presiona la tecla `Windows` + `R`, escribe `optionalfeatures` y presiona **Enter**.
   - Desmarca las siguientes casillas:
     - **Hyper-V** (todas sus opciones)
     - **Plataforma de hipervisor de Windows** (*Windows Hypervisor Platform*)
     - **Plataforma de máquina virtual** (*Virtual Machine Platform*)
     - **Protección de aplicaciones de Windows Defender** (*Guardia de aplicaciones*)
   - Haz clic en **Aceptar** y reinicia el sistema cuando lo solicite.

3. **Desactivar la protección por hipervisor mediante CMD (si persiste el aviso):**

   - Abre **Símbolo del sistema** (CMD) como **Administrador**.

   - Ejecuta el siguiente comando:

     

     ```
     bcdedit /set hypervisorlaunchtype off
     ```

   - Reinicia la PC.

*(Nota: Desactivar la Plataforma de máquina virtual o `hypervisorlaunchtype` deshabilitará temporalmente WSL2 o Sandbox en Windows 11 si los utilizas en el sistema anfitrión).*

> Si todas las casillas, ya desmarcadas... que alternativa tengo?

Si las casillas en *Características de Windows* ya están desmarcadas, la causa casi siempre es **Device Guard / Credential Guard** o la **Integridad de la Memoria** de Windows 11, que mantienen Hyper-V activo en segundo plano aunque el componente UI aparezca apagado.

Aplica las siguientes alternativas en orden para forzar a Windows a liberar AMD-V:

### 1. Desactivar la Integridad de la Memoria (HVCI)

Es la causa más común en Windows 11.

1. Abre el menú Inicio y busca **Seguridad de Windows**.
2. Ve a **Seguridad del dispositivo** > **Detalles de Aislamiento del núcleo**.
3. Desactiva el interruptor de **Integridad de la memoria**.
4. **Reinicia la PC** obligatoriamente.

### 2. Forzar el apagado del hipervisor vía Administrador

A veces el arranque de Windows mantiene el hipervisor cargado en memoria aunque no uses Hyper-V.

1. Haz clic derecho en el menú Inicio y selecciona **Terminal (Administrador)** o **Símbolo del sistema (Administrador)**.

2. Ejecuta exactamente el siguiente comando:

   

   ```
   bcdedit /set hypervisorlaunchtype off
   ```

3. Reinicia la PC.

*(Si en el futuro necesitas reactivar WSL2 o Windows Sandbox, puedes revertirlo ejecutando `bcdedit /set hypervisorlaunchtype auto` y reiniciando).*

### 3. Utilizar la herramienta oficial de VMware (`DG_Readiness`)

Microsoft proporciona un script específico para limpiar las directivas de *Device Guard* / *Credential Guard* que bloquean VMware:

1. Descarga la herramienta oficial **Device Guard and Credential Guard Readiness Tool** desde el sitio de Microsoft.

2. Abre PowerShell como **Administrador**.

3. Ejecuta el script desactivando las políticas de virtualización basada en seguridad:

   

   ```
   .\DG_Readiness_Tool_v3.6.ps1 -Disable
   ```

4. Reinicia el equipo cuando lo indique el script.

### 4. Desactivar la casilla en VMware (Solución rápida si no requieres anidación)

Si solo necesitas que la VM funcione con normalidad y **no vas a instalar Docker, KVM o WSL2 dentro de esa VM**:

1. Con la VM apagada, ve a **VM > Settings > Hardware > Processors**.
2. **Desmarca** la casilla `Virtualize Intel VT-x/EPT or AMD-V/RVI`.
3. Haz clic en **OK** y enciende la VM. El mensaje desaparecerá de inmediato.

#### Error en la consola de Powershell

---

```text
PS D:\dgreadiness_v3.6\dgreadiness_v3.6> .\DG_Readiness_Tool_v3.6.ps1 -Disable
.\DG_Readiness_Tool_v3.6.ps1 : No se puede cargar el archivo
D:\dgreadiness_v3.6\dgreadiness_v3.6\DG_Readiness_Tool_v3.6.ps1 porque la ejecución de scripts está deshabilitada en
este sistema. Para obtener más información, consulta el tema about_Execution_Policies en
https:/go.microsoft.com/fwlink/?LinkID=135170.
En línea: 1 Carácter: 1
+ .\DG_Readiness_Tool_v3.6.ps1 -Disable
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

Ese error ocurre porque PowerShell tiene bloqueada la ejecución de scripts por política de seguridad predeterminada (*ExecutionPolicy*).

Para ejecutar el script sin alterar permanentemente la seguridad de tu sistema, puedes omitir la restricción solo para esta sesión o cambiar la política temporalmente.

### Opción 1: Ejecutar omitiendo la política (Recomendada)

Ejecuta el script indicándole a PowerShell que ignore la política de ejecución únicamente para este proceso:

```
powershell -ExecutionPolicy Bypass -File .\DG_Readiness_Tool_v3.6.ps1 -Disable
```

### Opción 2: Cambiar la política temporalmente en tu ventana de PowerShell

1. En la misma ventana de PowerShell (que debe estar abierta como **Administrador**), cambia la política para la sesión actual:

   

   ```
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
   ```

2. Presiona `S` (o `Y`) para confirmar el cambio.

3. Vuelve a ejecutar el script de VMware/Microsoft:

   

   ```
   .\DG_Readiness_Tool_v3.6.ps1 -Disable
   ```

### ¿Qué esperar después de ejecutarlo?

El script modificará los registros del sistema y las variables de UEFI para deshabilitar Device Guard / Credential Guard.

- Al finalizar, el script te pedirá **reiniciar la PC**.
- Durante el reinicio, es muy probable que veas una pantalla negra con texto del firmware/BIOS preguntando si confirmas la deshabilitación de Credential/Device Guard (presionando `F3` o `Windows Key` según las instrucciones en pantalla). Confirma para aplicar los cambios.







