# 🛡️ Active Directory Purple Team Lab 🛡️

A hands-on home lab simulating a small Active Directory environment demonstrating 4 real world Kerberos-based attacks and their corresponding Wazuh SIEM detections for each attack, built with guidance from a guide to practice both offensive and defensive security skills ahead of applying for Cyber Security and Technology degree apprenticeships. 

## Environment

- **Domain Controller:** Windows Server 2022, hosting the Active Directory and DNS - `DC-01`, `10.10.10.10`, domain - `lab.local`

- **Windows 11 Client:** The target machine - `10.10.10.20`

- **Kali Linux:** Attacker machine running various attacks such as Impacket, Hashcat and John the Ripper - `10.10.10.30`

- **Wazuh SIEM:** Central monitoring hub to detect the attacks and collect logs - `10.10.10.40`

- All machines were isolated on a VirtualBox internal network (`purple-lab`).


## Attacks

- [1. Kerberoasting](./1-kerberoasting/) | Service account SPN abuse | T1558.003 |
- [2. AS-REP Roasting](./2-asreproasting/) | Stealing passwords via disabled pre-authentication | T1558.004 |
- [3. Pass The Hash](./3-pass-the-hash/) | Reusing stolen NTLM hashes directly | T1550.002 |
- [4. Golden Ticket](./4-golden-ticket/) | Forging TGTs using krbtgt keys | T1558.001 |


## What I learned:

- An Active Directory is a directory, computers and service accounts all tied together by Kerberos handling authentication.
- How kerberoasting works, that any logged in domain user can request a ticket for a service account with an SPN attached, then the ticket can be cracked offline later.
- That AS-REP Roasting targets accounts with Kerberos pre-authentication disabled so anyone connected to the network can pull that account's ticket data with no credentials or login required.
- Pass-The-Hash is where an attacker steals a user's NTLM password hash and uses it to log into other systems on a network without needing to crack their passwords. It utilises lateral movement to go from low level users to higher targets.
- About Golden Ticket attacks and it's when attackers create a Kerberos TGT (ticket granting ticket) which gives them unrestricted admin access to ever computer, server and domain controller on the entire Active Directory.
- Kerberos rejects any requests if my Kali Linux's clock is not on sync with the domain controller, this caused repeated failures while carrying out the attacks and had me having to fix the clock skew.
- Joining my windows 11 client to the lab.local domain and the Pass-The-Hash attempt failed due to unknown connection timeouts which traced to Windows Defender Firewall blocking traffic between machines that have never communicated before.
- The Domain Controller's Windows server does not let you forge a golden ticket for a username that does not exist in the active directory, it triggered a `KDC_ERR_TGT_REVOKED` error which made me forge a golden ticket for a real account such as the Admin (`Administrator`).
- Event IDs matter alot for detection, 4769 is for service ticket requests, 4768 is for TGT requests and 4624 for logons, each attack leaves a different trail.
- Reusing VM's from previous projects/labs caused alot of issues with wrong static IPs, unknown Wazuh password etc.
- Using the impacket libary and its tools like `GetUserSPNs`, `GetNPUsers`, `secretsdump`, `wmiexec`, `ticketer`, `lookupsid`.
- A domain SID is a unique ID for the whole domain.
- Using MITRE ATT&CK IDs to replicate how security teams communicate and categorise threats in the real world.



