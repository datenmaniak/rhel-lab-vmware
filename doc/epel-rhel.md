# EPEL en RHLE



## Es permitido instalar repositorios EPEL en RHLE ?



Sí, **está permitido instalar EPEL en RHEL**, y Fedora mantiene EPEL específicamente para RHEL. Sin embargo, EPEL **no forma parte de RHEL ni está cubierto por el soporte oficial de Red Hat**; sus paquetes tienen soporte comunitario y no están dentro del alcance normal de soporte de producción de Red Hat.fedoraproject+1

## Consideraciones importantes

- EPEL intenta evitar conflictos o reemplazos de paquetes incluidos en RHEL.[fedoraproject](https://docs.fedoraproject.org/en-US/epel/)
- Red Hat no garantiza SLA, correcciones ni soporte para problemas causados por paquetes de EPEL.[redhat](https://www.redhat.com/en/blog/whats-epel-and-how-do-i-use-it)
- En servidores críticos o sujetos a certificaciones, revisa antes las políticas internas, del cliente o del fabricante.
- Conviene habilitar también **CodeReady Builder** —llamado `crb` en derivados como Rocky o AlmaLinux— porque algunos paquetes de EPEL dependen de bibliotecas presentes allí.[docs.fedoraproject](https://docs.fedoraproject.org/en-US/epel/getting-started/)

## Ejemplo para RHEL 9

```bash
sudo subscription-manager repos \
  --enable codeready-builder-for-rhel-<VERSION>-$(arch)-rpms
```

**(RECOMENDADO)**

Reemplaza el `<VERSION>` por la versión instalado en tu sistema. 

```bash
sudo dnf install \
  https://dl.fedoraproject.org/pub/epel/epel-release-latest-<VERSION>.noarch.rpm

sudo dnf makecache
```

Para RHEL 8, cambia `rhel-9` por `rhel-8`:  

```
bashsudo subscription-manager repos \
  --enable codeready-builder-for-rhel-<VERSION>-$(arch)-rpms
```

Verifica primero la versión:

```
bashcat /etc/redhat-release
rpm -E '%{rhel}'
```

En resumen: **sí puedes usar EPEL**, pero debes tratarlo como un repositorio comunitario adicional, no como parte del producto soportado oficialmente por Red Hat.

