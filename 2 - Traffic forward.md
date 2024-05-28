# Настройка перенаправления трафика
## ISP:
```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1/g' /etc/net/sysctl.conf
systemctl restart network
```
## HQ-R:
```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1/g' /etc/net/sysctl.conf
systemctl restart network
```
## BR-R:
```sh
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1/g' /etc/net/sysctl.conf
systemctl restart network
```
