## ⚔️ 4. Golden Ticket ⚔️
#### ***MITRE ATT&CK - T1558.001***

## The Setup:

This attack targets the krbtgt account which signs every single Kerberos ticket issued in the whole domain. It's hash or the
TGT (Ticket Granting Ticket) and is the most powerful key. If an attacker gets access to this hash, they can forge a ticket
and impersonate any account without ever cracking a password or any credentials.

I got this hash from the Pass-The-Hash attack output.


## The Attack:

Using the krbtgt hash and my domains SID which I got using:

<img width="684" height="181" alt="Screenshot 2026-10-03 154529" src="https://github.com/user-attachments/assets/83e39821-6a29-4d7f-8cb6-f3c182fa986a" />


I attempted to forge a ticket for a made up account (`admin_fake`) which did not end up working so I had to switch to using
a legitimate, existing account such as the Administrator:

<img width="1173" height="294" alt="Screenshot 2026-10-03 155353" src="https://github.com/user-attachments/assets/73d72620-1e38-4ecd-b0e0-8955b4ec780c" />


I then used the forged ticket to get access to a shell on the DC itself and confirmed it using `whoami`:

<img width="613" height="189" alt="Screenshot 2026-10-03 162443" src="https://github.com/user-attachments/assets/b9ef87ab-0e37-4cf8-bade-7ec50c9c324f" />


## The Detection:

Windows logs this as Event ID 4769 which is the same as Kerberoasting and a forged ticket looks identical to a real one in the
SIEM logs and no field gives it away, the wazuh rule just flags any ticket request made for the admin account.

<img width="921" height="489" alt="Screenshot 2026-10-03 173443" src="https://github.com/user-attachments/assets/8de7d962-2ca8-49c8-8231-238bab5f6b85" />


(The custom Wazuh rule for this (100103) is in [wazuh-rules](/wazuh-rules/local_rules.xml))


## How to prevent this:

- Rotate the krbtgt password regularly which invalidates any forged tickets.
- Keep domain controller updated and download released patches.
