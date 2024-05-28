# Все команды, разбитые по устройствам
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
echo 10.0.3.100/24 > /etc/net/ifaces/eth0/ipv4address
echo 2001:33::100/64 > /etc/net/ifaces/eth0/ipv6address
echo default via 10.0.33.1 > /etc/net/ifaces/eth0/ipv4route
echo default via 2001:33::1 > /etc/net/ifaces/eth0/ipv6route
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
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables-save >> /etc/sysconfig/iptables
systemctl enable --now iptables
```
