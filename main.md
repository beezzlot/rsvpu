https://disk.yandex.ru/i/t62YChVMT22JjQ

# Alt Linux Network

cd ens18

sed -i -E 's/dhcp|dhcp4/static/' options

cat options

BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no

cat ipv4address
10.0.10.10/24

cat ipv4route
default via 10.0.10.10



systemclt restart network


# Alt Linux OVS

ovs-vsctl add-br SW-HQ
ovs-vsctl add-port SW-HQ ens18 trunks=100,200,300 vlan_mode=trunk
ovs-vsctl add-port SW-HQ ens19 tag=200
ovs-vsctl add-port SW-HQ vlan300 -- set interface vlan300 type=internal
ovs-vsctl set port vlan300 tag=300

mkdir vlan300

cat options

```
BOOTPROTO=static
TYPE=ovsport
BRIDGE=SW-HQ
VID=300
ONBOOT=yes
```

А для остальных подинтерфейсов на копируй с ens18

# LVM

Зеркальный

pvcreate /dev/sd{b,c}. 

vgcreate mirror-vg /dev/sd{b,c}.

lvcreate -l +100%FREE -m1 -n lv-mirror1 mirror-vg

mkfs -t ext4 /dev/mirror-vg/lv-mirror1.


Шифрованный

dd if=/dev/urandom of=/etc/secretkey bs=512 count=4

cryptsetup luksFormat /dev/sdb /etc/secretkey

crypsetup luksOpen /dev/sdb cryptlvm1 --key-file /etc/secretkey
crypsetup luksOpen /dev/sdb cryptlvm2 --key-file /etc/secretkey


pvcreate /dev/mapper/cryptlvm{1,2}

vgcreate vg_crypt /dev/mapper/cryptlvm{1,2}

lvcreate -i 2 -I 4 -L +100%FREE -n vl_crypt vg_crypt

 
/etc/crypttab

cryptlvm1 /dev/sdb /etc/secretkey luks
cryptlvm2 /dev/sdc /etc/secretkey luks

/etc/fstab - как обычно

# NAT Eltex
```
    vesr(config)# object-group network LOCAL1
    vesr(config-object-group-network)# ip prefix 10.10.2.0/24 - внутрення подсеть, которую планируешь натить
    vesr(config)# object-group network WAN1
    vesr(config-object-group-network)# ip address-range 192.168.122.100 - внешний адрес, в который планируешь натить
    vesr(config)# security zone UNTRUST
    vesr(config)# security zone TRUST
    vesr(config)# int g 1/0/1 - интерфейс в локальную сеть
    vesr(config-if)# security-zone TRUST 
    vesr(config)# int g 1/0/2 - интерфейс во внешнюю сеть
    vesr(config-if)# security-zone UNTRUST 
    vesr(config)# security zone-pair TRUST UNTRUST
    vesr(config-sec-zone-pair)# rule 1
    vesr(config-sec-zone-pair-rule)# action permit
    vesr(config-sec-zone-pair-rule)# match source-address LOCAL1
    vesr(config-sec-zone-pair-rule)# enable
    vesr(config)# nat source
    vesr(config-nat-source)# pool WAN1
    vesr(config-nat-source-pool)# ip address-range 192.168.122.100 - внешний адрес, в который планируешь натить
    vesr(config)# ruleset SNAT1
    vesr(config-ruleset)# to zone UNTRUST
    vesr(config-ruleset) rule 1
    vesr(config-ruleset-rule) match source-address LOCAL1
    vesr(config-ruleset-rule) action source-nat pool WAN1
    vesr(config-ruleset-rule) enable
    
    vesr# show ip nat translations
```
```
config t
ip dhcp-server
ip dhcp-server pool CLI
(config-dhcp) network 10.0.10.32/27
(config-dhcp) domain-name company.prof
(config-dhcp) address-range 10.0.10.45-10.0.10.50
(config-dhcp) default-router 10.0.10.33
(dns-server) 10.0.10.10,10.0.20.10
(dhcp-server) end
   
```

```
    vesr# show ip dhcp server pool 
    vesr# show ip dhcp server pool LOCAL
```

```
    vesr(config)# tunnel gre 1
    vesr(config-gre)# ttl 64
    vesr(config-gre)# ip ospf instance 1
    vesr(config-gre)# ip ospf area 0.0.0.1
    vesr(config-gre)# ip ospf 
    vesr(config-gre)# enable    
    ```

```
    vesr(config)# router ospf 1
    vesr(config-ospf)# router-id 10.5.5.1 - адрес твоего туннельного интерфейса
    vesr(config-ospf)# area 0.0.0.1
    vesr(config-ospf-area)# network 10.5.5.0/30 
    vesr(config-ospf-area)# enable
    vesr(config-ospf-area)# exit
    vesr(config-ospf)# enable
    ```

 На маршрутизаторах RTR-HQ и RTR-BR создавать пользователей не нужно. Главное - включить на них SSH сервер командой ip ssh server.
Распакуем архив командой tar xzvf esr-ansible-2021-06-11.tar.gz, перейдем в директорию esr-ansible командой cd esr-ansible/  и запустим скрипт установки командой python3 install.py.

Ansible:

![21](https://github.com/beezzlot/rsvpu/assets/57652313/1d90d481-8253-4cc2-bdfa-10aa358b350a)

 
FreeIPA


Установим пакет freeipa-server при помощи команды apt-get install freeipa-server -y. После этого отключаем службу ahttpd, чтобы избежать конфликта между alterator-fbi и freeipa, командой systemctl stop ahttpd.service. 
Создадим доменный контроллер на SRV-HQ при помощи команды ipa-server-install.


На CLI-HQ установим пакет freeipa-client при помощи команды apt-get update && apt-get install freeipa-client -y.
 Запустим скрипт интерактивного добавления клиента при помощи команды ipa-client-install --enable-dns-update.



