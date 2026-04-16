\## Lab 04 – Enterprise LAN Security Assessment



\### Name: Binyamin Seifu  

\### Course: IT 520  

\### Date: \[Enter Date]



\---



\## Part 1 – Network Architecture \& Topology



This network was designed to simulate an enterprise environment for Winslow Bay Municipal Utility District (WBMUD). The network includes three main segments: IT Operations, Operational Technology (OT), and a Remote Pump Station. A DMZ attacker node was also included to simulate external threats.



\- IT Network: 10.1.10.0/24  

\- OT Network: 192.168.50.0/24  

\- Remote Network: 172.16.5.0/24  

\- Attacker Node: 203.0.113.100  



!\[Network Topology](topology.png)



\---



\## Part 2 – Protocol Security Analysis



\### TCP vs UDP



TCP provides a connection-oriented communication method using a three-way handshake to establish a reliable connection. This allows for better tracking and visibility of communication sessions, making it more secure and easier to monitor. UDP, on the other hand, is connectionless and does not establish a session before sending data. While UDP is faster, it lacks reliability and makes it harder to detect malicious activity due to the absence of connection tracking.



\---



\### HTTP vs HTTPS



HTTP transmits data in plain text, making it vulnerable to interception through man-in-the-middle attacks. HTTPS improves security by encrypting communication using TLS/SSL. This ensures that sensitive data such as login credentials and system commands cannot be easily intercepted or modified by attackers.



\---



\### SSH vs Telnet



Telnet sends data, including credentials, in plain text, making it highly insecure. SSH replaces Telnet by encrypting all communication between devices. This ensures that login credentials and commands cannot be intercepted, making SSH the preferred protocol for secure remote management in enterprise environments.



\---



\## Part 3 – Shellshock Vulnerability Assessment



The Shellshock vulnerability (CVE-2014-6271) is a critical flaw in the GNU Bash shell that allows attackers to execute arbitrary commands through specially crafted environment variables. This vulnerability becomes especially dangerous when a web server uses CGI scripts, as user-supplied input can be passed to Bash without proper validation.



In this lab, the attacker (203.0.113.100) targets the OT web server by sending a malicious HTTP request to the CGI endpoint (/cgi-bin/test.cgi). The attacker crafts a request using a modified User-Agent header containing a Bash function payload. When the Apache web server processes this request, it passes the input to the Bash shell. Due to the vulnerability, Bash incorrectly interprets the payload and executes the injected command, allowing remote code execution.



This attack involves multiple components of the LAMP stack. The Linux operating system provides the Bash shell, which contains the vulnerability. Apache acts as the web server that receives and processes HTTP requests. The CGI script serves as the bridge between the web server and the Bash shell, allowing user input to be executed. Together, these components create an attack path that enables exploitation.



If successfully exploited, this vulnerability could allow an attacker to gain full control of the OT web server. From there, the attacker could access sensitive SCADA systems, manipulate control processes, exfiltrate data, or move laterally to other devices within the network.



To mitigate this vulnerability, systems must be regularly updated to ensure Bash is patched. A Web Application Firewall (WAF) should be implemented to block malicious requests, and network segmentation should be enforced to limit access between IT and OT environments.



Organizations such as the SEI CERT Coordination Center play a critical role by identifying vulnerabilities, publishing advisories like VU#252743, and helping coordinate patch distribution.



\---



\## Part 4 – Incident Response



\### Attack Path Diagram



!\[Attack Path Diagram](attack.png)



\---



\### Root Cause Analysis



| Category | What Went Wrong | Missing Security Control |

|----------|----------------|--------------------------|

| Unpatched Software | Bash was outdated and vulnerable to Shellshock | Patch management and vulnerability scanning |

| Network Segmentation | OT network was accessible from external sources | Network isolation and firewall controls |

| Access Controls | Weak validation allowed malicious input execution | Input validation and least privilege |

| Monitoring \& Detection | No alerts for suspicious activity | IDS/IPS and logging systems |



\---



\### Remediation Plan



Following the identification of the Shellshock vulnerability, immediate containment actions should include isolating the affected web server, blocking malicious IP addresses, and disabling vulnerable services. 



Short-term fixes involve patching the Bash vulnerability, resetting credentials, and scanning systems for malicious activity. 



Long-term improvements should include implementing network segmentation, enforcing least privilege access, and applying defense-in-depth strategies.



Monitoring should be enhanced using intrusion detection systems and centralized logging to detect anomalies early. 



Finally, organizations should improve policies and provide staff training to ensure better incident response and prevention in the future.



\---



\## Part 5 – OSI Model Mapping



| Component | OSI Layer | Explanation |

|----------|----------|-------------|

| Routers | Layer 3 | Handle IP routing between networks |

| Switches | Layer 2 | Forward frames using MAC addresses |

| HTTP/HTTPS | Layer 7 | Application layer communication |

| TCP/UDP | Layer 4 | Transport layer protocols |

| Ping (ICMP) | Layer 3 | Network layer connectivity testing |



\---



\## Reflection



This lab provided a strong understanding of how networking and cybersecurity are interconnected. By building the network in Packet Tracer, I learned how devices communicate across different segments and how routing enables connectivity.



The Shellshock vulnerability demonstrated how a weakness in one component can compromise an entire system. It highlighted the importance of patch management and secure configurations.



Overall, this lab reinforced the importance of defense-in-depth, proper monitoring, and strong network design in protecting critical infrastructure systems.

