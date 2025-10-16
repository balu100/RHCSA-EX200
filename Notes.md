# RHCSA Quick‑Glance Cheatsheet

> Minimal commands and config snippets in a **quick‑scan** format. Copy/paste friendly for exam practice.

---

## GLANCE LEGEND

```text
[CHOOSE] you decide value        [COMMON] typical default        [MUST] required syntax/behavior
[EASIER] shortest RHCSA-OK form  [ALT] acceptable alternative
```

---

## SELinux — `semanage`

```bash
# Allow nonstandard HTTP port
semanage port -a -t http_port_t -p tcp 85   # [EASIER]
# [CHOOSE] type=http_port_t / ssh_port_t etc.  [CHOOSE] proto=tcp|udp  [CHOOSE] port=number

# Reference examples
man semanage /examples
```

---

## Podman → systemd integration

```bash
# Build image from remote repo
podman image build -t randomtag -f http://repo.podman.home/container   # [EASIER]

# Generate service and enable as user
podman generate systemd --new --files --name web.service
mkdir -p ~/.config/systemd/user
mv web.service ~/.config/systemd/user/
podman stop web.service
systemctl --user enable --now web.service
loginctl enable-linger   # [COMMON] keep user service running after logout
```

---

## Password & Login Policy

```bash
# /etc/security/pwquality.conf
minlen = 8   # [COMMON] minimal length

# /etc/login.defs (defaults for NEW users)
PASS_MAX_DAYS 999  # [CHOOSE] aging limit
```

---

## GRUB Configuration

```bash
# Edit defaults then rebuild config
/etc/default/grub

grub2-mkconfig -o /boot/grub2/grub.cfg   # [MUST]
```

---

## LVM + Filesystem

```bash
# Create LVM stack
pvcreate /dev/vdb
vgcreate -s 20M my_vg_name /dev/vdb
lvcreate -n my_lv_name -L 1G my_vg_name
mkfs.xfs /dev/my_vg_name/my_lv_name

# Mount persistently
/dev/my_vg_name/my_lv_name /data xfs defaults 0 0

# Extend logical volume + filesystem in one step
lvextend -r -L +500M /dev/vgdata/lvdata   # [EASIER]
```

---

## AutoFS

```bash
# /etc/auto.master
/mnt/netdir /etc/auto.home --timeout 30   # [COMMON]

# /etc/auto.home
bobby  -rw,sync  repo.home:/home/bobby   # [CHOOSE] user mapping
```

---

## Flatpak Basics

```bash
# 1. Enable Flathub repo
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# 2. List configured repos
flatpak remotes

# 3. Install app
flatpak install flathub org.gnome.Calculator -y

# 4. List installed apps
flatpak list

# 5. Run an app
flatpak run org.gnome.Calculator

# 6. Remove app
flatpak uninstall org.gnome.Calculator -y

# 7. Remove repo
flatpak remote-delete flathub
```

---

## Shell Scripting — Minimal Exam Scope

```bash
# 1. Run script
chmod +x script.sh
./script.sh

# 2. Variables & arguments
#!/bin/bash
echo "Script name: $0"
echo "First arg: $1"
echo "Second arg: $2"

# 3. Conditional (if)
#!/bin/bash
if [ -f /etc/passwd ]; then
  echo "File exists"
else
  echo "Missing"
fi

# 4. Loop (for)
#!/bin/bash
for user in user1 user2 user3; do
  echo "Checking $user"
done

# 5. Command substitution
DATE=$(date +%F)
echo "Today is $DATE"

# 6. Exit status
ls /etc/hosts >/dev/null 2>&1
if [ $? -eq 0 ]; then echo OK; else echo FAIL; fi
```

---

> **Result:** All commands organized for RHCSA 9→10 crossover prep, Podman retained for backward compatibility.
