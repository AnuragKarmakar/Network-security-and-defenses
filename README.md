# Network-security-and-defenses
Providing network security for a conceptualised e-commerce company

# Layered Network Defense for Luna Bags

## Overview

This project implements a comprehensive network defense strategy for Luna Bags, an eco-friendly fashion brand expanding its digital footprint. The solution integrates HTTPS encryption, firewall access control, and an intrusion detection system (IDS) using Snort to secure the company's online operations and internal resources.

## Project Goals

* Secure web transactions using HTTPS.
* Implement strict firewall policies to regulate traffic.
* Deploy Snort for real-time intrusion detection.

## Network Infrastructure

The network is divided into two zones:

* **DMZ (10.9.0.0/24)**:

  * Public-facing Apache Web Server: `10.9.0.5`
  * Attacker/Test Machine: `10.9.0.1`

* **Internal Network (192.168.60.0/24)**:

  * Employee and sensitive systems: `192.168.60.5`, `192.168.60.6`, `192.168.60.7`

* **Router**: Gateway between networks (`10.9.0.11` / `192.168.60.11`), enforcing firewall rules and hosting Snort IDS.

## Technical Implementation

### 1. HTTPS Configuration

* Installed and configured Apache2 and OpenSSL.
* Generated self-signed certificates for HTTPS.

Commands:

```bash
apt update -y
apt install apache2 openssl -y
mkdir -p /etc/apache2/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/apache2/ssl/apache.key \
  -out /etc/apache2/ssl/apache.crt \
  -subj "/C=US/ST=NY/L=NewYork/O=LunaBags/CN=10.9.0.5"
a2enmod ssl
a2ensite default-ssl
nano /etc/apache2/sites-available/default-ssl.conf # Update SSL paths
service apache2 restart
```

### 2. Firewall Setup (iptables)

* Cleared existing firewall settings.
* Configured rules to allow necessary web traffic and protect internal resources.

Commands:

```bash
iptables -F
iptables -X
iptables -t nat -F
iptables -P FORWARD DROP
iptables -A FORWARD -p tcp -d 10.9.0.5 --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp -d 10.9.0.5 --dport 443 -j ACCEPT
iptables -A FORWARD -s 192.168.60.0/24 -d 192.168.60.0/24 -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -p icmp -m limit --limit 1/s --limit-burst 5 -j ACCEPT
iptables -t nat -A POSTROUTING -s 10.9.0.0/24 -o eth0 -j MASQUERADE
iptables -L -v
```

### 3. Snort Intrusion Detection System

* Installed and configured Snort for real-time network monitoring.
* Created custom detection rules for common attack scenarios.

Commands:

```bash
apt update -y
apt install snort -y
nano /etc/snort/snort.conf # Update HOME_NET
```

**Snort Custom Rules (`/etc/snort/rules/local.rules`):**

```snort
alert tcp any any -> 10.9.0.5 80 (msg:"LunaBags: Hidden file access"; content:"GET /."; http_method; sid:1000001;)
alert tcp any any -> 10.9.0.5 80 (msg:"LunaBags: Command Injection"; content:"cmd="; nocase; content:";"; sid:1000002;)
alert tcp any any -> any any (msg:"LunaBags: Nmap Stealth Scan"; flags:S; threshold:type threshold, track by_src, count 5, seconds 3; sid:1000003;)
alert tcp any any -> 10.9.0.5 80 (msg:"LunaBags: Admin Panel Access"; content:"/admin"; http_uri; sid:1000004;)
```

Run Snort:

```bash
snort -A console -q -c /etc/snort/snort.conf -i eth0
```

## Testing & Validation

* Performed HTTPS connection tests using `curl`.
* Verified firewall effectiveness using `iptables -L -v`.
* Simulated attack scenarios including command injection, unauthorized file access, and stealth scans using tools like `curl` and `nmap`.

## Recommendations

* Replace self-signed certificate with a certificate from a trusted authority.
* Regularly update firewall policies.
* Keep Snort rules current and integrate alerts into centralized monitoring.

## Project Reflection

This project enhanced practical skills in network security, firewall management, and IDS tuning, emphasizing the importance of layered defense strategies.

## Contributors

* Anurag Karmakar
* Asim Pathak

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
