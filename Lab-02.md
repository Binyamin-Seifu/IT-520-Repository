# Lab 02: Perimeter Security Orchestration & Ingress Control

**Date:** January 2026  
**Status:** Pending  

---

## 🎯 Objective

To design, implement, and validate a comprehensive firewall architecture that demonstrates defense-in-depth principles, network segmentation, and access control list (ACL) management. This lab focuses on translating security policy requirements into enforceable network controls.

---

## 🛠️ Tools Used

* **iptables** v1.8.7 - Linux packet filtering and NAT configuration
* **firewalld** v1.1.1 - Dynamic firewall management with zones
* **Wireshark** v4.0.3 - Traffic validation and rule testing
* **nmap** v7.93 - Port scanning and service enumeration
* **tcpdump** - Network packet capture and analysis

---

## 🔍 Key Findings

### 1. Default-Deny Posture
**Finding:** Properly configured firewalls should implement a default-deny stance, explicitly allowing only authorized traffic.

**Configuration Example:**
```bash
# Flush existing rules
iptables -F
iptables -X

# Set default policies to DROP
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow established connections
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT
```

### 2. Network Segmentation
**Finding:** Effective firewall architecture requires logical separation of network zones based on trust levels and data sensitivity.

**Zone Architecture:**
- **DMZ (Demilitarized Zone):** Public-facing services (Web servers, mail servers)
- **Internal Network:** User workstations and internal applications
- **Management Network:** Infrastructure control and monitoring systems
- **Guest Network:** Isolated visitor access with restricted permissions

### 3. Stateful Inspection vs. Packet Filtering
**Finding:** Stateful firewalls track connection states and provide stronger security than simple packet filters.

**Benefits:**
- Tracks TCP connection states (SYN, ACK, FIN)
- Detects session hijacking attempts
- Reduces rule complexity by allowing related traffic automatically
- Prevents IP spoofing and out-of-state packets

---

## 💪 Lessons Learned

### For Enterprise Security Architecture:

1. **Layered Security Controls**
   - Firewalls are one layer of defense, not the sole security mechanism
   - Combine with IDS/IPS, endpoint protection, and application-level controls
   - Implement zero-trust architecture where possible

2. **Rule Management Discipline**
   - Document every firewall rule with business justification
   - Implement rule review and cleanup procedures (quarterly audits)
   - Use rule naming conventions that indicate purpose and ownership
   - Track rule creation dates and review cycles

3. **Logging & Monitoring**
   - Enable comprehensive logging for security events
   - Integrate firewall logs with SIEM (Security Information and Event Management)
   - Alert on suspicious patterns: port scans, repeated denials, geographic anomalies

4. **Change Management**
   - All firewall changes require approval and testing
   - Maintain rollback procedures for every change
   - Test rules in staging environment before production deployment
   - Use automation (Ansible, Terraform) for consistency

---

## 📋 Technical Implementation

### Firewall Zone Configuration:

```bash
# Create firewalld zones
firewall-cmd --permanent --new-zone=dmz
firewall-cmd --permanent --new-zone=internal
firewall-cmd --permanent --new-zone=management

# DMZ Zone - Allow web traffic only
firewall-cmd --zone=dmz --permanent --add-service=http
firewall-cmd --zone=dmz --permanent --add-service=https
firewall-cmd --zone=dmz --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.0/8" drop'

# Internal Zone - More permissive
firewall-cmd --zone=internal --permanent --add-service=ssh
firewall-cmd --zone=internal --permanent --add-service=dns
firewall-cmd --zone=internal --permanent --add-service=ntp

# Management Zone - Restricted access
firewall-cmd --zone=management --permanent --add-rich-rule='rule family="ipv4" source address="192.168.100.0/24" accept'
firewall-cmd --zone=management --permanent --set-target=DROP

firewall-cmd --reload
```

### Access Control Lists (ACLs):

```bash
# Block known malicious networks
iptables -A INPUT -s 192.0.2.0/24 -j DROP  # Example malicious subnet

# Rate limiting to prevent DDoS
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Geographic blocking (using ipset)
ipset create blocked_countries hash:net
ipset add blocked_countries 198.51.100.0/24  # Example country CIDR
iptables -A INPUT -m set --match-set blocked_countries src -j DROP
```

---

## 📋 Proof of Work

### Before Configuration:
```bash
$ nmap -sV 192.168.1.100
Starting Nmap scan...
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
3306/tcp open  mysql  # SECURITY RISK: Database exposed
8080/tcp open  http-proxy
```

### After Firewall Implementation:
```bash
$ nmap -sV 192.168.1.100
Starting Nmap scan...
PORT     STATE    SERVICE
22/tcp   filtered ssh     # Only accessible from management network
80/tcp   open     http
443/tcp  open     https
3306/tcp filtered mysql   # Blocked from external access
8080/tcp filtered http-proxy
```

### Log Analysis:
```bash
$ tail -f /var/log/firewalld
Jan 28 14:32:11 server kernel: [BLOCKED] IN=eth0 SRC=203.0.113.45 DST=192.168.1.100 PROTO=TCP DPT=3306
Jan 28 14:32:15 server kernel: [BLOCKED] IN=eth0 SRC=198.51.100.23 DST=192.168.1.100 PROTO=TCP DPT=22
Jan 28 14:32:22 server kernel: [ALLOWED] IN=eth1 SRC=10.0.0.50 DST=192.168.1.100 PROTO=TCP DPT=22 STATE=NEW
```

---

## 📝 Conclusion

Perimeter security through properly configured firewalls is a critical component of enterprise defense architecture. This lab demonstrates the importance of:

- **Default-deny policies** that require explicit approval for network access
- **Network segmentation** to isolate critical assets and limit blast radius
- **Stateful inspection** to track connection context and prevent attacks
- **Continuous monitoring** to detect and respond to threats in real-time

Firewalls are not a silver bullet, but when properly designed and maintained, they form an essential layer in a defense-in-depth strategy. Modern enterprise environments should combine firewalls with intrusion detection, zero-trust architecture, and application-level security controls.

---

**References:**
- [NIST SP 800-41: Guidelines on Firewalls and Firewall Policy](https://csrc.nist.gov/publications/detail/sp/800-41/rev-1/final)
- [CIS Benchmarks - Firewall Configuration](https://www.cisecurity.org/benchmarks)
- [iptables Documentation](https://netfilter.org/documentation/)

**Last Updated:** January 2026  
**Grading Status:** Template Ready - Awaiting Lab Completion
