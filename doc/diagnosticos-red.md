# Diagnósticos  y soluciones de red



> He llegado hasta la Fase 4. 
>
> Se han seguidos los pasos de configuración. Sin embargo, no ha sido posible conectar la interfaz de red.



Entiendo, llegar hasta la Fase 4 y quedarse sin red es frustrante. Vamos a resolverlo. Como mencionas que es una máquina virtual (tu ESXi anidado en Workstation Pro), el problema suele ser la configuración en el host, no los drivers del ESXi en sí, ya que el adaptador "vmxnet3" que usas es virtual.

### 🔍 Diagnóstico Rápido

Cuando un ESXi anidado no tiene red, el culpable casi siempre es la configuración de red en VMware Workstation. Tu documento usa "Bridged" para que ESXi tenga su propia IP en tu red física. Si esto falla, es porque Workstation no está enlazando correctamente ese adaptador virtual a tu tarjeta de red física de Windows.

### 🛠️ Pasos para Solucionarlo

**1. Verifica el Editor de Red de VMware Workstation**

Este es el paso más importante . En tu máquina Windows:

1. Abre VMware Workstation Pro.
2. Ve al menú **Editar** > **Editor de red virtual**.
3. Haz clic en **Cambiar configuración** (necesitas permisos de administrador).
4. En la lista, busca el adaptador **VMnet0** (que normalmente es el de tipo "Bridged").
5. En la sección **"Información de VMnet"** que aparece abajo, asegúrate de que en el desplegable **"Conectar a:"** no esté en **Automático**. El problema suele ser que Workstation elige el adaptador incorrecto (por ejemplo, un adaptador de VPN o Bluetooth en lugar de tu Wi-Fi o Ethernet real) .
   - Cámbialo a tu adaptador de red físico activo (por ejemplo, "Realtek PCIe GbE Family Controller" o "Intel(R) Wi-Fi 6 AX200").
6. Haz clic en **Aplicar** y **Aceptar**.

**2. Apaga y Vuelve a Encender la VM**

Dentro de la consola de tu VM (la de ESXi), apágala completamente y luego enciéndela de nuevo. Esto forzará a que el adaptador virtual (vmnic0) reconozca el nuevo puente.

**3. Valida la Configuración desde la DCUI**

Cuando la VM de ESXi arranque, entra a la pantalla amarilla (DCUI) y presiona **F2** para ir a la configuración de red:

- Ve a **"Configure Management Network"**.
- Verifica que en **"Network Adapters"** esté seleccionado tu **vmnic0** .
- Revisa que tu **IPv4 Configuration** esté con la IP estática que configuraste (`192.168.1.50` o similar), y que la máscara y puerta de enlace (`192.168.1.1`) sean correctas.

### ⚠️ Si el problema persiste: Otros posibles culpables

1. **Conflicto con Hyper-V**: Si en tu Windows tienes Hyper-V, WSL o el Sandbox de Windows activados, pueden bloquear el acceso de VMware a la red. Asegúrate de que estén desactivados desde "Activar o desactivar las características de Windows" (`optionalfeatures`).

2. **Firewall de Windows**: El firewall puede estar bloqueando la conexión de la VM. Puedes desactivarlo temporalmente para hacer una prueba.

3. **Reiniciar Servicios**: En tu Windows, abre una consola como Administrador y ejecuta para reiniciar los servicios de red de VMware:

   

   ```
   net stop "VMware NAT Service"
   net stop "VMware DHCP Service"
   net start "VMware DHCP Service"
   net start "VMware NAT Service"
   ```

   

### 💡 Prueba de Verificación Rápida

Una vez que hayas cambiado la configuración del puente y reiniciado la VM, el primer paso desde la consola de la VM de ESXi es ejecutar el comando para ver si el adaptador es detectado:



```
esxcli network nic list
```



Si el comando no muestra ninguna interfaz, el problema es la conexión entre Workstation y Windows . Si la lista muestra `vmnic0` pero no tienes IP, el problema es la configuración de red dentro de ESXi (que ya revisaste en el paso 3).

Cuéntame qué ves en el editor de red de Workstation y si el comando `esxcli network nic list` te muestra alguna interfaz. Con eso, podemos enfocar la solución al 100%.



#### Advertencia | Error

> No permite elegir "Bridged".  Se presenta una mensaje:  Cannot change network to bridged: There are no un-bridged host network adapters.



El mensaje de error que ves, "Cannot change network to bridged: There are no un-bridged host network adapters", es un problema clásico de VMware Workstation. Significa que el software no puede encontrar un adaptador de red físico en tu Windows que esté disponible para ser "puenteado" (bridged) .

No te preocupes, vamos a solucionarlo paso a paso.

### 🛠️ Solución Paso a Paso

La causa más común es que el **"VMware Bridge Protocol"** esté deshabilitado en tu adaptador de red físico (Wi-Fi o Ethernet). Sigue estos pasos para verificar y habilitarlo:

1. **Abre las conexiones de red de Windows**: Presiona `Win + R`, escribe `ncpa.cpl` y presiona Enter .
2. **Accede a las propiedades de tu adaptador**: Busca el adaptador que estás usando para conectarte a Internet (puede ser "Wi-Fi" o "Ethernet"). Haz clic derecho sobre él y selecciona **"Propiedades"** .
3. **Habilita el Protocolo VMware**: En la lista de elementos que se muestra, busca **"VMware Bridge Protocol"**. Si aparece desmarcado, **márcalo**. Si no aparece en la lista, es posible que necesites reparar la instalación de VMware .
4. **Guarda los cambios**: Haz clic en **"Aceptar"** para guardar la configuración.

Una vez hecho esto, cierra y vuelve a abrir el Editor de Red Virtual de VMware y comprueba si ahora puedes seleccionar el modo "Bridged". 

### ⚙️ Si el problema persiste

Si el paso anterior no funciona, prueba estas soluciones en orden:

- **Reinicia el Servicio de VMware Bridge**: Abre el administrador de servicios (escribe `services.msc` en `Win + R`), busca **"VMware Bridge Service"**, haz clic derecho y selecciona **"Reiniciar"**. Asegúrate de que su tipo de inicio sea "Automático" .
- **Restaura la Configuración de Red Predeterminada**:
  1. Abre VMware Workstation como Administrador.
  2. Ve a `Edit > Virtual Network Editor`.
  3. Haz clic en **"Restore Defaults"**. Esto recreará los adaptadores de red virtuales, lo que suele solucionar el problema .
- **Verifica la selección manual del adaptador**:
  1. En el Virtual Network Editor, selecciona VMnet0.
  2. En el menú desplegable "Bridged to", cámbialo de "Automatic" a tu adaptador de red físico específico (por ejemplo, "Intel(R) Wi-Fi 6 AX200" o "Realtek Gaming GbE Family Controller") y aplica los cambios .
- **Reinstalación de los adaptadores VMware**: En ocasiones, reinstalar el programa desde el Panel de Control > Programas y Características y seleccionar la opción **"Reparar"** puede resolver problemas de componentes dañados .

------

### ⚠️ Posibles conflictos a tener en cuenta

- **Hyper-V y otras plataformas de virtualización**: Si tienes Hyper-V, WSL o el Sandbox de Windows activados, pueden entrar en conflicto con las funciones de red de VMware. Puedes desactivarlos temporalmente desde "Activar o desactivar las características de Windows" (`optionalfeatures`) para descartar este problema .
- **VPNs**: Algunas VPN crean sus propios adaptadores de red virtuales que pueden interferir. Prueba a desconectar y cerrar completamente tu VPN para ver si el problema se resuelve .

------

Prueba estos pasos en el orden indicado. El más efectivo suele ser habilitar el "VMware Bridge Protocol" o restaurar la configuración predeterminada en el Editor de Red Virtual.

Cuéntame cómo te va. Si después de estos pasos sigues sin poder configurarlo, podemos explorar soluciones más avanzadas, como una reinstalación completa de VMware.



#### Durante la configuracion en Windows 11

> Qué hacer en caso que no se encuentre "VMware Bridge Service" en la lista al buscar en **`services.msc`** ?



Que no aparezca el "VMware Bridge Service" indica que el componente de red puente de VMware no está instalado o está dañado . Esto es algo que podemos solucionar.

Te sugiero intentar estas soluciones en el orden que te presento. La primera es la más directa y suele funcionar.

### ✅ Solución 1: Instalar Manualmente el Protocolo (la más eficaz)

Este es el método más recomendado para resolver el problema cuando el servicio no existe .

1. **Abre las Conexiones de Red**: Presiona `Win + R`, escribe `ncpa.cpl` y pulsa Enter.
2. **Accede a Propiedades**: Haz clic derecho sobre tu adaptador de red activo (Wi-Fi o Ethernet) y selecciona **"Propiedades"**.
3. **Instala el Protocolo**:
   - Haz clic en el botón **"Instalar..."**.
   - En la nueva ventana, selecciona **"Servicio"** y haz clic en **"Agregar..."** .
   - Haz clic en **"Examinar..."** y navega hasta la carpeta de instalación de VMware Workstation (normalmente `C:\Program Files (x86)\VMware\VMware Workstation\`).
   - Busca y selecciona el archivo **`netbridge.inf`** . Luego haz clic en **"Abrir"**.
   - Asegúrate de que **"VMware Bridge Protocol"** esté seleccionado y haz clic en **"Aceptar"** .
4. **Reinicia**: Cierra todas las ventanas y **reinicia tu computadora** para que los cambios surtan efecto .

Después de reiniciar, abre el Editor de Red Virtual de VMware e intenta configurar el modo "Bridged" de nuevo.

### 🛠️ Solución 2: Reparar la Instalación de VMware

Esta opción es más simple y restaura todos los componentes, incluidos los de red, sin perder tus máquinas virtuales.

1. Abre el **Panel de Control** y ve a **"Programas y características"**.
2. Busca **"VMware Workstation Pro"** en la lista.
3. Haz clic derecho sobre él y selecciona **"Cambiar"**.
4. En el instalador, elige la opción **"Reparar"** y sigue las instrucciones .
5. Una vez termine, reinicia tu PC.

### ⚙️ Solución 3: Restaurar la Configuración Predeterminada

Este método reconstruye los adaptadores de red virtuales desde cero.

1. Abre VMware Workstation Pro.
2. Ve al menú **"Editar"** > **"Editor de red virtual"**.
3. Haz clic en **"Cambiar configuración"** (si aparece) para obtener permisos de administrador .
4. Haz clic en el botón **"Restaurar valores predeterminados"** . Espera a que el proceso termine.
5. **Reinicia tu computadora** .

------

> **Nota importante sobre VPNs y Antivirus:** Si usas una VPN, asegúrate de cerrarla completamente antes de probar cualquiera de estas soluciones, ya que puede crear adaptadores de red virtuales que interfieran con el proceso de VMware .

Si después de probar estas soluciones el servicio sigue sin aparecer, la opción más segura es **desinstalar y reinstalar VMware Workstation Pro desde cero**. Asegúrate de tener a mano tu clave de licencia.

Una vez que soluciones este problema y puedas configurar el modo "Bridged", continuaremos con la instalación de ESXi desde donde lo dejaste. ¿Qué te parece si pruebas el primer método y me cuentas cómo te va?
