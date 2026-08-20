# Guía de despliegue RHEL-VMware

**Tema central**

Diseño e implementación de un laboratorio de virtualización de bajo consumo de recursos (usando VMware) orientado a la preparación práctica para certificaciones en Red Hat Enterprise Linux (RHEL).



> Consideraciones: 
>
> 1. Una persona que sabe de Linux pero no tiene mucha experiencia  trabajando en producción con virtualización utilizando VMware. 
> 2. Se necesita un entorno simulado para ensayar y reforzar conocimientos de administración de sistemas basados en Red Hat Linux Enterprise para optar a exámenes de certificaciones. 
> 3. Se dispone de un Hardware con limitaciones.



Desde la perspectiva de un jefe de operaciones e IT, **la impresión inicial es positiva, pragmática y realista**. Revela a un profesional con una base sólida en el sistema operativo central (Linux), que reconoce honestamente sus vacíos prácticos en hipervisores de nivel empresarial (VMware) y que busca cerrar esa brecha mediante la práctica en laboratorio antes de enfrentarse a un entorno de certificación o producción.

El enfoque de simular un entorno en hardware limitado demuestra iniciativa y capacidad de adaptación. Sin embargo, en un entorno de operaciones real, la limitación de hardware obliga a optimizar recursos al máximo (optando por instalaciones Minimal/headless, gestión mediante CLI o herramientas ligeras de orquestación local) para garantizar que los nodos simulados ejecuten las cargas de trabajo necesarias para el examen sin colapsar la máquina anfitriona.



**Preámbulo**

> La validación de competencias en la administración de sistemas Red Hat Enterprise Linux (RHEL) mediante certificaciones profesionales requiere una preparación eminentemente práctica y la exposición a escenarios reales de operación. Sin embargo, la brecha habitual entre el dominio de la línea de comandos de Linux y el despliegue de infraestructura sobre plataformas de virtualización de nivel empresarial como VMware suele presentar un reto técnico, especialmente cuando se trabaja bajo restricciones de hardware local.
>
> El presente documento establece los lineamientos técnicos, la estrategia de dimensionamiento de recursos y el diseño de la arquitectura para implementar un entorno de pruebas virtualizado eficiente. El objetivo principal es maximizar el rendimiento del hardware disponible, garantizando un laboratorio funcional, ágil y representativo de los escenarios evaluados en los exámenes de certificación de Red Hat.



- [Escenarios de virtualización](/doc/escenarios-virtualizacion.md)
- [Soluciones disponibles](/doc/soluciones-disponibles.md)
- [Evaluacion/elección del Hardware](/doc/evaluacion-hardware.md)
- [Hardware disponible](/doc/hardware-disponible.md)
- [Evaluando la única ruta](/doc/evaluando-unica-ruta.md)
- [Justificación de esta iniciativa](/doc/justificacion-proyecto.md)
- [Virtualización anidada](/doc/virtualizacion-anidada.md)
- [Guía de Implementación](/doc/despliegue.md)
- [Problemas de red durante la implementación](/doc//diagnosticos-red.md)
- [Gajes del oficio con sistemas MS Windows](/doc//gajes-del-oficio-sistemas-windows.md)
- [Snapshot y Almacenamiento adicional](/doc/snapshot_add-disk.md)




## Recomendaciones

#### Instalación en Windows 11

A fin de lograr una implementación exitosa de **VMware ESXi 8.0** con todos los componentes de (Bridges, servicios), es importante seguir una secuencia de instalación:

1.  VMware WorkStation 17.6.4
2.  VMware WorkStation 26H1

La primera instalación comprende todos los componentes esenciales para la redes en modo **bridge**.  La versión **26H1** al detectar una versión previamente instalada, realizará un **upgrade**.

---

#### Optimiza la consola de comandos

Para quienes estamos acostumbrados a una consola de comandos para conectar a servidores o ejecutar tareas en nuestra propia estación de trabajo en Linux, cambiar ese entorno de trabajo a Windows puede resultar algunas veces una experiencia chocante e improductiva.

Sugiero PSMUX para mejorar la experiencia de usuario. 

1. Revisa el enlace y actívalo ahora:

[Instalación](/doc/use-psmux.md)

2. Abre CMD o Powershell:

```cmd
Microsoft Windows [Version 10.0.26200.9168]
(c) Microsoft Corporation. All rights reserved.

D:\>psmux
```











---

Por: Willians Patiño 



