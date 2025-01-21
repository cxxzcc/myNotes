

# Introduction

* Confidentiality 
* Integrity 
* Availability


Agile Security Practices
* Inception practices: at the start of the Agile project
	* Risk Assessment
	* Requirements Definition
	* Incident Response 
* Iteration practices: should be performed in every release
	* Threat Assessment
	* Code Review
	* Design Review
* Regular practices: on multiple sprints during the project
	* Dynamic Security Testing
	* Fuzz Testing (misuse)


Top 10 recommendations 
1. Earn or give, but never assume trust 
2. User authentication that cannot be bypassed or tampered 
3. Authorize after you authenticate 
4. Strictly separate data and control instructions 
5. All data must be explicitly validated 
6. Use cryptography correctly 
7. Identify sensitive data and how to handle it 
8. Always consider the users 
9. Understand how external components affect attack surface 
10. Be flexible when considering future objects and actors






# Network vulnerabilities

Physical Layer：Sniffers
Data link layer：MAC flooding  ARP spoofing/poisoning
Network layer：IP spoofing
Transport layer：TCP hijack  SYN flooding
Application layer： Remote Code Execution (RCE)


Kaminsky: Lack of authenticity of DNS messages



# firewall

iptables
* chains：Input, Output, Forward
* tables：NAT, Mangle, Filter
* actions：Drop, Accept, Modify



> HTTP uses TCP on port 80
> ＞1023
> DNS uses UDP on port 53






