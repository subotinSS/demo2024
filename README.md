# Все команды, разбитые по устройствам
- [CLI](#CLI)
- [ISP](#ISP)
- [HQ-R](#HQ-R)
- [BR-R](#BR-R)
- [HQ-SRV](#HQ-SRV)
- [BR-SRV](#BR-SRV)

# Таблица IP адресов
<table>
  <tr>
    <td>Имя устройства</td>
    <td>Интерфейс</td>
    <td>IPv4</td>
    <td>IPv6</td>
    <td>NIC</td>
  </tr>
  <tr>
    <td>CLI</td>
    <td>eth0</td>
    <td>10.0.1.100/24</td>
    <td>2001:11::100/64</td>
    <td>ISP-CLI</td>
  </tr>
  <tr>
    <td rowspan="4">ISP</td>
    <td>eth0</td>
    <td>DHCP</td>
    <td>-</td>
    <td>INTERNET</td>
  </tr>
  <tr>
    <td>eth1</td>
    <td>10.0.1.1/24</td>
    <td>2001:11::1/64</td>
    <td>ISP-CLI</td>
  </tr>
  <tr>
    <td>eth2</td>
    <td>10.0.2.1/24</td>
    <td>2001:22::1/64</td>
    <td>ISP-HQ</td>
  </tr>
  <tr>
    <td>eth3</td>
    <td>10.0.3.1/24</td>
    <td>2001:33::1/64</td>
    <td>ISP-BR</td>
  </tr>
  <tr>
    <td rowspan="3">HQ-R</td>
    <td>eth0</td>
    <td>10.0.2.100/24</td>
    <td>2001:22::100/64</td>
    <td>ISP-HQ</td>
  </tr>
  <tr>
    <td>eth1</td>
    <td>192.168.200.2/26</td>
    <td>2000:100::2/122</td>
    <td>HQ-SRV</td>
  </tr>
  <tr>
    <td>wg0</td>
    <td>10.0.0.1/24</td>
    <td>2001:100::1/64, fe80::200:ff:fe00:1/64</td>
    <td>TUNNEL</td>
  </tr>
  <tr>
    <td>HQ-SRV</td>
    <td>eth0</td>
    <td>10.0.2.100/24</td>
    <td>2001:22::100/64</td>
    <td>HQ-R</td>
   </tr>
  <tr>
    <td rowspan="3">BR-R</td>
    <td>eth0</td>
    <td>10.0.3.100/24</td>
    <td>2001:33::100/64</td>
    <td>ISP-BR</td>
  </tr>
 <tr>
    <td>eth1</td>
    <td>172.16.200.2/28</td>
    <td>2000:200::2/124</td>
    <td>BR-SRV</td>
  </tr>
  <tr>
    <td>wg0</td>
    <td>10.0.0.2/24</td>
    <td>2001:100::2/64, fe80::200:ff:fe00:2/64</td>
    <td>TUNNEL</td>
  </tr>
   <tr>
    <td>BR-SRV</td>
    <td>eth0</td>
    <td>172.16.200.1/28</td>
    <td>2000:200::1/124</td>
    <td>BR-R</td>
   </tr>
</table>

## CLI
```sh
hostnamectl set-hostname cli;exec bash
```
```sh
useradd admin && echo P@ssw0rd | passwd admin --stdin
```
```sh
mkdir /etc/net/ifaces/eth0
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPV4=YES
CONFIG_IPV6=YES
EOF
echo 10.0.1.100/24 > /etc/net/ifaces/eth0/ipv4address
echo 2001:11::100/64 > /etc/net/ifaces/eth0/ipv6address
echo default via 10.0.1.1 > /etc/net/ifaces/eth0/ipv4route
echo default via 2001:11::1 > /etc/net/ifaces/eth0/ipv6route
echo nameserver 192.168.200.1 192.168.100.1 > /etc/net/ifaces/eth0/resolv.conf
systemctl restart network
```
## ISP
```sh
hostnamectl set-hostname isp;exec bash
```
```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1/g' /etc/net/sysctl.conf
```
```sh
sed -i 's/CONFIG_IPV6=.*/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
mkdir /etc/net/ifaces/eth{0..3}
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
BOOTPROTO=dhcp
EOF
cat <<EOF > /etc/net/ifaces/eth1/options
TYPE=eth
BOOTPROTO=static
EOF
cp /etc/net/ifaces/eth1/options /etc/net/ifaces/eth2/
cp /etc/net/ifaces/eth1/options /etc/net/ifaces/eth3/
echo 10.0.1.1/24 > /etc/net/ifaces/eth1/ipv4address
echo 2001:11::1/64 > /etc/net/ifaces/eth1/ipv6address
echo 10.0.2.1/24 > /etc/net/ifaces/eth2/ipv4address
echo 2001:22::1/64 > /etc/net/ifaces/eth2/ipv6address
echo 192.168.200.0/26 via 10.0.2.100 > /etc/net/ifaces/eth2/ipv4route
echo 2000:100::/122 via 2001:22::100 > /etc/net/ifaces/eth2/ipv6route
echo 10.0.3.1/24 > /etc/net/ifaces/eth3/ipv4address
echo 2001:33::1/64 > /etc/net/ifaces/eth3/ipv6address
echo 172.16.200.0/28 via 10.0.3.100 > /etc/net/ifaces/eth3/ipv4route
echo 2000:200::/124 via 2001:33::100 > /etc/net/ifaces/eth3/ipv6route
systemctl restart network
```
```sh
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables-save >> /etc/sysconfig/iptables
systemctl enable --now iptables
```
```sh
apt-get install iperf3 -y
systemctl enable --now iperf3
```
## HQ-R
```sh
hostnamectl set-hostname hq-r.hq.work;exec bash
```
```sh
useradd admin && echo P@ssw0rd | passwd admin --stdin
useradd network_admin && echo P@ssw0rd | passwd network_admin --stdin
```
```sh
mkdir /opt/backup
mkdir /etc/scripts
cat <<EOF > /etc/scripts/backup-script.sh
#!/bin/bash
backup_files="/home /etc"
dest="/opt/backup"
day=\$(date +%A-%F)
hostname=\$(hostname -s)
archive_file="\$hostname-\$day.tgz"
tar czf \$dest/\$archive_file \$backup_files
EOF
chmod +x /etc/scripts/backup-script.sh
/etc/scripts/backup-script.sh
```
```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1/g' /etc/net/sysctl.conf
```
```sh
sed -i 's/CONFIG_IPV6=.*/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
mkdir /etc/net/ifaces/eth{0..1}
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
BOOTPROTO=static
EOF
cp /etc/net/ifaces/eth0/options /etc/net/ifaces/eth1/
echo 10.0.2.100/24 > /etc/net/ifaces/eth0/ipv4address
echo 2001:22::100/64 > /etc/net/ifaces/eth0/ipv6address
echo default via 10.0.2.1 > /etc/net/ifaces/eth0/ipv4route
echo default via 2001:22::1 > /etc/net/ifaces/eth0/ipv6route
echo 192.168.200.2/26 > /etc/net/ifaces/eth1/ipv4address
echo 2000:100::2/122 > /etc/net/ifaces/eth1/ipv6address
echo nameserver 192.168.200.1 192.168.100.1 > /etc/net/ifaces/eth0/resolv.conf
systemctl restart network
```
```sh
apt-get update
apt-get install iperf3 -y
iperf3 -c 10.0.2.1 --get-server-output
```
<!---
Wireguard keys
PrivateKeyServer = EC8jPEudJhau5nBa6CB2OPQxH6GHqvRYNUw6M11kNV4=
PublicKeyServer = 5ewEdGg2jcsE5Pht2O1X06RsvBwufxdev8SLgvJGKks=
PrivateKeyClient = 0NoAhQRWFxULuptD5tiqyhZcZIdNi66IGxPOu25LWlw=
PublicKeyClient = n50LXW13DNK8goptE84NyLJ2OIVlwVr2tVlezgtgAHw=
-->
```sh
apt-get install wireguard-tools wireguard-tools-wg-quick -y
mkdir /etc/wireguard
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
PrivateKey = EC8jPEudJhau5nBa6CB2OPQxH6GHqvRYNUw6M11kNV4=
Address = 10.0.0.1/24, 2001:100::1/64, fe80::200:ff:fe00:1/64
ListenPort = 1234
Table = off
[Peer]
PublicKey = n50LXW13DNK8goptE84NyLJ2OIVlwVr2tVlezgtgAHw=
AllowedIPs = 0.0.0.0/0, ::/0
EOF
systemctl restart wg-quick@wg0
systemctl enable wg-quick@wg0
```
```sh
apt-get install frr -y
sed -i 's/ospfd=.*/ospfd=yes/g' /etc/frr/daemons
sed -i 's/ospf6d=.*/ospf6d=yes/g' /etc/frr/daemons
cat <<EOF > /etc/frr/frr.conf
interface wg0
 ipv6 ospf6 area 0
exit
!
interface eth1
 ipv6 ospf6 area 0
exit
!
router ospf
 ospf router-id 1.1.1.1
 network 10.0.0.0/24 area 0
 network 192.168.200.0/26 area 0
exit
!
router ospf6
 ospf6 router-id 11.11.11.11
exit
EOF
systemctl enable --now frr
systemctl restart frr
```
```sh
apt-get install chrony -y
cat <<EOF > /etc/chrony.conf
server 127.0.0.1 iburst
local stratum 5
allow all
EOF
systemctl enable --now chronyd
systemctl restart chronyd
```
```sh
apt-get install dhcp-server -y
sed -i 's/DHCPDARGS=.*/DHCPDARGS=eth1/g' /etc/sysconfig/dhcpd
cat <<EOF > /etc/dhcp/dhcpd.conf
default-lease-time 6000;
max-lease-time 72000;
authoritative;
subnet 192.168.200.0 netmask 255.255.255.192 {
  range 192.168.200.3 192.168.200.62;
  option domain-name-servers 192.168.200.1;
  option routers 192.168.200.2;
}
host hq-srv {
  hardware ethernet aa:bb:cc:dd:ee:ff;
  fixed-address 192.168.200.1;
}
EOF
systemctl enable --now dhcpd
systemctl restart dhcpd
```
## BR-R
```sh
hostnamectl set-hostname br-r.branch.work;exec bash
```
```sh
useradd branch_admin && echo P@ssw0rd | passwd branch_admin --stdin
useradd network_admin && echo P@ssw0rd | passwd network_admin --stdin
```
```sh
mkdir /opt/backup
mkdir /etc/scripts
cat <<EOF > /etc/scripts/backup-script.sh
#!/bin/bash
backup_files="/home /etc"
dest="/opt/backup"
day=\$(date +%A-%F)
hostname=\$(hostname -s)
archive_file="\$hostname-\$day.tgz"
tar czf \$dest/\$archive_file \$backup_files
EOF
chmod +x /etc/scripts/backup-script.sh
/etc/scripts/backup-script.sh
```
```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1/g' /etc/net/sysctl.conf
```
```sh
sed -i 's/CONFIG_IPV6=.*/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
mkdir /etc/net/ifaces/eth{0..1}
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
BOOTPROTO=static
EOF
cp /etc/net/ifaces/eth0/options /etc/net/ifaces/eth1/
echo 10.0.3.100/24 > /etc/net/ifaces/eth0/ipv4address
echo 2001:33::100/64 > /etc/net/ifaces/eth0/ipv6address
echo default via 10.0.3.1 > /etc/net/ifaces/eth0/ipv4route
echo default via 2001:33::1 > /etc/net/ifaces/eth0/ipv6route
echo 172.16.200.2/28 > /etc/net/ifaces/eth1/ipv4address
echo 2000:200::2/124 > /etc/net/ifaces/eth1/ipv6address
echo nameserver 192.168.200.1 192.168.100.1 > /etc/net/ifaces/eth0/resolv.conf
systemctl restart network
```
<!---
Wireguard keys
PrivateKeyServer = EC8jPEudJhau5nBa6CB2OPQxH6GHqvRYNUw6M11kNV4=
PublicKeyServer = 5ewEdGg2jcsE5Pht2O1X06RsvBwufxdev8SLgvJGKks=
PrivateKeyClient = 0NoAhQRWFxULuptD5tiqyhZcZIdNi66IGxPOu25LWlw=
PublicKeyClient = n50LXW13DNK8goptE84NyLJ2OIVlwVr2tVlezgtgAHw=
-->
```sh
apt-get update
apt-get install wireguard-tools wireguard-tools-wg-quick -y
mkdir /etc/wireguard
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
PrivateKey = 0NoAhQRWFxULuptD5tiqyhZcZIdNi66IGxPOu25LWlw=
Address = 10.0.0.2/24, 2001:100::2/64, fe80::200:ff:fe00:2/64
Table = off
[Peer]
PublicKey = 5ewEdGg2jcsE5Pht2O1X06RsvBwufxdev8SLgvJGKks=
Endpoint = 10.0.2.100:1234
PersistentKeepalive = 10
AllowedIPs = 0.0.0.0/0, ::/0
EOF
systemctl restart wg-quick@wg0
systemctl enable wg-quick@wg0
```
```sh
apt-get install frr -y
sed -i 's/ospfd=.*/ospfd=yes/g' /etc/frr/daemons
sed -i 's/ospf6d=.*/ospf6d=yes/g' /etc/frr/daemons
cat <<EOF > /etc/frr/frr.conf
interface wg0
 ipv6 ospf6 area 0
exit
!
interface eth1
 ipv6 ospf6 area 0
exit
!
router ospf
 ospf router-id 2.2.2.2
 network 10.0.0.0/24 area 0
 network 172.16.200.0/28 area 0
exit
!
router ospf6
 ospf6 router-id 22.22.22.22
exit
EOF
systemctl enable --now frr
systemctl restart frr
```
```sh
apt-get install chrony -y
echo server 192.168.200.2 iburst > /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
```
## HQ-SRV
```sh
hostnamectl set-hostname hq-srv.hq.work;exec bash
```
```sh
useradd admin && echo P@ssw0rd | passwd admin --stdin
```
```sh
sed -i 's/CONFIG_IPV6=.*/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
mkdir /etc/net/ifaces/eth0
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
BOOTPROTO=dhcp,static
EOF
echo 192.168.200.1/26 > /etc/net/ifaces/eth0/ipv4address
echo 2000:100::1/122 > /etc/net/ifaces/eth0/ipv6address
echo default via 192.168.200.2 > /etc/net/ifaces/eth0/ipv4route
echo default via 2000:100::2 > /etc/net/ifaces/eth0/ipv6route
echo address aa:bb:cc:dd:ee:ff > /etc/net/ifaces/eth0/iplink
echo nameserver 192.168.200.1 192.168.100.1 > /etc/net/ifaces/eth0/resolv.conf
systemctl restart network
```
```sh
apt-get update
apt-get install chrony -y
echo server 192.168.200.2 iburst > /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
```
```sh
apt-get install clamav clamav-db -y
echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' > /root/infected.txt
echo '0 0 * * * clamscan -ir /  >> /root/clamav-scan.log' > /etc/cron.d/clamav-scan
```
```sh
apt-get install bind -y
cat <<EOF > /etc/bind/zone/hq.work
\$TTL 1D
@ IN SOA hq-srv.hq.work. root.hq.work. (
1;
12H;
1H;
1W;
1H;
)
@ IN NS hq-srv.hq.work.
hq-srv IN A 192.168.200.1
hq-r IN A 192.168.200.2
EOF
cat <<EOF > /etc/bind/zone/branch.work
\$TTL 1D
@ IN SOA hq-srv.branch.work. root.branch.work. (
1;
12H;
1H;
1W;
1H;
)
 IN NS hq-srv.hq.work.
hq-srv IN A 172.16.200.1
br-srv IN A 172.16.200.1
br-r IN A 172.16.200.2
EOF
cat <<EOF > /etc/bind/zone/domain.work
\$TTL 1D
@ IN SOA hq-srv.domain.work. root.domain.work. (
1;
12H;
1H;
1W;
1H;
)
 IN NS hq-srv.domain.work.
hq-srv IN A 192.168.200.1
EOF
cat <<EOF > /etc/bind/zone/hq.192
\$TTL 1D
@ IN SOA hq-srv.hq.work. root.hq.work. (
1;
12H;
1H;
1W;
1H;
)
@ IN NS hq-srv.hq.work.
1 IN PTR hq-srv
2 IN PTR hq-r
EOF
cat <<EOF > /etc/bind/zone/branch.172
\$TTL 1D
@ IN SOA hq-srv.hq.work. root.hq.work. (
1;
12H;
1H;
1W;
1H;
)
@ IN NS hq-srv.branch.work.
1 IN PTR br-srv
2 IN PTR br-r
EOF
cat <<EOF > /etc/bind/local.conf
zone "hq.work" {
type master;
file "/etc/bind/zone/hq.work";
};
zone "branch.work" {
type master;
file "/etc/bind/zone/branch.work";
};
zone "domain.work" {
type master;
file "/etc/bind/zone/domain.work";
};
zone "200.168.192.in-addr.arpa" {
type master;
file "/etc/bind/zone/hq.192";
};
zone "200.16.172.in-addr.arpa" {
type master;
file "/etc/bind/zone/branch.172";
};
EOF
sed -i 's/127.0.0.1/any/g' /etc/bind/options.conf
sed -i 's/::1/any/g' /etc/bind/options.conf
sed -i 's/\/\/forwarders .*/forwarders { 192.168.100.1; };/g' /etc/bind/options.conf
sed -i 's/::1/any/g' /etc/bind/options.conf
sed -i 's/\/\/allow-query .*/allow-query { any; };/g' /etc/bind/options.conf
sed -i 's/\/\/allow-query-cache .*/allow-query-cache { any; };/g' /etc/bind/options.conf
sed -i 's/\/\/interface-interval .*/dnssec-validation no;/g' /etc/bind/options.conf
chmod 777 /etc/bind/zone/ -R
systemctl enable --now bind
systemctl restart bind
```
```sh
apt-get install docker-engine docker-compose -y
systemctl enable --now docker.service
cat << EOF > /home/user/wiki.yaml
version: '3'
services:
  wiki:
    image: mediawiki
    restart: always
    ports:
      - 8080:80
    volumes:
      - ./LocalSettings.php:/var/www/html/LocalSettings.php
    links:
      - db
  db:
    image: mysql
    container_name: db
    environment:
      MYSQL_DATABASE: mediawiki
      MYSQL_USER: wiki
      MYSQL_PASSWORD: DEP@ssw0rd
      MYSQL_ROOT_PASSWORD: toor
EOF
docker-compose -f /home/user/wiki.yaml up -d
```
## BR-SRV
```sh
hostnamectl set-hostname br-srv.branch.work;exec bash
```
```sh
useradd network_admin && echo P@ssw0rd | passwd admin --stdin
useradd branch_admin && echo P@ssw0rd | passwd admin --stdin
```
```sh
sed -i 's/CONFIG_IPV6=.*/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
mkdir /etc/net/ifaces/eth0
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
BOOTPROTO=static
EOF
echo 172.16.200.1/28 > /etc/net/ifaces/eth0/ipv4address
echo 2000:200::1/124 > /etc/net/ifaces/eth0/ipv6address
echo default via 172.16.200.2 > /etc/net/ifaces/eth0/ipv4route
echo default via 2000:200::2 > /etc/net/ifaces/eth0/ipv6route
echo nameserver 192.168.200.1 192.168.100.1 > /etc/net/ifaces/eth0/resolv.conf
systemctl restart network
```
```sh
apt-get update
apt-get install chrony -y
echo server 192.168.200.2 iburst > /etc/chrony.conf
systemctl enable --now chronyd
systemctl restart chronyd
```
```sh
apt-get install clamav clamav-db -y
echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' > /root/infected.txt
echo '0 0 * * * clamscan -ir /  >> /root/clamav-scan.log' > /etc/cron.d/clamav-scan
```
```sh
apt-get install MySQL-server moodle moodle-apache2 moodle-local-mysql -y
systemctl enable --now mysqld httpd2
PASSWORD="" mysql -uroot <<EOF
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED WITH mysql_native_password BY 'moodlepasswd';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EOF
```



