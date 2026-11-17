# Network Troubleshooting Commands Cheat Sheet (Linux)

A simple guide to the network commands you need most. Each one has a short explanation, the options you'll use most, and examples.

> Tip: Some commands need `sudo` to show full details (for example the process name in `ss -p`, `nmap -sS`, `tcpdump`).

---

## Table of Contents

1. [Troubleshooting Flow (Where to Start)](#1-troubleshooting-flow-where-to-start)
2. [Interfaces and IP Addresses](#2-interfaces-and-ip-addresses)
3. [Basic Connectivity](#3-basic-connectivity)
4. [Path and Latency](#4-path-and-latency)
5. [DNS](#5-dns)
6. [Routing](#6-routing)
7. [ARP / Neighbors (Layer 2)](#7-arp--neighbors-layer-2)
8. [Open Ports and Connections (Local)](#8-open-ports-and-connections-local)
9. [Testing a Remote Port](#9-testing-a-remote-port)
10. [Port Scanning with nmap](#10-port-scanning-with-nmap)
11. [HTTP / API Testing](#11-http--api-testing)
12. [SSL / TLS Certificates](#12-ssl--tls-certificates)
13. [Packet Capture](#13-packet-capture)
14. [Firewall](#14-firewall)
15. [Bandwidth and Traffic](#15-bandwidth-and-traffic)
16. [Network Config Tools](#16-network-config-tools)
17. [Important Files](#17-important-files)
18. [Common Problems and What to Run](#18-common-problems-and-what-to-run)
19. [Linux vs Windows Commands](#19-linux-vs-windows-commands)
20. [Common Ports to Remember](#20-common-ports-to-remember)

---

## 1. Troubleshooting Flow (Where to Start)

Go from the bottom up. Stop when you find the broken layer.

| Step | Question | Command |
|------|----------|---------|
| 1 | Is my interface UP and has an IP? | `ip addr show` |
| 2 | Do I have a default gateway? | `ip route` |
| 3 | Can I reach the gateway? | `ping <gateway-ip>` |
| 4 | Can I reach the internet by IP? | `ping 8.8.8.8` |
| 5 | Does DNS work? | `dig google.com` |
| 6 | Where does the path break? | `mtr google.com` / `traceroute` |
| 7 | Is the remote port open? | `nc -zv host 443` / `telnet host 443` |
| 8 | Is my service listening? | `ss -tunlp` |
| 9 | Is a firewall blocking? | `iptables -L -n -v` / `ufw status` |
| 10 | What's actually on the wire? | `tcpdump` |

Quick rule:
- `ping 8.8.8.8` works but `ping google.com` fails: **DNS problem**
- `ping` works but app fails: **port / firewall / service problem**
- Nothing works: **interface, IP, or gateway problem**

---

## 2. Interfaces and IP Addresses

### `ip addr show` (modern, recommended)
Shows all network interfaces, their IPs, and status.

```bash
ip addr show          # all interfaces
ip a                  # short form
ip a show eth0        # only eth0
ip -br a              # brief, one line per interface (very handy)
```

Example output (brief):
```
lo     UNKNOWN  127.0.0.1/8
eth0   UP       192.168.1.10/24
```
What to check: interface is `UP`, and it has the correct IP and subnet.

### `ifconfig` (old, still common)
Same idea as `ip addr`. Package: `net-tools`.

```bash
ifconfig              # active interfaces
ifconfig -a           # all, including down ones
ifconfig eth0         # one interface
```
Look at `RX errors` / `TX errors` / `dropped`. High numbers mean a cable, driver, or duplex problem.

### `ip link`
Shows link (Layer 2) status, MAC address, and MTU.

```bash
ip link show
sudo ip link set eth0 up      # bring interface up
sudo ip link set eth0 down    # bring interface down
ip -s link show eth0          # show packet/error stats
```

### Add / remove an IP (temporary, lost on reboot)
```bash
sudo ip addr add 192.168.1.50/24 dev eth0
sudo ip addr del 192.168.1.50/24 dev eth0
```

### `ethtool`
Checks the physical link: speed, duplex, cable detected.

```bash
sudo ethtool eth0
```
Check `Link detected: yes` and `Speed: 1000Mb/s`. `Link detected: no` = cable/port/switch issue.

### `hostname`
```bash
hostname          # machine name
hostname -I       # all IPs of this machine
```

---

## 3. Basic Connectivity

### `ping`
Sends ICMP packets to check if a host is reachable and how fast.

```bash
ping google.com
ping -c 4 8.8.8.8          # send only 4 packets
ping -i 0.2 -c 20 host     # faster interval
ping -s 1472 -M do host    # test MTU (no fragmentation)
```

Reading the result:
- `time=20 ms`: latency (round trip)
- `packet loss 0%`: good. Any loss = unstable network
- `Destination Host Unreachable`: no route or host is down
- `Request timeout`: host down, or ICMP blocked by a firewall

> Note: Many servers/clouds block ping. "No ping reply" does not always mean the host is down. Test the port too.

---

## 4. Path and Latency

### `traceroute`
Shows every router (hop) between you and the destination. Use it to find **where** traffic stops.

```bash
traceroute google.com
traceroute -n google.com      # no DNS lookups (faster)
traceroute -T -p 443 host     # use TCP port 443 (passes firewalls better)
```
`* * *` on a hop means that router didn't reply. If all hops after a point are `* * *`, that's where it breaks.

### `tracepath`
Like traceroute, no root needed, also shows MTU along the path.
```bash
tracepath google.com
```

### `mtr` (best of ping + traceroute)
Live, continuous traceroute with packet loss per hop. Great for intermittent problems.

```bash
mtr google.com
mtr -n google.com                  # no DNS
mtr -r -c 100 google.com           # report mode, 100 cycles (good to share)
mtr -T -P 443 google.com           # TCP mode
```

How to read it:
- Loss only on a middle hop but not at the end: usually fine (router just limits ICMP)
- Loss starts at a hop and **continues to the end**: real problem starts there
- Big jump in `Avg` latency: congestion or long distance link

---

## 5. DNS

### `dig` (best DNS tool)
Asks a DNS server and shows the full answer.

```bash
dig google.com                 # A record (IPv4)
dig google.com +short          # only the IP
dig AAAA google.com            # IPv6
dig MX gmail.com               # mail servers
dig NS google.com              # name servers
dig TXT google.com             # TXT records (SPF, verification)
dig CNAME www.example.com
dig @8.8.8.8 google.com        # ask a specific DNS server
dig -x 8.8.8.8                 # reverse lookup (IP -> name)
dig google.com +trace          # follow resolution from root servers
```
Check `status: NOERROR` (OK), `NXDOMAIN` (name doesn't exist), `SERVFAIL` (DNS server problem). `Query time` shows DNS speed.

Trick: if `dig @8.8.8.8 site.com` works but `dig site.com` doesn't, your **local DNS server** is the problem.

### `nslookup`
Simpler DNS lookup. Works on Linux and Windows.

```bash
nslookup google.com
nslookup google.com 8.8.8.8    # use specific DNS server
nslookup -type=MX gmail.com
nslookup 8.8.8.8               # reverse lookup
```

### `host`
Quickest, short output.
```bash
host google.com
host -t MX gmail.com
```

### Check which DNS server you use
```bash
cat /etc/resolv.conf
resolvectl status              # systemd-resolved systems
```

### Flush DNS cache
```bash
sudo resolvectl flush-caches   # systemd-resolved
```

---

## 6. Routing

### `ip route` (modern)
Shows where packets go. `default via` = your gateway.

```bash
ip route
ip r                                   # short
ip route get 8.8.8.8                   # which route/interface is used for this IP
sudo ip route add 10.0.0.0/24 via 192.168.1.1     # add route
sudo ip route del 10.0.0.0/24                     # delete route
sudo ip route add default via 192.168.1.1         # set default gateway
```

Example:
```
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10
```
No `default via` line = no internet access.

### `route -n` (old)
Same routing table, numeric (no DNS).
```bash
route -n
```
Look at the line with Destination `0.0.0.0` and Flags `UG`: that's the default gateway.

### `netstat -rn`
Also shows the routing table.
```bash
netstat -rn
```

---

## 7. ARP / Neighbors (Layer 2)

ARP maps an IP to a MAC address on the local network. Useful when you can't reach a device in the same subnet.

### `arp`
```bash
arp -n                     # show ARP table, numeric
arp -a                     # all entries
sudo arp -d 192.168.1.1    # delete an entry
```

### `ip neigh` (modern)
```bash
ip neigh
ip neigh show dev eth0
sudo ip neigh flush all    # clear cache
```
States: `REACHABLE` (good), `STALE` (old but OK), `FAILED` / `INCOMPLETE` (device not answering on Layer 2).

Tip: if two machines show the same MAC for different IPs, or one IP has a changing MAC, you may have an **IP conflict**.

### `arping`
Ping on Layer 2. Works even if ICMP is blocked.
```bash
sudo arping -I eth0 192.168.1.1
```

---

## 8. Open Ports and Connections (Local)

Use these to check: "Is my service actually listening?" and "Who is connected?"

### `ss` (modern, fast, recommended)
```bash
ss -tunlp          # most used: all listening TCP/UDP ports with process
ss -tlnp           # only TCP listening
ss -ulnp           # only UDP listening
ss -tanp           # all TCP connections with process
ss -s              # summary stats
ss -tn state established
ss -tnp | grep :443
ss -lnt sport = :8080        # filter by port
```

Flags explained:
| Flag | Meaning |
|------|---------|
| `-t` | TCP |
| `-u` | UDP |
| `-n` | numeric (don't resolve names/ports) |
| `-l` | listening only |
| `-p` | show process (needs sudo) |
| `-a` | all (listening + connected) |

Example:
```
LISTEN 0 128  0.0.0.0:22     0.0.0.0:*   users:(("sshd",pid=900))
LISTEN 0 511  127.0.0.1:3306 0.0.0.0:*   users:(("mysqld",pid=1200))
```
Important:
- `0.0.0.0:22` = open on **all** interfaces (reachable from outside)
- `127.0.0.1:3306` = **localhost only**. Remote clients can't connect. Very common cause of "connection refused".

### `netstat` (old, same idea)
```bash
netstat -antp       # all TCP connections + process
netstat -tunlp      # listening TCP/UDP + process
netstat -an | grep :80
netstat -s          # protocol statistics (errors, retransmits)
```

Connection states to know:
| State | Meaning |
|-------|---------|
| `LISTEN` | waiting for connections |
| `ESTABLISHED` | active connection |
| `TIME_WAIT` | recently closed (normal, but thousands = many short connections) |
| `CLOSE_WAIT` | remote closed, your app didn't. Many = **app bug** |
| `SYN_SENT` | trying to connect, no reply. Usually **firewall** |

Count connections by state:
```bash
ss -tan | awk '{print $1}' | sort | uniq -c
```

### `lsof`
Find which process uses a port.
```bash
sudo lsof -i :8080
sudo lsof -i -P -n | grep LISTEN
```

### `fuser`
```bash
sudo fuser 8080/tcp            # PID using the port
sudo fuser -k 8080/tcp         # kill it
```

---

## 9. Testing a Remote Port

"Can I connect to host X on port Y?" This is the most common app troubleshooting question.

### `telnet`
Opens a raw TCP connection.

```bash
telnet 192.168.1.20 22
telnet db.example.com 5432
```
Results:
- `Connected to ...`: port is **open**. Press `Ctrl + ]` then type `quit` to exit.
- `Connection refused`: host reachable, but **nothing listening** on that port (service down or bound to localhost)
- Hangs / `Connection timed out`: **firewall** dropping traffic, or wrong IP/route
- `Name or service not known`: **DNS** problem

Bonus: test SMTP or HTTP manually:
```bash
telnet example.com 80
GET / HTTP/1.1
Host: example.com
(press Enter twice)
```

### `nc` (netcat) (better than telnet for scripts)
```bash
nc -zv 192.168.1.20 22             # test one port
nc -zv 192.168.1.20 20-25          # test a range
nc -zv -w 3 host 443               # 3 sec timeout
nc -zvu 192.168.1.20 53            # UDP test (less reliable)
```

Test between two machines (check firewall in the middle):
```bash
# Server side: listen on port 9000
nc -l 9000
# Client side: connect and type something
nc server-ip 9000
```

### `curl` for port test
```bash
curl -v telnet://host:5432
```

### Bash only (no tools installed)
```bash
timeout 3 bash -c '</dev/tcp/192.168.1.20/22' && echo OPEN || echo CLOSED
```

---

## 10. Port Scanning with nmap

Scans a host or network to find open ports and services. **Only scan systems you own or have permission to scan.**

```bash
nmap 192.168.1.20                  # top 1000 TCP ports
nmap -p 22,80,443 192.168.1.20     # specific ports
nmap -p 1-65535 192.168.1.20       # all ports (or -p-)
nmap -F 192.168.1.20               # fast, top 100 ports
nmap -sV 192.168.1.20              # detect service + version
nmap -O 192.168.1.20               # detect OS (sudo)
nmap -A 192.168.1.20               # OS, versions, scripts, traceroute
nmap -sn 192.168.1.0/24            # ping sweep: which hosts are alive
sudo nmap -sS 192.168.1.20         # SYN (stealth) scan, fast
sudo nmap -sU -p 53,161 host       # UDP scan
nmap -Pn 192.168.1.20              # skip ping (host blocks ICMP)
nmap --script ssl-enum-ciphers -p 443 host   # check TLS ciphers
```

Port states:
| State | Meaning |
|-------|---------|
| `open` | service is listening |
| `closed` | reachable, nothing listening |
| `filtered` | firewall blocking, no reply |

---

## 11. HTTP / API Testing

### `curl`
```bash
curl https://example.com
curl -I https://example.com            # headers only (status code)
curl -v https://example.com            # verbose: DNS, TCP, TLS, headers
curl -k https://self-signed.local      # ignore SSL errors (testing only)
curl -L http://example.com             # follow redirects
curl -o /dev/null -s -w "%{http_code}\n" https://example.com   # just status code
curl --resolve example.com:443:1.2.3.4 https://example.com     # force IP (test before DNS change)
curl -x http://proxy:3128 https://example.com                  # via proxy
curl -X POST -H "Content-Type: application/json" -d '{"a":1}' https://api.example.com
```

Measure where time is spent:
```bash
curl -o /dev/null -s -w "dns:%{time_namelookup} connect:%{time_connect} tls:%{time_appconnect} total:%{time_total}\n" https://example.com
```

### `wget`
```bash
wget https://example.com/file.zip
wget --spider https://example.com      # check if URL exists, no download
```

---

## 12. SSL / TLS Certificates

### `openssl s_client`
```bash
openssl s_client -connect example.com:443 -servername example.com
# Show certificate expiry dates
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates
# Show issuer and subject
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -issuer -subject
```
Common errors: expired certificate, wrong hostname, missing intermediate certificate.

---

## 13. Packet Capture

### `tcpdump`
Shows the real packets. Use it when everything "looks right" but still fails.

```bash
sudo tcpdump -i eth0                       # all traffic on eth0
sudo tcpdump -i any port 443               # only port 443
sudo tcpdump -i eth0 host 192.168.1.20     # only this host
sudo tcpdump -i eth0 -nn port 53           # DNS traffic, numeric
sudo tcpdump -i eth0 icmp                  # only ping
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'   # SYN packets
sudo tcpdump -i eth0 -w capture.pcap       # save to file (open in Wireshark)
sudo tcpdump -r capture.pcap               # read file
```

What to look for:
- You see `SYN` going out but no `SYN-ACK` back: **firewall or remote not listening**
- You see `RST` coming back: **port closed / connection rejected**
- No packets at all: traffic isn't leaving (routing/interface issue)

### Wireshark / `tshark`
GUI (Wireshark) or CLI (`tshark`) for deep packet analysis of `.pcap` files.

---

## 14. Firewall

### `iptables`
```bash
sudo iptables -L -n -v            # list rules with counters
sudo iptables -L -n --line-numbers
sudo iptables -t nat -L -n -v     # NAT rules (port forwarding)
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT   # allow port
```
Watch the packet counters: if a `DROP` rule counter increases while you test, that rule is blocking you.

### `nft` (nftables, newer)
```bash
sudo nft list ruleset
```

### `ufw` (Ubuntu)
```bash
sudo ufw status verbose
sudo ufw allow 22/tcp
sudo ufw allow from 10.0.0.0/24 to any port 5432
sudo ufw deny 23
```

### `firewalld` (RHEL / CentOS / Rocky)
```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

> Cloud reminder: on AWS / Azure / GCP also check **Security Groups / NSGs / VPC firewall rules**. The OS firewall can be open while the cloud firewall blocks.

---

## 15. Bandwidth and Traffic

### `iperf3`
Measures real throughput between two machines.
```bash
iperf3 -s                       # on server
iperf3 -c server-ip             # on client (upload test)
iperf3 -c server-ip -R          # download test
iperf3 -c server-ip -u -b 100M  # UDP test, shows jitter/loss
```

### `iftop` / `nload` / `bmon`
Live bandwidth per connection / per interface.
```bash
sudo iftop -i eth0
nload eth0
```

### `nethogs`
Bandwidth per **process**.
```bash
sudo nethogs eth0
```

### `vnstat`
Traffic history (daily/monthly).
```bash
vnstat
```

---

## 16. Network Config Tools

### `nmcli` (NetworkManager)
```bash
nmcli device status
nmcli connection show
nmcli device wifi list
sudo nmcli connection up "MyConnection"
sudo nmcli connection reload
```

### Restart networking
```bash
sudo systemctl restart NetworkManager     # desktop / RHEL
sudo systemctl restart systemd-networkd   # systemd-networkd systems
sudo netplan apply                        # Ubuntu servers (netplan)
```

### DHCP
```bash
sudo dhclient -r eth0     # release IP
sudo dhclient eth0        # get new IP
```

### `whois`
Who owns a domain or IP.
```bash
whois example.com
whois 8.8.8.8
```

---

## 17. Important Files

| File | Purpose |
|------|---------|
| `/etc/hosts` | Local name to IP mapping (checked before DNS) |
| `/etc/resolv.conf` | DNS servers in use |
| `/etc/nsswitch.conf` | Order of lookup (files, dns) |
| `/etc/netplan/*.yaml` | Ubuntu network config |
| `/etc/sysconfig/network-scripts/` | Old RHEL/CentOS config |
| `/etc/NetworkManager/` | NetworkManager config |
| `/etc/services` | Port number to service name list |

Tip: a wrong entry in `/etc/hosts` is a classic "DNS works with dig but app goes to the wrong IP" issue. `dig` ignores `/etc/hosts`; use `getent hosts name` to see what apps actually resolve.

```bash
getent hosts example.com
```

---

## 18. Common Problems and What to Run

### "No internet at all"
```bash
ip -br a                  # has IP? interface UP?
ip route                  # has default gateway?
ping -c 3 <gateway>       # gateway reachable?
ping -c 3 8.8.8.8         # internet by IP?
dig google.com            # DNS?
```

### "Website / domain not opening"
```bash
dig site.com +short
dig @8.8.8.8 site.com +short     # compare with public DNS
getent hosts site.com            # check /etc/hosts effect
curl -Iv https://site.com
```

### "Can't connect to my server/app on port X"
```bash
# On the server:
ss -tunlp | grep :X              # listening? on 0.0.0.0 or 127.0.0.1?
sudo iptables -L -n -v           # firewall?
sudo ufw status
# From the client:
nc -zv server-ip X
telnet server-ip X
nmap -Pn -p X server-ip
```
- `refused` = service not running or bound to localhost
- `timeout` = firewall / security group / routing

### "Network is slow"
```bash
mtr -r -c 100 destination        # loss / latency per hop
ping -c 50 destination           # jitter and loss
iperf3 -c server-ip              # real bandwidth
ip -s link show eth0             # interface errors
sudo ethtool eth0                # speed / duplex
sudo iftop -i eth0               # who uses bandwidth
```

### "Port already in use" error
```bash
sudo ss -tlnp | grep :8080
sudo lsof -i :8080
sudo kill <PID>
```

### "Can't reach a device in the same subnet"
```bash
ping 192.168.1.30
ip neigh                          # MAC learned? FAILED?
sudo arping -I eth0 192.168.1.30
```

### "Large files / some sites hang, small requests work" (MTU issue)
```bash
ping -M do -s 1472 destination   # reduce size until it works
tracepath destination            # shows path MTU
```

### "Too many connections / app hanging"
```bash
ss -s
ss -tan | awk '{print $1}' | sort | uniq -c   # many CLOSE_WAIT = app bug
```

---

## 19. Linux vs Windows Commands

| Task | Linux | Windows |
|------|-------|---------|
| Show IP | `ip a` / `ifconfig` | `ipconfig /all` |
| Ping | `ping -c 4 host` | `ping host` |
| Trace route | `traceroute host` | `tracert host` |
| Trace + loss | `mtr host` | `pathping host` |
| DNS lookup | `dig` / `nslookup` | `nslookup` / `Resolve-DnsName` |
| Flush DNS | `resolvectl flush-caches` | `ipconfig /flushdns` |
| Routing table | `ip route` / `route -n` | `route print` |
| ARP table | `ip neigh` / `arp -n` | `arp -a` |
| Open ports | `ss -tunlp` | `netstat -ano` |
| Test port | `nc -zv host port` | `Test-NetConnection host -Port 443` |
| Renew DHCP | `dhclient` | `ipconfig /release` then `/renew` |

---

## 20. Common Ports to Remember

| Port | Service |
|------|---------|
| 20/21 | FTP |
| 22 | SSH / SFTP |
| 23 | Telnet |
| 25 / 587 / 465 | SMTP (mail) |
| 53 | DNS (UDP + TCP) |
| 67/68 | DHCP |
| 80 | HTTP |
| 110 / 995 | POP3 |
| 123 | NTP |
| 143 / 993 | IMAP |
| 161 | SNMP |
| 389 / 636 | LDAP / LDAPS |
| 443 | HTTPS |
| 445 | SMB |
| 1433 | MS SQL Server |
| 1521 | Oracle DB |
| 2049 | NFS |
| 3306 | MySQL / MariaDB |
| 3389 | RDP |
| 5432 | PostgreSQL |
| 5672 | RabbitMQ |
| 6379 | Redis |
| 8080 / 8443 | Alternate HTTP / HTTPS |
| 9092 | Kafka |
| 9200 | Elasticsearch |
| 27017 | MongoDB |

---

## Install Missing Tools

```bash
# Ubuntu / Debian
sudo apt install net-tools iproute2 dnsutils traceroute mtr nmap netcat-openbsd telnet tcpdump curl iperf3 iftop ethtool lsof

# RHEL / CentOS / Rocky
sudo dnf install net-tools iproute bind-utils traceroute mtr nmap nmap-ncat telnet tcpdump curl iperf3 iftop ethtool lsof
```

---

**Golden rule:** Check from the bottom up: **Cable/Link, then IP, then Gateway, then DNS, then Port, then Firewall, then Application.**
