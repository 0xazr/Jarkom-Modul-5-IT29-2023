# Jarkom Modul 5 | IT29 | 2023
- M. Azril Fathoni (5027211002)
- Ridho Husni Indrawan (5027211043)

## Topologi
![image](https://github.com/0xazr/Jarkom-Modul-5-IT29-2023/assets/54212814/279d5684-c93c-4451-9070-e9eb05d6a301)
## Pembagian Subnet
| Subnet | Total IP | Netmask |     Address    |  Subnet Mask    |
|--------|----------|---------|----------------|-----------------|
|  A1    | 514      |  /22    | 10.78.8.0/22   | 255.255.252.0   | 
|  A2    | 1023     |  /21    | 10.78.0.0/21   | 255.255.248.0   |
|  A3    | 2        |  /30    | 10.78.14.128/30| 255.255.255.252 |
|  A4    | 2        |  /30    | 10.78.14.132/30| 255.255.255.252 |
|  A5    | 2        |  /30    | 10.78.14.136/30| 255.255.255.252 |
|  A6    | 2        |  /30    | 10.78.14.140/30| 255.255.255.252 |
|  A7    | 256      |  /23    | 10.78.12.0/23  | 255.255.254.0   |
|  A8    | 66       |  /25    | 10.78.14.0/25  | 255.255.255.128 |
|  A9    | 2        |  /30    | 10.78.14.144/30| 255.255.255.252 |
|  A10   | 2        |  /30    | 10.78.14.148/30| 255.255.255.252 |
|  A11   | 2        |  /30    | 10.78.14.152/30| 255.255.255.252 |
|  Total | 1873     |  /20    |      -         | 255.255.248.0   |

## Routing
### Heiter
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.78.14.130
```
### Aura
Kanan: A2, A1
```
route add -net 10.78.0.0 netmask 255.255.248.0 gw 10.78.14.129
route add -net 10.78.8.0 netmask 255.255.252.0 gw 10.78.14.129
```
Bawah: A5, A6, A7, A8, A9, A10
```
route add -net 10.78.14.136 netmask 255.255.255.252 gw 10.78.14.134
route add -net 10.78.14.140 netmask 255.255.255.252 gw 10.78.14.134
route add -net 10.78.12.0 netmask 255.255.254.0 gw 10.78.14.134
route add -net 10.78.14.0 netmask 255.255.255.128 gw 10.78.14.134
route add -net 10.78.14.144 netmask 255.255.255.252 gw 10.78.14.134
route add -net 10.78.14.148 netmask 255.255.255.252 gw 10.78.14.134
```
### Frieren
Atas: A11, A3, A2, A1
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.78.14.133
```
Kiri: A7, A8, A9, A10
```
route add -net 10.78.12.0 netmask 255.255.254.0 gw 10.78.14.142
route add -net 10.78.14.0 netmask 255.255.255.128 gw 10.78.14.142
route add -net 10.78.14.144 netmask 255.255.255.252 gw 10.78.14.142
route add -net 10.78.14.148 netmask 255.255.255.252 gw 10.78.14.142
```
### Himmel
Kiri: A9, A10
```
route add -net 10.78.14.144 netmask 255.255.255.252 gw 10.78.14.2
route add -net 10.78.14.148 netmask 255.255.255.252 gw 10.78.14.2
```
Kanan
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.78.14.141
```
### Fern
```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.78.14.1
```

### Initial Setup
#### Richter (DNS Server)
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update -y
apt-get install bind9 -y
cat <<EOF > /etc/bind/named.conf.options
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
            192.168.122.1;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        # dnssec-validation no;
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035

        listen-on-v6 { any; };
};
EOF
service bind9 start
```
#### Revolte (DHCP Server)
```
echo nameserver 10.78.14.146 > /etc/resolv.conf
apt-get update -y
apt-get install isc-dhcp-server -y
service isc-dhcp-server start
sed -i 's/INTERFACESv4=""/INTERFACESv4="eth0"/g' /etc/default/isc-dhcp-server
cat << EOF > /etc/dhcp/dhcpd.conf
ddns-update-style none;
default-lease-time 600;
max-lease-time 7200;
authoritative;
log-facility local7;

subnet 10.78.14.148 netmask 255.255.255.252 {
    range 10.78.14.149 10.78.14.150;
    option routers 10.78.14.149;
    option domain-name-servers 10.78.14.146;
    default-lease-time 720;
    max-lease-time 5760;
}

subnet 10.78.14.0 netmask 255.255.255.128 {
    range 10.78.14.2 10.78.14.126;
    option routers 10.78.14.1;
    option domain-name-servers 10.78.14.146;
    default-lease-time 720;
    max-lease-time 5760;
}

subnet 10.78.12.0 netmask 255.255.254.0 {
    range 10.78.12.2 10.78.13.254;
    option routers 10.78.12.1;
    option domain-name-servers 10.78.14.146;
    default-lease-time 720;
    max-lease-time 5760;
}

subnet 10.78.0.0 netmask 255.255.248.0 {
    range 10.78.0.2 10.78.7.254;
    option routers 10.78.0.1;
    option domain-name-servers 10.78.14.146;
    default-lease-time 720;
    max-lease-time 5760;
}

subnet 10.78.8.0 netmask 255.255.252.0 {
    range 10.78.8.3 10.78.11.254;
    option routers 10.78.8.1;
    option domain-name-servers 10.78.14.146;
    default-lease-time 720;
    max-lease-time 5760;
}
EOF
service isc-dhcp-server restart
```
#### Himmel (DHCP Relay)
```
echo nameserver 10.78.14.146 > /etc/resolv.conf
apt-get update -y
apt-get install isc-dhcp-relay -y
service isc-dhcp-relay start
cat <<EOF > /etc/default/isc-dhcp-relay
SERVERS="10.78.14.150"
INTERFACES="eth1 eth2"
OPTIONS=""
EOF
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
service isc-dhcp-relay restart
```
#### Heiter (DHCP Relay)
```
echo nameserver 10.78.14.146 > /etc/resolv.conf
apt-get update -y
apt-get install isc-dhcp-relay -y
service isc-dhcp-relay start
cat <<EOF > /etc/default/isc-dhcp-relay
SERVERS="10.78.14.150"
INTERFACES="eth1 eth2"
OPTIONS=""
EOF
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
service isc-dhcp-relay restart
```

### Soal 1
#### Aura
```
nat_ip=$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source $nat_ip
```

### Soal 2
#### Fern
```
iptables -A INPUT -p udp -j DROP
iptables -A INPUT -p tcp ! --dport 8080 -j DROP
iptables -A INPUT -p tcp -j DROP
```

### Soal 3
```
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

### Soal 4
```
iptables -A INPUT -p tcp --dport 22 -m iprange --src-range 10.21.8.3-10.21.10.5 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

### Soal 5
```
iptables -A INPUT -p tcp --dport 80 -m time --timestart 08:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j DROP
```

### Soal 6
```
iptables -A INPUT -p tcp --dport 80 -m time --timestart 12:00 --timestop 13:00 --weekdays Mon,Tue,Wed,Thu -j DROP
iptables -A INPUT -p tcp --dport 80 -m time --timestart 11:00 --timestop 13:00 --weekdays Fri -j DROP
```
