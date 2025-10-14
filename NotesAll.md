# RHCSA Quick‑Glance Cheatsheet

> Minimal commands and config snippets in a **quick‑scan** format. Copy/paste friendly for exam practice.

---

## GLANCE LEGEND

```text
[CHOOSE] you decide value        [COMMON] typical default        [MUST] required syntax/behavior
[EASIER] shortest RHCSA-OK form  [ALT] acceptable alternative
```

---

## AutoFS

```bash
# /etc/auto.master
/automount  /etc/auto.automount  --timeout=30
# /automount          [CHOOSE] parent mount root
# /etc/auto.automount [CHOOSE] map file path
# --timeout=30        [CHOOSE] idle unmount seconds (COMMON: 300)

# /etc/auto.automount
public  -ro,sync  nfs.lab.com:/public
# public        [CHOOSE] subdir name → becomes /automount/public
# -ro,sync      [CHOOSE] mount opts (ALT: -rw,soft / -rw,hard / -nosuid / -noexec)
# nfs.lab.com   [CHOOSE] server FQDN/IP
# :/public      [CHOOSE] exported path on server

# Result: touching /automount/public triggers mount; unmount after timeout.
```

---

## ACLs

```bash
# View
ls -lahi                      # [MUST] quick view
getfacl FILE_OR_DIR          # [MUST] show ACLs

# Grant user / group
setfacl -m u:harry:rwx  /data/file.txt   # [EASIER]
setfacl -m g:devs:rx    /data            # [EASIER]
# PERMS = r,w,x → combine as rwx / r-x / rw- / r--

# Remove / reset
setfacl -x u:harry /data/file.txt        # remove entry
setfacl -b          /data/file.txt       # wipe all ACLs

# Default ACLs on a dir (inherit to new files/dirs)
setfacl -m d:u:harry:rw /shared

# Copy ACLs
getfacl source | setfacl --set-file=- target
```

---

## LVM

```bash
# CHOOSE disk layout (both valid on RHCSA)

# No partitioning  [EASIER]
pvcreate /dev/sdb
vgcreate vgdata /dev/sdb
lvcreate -n lvdata -L 2G vgdata

# With partitioning  [ALT]
cfdisk /dev/sdb      # make sdb1 = Linux LVM (8e00), write, quit
pvcreate /dev/sdb1
vgcreate vgdata /dev/sdb1
lvcreate -n lvdata -L 2G vgdata

# Filesystem + mount
mkfs.xfs /dev/vgdata/lvdata                        # [CHOOSE] FS (ALT: mkfs.ext4 …)
mkdir -p /data
mount /dev/vgdata/lvdata /data
echo '/dev/vgdata/lvdata /data xfs defaults 0 0' >> /etc/fstab   # [ALT]: use UUID=

# Online extend LV + filesystem in one step (ext4 or XFS)
lvextend -r -L +500M /dev/vgdata/lvdata            # -r auto-runs xfs_growfs/resize2fs
# [TIP] -L = absolute size;  -l = extents/percent (e.g., -l +100%FREE)

# Add a new disk → grow VG → extend LV
pvcreate /dev/sdd
vgextend vgdata /dev/sdd
lvextend -r -l +100%FREE /dev/vgdata/lvdata

# Inspect
pvs; vgs; lvs; lsblk

# Notes: XFS grow only; ext4 grow online, shrink offline (rare on RHCSA).
```

---

## SELinux: `semanage`

```bash
# List known ports for a service type
semanage port -l | grep http

# Allow nonstandard port
semanage port -a -t http_port_t -p tcp 8081         # [EASIER]
# [CHOOSE] type: http_port_t / ssh_port_t / dns_port_t …
# [CHOOSE] proto: tcp|udp  [CHOOSE] port: number

# Modify / delete
semanage port -m -t http_port_t -p tcp 8081
semanage port -d -t http_port_t -p tcp 8081

# Persistent file contexts
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web
```

---

## find — RHCSA quick patterns

```bash
# Copy files owned by USER to /DEST (flat dest)  [EASIER memory]
find / -xdev -user USER -type f -exec cp -a {} /DEST \;

# Keep directory tree + attrs (if task requires original paths)
find / -xdev -user USER -type f -exec cp --parents -a -t /DEST {} +

# Recent big logs → list
find /PATH -type f -name "*.log" -size +10M -mtime -7 -print

# Fix dirs with 000 perms so they’re enterable
find /PATH -type d -perm -000 -exec chmod o+x {} +

# SUID audit (common)
find / -xdev -perm -4000 -type f -ls
```

---

## Password aging

```bash
# /etc/login.defs (defaults for NEW users)
PASS_MIN_DAYS 0
PASS_MAX_DAYS 20
PASS_WARN_AGE 7
# [CHOOSE] numbers. Labels fixed. Applies to users created AFTER change.

# Existing user (often needed)
chage -m 0 -M 20 -W 7 USER   # [EASIER]
```

---

## DNF local repos

```ini
# /etc/yum.repos.d/dvd.repo
[BaseOS]
name=BaseOS
baseurl=http://domain.tld/x/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://domain.tld/x/AppStream
enabled=1
gpgcheck=0
# [CHOOSE] section headers are conventional, not mandatory.
# [CHOOSE] name= is a label; baseurl must match your served path.
# [CHOOSE] gpgcheck=0 only if no keys; otherwise 1 + gpgkey=FILE/URL.
```

---

## Packages + Chrony

```bash
# EASIER installs (lean default)
dnf install -y policy* mod_ssl httpd mandb chrony tuned

# Optional bulk (only if task appears)
dnf install -y autofs* lvm2* acl* firewalld* podman* nfs* NetworkManager-tui* man*

# /etc/chrony.conf
server 192.0.0.1 iburst     # [CHOOSE] server/pool; 'iburst' = fast initial sync
# ALT: pool pool.ntp.org iburst
# Check: systemctl enable --now chronyd; chronyc sources -v
```

---

## Podman → systemd

```bash
podman generate systemd --new --files --name EXAMPLE1
# Output .service in CWD → move:
install -m 644 EXAMPLE1.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now EXAMPLE1
# [CHOOSE] container name; flags fixed in meaning.
```

---

## Help quick‑glance

```bash
man CMD                  # open manual
man 5 crontab            # section hint: 1=cmds, 5=formats, 8=admin
man -k KEYWORD           # search manuals by keyword
whatis CMD               # one-line summary
CMD --help               # builtin help

dnf search KEYWORD       # find package by name/summary
dnf provides '*/semanage'# which package ships a file/binary
which CMD                # where the command resolves from
```
