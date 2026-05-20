+++
date = '2026-05-20T09:51:20+08:00'
draft = false
title = 'RedHat_RH358'
slug = "oko1bg25nv"
description = "RedHat_RH358"
summary = "RedHat_RH358"
categories = ["📒考试"]
tags = ["RedHat"]
comments = true
series = ["RedHat 认证考试"]
showSeries = true
+++

# RH358 试题解法 - Linux网络服务自动化部署

## 0 环境准备

```bash
# 创建虚拟机
for i in servera serverb serverc serverd control; do
qemu-img create -f qcow2 /virtual/rh358-${i}-vda.qcow2 40G
sudo virt-install --name rh358-${i} --memory 512 --vcpus 4 --import --os-variant rhel8.7 --disk path=/virtual/rh358-${i}-vda.qcow2,bus=virtio --location http://10.173.28.219/web/cdrom/RedHat/8.9/ --network bridge=net-172.24.1.0,model=virtio --extra-args ks=http://10.173.28.219/web/ks/rhel-8.9.cfg --graphics vnc --noautoconsole
done

# 创建网络
for i in 172.24 172.25 192.168; do
sudo bash -c "cat > /usr/share/libvirt/networks/net-${i}.1.0.xml" << END
<network>
    <name>net-${i}.1.0</name>
    <bridge name="net-${i}.1.0"></bridge>
    <forward></forward>
    <ip address="${i}.1.1" netmask="255.255.255.0">
        <dhcp>
            <range start="${i}.1.50" end="${i}.1.99"></range>
        </dhcp>
    </ip>
</network>
END
virsh net-define /usr/share/libvirt/networks/net-${i}.1.0.xml
virsh net-start net-${i}.1.0
virsh net-autostart net-${i}.1.0
done

# 开机
for i in servera serverb serverc serverd control; do
sudo virsh start rh358-${i}
done

# 设置hosts
cat > /etc/hosts << END
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.24.1.101 node1.domain1.example.com
172.24.1.102 node2.domain1.example.com
172.24.1.103 node3.domain1.example.com
172.24.1.104 node4.domain1.example.com
172.24.1.101 www.domain1.example.com 
172.24.1.105 control.domain1.example.com
172.24.1.254 rhgls.domain1.example.com
10.2.0.254 utility.domain1.example.com
END

# 设置 servera serverb serverc serverd control 节点的IP
nmcli con mod enp1s0 ipv4.add 172.24.1.101/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.1.101/24
nmcli con up enp1s0

nmcli con mod enp1s0 ipv4.add 172.24.1.102/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.1.102/24
nmcli con up enp1s0 

nmcli con mod enp1s0 ipv4.add 172.24.1.103/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.1.103/24
nmcli con up enp1s0 

nmcli con mod enp1s0 ipv4.add 172.24.1.104/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.1.104/24
nmcli con up enp1s0 

nmcli con mod enp1s0 ipv4.add 172.24.1.105/24 ipv4.gate 172.24.1.1 ipv4.dns 172.24.1.1 ipv4.meth man
nmcli con mod enp1s0 +ipv4.add 192.168.1.105/24
nmcli con up enp1s0

# 设置 servera named 服务
yum install bind -y
vi /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
    listen-on port 53 { 172.24.1.101; };
    listen-on-v6 port 53 { ::1; };
    directory   "/var/named";
    dump-file   "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    secroots-file   "/var/named/data/named.secroots";
    recursing-file  "/var/named/data/named.recursing";
    allow-query     { any; };

    /* 
     - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
     - If you are building a RECURSIVE (caching) DNS server, you need to enable 
       recursion. 
     - If your recursive DNS server has a public IP address, you MUST enable access 
       control to limit queries to your legitimate users. Failing to do so will
       cause your server to become part of large scale DNS amplification 
       attacks. Implementing BCP38 within your network would greatly
       reduce such attack surface 
    */
    recursion yes;

    dnssec-enable yes;
    dnssec-validation yes;

    managed-keys-directory "/var/named/dynamic";

    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";

    /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
    include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
    type hint;
    file "named.ca";
};
zone "remote1.example.com." IN {
       type master;
       file "ex358-remote-reverse.zone";
};

zone "1.168.192.in-addr.arpa." IN {
       type master;
       file "ex358-remote-reverse.zone";
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

vi /var/named/ex358-remote-reverse.zone
$TTL 3h
remote1.example.com. IN SOA node1.remote1.example.com. root.node1.remote1.example.com. (2022030501 1H 5M 1W 1M)
1.168.192.in-addr.arpa. IN SOA node1.remote1.example.com. root.node1.remote1.example.com. (2022030501 1H 5M 1W 1M)
remote1.example.com. IN NS node1.remote1.example.com.
1.168.192.in-addr.arpa. IN NS node1.remote1.example.com.

# 制作快照
for i in servera serverb serverc serverd control; do
sudo virsh destroy rh358-${i}
sudo virsh snapshot-create-as rh358-${i} snapshot1
done
```

## 1. 配置防火墙

![image-20241011204428993](https://typora.fanyops.com/documents/2024/1011/image-20241011204428993.png)

```bash
#node1
firewall-cmd --set-default-zone=trusted
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=172.25.1.0/24 reject'
firewall-cmd --reload

#node2
firewall-cmd --set-default-zone=trusted
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=172.25.1.0/24 reject'
firewall-cmd --reload
```

## 2. 配置IPv6地址

![image-20241011204412189](https://typora.fanyops.com/documents/2024/1011/image-20241011204412189.png)

```bash
#node1
nmcli connection modify 'eth0' ipv6.method manual ipv6.addresses 2001:ac18::106/64
nmcli con up eth0
#node2
nmcli connection modify 'eth0' ipv6.method manual ipv6.addresses 2001:ac18::107/64
nmcli con up eth0
```

## 3. 配置dhcp服务器

![image-20241011204542298](https://typora.fanyops.com/documents/2024/1011/image-20241011204542298.png)

```bash
#node1
yum install dhcp-server -y
cat > /etc/dhcp/dhcpd.conf <<END
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.150 192.168.1.254;
    option domain-name-servers 192.168.1.6;
    option domain-name "remote1.example.com";
    option routers 192.168.1.100;
    default-lease-time 600;
    max-lease-time 3000;
}
host node5 {
    hardware ethernet 52:54:c0:a8:01:0a;
    fixed-address 192.168.1.10;
}
host node6 {
    hardware ethernet 52:54:c0:a8:01:0b;
    fixed-address 192.168.1.11;
}
END
systemctl enable dhcpd --now
```

## 4. 配置DNS服务器

![image-20241011204504302](https://typora.fanyops.com/documents/2024/1011/image-20241011204504302.png)

```bash
cat /var/named/ex358-remote-reverse.zone
$TTL 3h
remote1.example.com. IN SOA node1.remote1.example.com. root.node1.remote1.example.com. (2022030501 1H 5M 1W 1M)
1.168.192.in-addr.arpa. IN SOA node1.remote1.example.com. root.node1.remote1.example.com. (2022030501 1H 5M 1W 1M)
remote1.example.com. IN NS node1.remote1.example.com.
1.168.192.in-addr.arpa. IN NS node1.remote1.example.com.

101 PTR node1.remote1.example.com.
10 PTR node5.remote1.example.com.
11 PTR node6.remote1.example.com.
node1 A 192.168.1.101
node5 A 192.168.1.10
node6 A 192.168.1.11

systemctl restart named
systemctl enable named

cat > /etc/resolv.conf <<END
 nameserver 172.24.1.101
END
```

## 5. 配置SMB

![image-20241011204606710](https://typora.fanyops.com/documents/2024/1011/image-20241011204606710.png)

```bash
yum install samba* cifs-utils -y
mkdir /common
chcon -t samba_share_t /common/
[root@node1 ~]# cat /etc/samba/smb.conf
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.
[global]
    workgroup = STAFF
    security = user
    passdb backend = tdbsam
    printing = cups
    printcap name = cups
    load printers = yes
    cups options = raw
[homes]
    comment = Home Directories
    valid users = %S, %D%w%S
    browseable = No
    read only = Noinherit acls = Yes
[printers]
    comment = All Printers
    path = /var/tmp
    printable = Yes
    create mask = 0600
    browseable = No
[print$]
    comment = Printer Drivers
    path = /var/lib/samba/drivers
    write list = @printadmin root
    force group = @printadmin
    create mask = 0664
    directory mask = 0775
[common]
    comment = common
    path = /common
    valid users = andy

useradd -s /sbin/nologin andy
smbpasswd -a andy
systemctl enable smb --now
```

## 6. 配置SMB多用户挂载

![image-20241011204733625](https://typora.fanyops.com/documents/2024/1011/image-20241011204733625.png)

```bash
#node1
mkdir /miscellaneous
chmod 777 /miscellaneous
chcon -t samba_share_t /miscellaneous

vim /etc/samba/smb.conf
[miscellaneous]
comment = miscellaneous
path = /miscellaneous
valid users = silene, akira
write list = akira

useradd -s /sbin/nologin silene
useradd -s /sbin/nologin akira
smbpasswd -a silene
smbpasswd -a akira
systemctl restart smb

#node2
mkdir /mnt/multi
yum install cifs-utils -y
echo "//node1.domain1.example.com/miscellaneous /mnt/multi cifs multiuser,sec=ntlmssp,username=silene,password=flectrag 0 0`" >> /etc/fstab
mount -a
useradd smbtest
su - smbtest
cifscreds add -u akira node1.domain1.example.com
touch /mnt/multi/test.txt
```

## 7. 配置nfs服务器

![image-20241011205228309](https://typora.fanyops.com/documents/2024/1011/image-20241011205228309.png)

```bash
mkdir /public
yum install nfs-utils
chmod 777 /public/
echo "/public 172.24.1.0/255.255.255.0(ro)" > /etc/exports
systemctl enable nfs-server --now
```

## 8. 配置NFS挂载

![image-20241011205127680](https://typora.fanyops.com/documents/2024/1011/image-20241011205127680.png)

```bash
#node2
mkdir /mnt/nfsmount
echo "node1.domain1.example.com:/public /mnt/nfsmount nfs defaults 0 0" >> /etc/fstab
mount -a
```

## 9. 部署web服务器

![image-20241011205252948](https://typora.fanyops.com/documents/2024/1011/image-20241011205252948.png)

```bash
yum install httpd -y
cd /var/www/html/
wget http://utility.domain1.example.com/materials/station.html
mv station.html index.html
systemctl enable httpd --now

#node2
[root@node2 ~]# curl node1.domain1.example.com
gls is very pool, but very handsome.
```

## 10. 配置web加密

![image-20241011205326555](https://typora.fanyops.com/documents/2024/1011/image-20241011205326555.png)

```bash
cd /etc/pki/tls/certs/
wget http://utility.domain1.example.com/materials/node1.crt
wget http://utility.domain1.example.com/materials/domain1.crt
cd /etc/pki/tls/private/
wget http://utility.domain1.example.com/materials/node1.key
yum install mod_ssl -y
vim /etc/httpd/conf.d/v.conf
Listen 443
<VirtualHost *:80>
ServerName node1.domain1.example.com
DocumentRoot /var/www/html
</VirtualHost>
<VirtualHost *:443>
ServerName node1.domain1.example.com
DocumentRoot /var/www/html
SSLEngine on
SSLCertificateFile /etc/pki/tls/certs/node1.crt
SSLCertificateKeyFile /etc/pki/tls/private/node1.key
SSLCACertificateFile /etc/pki/tls/certs/domain1.crt
</VirtualHost>
httpd -t
systemctl restart httpd
#node2
[root@node2 ~]# curl node1.domain1.example.com
gls is very pool, but very handsome.
[root@node2 ~]# curl -k https://node1.domain1.example.com
gls is very pool, but very handsome.
```

## 11. 配置虚拟主机

![image-20241011205348284](https://typora.fanyops.com/documents/2024/1011/image-20241011205348284.png)

```bash
mkdir /var/www/virtual
cd /var/www/virtual/
wget http://utility.domain1.example.com/materials/www.html
mv www.html index.html
vim /etc/httpd/conf.d/v.conf
<VirtualHost *:80>
ServerName www.domain1.example.com
DocumentRoot /var/www/virtual
</VirtualHost>
systemctl restart httpd
setfacl -m u:andy:rwx /var/www/virtual
#node2
[root@node2 ~]# curl www.domain1.example.com
ex358 exam is very simple.
```

## 12. 配置web访问内容

![image-20241011205411621](https://typora.fanyops.com/documents/2024/1011/image-20241011205411621.png)

```bash
mkdir /var/www/html/secret
mkdir /var/www/virtual/secret
cd /var/www/html/secret/
wget http://utility.domain1.example.com/materials/private.html
mv private.html index.html
cd /var/www/virtual/secret/
wget http://utility.domain1.example.com/materials/private.html
mv private.html index.html
vim /etc/httpd/conf.d/v.conf
<Directory /var/www/html/secret>
Order Allow,Deny
Allow from 127.0.0.1
Allow from 172.24.1.101
Allow from 192.168.1.101
</Directory>
<Directory /var/www/virtual/secret>
Order Allow,Deny
Allow from 127.0.0.1
Allow from 172.24.1.101
Allow from 192.168.1.101
</Directory>
systemctl restart httpd
#node1,2
[root@node1 secret]# curl www.domain1.example.com/secret/index.html
web exam have a lot of score.
[root@node1 secret]# curl node1.domain1.example.com/secret/index.html
web exam have a lot of score.
[root@node2 ~]# curl www.domain1.example.com/secret/index.html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title></head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
[root@node2 ~]# curl node1.domain1.example.com/secret/index.html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
[root@node2 ~]#
```

## 13. 配置iscsi target

![image-20241011205432834](https://typora.fanyops.com/documents/2024/1011/image-20241011205432834.png)

```bash
[root@node1 virtual]# fdisk /dev/vdb 
Command (m for help): n
Partition number (1-4, default 1): 1
First sector (2048-20971519, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-20971519, default 20971519): +3272M
Command (m for help): t
Hex code (type L to list all codes): 8e
Command (m for help): p
Command (m for help): w

partprobe /dev/vdb
yum -y install lvm2
pvcreate /dev/vdb1
vgcreate iscsi /dev/vdb1
lvcreate -L 3072M -n iscsi_vol iscsi
yum install targetcli -y
targetcli
/backstores/block create iscsi_vol /dev/iscsi/iscsi_vol
/iscsi create iqn.2022-03.com.example.domain1:node1
/iscsi/iqn.2022-03.com.example.domain1:node1/tpg1/luns create /backstores/block/iscsi_vol
/iscsi/iqn.2022-03.com.example.domain1:node1/tpg1/acls create iqn.2022-03.com.example.domain1:node2
exit
systemctl enable target.service
```

## 14. 配置 iSCSI initiator

![image-20241011205452157](https://typora.fanyops.com/documents/2024/1011/image-20241011205452157.png)

```bash
yum -y install iscsi-initiator-utils
vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2022-03.com.example.domain1:node2
iscsiadm -m discovery -t st -p 172.24.1.101
iscsiadm -m node -T iqn.2022-03.com.example.domain1:node1 -p 172.24.1.101 -l

[root@node2 ~]# fdisk /dev/sda 
Command (m for help): n
Select (default p): 
Partition number (1-4, default 1): 
First sector (2048-6291455, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-6291455, default 6291455): +1700M
Command (m for help): w

mkfs -t ext3 /dev/sda1
mkdir /mnt/data
echo "UUID=`blkid | grep -i sda1 | cut -d '"' -f2` /mnt/data ext3 defaults,_netdev 0 0" >> /etc/fstab
mount -a
systemctl enable iscsi iscsid --now
```

## 15. 配置数据库

![image-20241011205517509](https://typora.fanyops.com/documents/2024/1011/image-20241011205517509.png)

```bash
yum install mariadb-server -y
vim /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
skip-networking=true
systemctl enable mariadb --now
mysql_secure_installation
Set root password? [Y/n] Y
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] Y
wget http://utility.domain1.example.com/materials/users.mdb
mysql -u root -pflectrag -e 'create database Contacts;'
mysql -u root -pflectrag Contacts < users.mdb
mysql -u root -pflectrag -e "grant select on Contacts.* to Luigi@'localhost' identified by 'flectrag';"
systemctl restart mariadb.service
```

## 16. 查询数据库1

![image-20241011205535064](https://typora.fanyops.com/documents/2024/1011/image-20241011205535064.png)

```bash
select first_name from User_Logins,User_Names where User_Names.user_id=User_Logins.id and User_Logins.User_Pass="tangerine";
```

## 17. 查询数据库2

![image-20241011205553605](https://typora.fanyops.com/documents/2024/1011/image-20241011205553605.png)

```bash
select * from User_Contacts,User_Names where User_Contacts.id=User_Names.user_id and User_Contacts.Location='Japan' and User_Names.first_name='cangjing';
```

## 18. 配置主机防火墙

![image-20241011205612450](https://typora.fanyops.com/documents/2024/1011/image-20241011205612450.png)

```bash
ssh admin@control
mkdir ansible
    vim /home/admin/ansible/firewall.yml
---
  - hosts: nodes
    become: yes
    tasks:
    - firewalld:
  zone: trusted
  rich_rule: rule family=ipv4 source address="172.25.1.0/24" reject
  permanent: yes
  state: enabled
  immediate: y
 - shell: firewall-cmd --set-default-zone=trusted
ansible-playbook firewall.yml
```

## 19. 配置nginx服务

![image-20241011205629242](https://typora.fanyops.com/documents/2024/1011/image-20241011205629242.png)

```bash
wget http://utility.domain1.example.com/materials/webserver.conf.j2
wget http://utility.domain1.example.com/materials/index.html.j2
vim /home/admin/ansible/nginx.yml
---
  - hosts: nodes
 become: yes
 tasks:
   - name: install the latest version of nginx
  yum:
    name: nginx
    state: latest
   - name: Template a file to /etc/nginx/conf.d/
  template:
    src: webserver.conf.j2
    dest: /etc/nginx/conf.d/webserver.conf
    mode: 0644
   - name: Create a directory if it does not exist
  file:
    path: /srv/www/html
    state: directory
    mode: '0755'
    recurse: yes
   - name: Template a file to /srv/www/html
  template:
    src: index.html.j2
    dest: /srv/www/html/index.html
    mode: 0644
   - name: Restart service nginx, in all cases
  service:
    name: nginx
    state: restarted
    enabled: yes
ansible-playbook nginx.yml
```

## 20. 配置本地邮件服务

![image-20241011205649448](https://typora.fanyops.com/documents/2024/1011/image-20241011205649448.png)

```bash
yum install rhel-system-roles -y
su - admin
mkdir ansible/roles
cd ansible
vim ansible.cfg
[defaults]
roles_path = roles
cp -r /usr/share/ansible/roles/rhel-system-roles.postfix/ roles/
vim /home/admin/ansible/nullclient.yml
---
  - hosts: nodes
 become: yes
 vars:
   postfix_conf:
  myorigin: "domain1.example.com"
  mydestination: ""
  mynetworks: "127.0.0.0/8"
  relayhost: "[rhgls.domain1.example.com]"
  inet_interfaces: "loopback-only"
 roles:
   - rhel-system-roles.postfix
ansible-playbook nullclient.yml
```

## 21. 配置打印服务

![image-20241011205707910](https://typora.fanyops.com/documents/2024/1011/image-20241011205707910.png)

```bash
vim /home/admin/ansible/printing.yml
---
  - hosts: nodes
 become: yes
 tasks:
   - name: Install a list of packages
  yum:
    name:
    - avahi
    - cups
    state: present
   - name: Enable service cpus, and not touch the state
     service:
    name: cups
    enabled: yes
    state: started
   - command: lpadmin -p "pqueue" -v "ipp://utility.domain1.example.com:8000/ipp/print/ipp-everywhere-pdf" -E
   - command: lpadmin -d "pqueue
ansible-playbook printing.yml
```

