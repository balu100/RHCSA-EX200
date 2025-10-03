# Complete Compact RHCSA (EX200) Study Guide

---

### **1. Essential Tools & File Management**

* **Navigation & Inspection:**

  * `ls -lathr`: List files (long, all, human-readable, by time, reversed).
  * `pwd`, `cd [~, -, .., /]`: Navigate directories.
  * `id`, `whoami`, `who`, `w`, `last`, `lastb`: User/login info.
  * `uname -a`, `hostnamectl`, `timedatectl`, `lscpu`: System info.
  * `grep [-nviEw] 'pattern' file`: Search text.
  * `find /path -name "file.txt"`: Find files by name.
  * `which`, `whatis`, `apropos`: Find command info.
  * `lsmod`, `modprobe modulename`: Kernel modules.
  * `lsusb`, `lspci`: Inspect hardware devices.

* **File/Directory Manipulation:**

  * `touch file`: Create empty file.
  * `mkdir -p /path/to/dir`: Create directories recursively.
  * `cp [-r] source dest`: Copy files/directories.
  * `mv source dest`: Move/rename.
  * `rm [-rf] file/dir`: Remove files/directories (forceful, recursive).

* **Links:**

  * `ln -s target link_name`: Soft link (symbolic).
  * `ln target link_name`: Hard link.

* **Archiving & Compression:**

  * `tar -cvzf archive.tar.gz /path`: **C**reate **v**erbose **z**ipped **f**ile.
  * `tar -xvzf archive.tar.gz`: **X**tract **v**erbose **z**ipped **f**ile.
  * (`-j` for `bzip2`, `-x` to extract, `-c` to create).

* **Redirection & Piping:**

  * `>`: Redirect stdout (overwrite).
  * `>>`: Redirect stdout (append).
  * `2>`: Redirect stderr.
  * `&>` or `2>&1`: Redirect stdout & stderr.
  * `<`: Redirect stdin.
  * `|`: Pipe output of one command to another.

* **Permissions (ugo/rwx):**

  * `chmod [ugoa][+-=][rwx] file`: Symbolic permissions.
  * `chmod 755 file`: Octal (Owner:rwx, Group:r-x, Other:r-x).
  * `chown user:group file`: Change owner/group (`-R` for recursive).
  * **Special:** `chmod u+s` (SUID), `g+s` (SGID), `+t` (Sticky Bit).

* **Documentation:** `man`, `info`, `/usr/share/doc`.

* **Text Editor (Vim):**

  * Proficient use of `vim` (insert, edit, save, quit).

* **Simple Shell Scripting:**

  * `#!/bin/bash`: Shebang indicates script interpreter.
  * `$1`, `$2`: Positional parameters.
  * `$?`: Exit code of last command.
  * `if [ ... ]; then ... fi`: Conditional logic.
  * `for i in ...; do ... done`: Loop.

---

### **2. Operate Running Systems**

* **Boot Process:**

  * Interrupt GRUB (`e`).
  * Reset root password: `rw init=/bin/bash`, `Ctrl+X`, `passwd`, `touch /.autorelabel`, `exec /sbin/init`.

* **Systemd Management:**

  * `systemctl [start|stop|restart|status] service`
  * `systemctl [enable|disable|mask] service`
  * `systemctl get-default`, `set-default graphical.target`
  * `systemctl isolate rescue.target`: Switch runlevel.

* **Process Management:**

  * `ps aux`, `top`
  * `kill PID`, `pkill name`, `killall name`
  * `kill -9 PID`: Force kill.
  * `nice -n 10 cmd`, `renice -n 5 PID`

* **Logging:**

  * `journalctl`, `journalctl -u service`, `journalctl -f`
  * Persistent: `mkdir -p /var/log/journal`, `systemctl restart systemd-journald`
  * Legacy logs: `/var/log/messages`, `/var/log/secure`, `/var/log/cron`

* **Tuning Profiles:**

  * `tuned-adm active`
  * `tuned-adm recommend`
  * `tuned-adm profile <profile>`

---

### **3. Local Storage Management (LVM, VDO, Stratis)**

* **Partitioning:**

  * `lsblk`, `blkid`
  * `fdisk /dev/sdx` (MBR), `gdisk /dev/sdx` (GPT)
  * `partprobe`: Reload partition table

* **LVM:**
  * Create: `pvcreate /dev/sd[b-c]1`, `vgcreate my_vg /dev/sd[b-c]1`, `lvcreate -n my_lv -L 10G my_vg`
  * Extend: `lvextend -r -L +5G /dev/my_vg/my_lv`   # grows LV + FS (ext4/XFS)
  * Reduce (⚠ ext4 only): `umount`, `e2fsck -f`, `resize2fs <smaller_size>`, `lvreduce -L <smaller_size>`, `e2fsck`, `mount`
  * Display: `pvs`, `vgs`, `lvs`, `lsblk -f`

* **VDO:**

  * `dnf install vdo kmod-kvdo`
  * `vdo create --name=my_vdo --device=/dev/sdx --vdoLogicalSize=100G`
  * Format and mount `/dev/mapper/my_vdo`

* **Stratis:**

  * `dnf install stratisd stratis-cli`
  * `systemctl enable --now stratisd`
  * `stratis pool create my_pool /dev/sdx`
  * `stratis filesystem create my_pool my_fs`
  * Mount: `/stratis/my_pool/my_fs`

---

### **4. Filesystems**

* **Creation:**

  * `mkfs.xfs /dev/device`
  * `mkfs.ext4 /dev/device`
  * `mkswap /dev/device`

* **Mounting:**

  * `mount /dev/device /mnt/point`
  * `umount /mnt/point`
  * `/etc/fstab` example:

    ```
    UUID=xxxx-xxxx  /data  xfs  defaults,noexec,nosuid,nodev  0 0
    ```
  * `swapon /dev/device`, `swapoff /dev/device`

* **NFS Client:**

  * `dnf install nfs-utils`
  * `mount -t nfs server:/export /mnt`
  * Add to `/etc/fstab`

* **AutoFS:**

  * `dnf install autofs`
  * `/etc/auto.master`: `/- /etc/auto.misc`
  * `/etc/auto.misc`: `localdir -fstype=nfs server:/export`
  * `systemctl enable --now autofs`

---

### **5. System Deployment & Maintenance**

* **DNF:**

  * `dnf install package`
  * `dnf remove package`
  * `dnf search keyword`
  * `dnf info package`
  * `dnf provides /path/file`
  * `dnf repolist`
  * `dnf module list`, `dnf module install name:stream`
  * `dnf history undo <id>`: Roll back transaction

* **Scheduling:**

  * `crontab -e`
  * Format: `min hour dom mon dow command`
  * Example: `*/15 * * * * echo "Hi" >> /tmp/log`

* **Time Service:**

  * `timedatectl`
  * `timedatectl list-timezones`, `set-timezone Europe/Budapest`
  * `timedatectl set-ntp true`
  * Service: `systemctl enable --now chronyd`

---

### **6. Networking**

* **nmcli:**

  * `nmcli device status`
  * `nmcli connection show`
  * `nmcli connection add con-name static ifname enp1s0 type ethernet ip4 192.168.1.10/24 gw4 192.168.1.1`
  * `nmcli con mod enp1s0 ipv4.dns 8.8.8.8 ipv4.method manual`
  * `nmcli con up enp1s0`

* **Hostname:**

  * `hostnamectl set-hostname server1.example.com`
  * `/etc/hosts`: Local mapping

* **firewalld:**

  * `firewall-cmd --get-active-zones`
  * `firewall-cmd --list-all`
  * `firewall-cmd --add-service=http --permanent`
  * `firewall-cmd --add-port=8080/tcp --permanent`
  * `firewall-cmd --change-interface=eth0 --zone=public --permanent`
  * Rich rule example:

    ```
    firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="http" accept' --permanent
    ```
  * `firewall-cmd --reload`

---

### **7. Users & Groups**

* **Users:**

  * `useradd bob`
  * `usermod -aG wheel bob`
  * `userdel -r bob`
  * `passwd bob`
  * `chage -l bob`

* **Groups:**

  * `groupadd developers`
  * `groupdel developers`
  * `gpasswd -a user group`
  * `gpasswd -d user group`

* **Sudo:** `visudo` to edit `/etc/sudoers`

---

### **8. Security**

* **SELinux:**

  * `getenforce`, `sestatus`
  * `setenforce 0|1`
  * Edit `/etc/selinux/config`
  * `ls -Z`, `restorecon -Rv /path`
  * `getsebool -a`, `setsebool -P boolean on`
  * **Non-standard directories:**

    ```
    semanage fcontext -a -t httpd_sys_content_t "/srv/website(/.*)?"
    restorecon -Rv /srv/website
    ```
  * Troubleshoot:

    * `ausearch -m avc -ts recent`
    * `ausearch -c processname`
    * `sealert -a /var/log/audit/audit.log`

* **SSH Key Authentication:**

  1. `ssh-keygen`
  2. `ssh-copy-id user@server`

* **ACLs:**

  * `setfacl -m u:user:rw- file`
  * `setfacl -m g:group:r-x dir`
  * `getfacl file`
  * `setfacl -x u:user file`

---

### **9. Containers (Podman)**

* **Image Management:**

  * `podman search ubi9`
  * `podman pull ubi9/ubi-minimal`
  * `podman images`
  * `podman inspect image`
  * `podman rmi image`

* **Container Lifecycle:**

  * `podman run -d --name myapp -p 8080:80 image`
  * `podman ps [-a]`
  * `podman stop container`, `podman start container`
  * `podman rm container`
  * `podman exec -it container bash`: Interactive shell inside

* **Persistent Storage:**

  * `podman run -v /host/path:/container/path:Z image`

* **systemd Service:**

  1. `podman generate systemd --name myapp > /etc/systemd/system/container-myapp.service`
  2. `systemctl daemon-reload`
  3. `systemctl enable --now container-myapp.service`
