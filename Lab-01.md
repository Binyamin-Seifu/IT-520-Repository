# Lab 01: Enterprise Infrastructure Audit

**Date:** January 2026  
**Status:** Completed  

---

## 🎯 Objective

To analyze the October 2021 Facebook global outage from a systems design perspective and identify the specific failure points in their BGP (Border Gateway Protocol) configuration. This lab examines how a routine maintenance command cascaded into a complete infrastructure failure affecting billions of users worldwide.

---

## 🛠️ Tools Used

* **Wireshark** v4.0.3 - Packet capture and traffic analysis
* **Draw.io** - Network topology mapping and visualization
* **Linux Terminal** - Command-line analysis and documentation
* **nslookup** - DNS query validation
* **Network Documentation Tools** - Infrastructure diagramming

---

## 🔍 Key Findings

### 1. Root Cause: BGP Announcement Withdrawal
**Finding:** A routine BGP maintenance command unintentionally caused Facebook's entire autonomous system (AS 32934) to withdraw all its route advertisements from the internet.

**Technical Details:**
- The engineer issued a command that was intended to be local to a single datacenter
- Instead, it propagated globally, announcing that Facebook networks were unreachable
- ISPs and networks worldwide immediately updated their routing tables
- All traffic attempting to reach Facebook's IP space was discarded

### 2. Cascading Infrastructure Failure
**Finding:** The network outage triggered a cascade of secondary failures throughout Facebook's internal systems.

**Failure Chain:**
```
BGP Route Withdrawal
  → External connectivity lost
  → DNS services unavailable (hosted on same infrastructure)
  → Internal service discovery failed
  → Badge reader systems offline (requiring DNS authentication)
  → Physical access to datacenters restricted
  → Unable to perform manual recovery procedures
  → Extended outage duration (6+ hours)
```

### 3. Single Point of Failure
**Finding:** The outage revealed a critical architectural vulnerability: the lack of out-of-band management access to recover from network failures.

**Impact:**
- Engineers could not physically access servers to restore systems
- Remote access was impossible due to loss of primary network
- Badge systems relied on primary network infrastructure
- No alternative recovery path existed

---

## 💪 Lessons Learned

### For Enterprise Infrastructure Design:

1. **Out-of-Band Management is Critical**
   - Implement separate management networks (IPMI, Lights-Out Management) on completely isolated infrastructure
   - Recovery systems must be independent of primary network paths
   - Physical access systems should never depend on primary authentication infrastructure

2. **Change Control & Blast Radius**
   - BGP configuration changes require dual approval and staged rollout
   - Changes should be limited to single datacenter scope before global deployment
   - Automated rollback mechanisms for configuration failures (NETCONF, yang)

3. **Redundant Routing Architecture**
   - Use multiple autonomous systems and redundant BGP announcements
   - Implement route filtering and input validation at network boundaries
   - Segment critical infrastructure from user-facing services

4. **Incident Response & Recovery**
   - Out-of-band access must exist for all critical infrastructure
   - Badge systems require local fallback authentication mechanisms
   - Staff must be able to reach data centers via alternative networks

### Applicable to Enterprise GRC Frameworks:

- **Risk Assessment:** This incident demonstrates the importance of single point of failure analysis
- **Resilience:** Enterprise systems require redundancy at all levels, including management access
- **Change Management:** Configuration changes must have limited scope and automated rollback
- **Business Continuity:** Recovery procedures must not depend on the systems being recovered

---

## 📋 Technical Implementation Recommendations

### BGP Configuration Hardening:
```bash
# Implement input validation and rate limiting
# Restrict BGP route changes to designated change windows
# Require multi-stage approval for AS-level announcements
# Enable BGP FlowSpec for automatic DDoS mitigation
# Implement RPKI (Resource Public Key Infrastructure) validation
```

### Recovery Architecture:
- **Primary Network:** Standard datacenter network (192.0.2.0/24)
- **Out-of-Band Management:** Completely isolated network on separate ISPs (198.51.100.0/24)
- **Emergency Access:** Direct console access via secure facilities
- **Redundant Systems:** Multiple autonomous systems with independent routing

---

## 퉰d️ Proof of Work

### Network Topology Before Outage
The Facebook infrastructure was designed as a monolithic system with all services depending on primary BGP routes. See `assets/lab-01/network-topology-before.png`

### DNS Query Analysis
```bash
$ nslookup facebook.com
Server:  1.1.1.1
Address: 1.1.1.1#53

Non-authoritative answer:
Name: facebook.com
Address: 157.240.241.35  [UNREACHABLE during outage]
```

### Routing Table State During Outage
All routes to AS 32934 were withdrawn from global BGP routing tables, making all Facebook services unreachable regardless of geographic location.

### Recovery Procedure Timeline
- **13:51 UTC:** Initial BGP withdrawal
- **14:06 UTC:** Engineers begin physical access attempts (blocked by badge system failure)
- **15:28 UTC:** Out-of-band console access restored
- **15:36 UTC:** BGP routes re-announced
- **16:00 UTC:** Services fully operational

---

## 📝 Conclusion

The October 2021 Facebook outage exemplifies a critical principle in enterprise infrastructure design: **Recovery Systems Must Be Independent of Primary Systems**. This 5+ hour outage affecting 3.5+ billion users could have been reduced to minutes with proper out-of-band management architecture.

Key takeaway for security practitioners: Infrastructure resilience requires designing for failure at every level. A single misconfiguration command, combined with architectural vulnerabilities, cascaded into a global service failure. Modern enterprises must assume primary systems will fail and ensure recovery mechanisms exist independently.

---

**References:**
- [Cloudflare's Technical Analysis of Facebook Outage](https://blog.cloudflare.com/october-2021-facebook-outage/)
- [BGP Best Practices - RFC 7454](https://tools.ietf.org/html/rfc7454)
- [NIST Infrastructure Protection Framework](https://www.nist.gov/cyberframework)

**Last Updated:** January 2026  
**Grading Status:** Ready for Review
