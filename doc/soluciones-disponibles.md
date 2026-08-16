# Soluciones disponibles 



> Existe  alguna solución de VMware   Hypervisor ESXi + vCenter que pueda ser descargada gratis. ?



Sí, existe. Se llama **VMware vSphere Hypervisor**, conocido simplemente como **ESXi Free**. Sin embargo, su situación cambió drásticamente tras la adquisición de VMware por Broadcom, así que te cuento el estado exacto en 2026.

------

## 1. Estado actual: volvió a ser gratis, pero con truco

Broadcom eliminó la licencia gratuita de ESXi en febrero de 2024, generando un gran revuelo en la comunidad. Tras un año de presión, en **abril de 2025** reintrodujeron la versión gratuita con el lanzamiento de **ESXi 8.0 Update 3e**.

**La clave**: ahora hay **dos ISOs distintas** para la misma versión 8.0U3e:



| ISO                                                   | Licencia incluida         | Uso                                |
| :---------------------------------------------------- | :------------------------ | :--------------------------------- |
| `VMware-VMvisor-Installer-8.0U3e-24677879.x86_64.iso` | **Free ESXi** embebida    | Home lab, aprendizaje              |
| `VMware-VMvisor-Installer-8.0U3e-24674464.x86_64.iso` | Modo evaluación (60 días) | Prueba Enterprise Plus por 60 días |

La versión gratuita tiene la licencia ya incrustada; no necesitas introducir clave manualmente 

------

## 2. Cómo obtenerlo

1. Crea una cuenta gratuita en el [Broadcom Support Portal](https://support.broadcom.com/).
2. Ve a **My Downloads** → **Free Software Downloads**.
3. Busca **"VMware vSphere Hypervisor"**.
4. Selecciona la versión **8.0U3e** y acepta los términos.
5. Descarga la ISO y la instalas en tu servidor físico (o anidada dentro de Workstation).

------

## 3. Limitaciones de ESXi Free (las que realmente importan)

La licencia es **perpetua** (no expira) y permite usar todo el hardware del host: CPUs físicas ilimitadas, cores ilimitados y RAM ilimitada. Pero tiene restricciones duras:

| Feature                      | ¿Disponible en Free? | Impacto                                                      |
| :--------------------------- | :------------------- | :----------------------------------------------------------- |
| **Máximo de vCPU por VM**    | **8 vCPU**           | Si necesitas una VM grande (ej: base de datos pesada), no arrancará |
| **vCenter Server**           | ❌ No                 | No puedes gestionar el host desde vCenter. Solo interfaz web local (Host Client) |
| **vMotion**                  | ❌ No                 | No migras VMs en caliente entre hosts                        |
| **HA / DRS**                 | ❌ No                 | Si el host físico muere, todo muere contigo                  |
| **API de backup (VADP)**     | ❌ No                 | Veeam, Nakivo, etc. no pueden hacer backup agentless de tus VMs |
| **API escritura (PowerCLI)** | ❌ Solo lectura       | Puedes consultar (`Get-VM`) pero no crear ni modificar por scripts |
| **Distributed Switch**       | ❌ No                 | Solo Standard vSwitch                                        |
| **Soporte oficial**          | ❌ No                 | Si se rompe, tu único salvavidas es Google y los foros       |

------

## 4. ESXi Free vs. Modo Evaluación (60 días)

Cuando instalas ESXi **sin licencia** (la ISO de evaluación), obtienes **Enterprise Plus completo por 60 días**: vCenter, vMotion, HA, DRS, APIs, todo. Es la mejor forma de aprender.

Pasados los 60 días, si no introduces una licencia (free o paga), las VMs encendidas siguen funcionando, pero **no puedes encender VMs apagadas** ni conectar el host a vCenter.

**Estrategia recomendada para aprender:**

1. Instala la ISO de **evaluación** (la de pago).
2. Durante 60 días, monta tu vCenter, juega con HA, vMotion, DRS, PowerCLI, backups.
3. Cuando venza, si solo quieres un lab perpetuo, aplicas la licencia **Free** y pierdes esas features, pero conservas tus VMs.

------

## 5. ¿Sirve ESXi Free para aprender lo de producción?

**Para los fundamentos del hypervisor:** sí. Aprenderás la interfaz de ESXi, crear VMs, datastores, redes básicas, VMware Tools.

**Para la operación enterprise:** **no**. Te faltará práctica con:

- vCenter (el 90% del trabajo en producción)
- vMotion y Storage vMotion
- Clusters, HA y DRS
- Automatización con PowerCLI/API
- Backups enterprise
- Redes distribuidas

------

## 6. Alternativa si quieres todo gratis (incluyendo vCenter)

Si tu objetivo es aprender **todo** el stack enterprise sin pagar miles de dólares, existe **VMUG Advantage**:

- **~$200/año** (precio aproximado).

- Te da licencias **vSphere Enterprise Plus**, **vCenter Standard**, **vSAN**, **NSX**, etc.

- Válido para home lab / uso personal (no producción comercial).

- Es el estándar de oro para quienes quieren certificaciones VCP o simular un datacenter real en casa 

  .

------

## Resumen ejecutivo



| Pregunta                           | Respuesta                                                    |
| :--------------------------------- | :----------------------------------------------------------- |
| ¿ESXi gratis existe en 2026?       | **Sí**, ESXi 8.0U3e Free vía Broadcom Support Portal.        |
| ¿Es perpetuo?                      | **Sí**, la licencia no expira.                               |
| ¿Puedo usarlo en producción?       | Legalmente sí, pero **no es recomendable**: sin backups, sin HA, sin soporte. |
| ¿Me prepara para trabajo real?     | **Parcialmente**. Aprenderás ESXi, pero no vCenter ni las features enterprise. |
| ¿Qué hago si quiero aprender todo? | Usa los **60 días de evaluación** + considera **VMUG Advantage** ($200/año). |

------

¿Quieres que te guíe paso a paso para descargar la ISO correcta y hacer tu primera instalación de ESXi Free, o prefieres que explique cómo montar un lab con **evaluación de 60 días + vCenter** para que toques lo más parecido a producción?



> Por qué se produce esta notificación en la pagina de descarga aunque haya sido elegido de la sección Free Software Downloads?
>
> #### Account verification is Pending. Please try after some time.



Ese mensaje es uno de los dolores de cabeza más comunes desde que Broadcom compró VMware. No es un error técnico de tu navegador; es un **filtro de cumplimiento comercial (Trade Compliance Screening)** que Broadcom aplica a todas las cuentas nuevas antes de permitir descargas. Te explico por qué pasa y, más importante, qué puedes hacer **ahora mismo**.

------

## 1. ¿Por qué aparece "Account verification is pending"?

Broadcom ejecuta un proceso automático llamado **Global Trade Management (GTM) screening** en cada cuenta nueva. Tu perfil queda bloqueado hasta que pase este filtro. Las causas más frecuentes son :



| Causa                       | Ejemplo que falla                                   |
| :-------------------------- | :-------------------------------------------------- |
| **Nombre incompleto**       | "J. Pérez" o solo iniciales                         |
| **Dirección incompleta**    | Solo P.O. Box, o calle sin número                   |
| **Caracteres no-inglés**    | Tildes, eñes (ñ), acentos en tu nombre o dirección  |
| **Screening GTM pendiente** | El sistema simplemente no ha terminado de revisarte |

> **Lo crítico:** Si usaste caracteres con tilde o la letra "ñ" en tu nombre o dirección, es muy probable que ese sea el bloqueo. Broadcom lo especifica explícitamente: *"Use only English characters and do not use special characters such as accent marks"*
>
> 

------

## 2. Solución oficial (la que Broadcom recomienda)

Sigue estos pasos en tu cuenta del [Broadcom Support Portal](https://support.broadcom.com/):

1. **Inicia sesión** y ve a **Terms and Conditions**.
2. Haz clic en el ícono de descarga del producto para llegar a la página **Trade Compliance and Download Conditions**.
3. **Actualiza tu perfil:**
   - Nombre completo (sin abreviaturas).
   - Dirección física completa: calle, número, ciudad, país. **Sin P.O. Box.**
   - **Todo en inglés**, sin tildes, eñes ni símbolos.
4. Haz clic en **Submit**.
5. **Espera.** El screening es automático, pero puede tardar desde minutos hasta varios días.

Si después de corregir tu perfil sigue bloqueado después de varios días, debes llamar al **Broadcom Global Customer Care** (no abrir un caso técnico, sino reportar un problema de acceso al portal web).

------

## 3. Alternativas si no quieres esperar o lidiar con Broadcom

Aquí te presento opciones ordenadas de más a menos recomendable:

### Opción A: Usar los VMware Hands-On Labs (gratis, inmediato)

Broadcom mantiene labs gratuitos en navegador donde puedes tocar **vCenter, ESXi, HA, vMotion, DRS** sin instalar nada. Es la forma más rápida de aprender el stack enterprise real sin descargar ISOs.

- **Pros:** Acceso inmediato, infraestructura real, sin verificaciones.
- **Contras:** Sesiones de duración limitada (unas horas), no puedes guardar configuraciones permanentes.

### Opción B: Proxmox VE (gratis, open source, sin verificaciones)

Si tu objetivo es **aprender virtualización enterprise** (clusters, HA, live migration, storage definido por software, redes VLAN), **Proxmox VE** es la alternativa más cercana a vSphere y completamente gratuita.



| Característica                | Proxmox VE                   | ESXi Free                |
| :---------------------------- | :--------------------------- | :----------------------- |
| Precio                        | Gratis (AGPL)                | Gratis                   |
| HA / Live Migration           | ✅ Sí                         | ❌ No                     |
| Gestión centralizada web      | ✅ Sí (nativa)                | ❌ Solo por host          |
| Storage definido por software | ✅ Ceph / ZFS integrado       | ❌ No                     |
| Backups                       | ✅ Proxmox Backup Server      | ❌ Sin API VADP           |
| Curva de aprendizaje          | Media (basado en Debian/KVM) | Baja (interfaz familiar) |

**Mi recomendación como tutor:** Si estás aprendiendo para un trabajo real, Proxmox te enseña más conceptos transferibles que ESXi Free (que está muy capado). Además, muchas empresas están migrando a Proxmox precisamente por el lío de Broadcom.

### Opción C: ESXi en modo evaluación (60 días, todo desbloqueado)

Si tienes la ISO de ESXi de cualquier fuente (incluso si un colega te la pasa), puedes instalarla **sin licencia** y tendrás **Enterprise Plus completo durante 60 días**: vCenter, vMotion, HA, DRS, APIs, todo.

- Instala ESXi en una VM anidada dentro de VMware Workstation.
- Descarga el **vCenter Server Appliance (VCSA)** y despliégalo.
- Tienes 60 días para aprender todo el stack enterprise.

> **Nota legal:** La ISO en sí no tiene costo; la licencia es lo que se paga. Tener la ISO no es ilegal. Lo que vence es el período de evaluación.

### Opción D: Workaround técnico (comunidad)

Existe un script de Tampermonkey publicado en GitHub que intercepta la respuesta del portal de Broadcom y elimina el campo `exportControlStatus`, permitiendo descargar directamente. Ha sido probado con ESXi, Workstation y vCenter.

- **Advertencia:** Es un workaround no oficial. Usarlo es bajo tu propia responsabilidad. Revisa el código antes de instalarlo.

------

## 4. Mi recomendación final como tutor

Dado que vienes de Linux y quieres prepararte para producción, te sugiero esta ruta **sin fricciones**:



| Paso  | Acción                                                       | Tiempo   |
| :---- | :----------------------------------------------------------- | :------- |
| **1** | Descarga e instala **VMware Workstation Pro** (es gratis ahora) en tu Linux. | 30 min   |
| **2** | Crea una VM anidada e instala **ESXi en modo evaluación** (60 días). | 1 hora   |
| **3** | Despliega **vCenter Server Appliance (VCSA)** dentro de esa VM. | 1 hora   |
| **4** | Juega 60 días con el stack completo: clusters, vMotion, HA, DRS. | 2 meses  |
| **5** | Paralelamente, instala **Proxmox VE** en otra VM o máquina vieja. | 1 hora   |
| **6** | Compara: VMware es el estándar enterprise, Proxmox es el futuro open source. | Continuo |

Así evitas el dolor de cabeza del portal de Broadcom, aprendes VMware real (no la versión capada), y descubres una alternativa que cada vez más empresas están adoptando.

------

¿Te gustaría que te guíe paso a paso para montar el **Laboratorio anidado de ESXi + vCenter en Workstation**, o prefieres que empecemos directamente con **Proxmox VE**?





> Cual es la ultima versión de VMware ESXi que puede utilizarse en producción.



La última versión de VMware ESXi disponible para producción en agosto de 2026 es **ESXi 9.1** (ESX 9.1.0.0200, build 25557999, liberada el 13 de julio de 2026). Sin embargo, hay un **cambio radical en el modelo de licenciamiento** que debes conocer antes de tomar cualquier decisión.

------

## 1. Versiones disponibles y su estado



| Versión      | Último Update         | Estado                       | Soporte General hasta     | Recomendación             |
| :----------- | :-------------------- | :--------------------------- | :------------------------ | :------------------------ |
| **ESXi 9.1** | 9.1.0.0200 (jul 2026) | ✅ **GA — Producción**        | ~octubre 2031*            | Última versión estable    |
| **ESXi 9.0** | 9.0.2.0 (ene 2026)    | ✅ GA — Producción            | ~septiembre 2030*         | Versión anterior estable  |
| **ESXi 8.0** | 8.0U3g (jul 2025)     | ✅ GA — Producción            | **11 de octubre de 2027** | Versión madura y probada  |
| **ESXi 7.x** | 7.0U3                 | ❌ **End of General Support** | 2 de octubre de 2025      | **NO usar en producción** |

*Estimado basado en el ciclo de vida estándar de 5 años de soporte general + 2 de orientación técnica.

------

## 2. El cambio más importante: vSphere 9 ya NO se vende solo

Este es el punto que cambia todo desde la adquisición de VMware por Broadcom:

> **A partir de vSphere 9.0, las ediciones Standard y Enterprise Plus dejaron de existir como productos independientes.**

### ¿Qué significa esto en la práctica?



| Antes (vSphere 8.x y anteriores)                             | Ahora (vSphere 9.x)                                       |
| :----------------------------------------------------------- | :-------------------------------------------------------- |
| Comprabas **vSphere Standard** o **Enterprise Plus** por socket | Ya no existen como SKUs independientes                    |
| Podías usar **ESXi Free** para labs                          | **ESXi Free NO existe para 9.x** — solo hasta 8.0U3       |
| Escalabas comprando más licencias de vSphere                 | Debes comprar **VVF 9** o **VCF 9** (paquetes integrados) |

### Las únicas opciones de licenciamiento para producción con 9.x:



| Producto                                | Qué incluye                        | Para quién                           |
| :-------------------------------------- | :--------------------------------- | :----------------------------------- |
| **VMware vSphere Foundation 9 (VVF 9)** | vSphere 9 + vSAN + Aria Operations | Entornos on-premise medianos/grandes |
| **VMware Cloud Foundation 9 (VCF 9)**   | VVF + NSX + Tanzu Kubernetes + más | Nube privada enterprise completa     |

------

## 3. ¿Qué versión debo usar en producción?

Depende de tu situación:

### Si ya tienes infraestructura VMware con licencias activas:

- **Si estás en vSphere 8.x:** Puedes quedarte en **ESXi 8.0U3g** hasta 2027. Es estable, probado y tienes soporte completo.
- **Si estás en vSphere 7.x:** Debes migrar **urgentemente**. El soporte general terminó en octubre de 2025.

### Si estás construyendo algo nuevo:

- **Con presupuesto enterprise:** Ve directo a **VVF 9.1** o **VCF 9.1**. Es el futuro de VMware y tiene features como NVMe memory tiering, vMotion offload a hardware, y topology-aware scheduling.
- **Con presupuesto limitado o lab personal:** Tu única opción oficial gratuita es **ESXi 8.0U3e Free**. La versión 9.x no tiene edición gratuita.

------

## 4. Novedades clave de ESXi 9.1 (si vas a 9.x)

Si decides ir a la última versión, estas son las mejoras más relevantes:



| Feature                          | Qué hace                                                     |
| :------------------------------- | :----------------------------------------------------------- |
| **NVMe Memory Tiering mejorado** | Usa SSDs NVMe como extensión de RAM, reduciendo costos hasta 40% |
| **vCenter Quick Patching**       | Parches de seguridad en vCenter con downtime cercano a cero  |
| **Live Patching para TPM**       | Aplica parches a ESXi sin reiniciar, incluso con TPM habilitado |
| **vMotion offload (Intel QAT)**  | El cifrado de vMotion se descarga a hardware, ahorrando ~70% de CPU |
| **EDR nativo en ESXi**           | Framework para que agentes de detección de amenazas corran directamente en el hypervisor |

------

## 5. Mi recomendación final como tutor



| Escenario                                         | Versión recomendada       | Por qué                                                      |
| :------------------------------------------------ | :------------------------ | :----------------------------------------------------------- |
| **Nuevo proyecto enterprise con licenciamiento**  | **VVF 9.1** o **VCF 9.1** | Es la única opción disponible para 9.x y el camino que VMware/Broadcom está impulsando |
| **Entorno existente estable en 8.x**              | **Quédate en 8.0U3g**     | Tienes soporte hasta 2027; no hay presión inmediata          |
| **Lab personal / aprendizaje / presupuesto cero** | **ESXi 8.0U3e Free**      | La última versión con licencia gratuita; 9.x no tiene opción free |
| **Migración desde 7.x**                           | **8.0U3g** o **VVF 9.1**  | 7.x ya no tiene soporte; sal de ahí lo antes posible         |

------

## Resumen ejecutivo

- **Última versión de ESXi:** 9.1.0.0200 (julio 2026).
- **¿Puedo usarla en producción?** Sí, pero **solo como parte de VVF 9 o VCF 9**.
- **¿Hay versión gratuita de ESXi 9?** **No.** ESXi Free murió en la 8.0U3.
- **¿Y si trabajo en una empresa que ya tiene VMware?** Probablemente estén en 8.x y tengan tiempo hasta 2027 para planificar la migración a 9.x (con el nuevo modelo de licenciamiento).

------

¿Te gustaría que profundice en cómo funciona el **licenciamiento de VVF 9.1** (que reemplazó a Enterprise Plus), o prefieres que explique cómo montar un **lab con ESXi 8.0U3e Free** (tu única opción gratuita oficial)?

