# Все команды, разбитые по устройствам
- [CLI](#CLI)
- [ISP](#ISP)
- [HQ-R](#HQ-R)
- [BR-R](#BR-R)
- [HQ-SRV](#HQ-SRV)
- [BR-SRV](#BR-SRV)

## CLI
```sh
hostnamectl set-hostname cli;exec bash
```
```sh
useradd admin && echo P@ssw0rd | passwd admin --stdin
```
```sh
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
day=$(date +%A-%F)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"
tar czf $dest/$archive_file $backup_files
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
echo nameserver 192.168.100.1 > /etc/net/ifaces/eth1/resolv.conf
systemctl restart network
```
```sh
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
  hardware ethernet 00:50:00:00:03:00;
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
day=$(date +%A-%F)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"
tar czf $dest/$archive_file $backup_files
EOF
chmod +x /etc/scripts/backup-script.sh
/etc/scripts/backup-script.sh
```
```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1/g' /etc/net/sysctl.conf
```
```sh
sed -i 's/CONFIG_IPV6=/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
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
echo nameserver 192.168.100.1 > /etc/net/ifaces/eth1/resolv.conf
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
sed -i 's/CONFIG_IPV6=/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
mkdir /etc/net/ifaces/eth0
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
BOOTPROTO=dhcp,static
EOF
echo 192.168.200.1/26 > /etc/net/ifaces/eth0/ipv4address
echo 2000:100::1/122 > /etc/net/ifaces/eth0/ipv6address
echo default via 192.168.200.2 > /etc/net/ifaces/eth0/ipv4route
echo default via 2000:100::2 > /etc/net/ifaces/eth0/ipv6route
systemctl restart network
echo nameserver 192.168.100.1 > /etc/resolv.conf
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
sed -i 's/CONFIG_IPV6=/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
mkdir /etc/net/ifaces/eth0
cat <<EOF > /etc/net/ifaces/eth0/options
TYPE=eth
BOOTPROTO=static
EOF
echo 172.16.200.1/28 > /etc/net/ifaces/eth0/ipv4address
echo 2000:200::1/124 > /etc/net/ifaces/eth0/ipv6address
echo default via 172.16.200.2 > /etc/net/ifaces/eth0/ipv4route
echo default via 2000:200::2 > /etc/net/ifaces/eth0/ipv6route
systemctl restart network
echo nameserver 192.168.100.1 > /etc/resolv.conf
```
```sh
apt-get install clamav clamav-db -y
echo <<EOF > /root/infected.txt
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
EOF
cat <<EOF > /etc/cron.d/clamav-scan
# RUN system scanning at midnight
0       0       *       *       *       clamscan -ir /  >> /root/clamav-scan.log
EOF
```



