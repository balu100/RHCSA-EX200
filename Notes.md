# RHCSA EX200 – Compact Focus Guide

Targeted drill notes for these tasks:
- **Q3 SELinux/Firewall**
- **Q6 Autofs**
- **Q8 ACL**
- **Q10 Find & Copy**
- **Q16 Script**
- **Q19 LVM**
- **Q22 Containers (Podman)**

> Syntax uses RHEL 9 defaults. Replace placeholders in **UPPER_CASE**.

---

## Q3 — SELinux + Firewall

### SELinux: quick ops
```bash
# Show mode and contexts
getenforce; sestatus; id -Z; ps -eZ | head
ls -lZ /PATH

# Switch enforcing temporarily
setenforce 1        # or 0

# Permanent mode
vi /etc/selinux/config   # SELINUX=enforcing|permissive|disabled

# Fix mislabeled tree
restorecon -Rv /PATH

# Add custom context (persists)
semanage fcontext -a -t httpd_sys_content_t '/srv/www(/.*)?'
restorecon -Rv /srv/www

# Allow service feature via boolean
getsebool -a | grep httpd
setsebool -P httpd_can_network_connect on

# Add SELinux port mapping
semanage port -a -t http_port_t -p tcp 8080
semanage port -l | grep http
```

### Firewalld: quick ops
```bash
# Check zones and active
firewall-cmd --get-active-zones
firewall-cmd --list-all

# Open a known service in active zone
firewall-cmd --add-service=http --permanent
firewall-cmd --reload

# Open a port
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

# Verify
firewall-cmd --list-services; firewall-cmd --list-ports

# Rich rule example (source CIDR)
firewall-cmd --add-rich-rule='rule family=ipv4 source address=192.168.50.0/24 service name=ssh accept' --permanent
firewall-cmd --reload
```

**Checks**
- SELinux: `ausearch -m AVC -ts recent` shows denials. `sealert -a /var/log/audit/audit.log` for hints.
- Firewall: `ss -tulpen` confirms listener. `firewall-cmd --state` is running.

---

## Q6 — Autofs (NFS)

### Server side (on **NFS_SERVER**)
```bash
dnf -y install nfs-utils
mkdir -p /srv/share/docs
chmod 755 /srv/share
chmod 755 /srv/share/docs

echo '/srv/share/docs 192.168.0.0/16(ro)' >> /etc/exports
exportfs -rav
systemctl enable --now nfs-server
firewall-cmd --add-service=nfs --permanent && firewall-cmd --reload
```

### Client side (autofs)
```bash
dnf -y install autofs nfs-utils

# Master map
echo '/- /etc/auto.direct' >> /etc/auto.master.d/direct.autofs

# Direct map: mount at /mnt/docs on demand
mkdir -p /mnt/docs
cat >/etc/auto.direct <<'EOF'
/mnt/docs  -rw,soft,intr  NFS_SERVER:/srv/share/docs
EOF

systemctl enable --now autofs

# Verify: touch triggers mount
ls /mnt/docs; mount | grep /mnt/docs; systemctl status autofs --no-pager
```

**Notes**
- Indirect map alternative: `/NFS  /etc/auto.nfs` then lines like `docs -ro NFS_SERVER:/srv/share/docs` create `/NFS/docs`.
- If SELinux blocks, ensure `nfs_t` contexts are used by default. Client usually fine.

---

## Q8 — ACL

```bash
dnf -y install acl   # usually preinstalled

# Enable ACL on XFS (default is enabled). For ext4 ensure fstab has acl option.
# Set user and group ACLs
setfacl -m u:alice:rX,u:bob:rwX,g:devs:rx /data/project

# Default ACLs for new files
setfacl -m d:u:alice:rwX,d:g:devs:rwx /data/project

# Recursively apply to existing tree
setfacl -R -m g:qa:rx /data/project

# View
getfacl /data/project | less

# Remove entries
setfacl -x u:alice,g:devs /data/project
setfacl -b /data/project   # blow away all ACLs

# Mask controls the max effective bits beyond owner/group/other
setfacl -m m:rwx /data/project
```

**Pitfalls**
- Effective permissions can be reduced by **mask**. Use `getfacl` to debug.
- Ensure basic Unix owner/group bits do not contradict the task.

---

## Q10 — Find & Copy

### Common patterns
```bash
# Find by name, type, size, mtime
find /var/log -type f -name '*.log' -size +10M -mtime -3

# Find owned by user with mode bits
find /home -type f -user alice -perm -200   # writable by owner

# Find by content (text)
grep -RIl 'PATTERN' /path   # files with PATTERN, I=ignore binary, l=names

# Copy matches preserving attributes to single dir
find /src -type f -name '*.conf' -exec cp -a --parents -t /dest {} +
# or
rsync -aR --include='*.conf' --exclude='*' /src/ /dest/

# Delete empty files or dirs
find /tmp/old -type f -empty -delete
find /tmp/old -type d -empty -delete

# Safer: preview then exec
find /etc -name '*.rpmnew' -print
find /etc -name '*.rpmnew' -exec mv -v {} /root/rpmnew/ \;
```

**Checks**
- `tree /dest` or `rsync -anR` dry-run first.
- Use `-exec ... +` for batching speed.

---

## Q16 — Script (bash)

### Template you can adapt in seconds
```bash
#!/usr/bin/env bash
set -euo pipefail
usage(){ echo "Usage: $0 [-n NAME] [-f FILE]"; }
NAME=""
FILE=""
while getopts ":n:f:h" o; do
  case "$o" in
    n) NAME="$OPTARG";;
    f) FILE="$OPTARG";;
    h) usage; exit 0;;
    *) usage; exit 2;;
  esac
done

[[ -n "${NAME}" ]] || { echo "NAME required"; exit 2; }
[[ -n "${FILE}" ]] || { echo "FILE required"; exit 2; }

if [[ -f "$FILE" ]]; then
  echo "[$(date +%F_%T)] $NAME -> $(wc -l < "$FILE") lines"
else
  echo "no such file: $FILE"; exit 1
fi
```

### Patterns to memorize
```bash
# Read file line by line with safety
while IFS= read -r line; do echo "$line"; done < input.txt

# For loop on glob
for f in /etc/*.conf; do cp -a "$f" /backup/; done

# Case statement
case "$1" in start) systemctl start "$2";; stop) systemctl stop "$2";; *) echo usage; exit 2;; esac

# Exit codes
cmd || { echo "failed"; exit 1; }
```

---

## Q19 — LVM

### Create PV → VG → LV → FS
```bash
# Identify disk
ds="/dev/sdb"   # example

# Create PV and VG
pvcreate "$ds"
vgcreate vgdata "$ds"

# Create LV by size or percent free
lvcreate -n lvlogs -L 4G vgdata
# or use all free space
# lvcreate -n lvlogs -l 100%FREE vgdata

# Make FS and mount
mkfs.xfs /dev/vgdata/lvlogs
mkdir -p /logs
blkid /dev/vgdata/lvlogs >> /etc/fstab
# add line to fstab (XFS):
# UUID=<copied>  /logs  xfs  defaults  0 0
mount -a && mount | grep /logs
```

### Extend LV and FS
```bash
# Extend LV by size or percent
lvextend -r -L +2G /dev/vgdata/lvlogs    # -r grows FS online (xfs_growfs)
# or use ext4
# lvextend -L +2G /dev/vgdata/lvdata; resize2fs /dev/vgdata/lvdata
```

### Reduce ext4 only (not XFS)
```bash
umount /data
fsck -f /dev/vgdata/lvdata
resize2fs /dev/vgdata/lvdata 8G
lvreduce -L 8G /dev/vgdata/lvdata
mount /data
```

### Add new disk to VG
```bash
pvcreate /dev/sdc
vgextend vgdata /dev/sdc
pvs; vgs; lvs -a -o +devices
```

**Checks**
- `lsblk -f`, `vgs -o +vg_free`, `lvs -a -o +devices`.

---

## Q22 — Containers (Podman)

### Run a simple container
```bash
dnf -y install podman
podman run --name web -d -p 8080:80 docker.io/library/nginx:alpine
podman ps; curl -I http://127.0.0.1:8080
```

### Persistent data with volume
```bash
mkdir -p /srv/nginx/html
podman run --name web -d -p 8080:80 -v /srv/nginx/html:/usr/share/nginx/html:Z docker.io/library/nginx:alpine
# :Z applies SELinux label for container sharing
```

### Keep container across reboots via systemd (generate unit)
```bash
# Option A: generate unit from existing container
podman generate systemd --name web --files --new
mkdir -p /etc/systemd/system
mv container-web.service /etc/systemd/system/
systemctl enable --now container-web

# Option B (RHEL9+): Quadlet file
cat >/etc/containers/systemd/web.container <<'EOF'
[Container]
Image=docker.io/library/nginx:alpine
PublishPort=8080:80
Volume=/srv/nginx/html:/usr/share/nginx/html:Z
ContainerName=web
EOF
systemctl daemon-reload
systemctl enable --now web.container
```

### Build from Containerfile
```bash
cat >Containerfile <<'EOF'
FROM registry.access.redhat.com/ubi9/ubi-minimal
RUN microdnf -y install httpd && microdnf clean all
EXPOSE 8080
CMD ["/usr/sbin/httpd","-D","FOREGROUND","-f","/etc/httpd/conf/httpd.conf","-k","start"]
EOF
podman build -t myhttpd:1 .
podman run -d -p 8080:8080 --name myhttpd myhttpd:1
```

**Checks**
- `podman images`, `podman ps --all`, `systemctl status web.container`.
- SELinux denials: `journalctl -t audit`, add `:Z` to volumes.

---

## Speed Cards

### SELinux triage
1) Try `restorecon`. 2) Add `semanage fcontext` + `restorecon`. 3) Toggle boolean. 4) Map port.

### Firewalld triage
Add service → port → rich rule. Always `--permanent` then `--reload`.

### Autofs recipe
`auto.master.d` → direct map `/-` or indirect map `/NFS` → enable → test with `ls`.

### ACL recipe
`setfacl -m` user/group + default. Verify with `getfacl`.

### Find & copy
Use `cp -a --parents` or `rsync -aR`. Always dry-run with `-n`.

### LVM growth
`pvcreate` → `vgextend` → `lvextend -r`.

### Podman persistence
Prefer Quadlet on RHEL 9: drop `*.container` into `/etc/containers/systemd/`.

---

## Verification Checklist (fast)
- **SELinux**: `getenforce`=Enforcing, `semanage fcontext -l | grep PATH`, `semanage port -l`.
- **Firewall**: service/port listed after reload.
- **Autofs**: `mount | grep autofs`, access triggers mount.
- **ACL**: `getfacl` shows intended entries and mask.
- **Find/Copy**: result count equals `find ... | wc -l`.
- **LVM**: `lsblk -f` shows FS and mount; `lvs` size matches.
- **Containers**: service enabled; `curl` to exposed port works.



---

`ll = ls -lahis`   # max verbosity 

r=4, w=2, x=1
0=---
1=--x
2=-w-
3=-wx
4=r--
5=r-x
6=rw-
7=rwx

1xxx sticky → `t` in others exec

* 1777 `/tmp` → `drwxrwxrwt`

2xxx setgid → `s` in group exec

* 2755 → `drwxr-sr-x`
* 2770 → `drwxrws---`

4xxx setuid → `s` in owner exec

### Swap setup

mkswap /dev/sdX
echo '/dev/sdX swap swap defaults 0 0' >> /etc/fstab
swapon -a
swapon -s   # verify

### Crontab

`crontab -e`
`minute hour day_of_month month day_of_week command`

Example:
`0 3 * * * /usr/local/bin/backup.sh` → runs daily at 03:00

Ops: `*` any, `,` list, `-` range, `*/N` every N
Specials: `@reboot @hourly @daily @weekly @monthly @yearly`

Deny user: `echo user >> /etc/cron.deny`

### man quick info

`man -f <cmd>` → show manual section summary
Example: `man -f crontab` → shows `crontab (1)` and `crontab (5)`

* `(1)` = user command
* `(5)` = config file format
  Search text inside man: `/word`
  Next result: `n`

### umask

`umask` sets default file permissions (subtracts bits from 777/666).
Example: `umask 027` → new files 640, dirs 750.

To make it persistent for one user:

```bash
su - username
nano ~/.bash_profile
```

Add at end:

```bash
umask 027
```

Then re-login or `source ~/.bash_profile`.



---

### Q6 — Autofs: master map explained (why it exists)
- **Master map** tells `automount` *where to watch* and *which child map to use*.
  - Files: `/etc/auto.master` and drop-ins `/etc/auto.master.d/*.autofs`.
  - Syntax: `<mountpoint>  <mapfile>  [options]`.
- **Indirect map** (common): attaches submounts under a base dir.
  - Master: `/NFS  /etc/auto.nfs  --timeout=60`
  - Map `/etc/auto.nfs`:
    ```
    docs  -ro    NFS_SERVER:/srv/share/docs   # becomes /NFS/docs on access
    iso   -rw    NFS_SERVER:/srv/iso
    ```
- **Direct map**: mounts exact paths anywhere via `/-`.
  - Master: `/-  /etc/auto.direct  --timeout=60`
  - Map `/etc/auto.direct`:
    ```
    /mnt/docs   -rw,soft    NFS_SERVER:/srv/share/docs
    /srv/iso    -ro         NFS_SERVER:/srv/iso
    ```
- **Why Autofs vs fstab** ✔ on‑demand mount, ✔ auto‑unmount idle, ✔ faster boot if server is down, ✔ per‑map options. ⚠ Needs `autofs` service.

---

### Q19 — LVM: PV → VG → LV create path (clarified)
**Mental model**: **PV**=disks/partitions, **VG**=pool, **LV**=virtual disk (format and mount).

**Shortest full example (use whole disk)**
```bash
DSK=/dev/sdb
pvcreate $DSK                  # make a PV on the disk
vgcreate vgdata $DSK           # create pool
lvcreate -n lvlogs -L 4G vgdata
mkfs.xfs /dev/vgdata/lvlogs
mkdir -p /logs
UUID=$(blkid -s UUID -o value /dev/vgdata/lvlogs)
echo "UUID=$UUID  /logs  xfs  defaults  0 0" >> /etc/fstab
mount -a && lsblk -f | grep logs
```

**If you need a partitioned disk** (GPT + LVM flag)
```bash
parted -s /dev/sdb mklabel gpt mkpart primary 1MiB 100% set 1 lvm on
pvcreate /dev/sdb1
vgcreate vgdata /dev/sdb1
# then same as above
```

**Grow later**
```bash
# add disk and extend pool
pvcreate /dev/sdc; vgextend vgdata /dev/sdc
# extend LV and filesystem online (XFS)
lvextend -r -L +2G /dev/vgdata/lvlogs
```

**Pitfalls**
- XFS cannot shrink (use ext4 for shrink tasks).  
- Ensure fstab uses **UUID**.  
- Verify free extents: `vgs -o +vg_free`; mapping: `lvs -o +devices`.



---

# Easy defaults (exam speed mode)

### Networking — use `nmtui`
```
nmtui
# Edit connection → IPv4 Automatic (DHCP) or Manual (addr/prefix/gateway)
# Add DNS (e.g., 1.1.1.1 8.8.8.8)
# Activate connection, optionally Set system hostname
```
Quick checks: `ip a`, `ip r`, `ping -c2 1.1.1.1`, `ping -c2 example.com`.

### Firewall — open the port, reload
```
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-service=nfs --permanent   # when using NFS/autofs
firewall-cmd --reload
```
If a service exists, you can use `--add-service=<name>`; ports always work.

### SELinux — minimal fixes
```
restorecon -Rv /PATH
setsebool -P httpd_can_network_connect on     # common web/network need
# For Podman bind mounts add :Z
#   -v /srv/nginx/html:/usr/share/nginx/html:Z
```

### Autofs — one recipe (direct map)
```
dnf -y install autofs nfs-utils
printf '/- /etc/auto.direct
' > /etc/auto.master.d/direct.autofs
printf '/mnt/docs  -rw  NFS_SERVER:/srv/share/docs
' > /etc/auto.direct
systemctl enable --now autofs
ls /mnt/docs && mount | grep /mnt/docs
```
Use direct map for exact paths. Indirect maps are optional.

### Find & Copy — prefer rsync include tree
```
rsync -aR --include='*.conf' --exclude='*' /src/ /dest/
# Preview: rsync -anR ...
```

### LVM — mnemonic: PV → VG → LV → FS → mount → fstab
```
pvcreate /dev/sdb
vgcreate vgdata /dev/sdb
lvcreate -n lvdata -L 5G vgdata
mkfs.xfs /dev/vgdata/lvdata
mkdir -p /data
UUID=$(blkid -s UUID -o value /dev/vgdata/lvdata)
echo "UUID=$UUID /data xfs defaults 0 0" >> /etc/fstab
mount -a && lsblk -f | grep /data
```
Grow fast:
```
lvextend -r -L +2G /dev/vgdata/lvdata
```

### Containers (Podman) — shortest persistent web
```
podman run -d --name web -p 8080:80 \
  -v /srv/nginx/html:/usr/share/nginx/html:Z docker.io/library/nginx:alpine
podman generate systemd --name web --files --new
mv container-web.service /etc/systemd/system/
systemctl enable --now container-web
```

### Common service checks
```
systemctl status <name> --no-pager
ss -tulpen | grep -E ':80|:8080|:2049'
```

