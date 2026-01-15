| MITRE Tactic | Technique ID | Technique Name | App / SP Abuse Scenario | Defender XDR / Data Source |
|------------|--------------|---------------|--------------------------|----------------------------|
| Initial Access | T1078 | Valid Accounts | Stolen service principal credential used to authenticate | Entra ID Sign‑In Logs |
| Initial Access | T1528 | Steal Application Access Token | Token abuse after secret/cert compromise | Sign‑In Logs, CloudAppEvents |
| Persistence | T1098 | Account Manipulation | Adding credentials to an existing service principal | Entra ID Audit Logs |
| Persistence | T1136 | Create Account | Creating additional applications/SPs for durability | Entra ID Audit Logs |
| Privilege Escalation | T1098.003 | Add Application Credentials | Adding high‑privilege app secrets/certs | Audit Logs |
| Defense Evasion | T1550 | Use Alternate Authentication Material | App authenticates non‑interactively, bypassing MFA | Sign‑In Logs |
| Credential Access | T1552.004 | Private Keys | Certificate theft from pipeline, disk, or repo | Audit Logs, GitHub exposure |
| Discovery | T1087 | Account Discovery | Enumerating users via Microsoft Graph | Graph Activity Logs |
| Discovery | T1069 | Permission Groups Discovery | Enumerating directory roles via Graph | Graph Activity Logs |
| Collection | T1114 | Email Collection | App accesses Exchange mailboxes | Unified Audit Log |
| Collection | T1213 | Data from Cloud Storage | SharePoint / OneDrive enumeration | UAL, CloudAppEvents |
| Exfiltration | T1537 | Transfer Data to Cloud | Downloading large volumes via Graph | CloudAppEvents |
| Command & Control | T1071.001 | Web Protocols | App calling Graph from new ASN or geo | Sign‑In Logs |
| Impact | T1486 | Data Encrypted for Impact | App used to orchestrate ransomware‑adjacent actions | UAL, Sign‑In Logs |