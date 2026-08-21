# Durante el Simulacro EX200 — RHCSA

Anotaciones de comandos y configuraciones importantes para recordar  y de utilidad para futuras implementaciones.



## Autofs

**/etc/auto.master**

```bash
/shares/apps   /etc/shares.apps --timeout=60
```

**/etc/shares.apps**

```bash
shares  -fstype=ext4            :/data/apps
```

[Configure autofs Timeout](https://oneuptime.com/blog/post/2026-03-04-autofs-timeout-caching-settings-rhel-9/view)



## NFS

**nfs-utils**

```bash
/data/apps     192.168.1.0/24(rw,sync,no_root_squash)
```



## LVM

**Se crea el volumen físico PV:**

```bash
pvcreate /dev/sdb
pvs
```

**Grupo de volumen VG:**

```bash
vgcreate vg_production /dev/sdb
vgs
```

**Volumen lógico LV:**

```bash
 lvcreate -L8G -n /dev/vg_production/lv_apps
 lvcreate -L6G -n /dev/vg_production/lv_db
 lvcreate -L4G -n /dev/vg_production/lv_archive
 lvs
 
```

**Formateo:**

```\bash
mkfs.ext4 -L apps /dev/vg_production/lv_apps
mkfs.ext4 -L archive /dev/vg_production/lv_archive
mkfs.xfs -L db /dev/vg_production/lv_db
```

---



### Extracción rápida de los UUID para el montaje persistente 

```bash
 blkid | awk '/lv_/ {print $2, $3, $5} '
```

```text
LABEL="archive" UUID="7ab61686-fc51-486b-8447-04e622783078" TYPE="ext4"
LABEL="apps" UUID="acb6c751-16c3-4725-9d24-081140652437" TYPE="ext4"
LABEL="db" UUID="51f79cbf-3e46-435c-a37a-359c00c47246" TYPE="xfs"
```

#### Preparar el mapeo de las particiones

```bash
 blkid | awk '/lv_/ {print $2, $3, $5 }' > uuid.txt
```

Editar `uuid.txt`

```bash
cat uuid.txt
```
```text
#LABEL="archive"
UUID="7ab61686-fc51-486b-8447-04e622783078" /data/archive  ext4 defaults 0 0
#TYPE="ext4"

#LABEL="apps"
UUID="acb6c751-16c3-4725-9d24-081140652437" /data/apps ext4     defaults 0 0
#TYPE="ext4"

#LABEL="db"
UUID="51f79cbf-3e46-435c-a37a-359c00c47246"  /data/db  xfs      defaults 0 0
#TYPE="xfs"
````

#### Revisar antes de incluir en `/etc/fstab`:

```bash
cat uuid.txt >> /etc/fstab
```

**Comprobacion:**

```bash
mount -a
```

```bash
systemctl daemon-reload'
```























