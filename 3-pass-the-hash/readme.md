# ⚔️ 3. Pass The Hash ⚔️
#### ***MITRE ATT&CK - T1550.002***

## The Setup:

Unlike Kerberoasting and AS-REP roasting, Pass The Hash attacks don't rely on specifically misconfigured accounts. It demonstrates
a core weakness in NTLM authentication, Windows will accept a password's hash instead of the password for proof of identity. Once
an attacker has a hash they can utilise lateral movement to log into other machines even ever knowing the actual password.

## The Attack:

From Kali, using an account with privileges I dumped credential hashes directly from the Domain Controller using
`impacket-secretsdump lab.local/Administrator:'... (my actual admin password)'@10.10.10.10`.

<img width="1257" height="656" alt="Screenshot 2026-10-03 132656" src="https://github.com/user-attachments/assets/ab5adab5-13a3-4a91-a7c1-e6e247ef5e23" />

This simulates an attacker who has escalated to admin and is harvesting all the credentials in the domain. From the output I took
the Administrator accounts NTLM hash and used it directly to get access to a remote shell on the windows 11 client and comfirmed
it with `whoami`.

<img width="975" height="192" alt="Screenshot 2026-10-03 134022" src="https://github.com/user-attachments/assets/ccf946e6-95ad-4820-a8ac-9d9b3214040a" />


## The Detection:

Windows logs this as Event ID 4624 to show that an account was successfully logged on, the key differentiator from a normal
login is the LogonType field which would be `3` to indicate a network logon, used by remote tools like wmiexec rather than
someone typing an actual password.

<img width="1207" height="436" alt="Screenshot 2026-10-03 150136" src="https://github.com/user-attachments/assets/180d539d-4b05-4ba0-8f32-84e71ee8b523" />


(The custom Wazuh rule for this (100102) is in wazuh-rules/local_rules.xml)


## How to prevent this:

- By enabling Credential Guard which is a security feature that prevents NTLM hashes from being extracted from memory.
- Restrict high privilege accounts such as the Admin from being used for everyday logons.
- Closely Monitor for LogonType 3 logins using high privilege accounts since this is uncommon for real admin work.

  
