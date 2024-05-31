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

       vesr(config)#  ip dhcp-server - включили DHCP
       vesr(config)# ip dhcp-server  pool LOCAL - создали пул
       vesr(config-dhcp-server)#  network 10.10.2.0/24
       vesr(config-dhcp-server)#  address-range 10.10.2.2-10.10.2.10
       vesr(config-dhcp-server)# domain-name digital.skills
       vesr(config-dhcp-server)# default-router 10.10.2.1
       vesr(config-dhcp-server)# dns-server 10.113.38.100
       vesr(config)#  object-group service dhcp_server
       vesr(config-object-group)# port-range 67
       vesr(config)#  object-group service dhcp_client
       vesr(config-object-group)# port-range 68
       vesr(config)# security zone-pair TRUST self
       vesr(config-zone-pair)# rule 30
       vesr(config-zone-pair)# match protocol udp
       vesr(config-zone-pair)# match source-port dhcp_client
       vesr(config-zone-pair)# match destination-port dhcp_server
       vesr(config-zone-pair)# action permit
       vesr(config-zone-pair)# enable
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
    

