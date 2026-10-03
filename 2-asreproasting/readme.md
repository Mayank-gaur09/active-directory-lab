# ⚔️ 2. AS-REP Roasting ⚔️
#### ***MITRE ATT&CK - T1558.004***


## The Setup:

A second account `svc_backup` was created with the "Do not require Kerberos pre authentication` setting enabled to make it 
deliberately weak so that anyone on the network can request the account's ticket data with no credentials at all.

## The Attack:

From Kali with no domain credentials needed, I listed the target account in a file using `echo "svc_backup" > users.txt `
and ran it through Impacket.

<img width="1245" height="171" alt="Screenshot 2026-10-02 220224" src="https://github.com/user-attachments/assets/6356274e-8ffe-41ba-b797-fd3901f108b5" />


## The Detection:

Windows logs this as Event ID 4768 to show that a Kerberos authentication ticket was requested, the key differentiator from a
normal login is the PreAuthType field which is `0` if no pre-authentication was used, which is unusual for real logins.

<img width="993" height="569" alt="Screenshot 2026-10-02 232751" src="https://github.com/user-attachments/assets/0c696991-100c-4221-9104-9f5d8006d1fa" />

<img width="970" height="496" alt="Screenshot 2026-10-02 232816" src="https://github.com/user-attachments/assets/4ae1883a-55fe-420e-a663-e3c6662d1fa1" />


(The custom Wazuh rule for this (100101) is in [wazuh-rules/local_rules.xml](../wazuh-rules/local_rules.xml))


## Mitigation:

- Audit all domain accounts for "Do not require Kerberos pre-authentication" and disable it unless there’s a valid reason.
- Enforce strong passwords on every account.
- Monitor for Event ID 4768 entries specifically for PreAuthType 0 since it highlights when no pre-authentication was used.
