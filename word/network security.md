

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

Physical Layer：Sniffers -require promiscuous mode to work as intended
Data link layer：MAC flooding  ARP spoofing/poisoning
Network layer：IP spoofing
Transport layer：TCP hijack  SYN flooding
Application layer： Remote Code Execution (RCE)


Kaminsky: Lack of authenticity of DNS messages
* prevent
	* Change the source port in every request
	* Use DNSSEC

ARP Redirect Change the ARP table of a remote machine

# firewall

iptables : Tables contain Chains,Chains contain Rules
* tables：NAT, Mangle, Filter
* chains：Input, Output, Forward
* actions：Drop, Accept, Modify



> HTTP uses TCP on port 80
> ＞1023
> DNS uses UDP on port 53






DMZ (Demilitarized Zone) 
if public server is compromised, the other machines are still protected

NIDS
traffic that can be observed
detect long running attacks, detect suspicious flows,N

HIDS
observe traffic received at server, observe disk access anomalies, observe CPU and RAM usage anomalies,







# Cryptography


MAC - Integrity + Authentication



Diffie-Hellman algorithm achieves PFS (Perfect Forward Secrecy)
* If a and b are ephemeral
* A shared secret







# Secret key management

Replay Attacks - Sequence Numbers and Timestamps




# Public key management

PKI (Public Key Infrastructure)
* Secure creation of asymmetric key pairs
* Creation and distribution of public key certificates
* Definition and usage of certification chains
* Update, publication and query of certificate revocation lists


CRL (Certificate Revocation Lists)
OCSP (Online Certificate Status Protocol )






# Authentication
Kerberos - the user uses his password less times
* Authentication Service (AS) 
	* Allows client to login in Kerberos
	* Each client has a shared key with AS derived from password
* Ticket Granting Service (TGS)
	* Provides time-limited credentials (tickets) for several services/servers

**Kerberos ticket** serves both **authentication** and **authorization** purposes


# Wireless security
WEP protocol
* Authenticating with the same keystream
* The AP is not authenticated
* Integrity control is weak


WPA/WPA2
* RADIUS
* large networks


802.11i = WPA2 = WPA + AES-CCMP
* AES-CCMP Confidentiality authenticity


802.1X and EAP



# IPSec


IPsec（Internet Protocol Security）
* Authentication Header (AH)
	* Authentication. Integrity. Freshness
* Encapsulating Security Payload (ESP)
	* Confidentiality.Authentication. Integrity. Freshness

Sequence number in header

# TLS SSH

protect confidentiality and integrity of the payloads




Cipher Suite - A choice of encryption and integrity algorithms



TOFU
* It is secure only if there was no attack on the first connection to the server


