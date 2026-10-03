# ⚔️ 1. Kerberoasting ⚔️
#### ***MITRE ATT&CK - T1558.003***


## The Setup:

A service account `svc_sql` was made to simulate a SQL server service account which is commonly found in real AD environments 
with an SPN (Service Principle Name) attached to it using `setspn -A MSSQLSvc/dc01.lab.local:1433 LAB\svc_sql`.

Any authenticated domain user can request a Kerberos service ticket for an account with a SPN attached, that ticket is encrypted
using the accounts password hash so an unused account with a weak password can become an exploitable weak point.

## The Attack:

From Kali, using a low privilege compromised domain account (`mayank.gaur`) I requested a service ticket for the account
`svc_sql` using the impacket library.

<img width="1263" height="497" alt="Screenshot 2026-09-28 005335" src="https://github.com/user-attachments/assets/01f29589-1395-49c0-acae-34ad2b15e7b3" />

My first attempt returned an AES Encrypted ticket which is way harder to crack so I had to
downgrade the service account to RC4 using `Set-ADUser -Identity svc_sql -KerberosEncryptionType None` which simulates an older,
 abandoned service account. Re running the attack returned the RC4 Encrypted ticket hash which I ran through the tool John The
 Ripper against the rockyou.txt wordlist.

 <img width="759" height="218" alt="Screenshot 2026-09-28 014805" src="https://github.com/user-attachments/assets/d0576275-e347-4c61-b9c2-f54438155a95" />


 
## The Detection:

Window logs this as Event ID 4769 to show that a Kerberos service ticket was requested but normal logons also generate the same
event ID so the tickets encryption type is the main differentiator - `0x17` is the indicator for kerberoasting since normal 
traffic is `0x12`.

(The custom Wazuh rule for this (100100) is in [wazuh-rules/local_rules.xml](../wazuh-rules/local_rules.xml))



## How to prevent this:

- Replace standard service accounts with Group Managed Service Accounts (gMSAs) since they use 120 character complex passwords
that are also rotated on a regular basis.
- Monitor for RC4 encrypted ticket requests regularly.
- Use long random passwords for any standard service accounts.

