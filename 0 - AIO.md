# Все команды, разбитые по устройствам
[CLI](#CLI)
[ISP](#ISP)
[HQ-R](#HQ-R)
[BR-R](#BR-R)

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
sed -i 's/CONFIG_IPV6=/CONFIG_IPV6=YES/g' /etc/net/ifaces/default/options
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
```
```sh
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
```
```sh
systemctl restart network
```
```sh
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables-save >> /etc/sysconfig/iptables
systemctl enable --now iptables
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
```
```sh
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
apt-get install wireguard-tools wireguard-tools-wg-quick -y
mkdir /etc/wireguard
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
PrivateKey = EC8jPEudJhau5nBa6CB2OPQxH6GHqvRYNUw6M11kNV4=
#PublicKey = 5ewEdGg2jcsE5Pht2O1X06RsvBwufxdev8SLgvJGKks=
Address = 10.0.0.1/24, 2001:100::1/64, fe80::200:ff:fe00:1/64
ListenPort = 1234
Table = off
[Peer]
#PrivateKey = 0NoAhQRWFxULuptD5tiqyhZcZIdNi66IGxPOu25LWlw=
PublicKey = n50LXW13DNK8goptE84NyLJ2OIVlwVr2tVlezgtgAHw=
AllowedIPs = 0.0.0.0/0, ::/0
EOF
systemctl restart wg-quick@wg0
systemctl enable wg-quick@wg0
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
```
```sh
echo 10.0.3.100/24 > /etc/net/ifaces/eth0/ipv4address
echo 2001:33::100/64 > /etc/net/ifaces/eth0/ipv6address
echo default via 10.0.3.1 > /etc/net/ifaces/eth0/ipv4route
echo default via 2001:33::1/64 > /etc/net/ifaces/eth0/ipv6route
echo 172.16.200.2/28 > /etc/net/ifaces/eth1/ipv4address
echo 2000:200::2/124 > /etc/net/ifaces/eth1/ipv6address
echo nameserver 192.168.100.1 > /etc/net/ifaces/eth1/resolv.conf
systemctl restart network
```
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
