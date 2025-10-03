# Compact RHCSA (EX200) Study Guide

---

### **1. Essential Tools & File Management**

* List files: `ls -lathr` # long, all, human-readable, time-sorted, reverse

* Current dir: `pwd` # print working directory

* Change dir: `cd [~, -, .., /]` # home, prev, parent, root

* User info: `id`, `whoami` # UID/GID, current user

* Sessions: `who`, `w`, `last`, `lastb` # login details, history

* System info: `uname -a`, `hostnamectl`, `timedatectl`, `lscpu` # kernel, host, time, CPU

* Search text: `grep -nviEw 'pattern' file` # line numbers, invert, ignore-case, word

* Find file: `find /path -name file.txt` # search by name

* Command info: `which`, `whatis`, `apropos` # binary path, description, search

* Kernel modules: `lsmod`, `modprobe modulename` # list or load module

* Hardware: `lsusb`, `lspci` # USB and PCI devices

* Create file: `touch file` # empty file

* Make dir: `mkdir -p /path/to/dir` # recursive dirs

* Copy: `cp [-r] source dest` # copy files or directories

* Move: `mv source dest` # move or rename

* Remove: `rm [-rf] file/dir` # force recursive delete

* Symlink: `ln -s target link_name` # soft link

* Hard link: `ln target link_name` # hard link

* Archive: `tar -cvzf archive.tar.gz /path` # create gzipped tarball

* Extract: `tar -xvzf archive.tar.gz` # extract gzipped tarball

* Redirect stdout: `>` # overwrite

* Append stdout: `>>` # append

* Redirect stderr: `2>` # stderr only

* Both: `&>` or `2>&1` # stdout + stderr

* Redirect stdin: `<` # feed input

* Pipe: `|` # send output to next cmd

* Change perms: `chmod 755 file` # rwx r-x r-x

* Symbolic perms: `chmod u+x file` # add exec to user

* Change owner: `chown user:group file` # set ownership

* Special perms: `chmod u+s file` # SUID, `g+s` SGID, `+t` sticky

* Docs: `man`, `info`, `/usr/share/doc` # documentation sources

* Vim basics: `i` insert, `:wq` save+quit, `:q!` quit no save

* Bash script: `#!/bin/bash` # shebang

* Args: `$1`, `$2` # positional

* Exit code: `$?` # last command status

* If: `if [ cond ]; then ... fi`

* Loop: `for i in ...; do ... done`

---

### **2. Operate Running Systems**

* Boot reset root: `rw init=/bin/bash`, `Ctrl+X`, `passwd`, `touch /.autorelabel`, `exec /sbin/init` # reset root password

* Systemctl basic: `systemctl start|stop|restart|status service`

* Enable/disable: `systemctl enable|disable service`

* Default target: `systemctl get-default`, `systemctl set-default multi-user.target`

* Switch target: `systemctl isolate rescue.target` # single user

* Process list: `ps aux`, `top` # show processes

* Kill: `kill PID`, `pkill name`, `killall name` # terminate process

* Force kill: `kill -9 PID`

* Priority: `nice -n 10 cmd`, `renice -n 5 PID`

* Logs: `journalctl`, `journalctl -u service`, `journalctl -f` # full, service, follow

* Persistent logs: `mkdir -p /var/log/journal`, `systemctl restart systemd-journald`

* Legacy logs: `/var/log/messages`, `/var/log/secure`, `/var/log/cron`

* Tuned profile: `tuned-adm active` # current profile

* Recommend: `tuned-adm recommend` # suggest best profile

* Switch: `tuned-adm profile balanced` # apply profile

---

### **3. Local Storage Management (LVM, VDO, Stratis)**

* Show disks: `lsblk`, `blkid` # block layout, UUIDs

* Partition: `fdisk /dev/sdx` # MBR, `gdisk /dev/sdx` # GPT

* Reload table: `partprobe` # refresh kernel

* Create LVM: `pvcreate /dev/sd[b-c]1`, `vgcreate my_vg /dev/sd[b-c]1`, `lvcreate -n my_lv -L 10G my_vg`

* Extend LVM: `lvextend -r -L +5G /dev/my_vg/my_lv` # grow LV + filesystem (ext4/XFS)

* Shrink LVM (ext4 only): `umount`, `e2fsck -f`, `resize2fs <smaller>`, `lvreduce -L <smaller>`, `e2fsck`, `mount`

* Show LVM: `pvs`, `vgs`, `lvs`, `lsblk -f`

* VDO create: `vdo create --name=my_vdo --device=/dev/sdx --vdoLogicalSize=100G` # thin dedupe/compress

* Stratis create: `stratis pool create my_pool /dev/sdx`, `stratis filesystem create my_pool my_fs` # layered storage

---

### **4. Filesystems**

* Make FS: `mkfs.xfs /dev/device`, `mkfs.ext4 /dev/device`, `mkswap /dev/device`

* Mount: `mount /dev/device /mnt/point`, `umount /mnt/point`

* Fstab: `UUID=xxxx /data xfs defaults 0 0` # persistent mount

* Swap: `swapon /dev/device`, `swapoff /dev/device`

* NFS client: `mount -t nfs server:/export /mnt` # manual mount

* AutoFS: edit `/etc/auto.master`, `/etc/auto.misc` # automount

---

### **5. System Deployment & Maintenance**

* Install pkg: `dnf install pkg` # install package

* Remove pkg: `dnf remove pkg` # uninstall

* Search pkg: `dnf search keyword` # find

* Info: `dnf info pkg` # details

* File provider: `dnf provides /path/file` # which pkg owns file

* Repos: `dnf repolist`

* Modules: `dnf module list`, `dnf module install name:stream`

* Rollback: `dnf history undo <id>` # revert transaction

* Cron edit: `crontab -e` # edit jobs

* Cron format: `min hour dom mon dow cmd` # schedule

* Cron example: `*/15 * * * * echo hi >> /tmp/log`

* Time check: `timedatectl` # show time

* Timezone: `timedatectl set-timezone Europe/Budapest`

* Enable NTP: `timedatectl set-ntp true`

* Service: `systemctl enable --now chronyd`

---

### **6. Networking**

* Devices: `nmcli device status` # show NICs

* Connections: `nmcli connection show` # list configs

* Add static: `nmcli connection add con-name static ifname enp1s0 type ethernet ip4 192.168.1.10/24 gw4 192.168.1.1`

* DNS: `nmcli con mod enp1s0 ipv4.dns 8.8.8.8 ipv4.method manual`

* Bring up: `nmcli con up enp1s0`

* Hostname: `hostnamectl set-hostname server1.example.com`

* Hosts file: `/etc/hosts` # local name mapping

* Firewalld zones: `firewall-cmd --get-active-zones`

* List rules: `firewall-cmd --list-all`

* Add service: `firewall-cmd --add-service=http --permanent`

* Add port: `firewall-cmd --add-port=8080/tcp --permanent`

* Assign iface: `firewall-cmd --change-interface=eth0 --zone=public --permanent`

* Rich rule: `firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="http" accept' --permanent`

* Reload: `firewall-cmd --reload`

---

### **7. Users & Groups**

* Add user: `useradd bob`

* Add sudo: `usermod -aG wheel bob`

* Remove user: `userdel -r bob`

* Set password: `passwd bob`

* Password aging: `chage -l bob` # list settings

* Add group: `groupadd developers`

* Remove group: `groupdel developers`

* Add to group: `gpasswd -a user group`

* Remove from group: `gpasswd -d user group`

* Edit sudoers: `visudo` # safe edit

---

### **8. Security**

* SELinux status: `getenforce`, `sestatus`

* Enforce mode: `setenforce 1|0` # on/off

* Config file: `/etc/selinux/config`

* Contexts: `ls -Z`, `restorecon -Rv /path` # list, restore

* Booleans: `getsebool -a`, `setsebool -P boolean on` # check/set

* Custom dir: `semanage fcontext -a -t httpd_sys_content_t "/srv/web(/.*)?"`, `restorecon -Rv /srv/web`

* Audit logs: `ausearch -m avc -ts recent`, `sealert -a /var/log/audit/audit.log`

* SSH keys: `ssh-keygen`, `ssh-copy-id user@server`

* ACLs: `setfacl -m u:user:rw- file`, `getfacl file`, `setfacl -x u:user file`

---

### **9. Containers (Podman)**

* Search image: `podman search ubi9`

* Pull image: `podman pull ubi9/ubi-minimal`

* List images: `podman images`

* Inspect: `podman inspect image`

* Remove image: `podman rmi image`

* Run container: `podman run -d --name myapp -p 8080:80 image`

* List containers: `podman ps -a`

* Start/stop: `podman start|stop container`

* Remove: `podman rm container`

* Exec shell: `podman exec -it container bash` # open shell inside

* Volumes: `podman run -v /host/path:/cont/path:Z image`

* Systemd service: `podman generate systemd --name myapp > /etc/systemd/system/container-myapp.service`, then `systemctl daemon-reload && systemctl enable --now container-myapp.service`
