    ls -lh /etc/ssh
    systemctl stop ssh
    sudo ssh-keygen -A
    systemctl start ssh

    ip a
    sudo passwd ubuntu-server

    tmux

    tmux a

    sudo -iexport LANG=C
    apt update -y && apt full-upgrade -y

    cat /proc/partitions

## ZFS EFI boottal
    sgdisk -Z -n9:-8M:0 -t9:bf07 -c9:sda9-Reserved -n1:1M:+512 -t1:EF00 -c1:sda1-EFI -n2:0:0 -t2:bf01 -c2:sda2-zfs /dev/sda
    sgdisk -Z -n9:-8M:0 -t9:bf07 -c9:sdb9-Reserved -n1:1M:+512 -t1:EF00 -c1:sdb1-EFI -n2:0:0 -t2:bf01 -c2:sdb2-zfs /dev/sdb

## ZFS /boot boottal
    sgdisk -Z -n9:-8M:0 -t9:bf07 -c9:sda9-Reserved -n1:1M:+1023M -t1:8300 -c1:sda1-BOOT -n2:0:0 -t2:bf01 -c2:sda2-zfs /dev/sda
    sgdisk -Z -n9:-8M:0 -t9:bf07 -c9:sdb9-Reserved -n1:1M:+1023M -t1:8300 -c1:sdb1-BOOT -n2:0:0 -t2:bf01 -c2:sdb2-zfs /dev/sdb

apt install -y zfsutils-linux

mkdrir -p /target

## ZFS egy lemezzel
    zpool create -o ashift=12 -O atime=off -O canmount=off -O compression=lz4 -O normalization=formD -O mountpoint=/ -R /target systempool /dev/sda2

## ZFS mirror
    zpool create -o ashift=12 -O atime=off -O canmount=off -O compression=lz4 -O normalization=formD -O mountpoint=/ -R /target mirror /dev/sda2 /dev/sdb2

    zfs create -o canmount=off -o mountpoint=none systempool/ROOT
    zfs create -o canmount=noauto -o mountpoint=/ -o exec=on -o setuid=on -o devices=on systempool/ROOT/mint
    zfs mount systempool/ROOT/mint
    zfs create -o canmount=noauto -o mountpoint=/boot systempool/boot
    zfs mount systempool/boot

    df -h

    zpool set bootfs=systempool/ROOT/mint systempool

    zfs set exec=off systempool
    zfs set setuid=off systempool
    zfs set devices=off systempool

    zfs create -o canmount=off systempool/var
    zfs create systempool/var/lib
    zfs create systempool/var/lib/apt
    zfs create -o exec=on systempool/var/lib/dpkg
    zfs create systempool/var/log
    zfs create -o com.sun:auto-snapshot=false systempool/var/tmp
    zfs create -o com.sun:auto-snapshot=false systempool/var/cache
    zfs create -o com.sun:auto-snapshot=false systempool/var/cache/apt
    zfs create systempool/var/spool
    zfs create systempool/var/mail
    zfs create -o com.sun:auto-snapshot=false -o exec=on systempool/tmp
    zfs create -o exec=on systempool/root
    zfs create -o mountpoint=/home systempool/home
    zfs create -o mountpoint=/srv systempool/srv
    zfs create -o mountpoint=/opt systempool/opt
    zfs create -o exec=on systempool/usr

    chmod 1777 /target/tmp
    chmod 1777 /target/var/tmp

    apt install -y debootstrap

    debootstrap --include=zfsutils-linux tara /target

    mount --rbind /dev /target/dev
    mount --rbind /proc /target/proc
    mount --rbind /sys /target/sys
    chroot /target /bin/bash

(csomágtároló config, ha kell)
(hálózat konfig, ha kell)

    apt install language-pack-hu

    dpkg-reconfigure locales
    dpkg-reconfigure debconf
    dpkg-reconfigure tzdata

    apt install -y gdisk dosfstools zfs-initramfs

    echo PARTUUID=$(blkid -s PARTUUID -o value /dev/disk/by-partlabel/sda1-EFI /boot/efi vfat d 1 >> /etc/fstab)

