# Raidvm
В прикреплении прилагаю файл в котором автоматически добавляются диски, далее они примонтируются к ВМ виртуалбокс.
Также в vagrant файле скрипт устанавливает компоненты для работы с Raid и обновляет машину
box.vm.provision "shell", inline: <<-SHELL
               mkdir -p ~root/.ssh
               cp ~vagrant/.ssh/auth* ~root/.ssh
               apt-get update
               sudo ex +"%s@DPkg@//DPkg" -cwq /etc/apt/apt.conf.d/70debconf
               sudo dpkg-reconfigure debconf -f noninteractive -p critical
               apt-get install -y mdadm smartmontools hdparm gdisk
          SHELL

Вывод lsblk При создании:
root@testvm:/home/Raidvm# vagrant ssh
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-92-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Fri Mar 22 10:33:52 AM UTC 2024

  System load:  0.0                Processes:             140
  Usage of /:   12.4% of 30.34GB   Users logged in:       0
  Memory usage: 25%                IPv4 address for eth0: 10.0.2.15
  Swap usage:   0%                 IPv4 address for eth1: 192.168.56.101


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
vagrant@vm:~$ sudo su
root@vm:/home/vagrant# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  53.3M  1 loop /snap/snapd/19457
loop1                       7:1    0 111.9M  1 loop /snap/lxd/24322
loop2                       7:2    0  63.4M  1 loop /snap/core20/1974
sda                         8:0    0    64G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0    62G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0    31G  0 lvm  /
sdb                         8:16   0   250M  0 disk
sdc                         8:32   0   250M  0 disk
sdd                         8:48   0   250M  0 disk
sde                         8:64   0   250M  0 disk


Далее действия продолжает Ansible
box.vm.provision "ansible" do |ansible|
На примонтированных дисков создается разметка и далее создается Raid1. После этого когда Raid1 создан происходит его тестирование.
Форматируем его и монтируем в каталог /mnt
Вывод после отработки Ansible:

root@testvm:/home/Raidvm# vagrant ssh
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-92-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Fri Mar 22 10:37:24 AM UTC 2024

  System load:  0.51416015625      Processes:             175
  Usage of /:   12.4% of 30.34GB   Users logged in:       0
  Memory usage: 28%                IPv4 address for eth0: 10.0.2.15
  Swap usage:   0%                 IPv4 address for eth1: 192.168.56.101


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Fri Mar 22 10:36:48 2024 from 10.0.2.2
vagrant@vm:~$ sudo su
root@vm:/home/vagrant# lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
loop0                       7:0    0  63.4M  1 loop  /snap/core20/1974
loop1                       7:1    0  53.3M  1 loop  /snap/snapd/19457
loop2                       7:2    0 111.9M  1 loop  /snap/lxd/24322
sda                         8:0    0    64G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part  /boot
└─sda3                      8:3    0    62G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0    31G  0 lvm   /
sdb                         8:16   0   250M  0 disk
└─sdb1                      8:17   0    12M  0 part
  └─md0                     9:0    0  11.9M  0 raid1 /mnt
sdc                         8:32   0   250M  0 disk
└─sdc1                      8:33   0    12M  0 part
  └─md0                     9:0    0  11.9M  0 raid1 /mnt
sdd                         8:48   0   250M  0 disk
sde                         8:64   0   250M  0 disk
