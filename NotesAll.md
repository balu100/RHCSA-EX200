# RHCSA Dump → Combined Quick‑Glance Patterns

One canonical example per recurring task, in the same **quick‑scan** style. Adjust only the **[CHOOSE]** parts.

```text
[CHOOSE] you decide value        [COMMON] typical default        [MUST] required syntax/behavior
[EASIER] shortest RHCSA-OK form  [ALT] acceptable alternative
```

---

## Cron — daily at exact time

```bash
# For USER at 14:23 every day
crontab -u USER -e
23 14 * * * /bin/echo "Hello"
# [COMMON] service ensure
systemctl enable --now crond
```

---

## DNF repo — BaseOS + AppStream (HTTP)

```ini
# /etc/yum.repos.d/local.repo
[BaseOS]
name=BaseOS
baseurl=http://HOST/x/BaseOS
enabled=1
gpgcheck=0
[AppStream]
name=AppStream
baseurl=http://HOST/x/AppStream
enabled=1
gpgcheck=0
```

---

## Find & copy files owned by user

```bash
# Copy owned-by USER → /opt/dir, keep tree + attrs, stay on root FS
mkdir -p /opt/dir
find / -xdev -user USER -type f -exec cp --parents -a -t /opt/dir {} +   # [EASIER]
# ALT (slower, per-file): ... -exec cp -a {} /opt/dir \;
```

---

## LVM — create minimal LV, mount, extend

```bash
# No partitioning  [EASIER]
pvcreate /dev/sdb
vgcreate vgdata /dev/sdb
lvcreate -n lvdata -L 2G vgdata
mkfs.xfs /dev/vgdata/lvdata
mkdir -p /data
mount /dev/vgdata/lvdata /data
echo '/dev/vgdata/lvdata /data xfs defaults 0 0' >> /etc/fstab

# Online grow (any FS that supports online grow: XFS/ext4)
lvextend -r -L +5G /dev/vgdata/lvdata
```

---

## LVM — specific PE size + extents

```bash
# 16M PE, LV of 50 extents (50×16M = 800M)
pvcreate /dev/sdc
vgcreate -s 16M vg1 /dev/sdc
lvcreate -l 50 -n lv1 vg1
mkfs.ext4 /dev/vg1/lv1
mkdir -p /mnt/data
echo '/dev/vg1/lv1 /mnt/data ext4 defaults 0 0' >> /etc/fstab && mount -a
```

---

## Mount ISO permanently

```bash
mkdir -p /mnt/iso
echo '/root/exam.iso  /mnt/iso  iso9660  loop  0 0' >> /etc/fstab
mount -a
```

---

## Swap — file (fastest) and partition (ALT)

```bash
# File swap 600M  [EASIER]
fallocate -l 600M /swapfile || dd if=/dev/zero of=/swapfile bs=1M count=600
chmod 600 /swapfile && mkswap /swapfile
echo '/swapfile swap swap defaults 0 0' >> /etc/fstab
swapon -a

# ALT partition swap (assume /dev/vdb2)
mkswap /dev/vdb2
echo '/dev/vdb2 swap swap defaults 0 0' >> /etc/fstab && swapon -a
```

---

## Directory with group RW + SGID

```bash
# /home/admins: group=admin, rwx for owner+group, none for others, force group inheritance
mkdir -p /home/admins
chgrp admin /home/admins
chmod 2770 /home/admins       # 2 = SGID
```

---

## Users & groups (common combos)

```bash
groupadd admin
useradd -G admin natasha
useradd -G admin harry
useradd -s /sbin/nologin sarah
for u in natasha harry sarah; do echo password | passwd --stdin "$u"; done   # [EASIER]

# Specific UID example
useradd -u 3400 alex && echo redhat | passwd --stdin alex
```

---

## AutoFS — simple NFS map

```bash
# /etc/auto.master
/automount  /etc/auto.automount  --timeout=30

# /etc/auto.automount
public  -rw,soft  nfs.example.com:/export/public

systemctl enable --now autofs
# Access: /automount/public
```

---

## SELinux — allow nonstandard HTTP port + content label

```bash
semanage port -a -t http_port_t -p tcp 8081
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web
```

---

## Chrony — simple source

```bash
# /etc/chrony.conf (append one line)
server 192.0.2.1 iburst
systemctl enable --now chronyd
```

---

## Firewall — common services

```bash
firewall-cmd --add-service={ssh,http,https} --permanent
firewall-cmd --reload
```

---

## IP forwarding (router task)

```bash
sysctl -w net.ipv4.ip_forward=1
printf 'net.ipv4.ip_forward=1\n' >/etc/sysctl.d/99-router.conf
```

---

## Kernel update (keep old kernel)

```bash
dnf update -y kernel
# New kernel becomes default automatically on RHEL/Rocky 9
```

---

## "Install everything needed" — lean default + add‑ons

```bash
# Lean default (covers most tasks)
dnf install -y policy* mod_ssl httpd mandb

# Add‑ons when needed
dnf install -y autofs* lvm2* acl* firewalld* chrony* podman* nfs* NetworkManager-tui*
```

---

## Help quick‑glance

```bash
man CMD          # manual
man -k WORD      # keyword search
CMD --help       # quick usage
dnf search WORD  # package search
```
