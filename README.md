# Projectman-example


**TOPOLOGI:**
![image](https://github.com/Fenrir717/Projectman-example/assets/147627144/fc2f674b-2b70-4379-8bb7-0940ea93da03)

**SCENARIO:**
Proyek ini bertujuan untuk mengimplementasikan langkah-langkah keamanan pada tiga server virtual (VM), masing-masing bernama VM1, VM2, dan VM3. Berikut adalah konfigurasi keamanan yang akan diterapkan:

**1. VM1 (Honeypot):**
- IP: 192.168.20.1
- Berfungsi sebagai honeypot untuk mendeteksi potensi ancaman.
- Terinstal Honeypot dengan aturan iptables yang mengalihkan paket yang masuk ke VM2 (IP 192.168.20.2) dengan tujuan port 22, akan dialihkan ke VM1. Penyerang yang mencoba mengakses SSH di VM2 akan terjebak ke Honeypot di VM1.

**2. VM2 (Server Utama):**
- IP: 192.168.20.2
- Terinstal Web Server (CMS WordPress) dan Mail Server (Roundcube).
- Layanan dilindungi oleh UFW dan WAF (ModSecurity2) untuk meningkatkan keamanan.
- Akses SSH ditutup dan hanya dapat diakses melalui Port Knocking (Knockd) yang dikombinasikan dengan VPN Server (OpenVPN).
- SSH hanya dapat diakses melalui VPN dengan Subnet VPN 20.10.20.0/24 pada interface tap0.
- Interface tambahan untuk area lokal dengan IP 10.10.10.1, digunakan untuk komunikasi dengan VM3.
- Akan ditambahkan Monitoring Log Server dengan Promtail & LOKI dan Rsyslog yang akan divisualisasikan dengan Grafana.
- Penggunaan SSL Certificate pada layanan HTTP/Apache2 untuk mengamankan setiap halaman web. Web server menggunakan port 443, mail server (SMTPS 465 dan IMAPS 993), serta Grafana menggunakan HTTPS pada port 443.

**3. VM3 (Server Backup):**
- IP: 10.10.10.2
- Berfungsi sebagai server backup.
- Hanya dapat berkomunikasi dengan VM2 di jaringan lokal (10.10.10.1).
- Tidak terhubung langsung ke internet untuk meningkatkan keamanan.
- Melakukan backup konfigurasi secara rutin dan sinkronisasi konfigurasi secara live dengan VM2.

Proyek ini bertujuan untuk menciptakan lingkungan server yang aman dengan mengimplementasikan praktik keamanan yang canggih seperti honeypot, port knocking, VPN, dan konfigurasi otomatis backup. Seluruh konfigurasi akan didokumentasikan dengan baik di dalam repositori ini untuk memudahkan pengelolaan dan pemeliharaan sistem keamanan.



**Konfigurasi pada Setiap VM**
 [VM1](#1-Konfigurasi-Honeypot-pada-VM1)
 [VM2](#2-Konfigurasi-Server-pada-VM2)
 [VM3](#VM3)


## 1. Konfigurasi Honeypot pada VM1

### 1.1 Konfigurasi Adapter Network di VM1

**Langkah 1: Buka direktori utama Konfigurasi Adapter
```
nano /etc/network/interfaces
```
**Langkah 2: Edit File Konfigurasi seperti ini**
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface

allow-hotplug enp0s3
auto enp0s3
iface enp0s3 inet dhcp

auto enp0s8
iface enp0s8 inet static
 address 192.168.20.1
 netmask 255.255.255.0

```
**Langkah 3: Restart Layanan Networking**
```
systemctl restart networking
```

### 1.2 Konfigurasi Honeypot

**Langkah 1: Instalasi paket dan Depedensi yang dibutuhkan**
```
apt-get install postgresql
apt-get install python3-psycopg2
apt-get install libpq-dev
apt install python3-pip -y
apt install python3.11-venv
```
**Langkah 2: Instalasi Honeypots di virtual Environment Python3**
```
mkdir Honeypots
python3 -m venv Honeypots
cd Honeypots/
source bin/activate
pip3 install honeypots
```
**Langkah 3: Konfigurasi dan Running**
```
(Honeypots) root@VM1:~/Honeypots# honeypots -h
Qeeqbox/honeypots customizable honeypots for monitoring network traffic, bots activities, and username\password
credentials

Arguments:
  --setup               target honeypot E.g. ssh or you can have multiple E.g ssh,http,https
  --list                list all available honeypots
  --kill                kill all honeypots
  --verbose             Print error msgs

Honeypots options:
  --ip                  Override the IP
  --port                Override the Port (Do not use on multiple!)
  --username            Override the username
  --password            Override the password
  --config              Use a config file for honeypots settings
  --options             Extra options

General options:
  --termination-strategy {input,signal}
                        Determines the strategy to terminate by
  --test                Test a honeypot
  --auto                Setup the honeypot with random port

Chameleon:
  --chameleon           reserved for chameleon project
  --sniffer             sniffer - reserved for chameleon project
  --iptables            iptables - reserved for chameleon project
```
Kita akan Running Honeypots sesuai sekenario saja yaitu SSH
```
python3 -m honeypots --setup ssh --username root --password root --port 22 --options interactive
```

### Konfigurasi Rules IPTABLES

**Langkah 1: Instalasi paket iptables**
```
apt-get install iptables
apt-get install iptables-persistent
```
**Langkah 2: Rule Iptables untuk menghubungkan VM2 ke internet**
```
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```
**Langkah 3: Rule iptables untuk mengalihkan/meredirect paket yang masuk ke VM2 (IP 192.168.20.2) dengan tujuan port 22**
```
iptables -t nat -A PREROUTING -p tcp --dport 22 -j DNAT --to 192.168.20.1
```
**Langkah 4: Save Konfigurais iptables secara permanen dengan persistent**
```
iptables-save > /etc/iptables/rules.v4
```
or
```
dpkg-reconfigure iptables-persistent
```

## 2. Konfigurasi Server pada VM2

### Konfigurasi Adapter Network VM2

Apa saja yang diKonfiguras?

1. Web Server 
2. Mail Server 
3. Database Server
4. VPN Server
5. DNS Server
6. Port Knocking
7. Monitoring Server

### 2.1 Web Server

### 2.2 Instalasi dan Konfigurasi Apache2

### 2.3 Konfigurasi CMS Wordpress pada Apache2

### 2.4 Konfigurasi WAF(Web Application Firewall)

### 2.5 Mengamankan Halaman-Halaman Utama Wordpress dengan IP Filtering

### 2.6 Instalasi SSL Certificate pada Web server(443 HTTPS)


### 2.8 Mail Server

### 2.9 Instalasi dan Konfigurasi Postfix dan Dovecot

### 3.0 Konfigurasi Webmail Roundcube

### 3.1 Mengamankan Roundcube secara Umum

### 3.2 Instalasi SSL Certificate pada Protocol Mail Server (SMTPS dan IMAPS)

### 3.3 Database Server

### 3.2 Instalasi dan konfigurasi Mariadb dan Phpmyadmin

### 3.3 Mengamankan MariaDB dan phpmyadmin dengan UFW dan IP FIlTERING

### 3.4 Instalasi dan Konfigurasi OPENVPN

**Langkah 1: Instalasi paket Openvpn & iptables**
```
apt-get install -y openvpn easy-rsa iptables
```
**Langkah 2: Buka direktori easyrsa**
```
cd /usr/share/easy-rsa
```
**Langkah 3: Membuat file PKI**
```
./easyrsa init-pki
```
**Langkah 4: Membuat CA**
```
./easyrsa build-ca

* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.11 19 Sep 2023 (Library: OpenSSL 3.0.11 19 Sep 2023)


Enter New CA Key Passphrase:
Re-Enter New CA Key Passphrase:
Using configuration from /usr/share/easy-rsa/pki/93c96b9d/temp.9d426c95
.+.........+.........+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+...................+..+.+..................+.........+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+.....+.......+...+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...+..+.+........+.......+...+.........+............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...+...............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+...+............+......+.......+..+...+...............+..........+.....+..........+..+..........+..+....+......+...+......+.....+......+.+...+......+..............+......+.........+.+.....................+.....+....+...+.....+.+.....+.......+..+......................+.....+...............+.+........+....+......+.....+......+......+....+.........+..............+...+....+........+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:projectman.my.id

* Notice:

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/usr/share/easy-rsa/pki/ca.crt
```
**Langkah 5: Membuat CA certificate untuk Server**
```
./easyrsa build-server-full server1 nopass

* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.11 19 Sep 2023 (Library: OpenSSL 3.0.11 19 Sep 2023)

.....+....+...+.....+...+................+.....+.+.....+...+....+.....+......+......+.+...+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.....+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......+.....+....+.....................+...+...+............+..+...+.+.........+........+....+........+......+....+..+....+.....+.......+..+......+.......+.........+...+.......................+....+.....+...+.+............+..+......+.......+...+.....+......+.+...+..+.........+.........+....+...+...+..+.........+....+......+...........+...+.+......+..+..........+...+...+.....................+......+..+..........+...+.....+...+.........+.+..+....+...+...........+...............+.+...+..+.+..............+....+......+.....+.+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.+.+...+...........+.+......+...+.....+......+....+............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...+...........+....+...+..+..........+...........+.+......+..+.+.....+....+.....+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+...........+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
* Notice:

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/server1.req
key: /usr/share/easy-rsa/pki/private/server1.key


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 825 days:

subject=
    commonName                = server1


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes

Using configuration from /usr/share/easy-rsa/pki/957ade41/temp.1fbdc4d7
Enter pass phrase for /usr/share/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server1'
Certificate is to be certified until Feb 14 05:10:06 2026 GMT (825 days)

Write out database with 1 new entries
Database updated

* Notice:
Certificate created at: /usr/share/easy-rsa/pki/issued/server1.crt

```
**Langkah 6: Membuat Client Certificate**
```
./easyrsa build-client-full client1 nopass
*** Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.11 19 Sep 2023 (Library: OpenSSL 3.0.11 19 Sep 2023)

..+....+..............+............+...+.+...+..+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...+..+.+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.................+......+.........+...+..+......+...+............+.......+..+................+.....+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...+......+...+...........+.+........................+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+............+..+...............+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+............+.+..............+.+..+.............+.....+......+.........+.........+.+......+...+........+.........+......+.......+...+..+..........+............+......+..+.+.....+.........+......+.+......+...+......+...+...+..+...+....+...+...+........+......+...+.+...........+.........+............+....+........+............+.+..+...+...+....+..+...+...+.......+..+....+.................+.......+..................+..+............+.+...+.....+..........+...........+...+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
* Notice:

Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/client1.req
key: /usr/share/easy-rsa/pki/private/client1.key


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 825 days:

subject=
    commonName                = client1


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes

Using configuration from /usr/share/easy-rsa/pki/993657cf/temp.3261277e
Enter pass phrase for /usr/share/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'client1'
Certificate is to be certified until Feb 14 05:12:07 2026 GMT (825 days)

Write out database with 1 new entries
Database updated

* Notice:
Certificate created at: /usr/share/easy-rsa/pki/issued/client1.crt
**
```
**Langkah 7: Generate DH paramenter**
```
./easyrsa gen-dh

* Notice:
Using Easy-RSA configuration from: /usr/share/easy-rsa/pki/vars

* Notice:
Using SSL: openssl OpenSSL 3.0.11 19 Sep 2023 (Library: OpenSSL 3.0.11 19 Sep 2023)

Generating DH parameters, 2048 bit long safe prime
..........++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*++*

* Notice:

DH parameters of size 2048 created at /usr/share/easy-rsa/pki/dh.pem
```
**Langkah 8: Membuat TLS Auth Key**
```
openvpn --genkey secret ./pki/ta.key
```
**Langkah 9: Copy sample Config Openvpn**
```
cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/
```
**Langkah 10: Copy Certificate yang digenerate ke Direktori utama Openvpn**
```
cp -pR /usr/share/easy-rsa/pki/{issued,private,ca.crt,dh.pem,ta.key} /etc/openvpn/server/
```
**Langkah 11: Buka Direktori utama Openvpn**
```
nano /etc/openvpn/server/server.conf
```
**Langkah 12: Ubahlah isinya seperti**
```
# line 32 :  Menggunakan Port default 1194
port 1194
# line 35 : menggunakan Protocol UDP
;proto tcp
proto udp
# line 53 : ubah ke ke Tap 
dev tap
;dev tun

# line 78 : Arahkan ke direktori spesifik dari Certificate tadi
ca ca.crt
cert issued/server1.crt
key private/server1.key

# line 85 : Aarahkan ke lokasi file dh.pem
dh dh.pem

# line 101 : Subnet yang digunakan VPN saya akan menyesuaikan topologi
server 20.10.20.0 255.255.255.0

# line 142 : Change Routing table ke network local(yang menuju VM3)
push "route 10.10.10.0 255.255.255.0"

# line 231 : Settingan keepalive 
keepalive 10 120

# line 244 : File Auth Key
tls-auth ta.key 0

# line 281 : aktifkan persistent
persist-key
persist-tun

# line 306 : Log level ke 3 (0 - 9,artinya debug level)
verb 3
```
**Langkah 13: Membuat script bridge.sh**
```
nano /etc/openvpn/server/add-bridge.sh
```
isi seperti ini
```
#!/bin/bash

# network interface dari VPN
IF=enp0s8
# interface yang digunakan VPN tunnel
VPNIF=tap0

echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -A FORWARD -i ${VPNIF} -j ACCEPT
iptables -t nat -A POSTROUTING -o ${IF} -j MASQUERADE
```
**Langkah 14: Membuat script remove bridge.sh**
```
nano /etc/openvpn/server/remove-bridge.sh

#!/bin/bash
# sama seperti sebelumnya
IF=enp0s8
# interface VPN
VPNIF=tap0

echo 0 > /proc/sys/net/ipv4/ip_forward
iptables -D FORWARD -i  ${VPNIF} -j ACCEPT
iptables -t nat -D POSTROUTING -o ${IF} -j MASQUERADE
```
**Langkah 15: Ubah hak akses file dengan chmod**
```
chmod 700 /etc/openvpn/server/{add-bridge.sh,remove-bridge.sh}
```
**Langkah 16: Tambahkan Script Tersebut ke Service Openvpn**
```
nano /lib/systemd/system/openvpn-server@.service

# tambahkan ke section [service]
[Service]
.....
.....
ExecStartPost=/etc/openvpn/server/add-bridge.sh
ExecStopPost=/etc/openvpn/server/remove-bridge.sh
```
**Langkah 17: Restart Layanan**
```
systemctl daemon-reload
systemctl enable --now openvpn-server@server
```
### 3.5 Instalasi dan Konfigurasi Port Knocking(Knockd)

### 3.6 Instalasi dan Konfigurasi Monitoring Log Server(Loki,Promtail,Rsyslog)

### 3.7 Visualisasi Log Ke Grafana

### 3.8 Mengamankan Grafana dengan IP filtering dan SSL Cerfificate

### 3.9 Instalasi dan Konfigurasi DNS Server(bind9)


### 4.0 Konfigurasi Server Backup VM3

### Konfigurasi Adapter Network VM3

### 4.1 Instalasi Rsync

### 4.2 Membuat Script Backup Konfigurasi secara Rutin

