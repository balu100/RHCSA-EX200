# Compact RHCSA (EX200) Study Guide

---

### **1. Essential Tools & File Management**

#### Navigation & Inspection

* `ls -lathr` # list files long, all, human-readable, time-sorted, reverse
* `pwd` # print working directory
* `cd [~, -, .., /]` # home, previous, parent, root
* `id` # show UID/GID info
* `whoami` # show current user
* `who`, `w` # logged in users
* `last`, `lastb` # login and failed attempts
* `uname -a` # kernel + OS details
* `hostnamectl` # system hostname info
* `timedatectl` # system time info
* `lscpu` # CPU details
* `grep -nviEw 'pattern' file` # search text with options
* `find /path -name file.txt` # search file by name
* `which cmd` # path to command
* `whatis cmd` # short description
* `apropos keyword` # search manual pages
* `lsmod` # list kernel modules
* `modprobe modulename` # load module
* `lsusb` # list USB devices
* `lspci` # list PCI devices

#### File & Directory Manipulation

* `touch file` # create empty file
* `mkdir -p /path/to/dir` # create dirs recursively
* `cp -r src dest` # copy recursively
* `mv src dest` # move or rename
* `rm -rf file/dir` # force remove

#### Links

* `ln -s target link` # soft link
* `ln target link` # hard link

#### Archiving & Compression

* `tar -cvzf file.tar.gz /path` # create gzipped archive
* `tar -xvzf file.tar.gz` # extract gzipped archive

#### Redirection & Piping

* `>` # redirect stdout overwrite
* `>>` # redirect stdout append
* `2>` # redirect stderr
* `&>` or `2>&1` # redirect both stdout & stderr
* `<` # redirect stdin
* `|` # pipe

#### Permissions

* `chmod 755 file` # rwx r-x r-x
* `chmod u+x file` # add exec for user
* `chown user:group file` # change ownership
* `chmod u+s file` # SUID
* `chmod g+s file` # SGID
* `chmod +t dir` # sticky bit

#### Documentation

* `man cmd` # manual
* `info cmd` # info page
* `/usr/share/doc` # docs directory

#### Vim

* `i` # insert
* `:wq` # save & quit
* `:q!` # quit no save

#### Simple Shell Scripting

* `#!/bin/bash` # shebang
* `$1`, `$2` # positional parameters
* `$?` # exit code
* `if [ cond ]; then ... fi` # condition
* `for i in list; do ... done` # loop

---

### **2. Operate Running Systems**

#### Boot Process

* `rw init=/bin/bash`, `Ctrl+X`, `passwd`, `touch /.autorelabel`, `exec /sbin/init` # reset root password

#### Systemd Management

* `systemctl start|stop|restart|status service` # control services
* `systemctl enable|disable service` # enable/disable
* `systemctl get-default` # show default target
* `systemctl set-default multi-user.target` # set default target
* `systemctl isolate rescue.target` # switch to rescue

#### Process Management

* `ps aux` # process list
* `top` # interactive monitor
* `kill PID` # kill by PID
* `pkill name` # kill by name
* `killall name` # kill all matching
* `kill -9 PID` # force kill
* `nice -n 10 cmd` # set priority
* `renice -n 5 PID` # change priority

#### Logging

* `journalctl` # view logs
* `journalctl -u service` # service logs
* `journalctl -f` # follow logs
* `mkdir -p /var/log/journal` # enable persistent
* `systemctl restart systemd-journald` # restart journald
* Legacy: `/var/log/messages`, `/var/log/secure`, `/var/log/cron`

#### Tuning Profiles

* `tuned-adm active` # current profile
* `tuned-adm recommend` # suggest profile
* `tuned-adm profile balanced` # switch profile

---

### **3. Local Storage Management**

#### Partitioning

* `lsblk` # block layout
* `blkid` # UUIDs
* `fdisk /dev/sdx` # MBR
* `gdisk /dev/sdx` # GPT
* `partprobe` # reload partition table

#### LVM
* `pvcreate /dev/sdb1 /dev/sdc1` # init PV(s)  
* `vgcreate my_vg /dev/sdb1 /dev/sdc1` # create VG from PV(s)  
* `lvcreate -n my_lv -L 10G my_vg` # new LV  
* `lvextend -r -L +5G /dev/my_vg/my_lv` # extend LV + FS (ext4/XFS)  
* Shrink ext4: `umount`, `e2fsck -f`, `resize2fs <smaller>`, `lvreduce -L <smaller>`, `e2fsck`, `mount`  
* Show: `pvs` # PVs, `vgs` # VGs, `lvs` # LVs, `lsblk -f` # all block/FS  


#### VDO

* `vdo create --name=my_vdo --device=/dev/sdx --vdoLogicalSize=100G` # create VDO

#### Stratis

* `stratis pool create my_pool /dev/sdx` # create pool
* `stratis filesystem create my_pool my_fs` # create FS

---

### **4. Filesystems**

#### Creation

* `mkfs.xfs /dev/device` # create XFS
* `mkfs.ext4 /dev/device` # create ext4
* `mkswap /dev/device` # create swap

#### Mounting

* `mount /dev/device /mnt/point` # mount device
* `umount /mnt/point` # unmount
* `/etc/fstab`: `UUID=xxxx /data xfs defaults 0 0` # persistent mount
* `swapon /dev/device` # enable swap
* `swapoff /dev/device` # disable swap

#### NFS Client

* `mount -t nfs server:/export /mnt` # mount NFS

#### AutoFS

* Edit `/etc/auto.master`, `/etc/auto.misc` # configure automount

---

### **5. System Deployment & Maintenance**

#### DNF

* `dnf install pkg` # install
* `dnf remove pkg` # uninstall
* `dnf search keyword` # search
* `dnf info pkg` # info
* `dnf provides /path/file` # find package
* `dnf repolist` # list repos
* `dnf module list` # list modules
* `dnf module install name:stream` # install module
* `dnf history undo <id>` # rollback

#### Scheduling

* `crontab -e` # edit cron
* `*/15 * * * * echo hi >> /tmp/log` # run every 15 min

#### Time Service

* `timedatectl` # show time
* `timedatectl set-timezone Europe/Budapest` # set timezone
* `timedatectl set-ntp true` # enable NTP
* `systemctl enable --now chronyd` # enable chrony

---

### **6. Networking**

#### nmcli

* `nmcli device status` # NICs
* `nmcli connection show` # list connections
* `nmcli connection add con-name static ifname enp1s0 type ethernet ip4 192.168.1.10/24 gw4 192.168.1.1` # add static
* `nmcli con mod enp1s0 ipv4.dns 8.8.8.8 ipv4.method manual` # add DNS
* `nmcli con up enp1s0` # bring up

#### Hostname

* `hostnamectl set-hostname server1.example.com` # set hostname
* `/etc/hosts` # local mapping

#### firewalld

* `firewall-cmd --get-active-zones` # active zones
* `firewall-cmd --list-all` # list rules
* `firewall-cmd --add-service=http --permanent` # add http
* `firewall-cmd --add-port=8080/tcp --permanent` # add port
* `firewall-cmd --change-interface=eth0 --zone=public --permanent` # move iface
* `firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="http" accept' --permanent` # rich rule
* `firewall-cmd --reload` # reload rules

---

### **7. Users & Groups**

#### Users

* `useradd bob` # add user
* `usermod -aG wheel bob` # add to sudo
* `userdel -r bob` # remove user
* `passwd bob` # set password
* `chage -l bob` # show password aging

#### Groups

* `groupadd developers` # add group
* `groupdel developers` # delete group
* `gpasswd -a user group` # add user to group
* `gpasswd -d user group` # remove user from group

#### Sudo

* `visudo` # edit sudoers

---

### **8. Security**

#### SELinux

* `getenforce` # current mode
* `sestatus` # status
* `setenforce 1|0` # set enforce
* `/etc/selinux/config` # config file
* `ls -Z` # list contexts
* `restorecon -Rv /path` # restore context
* `getsebool -a` # list booleans
* `setsebool -P boolean on` # set boolean
* `semanage fcontext -a -t httpd_sys_content_t "/srv/web(/.*)?"` # label dir
* `restorecon -Rv /srv/web` # apply context
* `ausearch -m avc -ts recent` # search AVC
* `sealert -a /var/log/audit/audit.log` # analyze audit

#### SSH Keys

* `ssh-keygen` # generate keys
* `ssh-copy-id user@server` # copy keys

#### ACLs

* `setfacl -m u:user:rw- file` # set user acl
* `getfacl file` # view ACL
* `setfacl -x u:user file` # remove user ACL

---

### **9. Containers (Podman)**

#### Image Management

* `podman search ubi9` # search image
* `podman pull ubi9/ubi-minimal` # pull image
* `podman images` # list images
* `podman inspect image` # inspect image
* `podman rmi image` # remove image

#### Container Lifecycle

* `podman run -d --name myapp -p 8080:80 image` # run container
* `podman ps -a` # list containers
* `podman stop container` # stop
* `podman start container` # start
* `podman rm container` # remove
* `podman exec -it container bash` # shell inside

#### Persistent Storage

* `podman run -v /host/path:/container/path:Z image` # bind mount volume

#### systemd Service

* `podman generate systemd --name myapp > /etc/systemd/system/container-myapp.service` # gen unit
* `systemctl daemon-reload` # reload units
* `systemctl enable --now container-myapp.service` # enable service
