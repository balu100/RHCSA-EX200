## SELinux and semanage

```
semanage port -a -t http_port_t -p tcp 85
man semanage /examples
```

---

## Podman and Systemd Integration

```
podman image build -t randomtag -f http://repo.podman.home/container

podman generate systemd --new --files --name web.service
mkdir -p ~/.config/systemd/user
mv web.service ~/.config/systemd/user/
podman stop web.service
systemctl --user enable --now web.service
loginctl enable-linger
```

---

## Password and Security Settings

```
/etc/security/pwquality.conf
minlen = 8

cat /etc/login.defs
PASS_MAX_DAYS 999
```

---

## GRUB Configuration

```
/etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## LVM and Storage Management

```
pvcreate /dev/vdb
vgcreate -s 20M my_vg_name /dev/vdb
lvcreate -n my_lv_name -L 1G my_vg_name
mkfs.xfs /dev/my_vg_name/my_lv_name

/dev/my_vg_name/my_lv_name /data xfs defaults 0 0

lvextend -r -L +500M /dev/vgdata/lvdata
```

---

## AutoFS Configuration

```
/etc/auto.master
/mnt/netdir /etc/auto.home --timeout 30

/etc/auto.home
bobby  -rw,sync  repo.home:/home/bobby
```

---

## Flatpak Basics

```
# 1. Enable Flathub repo (only once)
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# 2. List configured repos
flatpak remotes

# 3. Install an app (example: Calculator)
flatpak install flathub org.gnome.Calculator -y

# 4. List installed apps
flatpak list

# 5. Run an app
flatpak run org.gnome.Calculator

# 6. Remove an app
flatpak uninstall org.gnome.Calculator -y

# 7. Remove a repo
flatpak remote-delete flathub
```

---

## Shell Scripting Basics

```
1. Run a script
chmod +x script.sh
./script.sh

2. Variables and arguments
#!/bin/bash
echo "Script name: $0"
echo "First arg: $1"
echo "Second arg: $2"

3. Conditional (if)
#!/bin/bash
if [ -f /etc/passwd ]; then
  echo "File exists"
else
  echo "Missing"
fi

4. Loop (for)
#!/bin/bash
for user in user1 user2 user3; do
  echo "Checking $user"
done

5. Command substitution
DATE=$(date +%F)
echo "Today is $DATE"

6. Exit status
ls /etc/hosts >/dev/null 2>&1
if [ $? -eq 0 ]; then echo OK; else echo FAIL; fi
```
