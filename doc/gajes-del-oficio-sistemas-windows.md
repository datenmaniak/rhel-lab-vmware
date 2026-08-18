# Aplicar cambios como administrador



> Soy un usuario estándar en Windows 11 por razones de seguridad.
>
> Necesito agregar algunos ajustes a la interfaz de red, e instalar servicios a través de la opciones 'Propiedades'
>
> Como invoco estas acciones en modo administrador sin cerrar mi sesión actual.?



Para realizar cambios a nivel de sistema en las interfaces de red sin cerrar tu sesión estándar, necesitas invocar el panel clásico de Conexiones de red (`ncpa.cpl`) desde una consola elevada.

**1.Ejecutar Símbolo del sistema (CMD) con privilegios:**

Presiona la tecla **Windows**, escribe `cmd`, localiza "Símbolo del sistema" y haz clic en **Ejecutar como administrador** en el panel derecho. Como alternativa, puedes presionar `Win + R`, escribir `cmd` y presionar **`Ctrl + Shift + Enter`**.

**2.Ingresar credenciales de administrador:**

Aparecerá el aviso del Control de cuentas de usuario (UAC). Como estás en una cuenta estándar, el sistema te pedirá que ingreses el nombre de usuario y la contraseña de una cuenta que sí tenga permisos de administrador en ese equipo.

**3.Lanzar el panel de Conexiones de red:**

Una vez que se abra la ventana negra de comandos (debería decir "Administrador: Símbolo del sistema" en la barra de título), escribe el comando `ncpa.cpl` y presiona **Enter**.

**4.Modificar las propiedades de la interfaz:**

Se abrirá la ventana de "Conexiones de red". Como esta ventana heredó los permisos de administrador de la consola, puedes hacer clic derecho sobre tu adaptador (Ethernet o Wi-Fi), seleccionar **Propiedades** y utilizar el botón **Instalar...** para agregar clientes, servicios o protocolos sin restricciones.

Puedes cerrar la ventana negra del Símbolo del sistema en cuanto se abra el panel de conexiones; el panel de red seguirá abierto con los permisos elevados.



### Aplica a cualquier invocación de configuraciones 

**Ejemplos:**

- **Servicios** `services.msc`

- **Administrar discos:**  `diskmgmt.msc`



