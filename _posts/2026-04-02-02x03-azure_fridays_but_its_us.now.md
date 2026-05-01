---
layout: post
title:  "02x03-azure_fridays_but_its_us.now"
date:   2026-04-02 21:00:00 +1100
categories: episodes
cover: 'assets/images/cover/cover-02x03.jpg'
navigation: True
subclass: 'post'
logo:
---
In this episode Koos discusses the Odido data breach in The Netherlands by hacker group Shinyhunters, one of the largest public data leaks in Dutch history. Touching on vishing, misconfigurations, and the importance of blocking Device Code Flow. Chris was inspired by a fellow MVP to take a look at common AD security mistakes and provides some detail on how to look for these in your environment.

<iframe src="https://player.rss.com/df3ndr/2682511?theme=dark&v=2" width="100%" height="202px" title="02x03-azure_fridays_but_its_us.now" frameBorder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen scrolling="no"><a href="https://rss.com/podcasts/df3ndr/2682511/">02x03-azure_fridays_but_its_us.now | RSS.com</a></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/_ExMOMOAuiA?si=pJSTqFFYY71viWfu" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## The Odido Breach

In February 2026, hackersgroup Shinyhunters breached Dutch telecom provider Odido (formerly known as T-Mobile, sold off in 2022), stealing approximately 90 GB of customer data covering 6.5 million people. Which is 1/3rd of the population. The data included passport numbers, bank account numbers, addresses, and sensitive helpdesk notes. The group demanded 1 million euros in ransom, which Odido refused to pay. The data was eventually published in full on March 1st.
It became the biggest public data breach in The Netherlands.

### How it started: Vishing

The attack began on February 3rd with a simple phone call. A Shinyhunters operative called Odido's customer helpdesk, impersonating an IT employee. The caller spoke fluent Dutch and persuaded a helpdesk employee to "log in" to what appeared to be a legitimate Odido website.

According to campaigns documented by BleepingComputer and Mandiant in early 2026, Shinyhunters has been combining *vishing* with abuse of the OAuth 2.0 Device Authorization Grant, also known as Device Code Flow. Vishing, a conjunction of "voice" and "phishing", is essentially phone-based social engineering. In these attacks, the attacker first generates a legitimate device code using a real Microsoft OAuth client ID. They then call the target, impersonate IT support, and instruct them to navigate to `microsoft.com/devicelogin` and enter the code. The victim lands on a legitimate Microsoft sign-in page, authenticates with their credentials, including MFA. And because everything runs through Microsoft’s authentication platform, the user won't notice anything suspicious going on. They approve their MFA and complete their sign-in. And the attacker receives a PRT (Primary Refresh Token) granting persistent access to the victim’s Microsoft Entra account and, through SSO, to every connected SaaS application.

In [episode s01e11](https://df3ndr.io/episodes/2025/10/01/01x11_trust_me_im_a_keyboard_hid.html) I visited the Social Engineering Village at DEFCON where I watched a live *vishing* contest. Participants used phone calls and social engineering tactics to trick real companies into revealing sensitive information. I was both pleasantly surprised that some employees showed signs of security awareness training, and alarmed at how skilled social engineers could still extract valuable information with enough flair and persuasion. The Odido breach is a textbook example of this playing out in the real world with devastating consequences.

There is no phishing page to detect, no malicious URL to block, and the victim completes MFA successfully on Microsoft's own infrastructure. And crucially: **even phishing-resistant MFA like passkeys does not help here**, because the user is authenticating on the real Microsoft domain. The passkey works exactly as designed, the problem is that the user is unknowingly authorizing someone else's session.

These attacks show why blocking Device Code Flow is not optional.

### What could've prevented this: several key misconfigurations

Once Shinyhunters had the helpdesk employee's credentials and tokens, they logged in as that employee and accessed Odido's customer management system (Salesforce). Several critical misconfigurations made this breach far worse than it perhaps needed to be. But in today's episode I want to focus on Device Code Flow specifically. If you're interested to know more details about the Odido attack specifically, I urge you to look at a great writeup by Maarten Goet which also contains loads of links to other external sources with details.

#### Links with details about the Odido hack

* [Shinyhunters & Odido - What Went Wrong and How to Defend (Maarten Goet)](https://maartengoet.org/archive/shinyhunters-odido-what-went-wrong-and-how-to-defend/)
* [ShinyHunters extortion gang claims Odido breach affecting millions (BleepingComputer)](https://www.bleepingcomputer.com/news/security/shinyhunters-extortion-gang-claims-odido-breach-affecting-millions/)
* [Why Odido's crisis is only getting bigger (Cybernews)](https://cybernews.com/security/why-odidos-crisis-only-getting-bigger-criminal-investigation/)
* [Odido: ShinyHunters leaks data (The Register)](https://www.theregister.com/2026/02/27/odido_shinyhunters_leaks/)
* [Hacker group ShinyHunters posts Odido customer data (NL Times)](https://nltimes.nl/2026/02/28/hacker-group-shinyhunters-posts-odido-customer-data-online-third-consecutive-day)
* [Voice Phishing for Data Extortion (Google Cloud Threat Intelligence)](https://cloud.google.com/blog/topics/threat-intelligence/voice-phishing-data-extortion)
* [Defending against ShinyHunters (Picus Security)](https://www.picussecurity.com/threat-database/defending-against-shinyhunters-tactics-and-breaches)

### Device Code Flow

One of the defensive recommendations from this breach (and from Microsoft itself) is to block Device Code Flow. This is a topic that deserves a deep dive because it is increasingly being abused by threat actors and I still see this enabled all of the time at my customers.

#### What is Device Code Flow?

Device Code Flow is an OAuth 2.0 authentication flow designed for devices that lack a local browser or have limited input capabilities. I think most of us have seen this being used in PowerShell scripts for example. But also devices like smart TVs, IoT devices, Microsoft Teams Room devices, printers, or digital signage.

Here's how the flow works under the hood:

**Step 1 - Device Authorization Request**
The device (or application) sends a `POST` request to the Microsoft Entra ID device authorization endpoint:

```
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/devicecode
Content-Type: application/x-www-form-urlencoded

client_id=<application-client-id>&scope=openid profile offline_access https://graph.microsoft.com/.default
```

The `client_id` identifies the application registration in Entra ID. This is where it gets interesting from an attack perspective: attackers can use well-known first-party Microsoft client IDs (e.g. the Microsoft Azure CLI or Microsoft Office app IDs) so the resulting consent screen looks completely trusted and familiar to the user.

**Step 2 - Device Authorization Response**
Entra ID responds with a JSON payload containing:

```json
{
  "device_code": "GMMhmHCXhWEzkobq...{long opaque string}",
  "user_code": "BRWC-MJNK",
  "verification_uri": "https://microsoft.com/devicelogin",
  "expires_in": 900,
  "interval": 5,
  "message": "To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BRWC-MJNK to authenticate."
}
```

* `device_code` - A long opaque token that the device keeps privately and uses to poll for completion.
* `user_code` - A short, human-readable code (e.g. `BRWC-MJNK`) that the user must enter. This is the code an attacker would hand to their victim.
* `verification_uri` - Always points to Microsoft's legitimate domain.
* `expires_in` - The device code is valid for 15 minutes (900 seconds) by default. If the user doesn't complete authentication within this window, the code expires.
* `interval` - The polling interval in seconds. The device should wait at least this many seconds between each poll.

**Step 3 - User authenticates on a separate device**
The user opens a browser on any device (phone, laptop, etc.), navigates to `https://microsoft.com/devicelogin`, and enters the user code. Microsoft's authentication platform takes over from here: the user signs in with their credentials, completes MFA if required, and sees a consent screen showing which application is requesting access and what permissions (scopes) it needs.

At this point, the user has no way to know *which physical device* initiated the request. They see a legitimate Microsoft page, a legitimate app name, and legitimate permission scopes. Nothing indicates that the device code was generated by an attacker.

**Step 4 - Device polls the token endpoint**
While the user is authenticating, the device continuously polls the token endpoint at the configured interval:

```
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:device_code
&client_id=<application-client-id>
&device_code=GMMhmHCXhWEzkobq...
```

During polling, the response contains `"error": "authorization_pending"` until the user completes or denies authentication, or the code expires.

**Step 5 - Token issuance**
Once the user completes authentication, the next poll returns the token payload:

```json
{
  "token_type": "Bearer",
  "scope": "openid profile offline_access https://graph.microsoft.com/.default",
  "expires_in": 4799,
  "access_token": "eyJ0eXAi...{JWT}",
  "refresh_token": "0.ARoAv4j5cvGGr0G...{opaque string}",
  "id_token": "eyJ0eXAi...{JWT}"
}
```

* `access_token` - A short-lived JWT (typically ~60-90 minutes) granting access to the requested resources.
* `refresh_token` - A long-lived token (up to 90 days, or indefinite with continuous access evaluation) that can be exchanged for new access tokens without user interaction. This is the real prize for an attacker: it provides persistent access long after the initial authentication.
* `id_token` - Contains claims about the authenticated user.

The critical design characteristic is that tokens are issued to whatever device holds the `device_code`, which is the device that initiated the flow, not the device where the user authenticated. This decoupling is what makes the flow useful for input-constrained devices, and it's exactly what makes it so dangerous when abused.

#### What Microsoft is doing to prevent Device Code Flow abuse?

Microsoft has taken several concrete steps as part of their **Secure Future Initiative**:

**1. Microsoft-Managed Conditional Access Policy (February 2025)**
Microsoft announced the rollout of a Microsoft-managed Conditional Access policy that **blocks Device Code Flow by default**. Key details:

* The policy is automatically created in tenants and starts in **report-only mode**.
* It targets organizations that **have not used Device Code Flow in the past 25 days**, ensuring it doesn't break existing legitimate use.
* After at least **45 days** in report-only mode, Microsoft automatically enables the policy (moves it to "On") unless the administrator explicitly opts out.
* Administrators can customize exclusions (e.g. specific users or break-glass accounts) but cannot delete the policy.
* This policy appears alongside other Microsoft-managed policies like "Block legacy authentication" and "Multifactor authentication for all users."

**2. Authentication Flows condition in Conditional Access**

Microsoft added "Authentication Flows" as a dedicated condition in Conditional Access policies, allowing administrators to explicitly target and block Device Code Flow. This enables granular control, for example:

* Block Device Code Flow for all users except a specific group that uses Teams Room devices.
* Block Device Code Flow everywhere except for specific network locations.
* Block Device Code Flow for all resources except specific applications.

**3. Protocol Tracking**
Microsoft implemented "protocol tracking" for sessions that use Device Code Flow. Once a session is established through Device Code Flow, that session is marked as "protocol tracked" and remains subject to authentication flow policy enforcement even through subsequent token refreshes. This prevents attackers from using Device Code Flow to get an initial token and then pivoting to other resources.

#### What you should do

1. **Check your sign-in logs** - Filter for Device Code Flow in the authentication protocol filter to understand if and how it's being used in your organization. You can run the following KQL query in Microsoft Sentinel or Log Analytics to get an overview:

```kusto
SigninLogs
| where AuthenticationProtocol == "deviceCode"
| project
    TimeGenerated,
    UserPrincipalName,
    AppDisplayName,
    ResourceDisplayName,
    IPAddress,
    Location = LocationDetails.city,
    DeviceDetail.operatingSystem,
    Status.errorCode,
    ConditionalAccessStatus,
    RiskLevelDuringSignIn,
    OriginalTransferMethod
| sort by TimeGenerated desc
```

   If you see results, investigate each entry. Pay attention to which applications (`AppDisplayName`) are using Device Code Flow and whether the users and locations are expected. Any sign-in from an unexpected user, app, or location is a red flag.

2. **Don't wait for the Microsoft-managed policy** - Create your own Conditional Access policy to block Device Code Flow now:

   * Assignments: All users (exclude break-glass accounts)
   * Target resources: All resources
   * Conditions: Authentication Flows -> Device code flow
   * Grant: Block access

3. **If you have legitimate use cases**, create a narrow exception. Only allow Device Code Flow for documented use cases (e.g. specific users, specific device platforms, specific network locations).

4. **Start in report-only mode** if you're unsure about impact, but move to enforcement quickly.

### Links

* [Block authentication flows with Conditional Access policy (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/conditional-access/policy-block-authentication-flows)
* [Authentication flows as a condition in Conditional Access (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-authentication-flows)
* [Microsoft-Managed Conditional Access Policies (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/conditional-access/managed-policies)
* [New Microsoft-managed policies to raise your identity security posture (Microsoft Tech Community)](https://techcommunity.microsoft.com/blog/microsoft-entra-blog/new-microsoft-managed-policies-to-raise-your-identity-security-posture/4286758)
* [Storm-2372 conducts device code phishing campaign (Microsoft Security Blog)](https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/)
* [How to protect against Device Code Flow abuse (Jeffrey Appel)](https://jeffreyappel.nl/how-to-protect-against-device-code-flow-abuse-storm-2372-attacks-and-block-the-authentication-flow/)
* [Blocking Device Code Flow in Microsoft Entra ID (Sentrium Security)](https://www.sentrium.co.uk/labs/blocking-device-code-flow-in-microsoft-entra-id)
* [Device Code Flow Phishing - Access granted (Proofpoint)](https://www.proofpoint.com/us/blog/threat-insight/access-granted-phishing-device-code-authorization-account-takeover)

## Common AD Security Mistakes

Koos' Odido Breach story is a great case in point that demonstrates how breaches don’t always require fancy exploits. More often than not, rely on common misconfigurations, mistakes or legacy configurations to get a foot in the door.

MVP [Spencer Alessi](https://spenceralessi.com/post/common-active-directory-security-mistakes-attackers-count-on) recently published a great post called 'Common Active Directory security mistakes attackers count on' which really resonated with me. I have spent countless hours working with organizations helping them correct most or all of these.

I thought it would be a good idea to review his list of mistakes for those in the back who may not have seen the post. I think his list is an great place to start if you are looking to inprove the security posture of your AD environment.

1) Weak or reused password
Weak and reused passwords remain one of the most exploited vectors in Active Directory environments. Attackers use credential stuffing, password spraying, and hash cracking to compromise accounts.

* Run a password audit using DSInternals or similar tools to identify accounts with weak or common passwords
* Cross-reference hashed credentials against the HaveIBeenPwned NTLM hash list to find breached passwords currently in use
* Check the Default Domain Password Policy and any Fine-Grained Password Policies for minimum length, complexity, and lockout settings
* Identify accounts with 'Password Never Expires' set — these are common targets since they are never forced to rotate
* Look for service accounts or admin accounts sharing passwords across systems

```powershell
# Retrieves the default password policy applied to all domain users (min length, complexity, lockout, history)
Get-ADDefaultDomainPasswordPolicy
# Lists all Fine-Grained Password Policies (PSOs) — these override the default policy for specific users/groups
Get-ADFineGrainedPasswordPolicy -Filter *
# Finds all user accounts with 'Password Never Expires' enabled — these accounts never get forced to rotate
Get-ADUser -Filter {PasswordNeverExpires -eq $true} | Select Name,SamAccountName
```

2) Assigning overly broad permissions on OU, Security Groups, and file shares
Misconfigured ACLs on AD objects are a primary enabler of privilege escalation. Attackers with write access to an OU, group, or user object can add members to privileged groups, reset passwords, or modify GPO links without needing Domain Admin credentials. Over-permissioned file shares similarly expose sensitive data and allow lateral movement.

* Use BloodHound / SharpHound to map all attack paths to Domain Admins and identify principals with excessive AD object permissions
* Audit ACLs on all Tier-0 and Tier-1 OUs using Get-Acl or PowerView's Get-ObjectAcl
* Check for non-admin accounts with GenericAll, GenericWrite, WriteOwner, WriteDACL, or DCSync rights (Replicating Directory Changes All)
* Audit security group membership, especially 'nested' groups that may inadvertently grant elevated access
* For file shares: use PowerShell or third-party tools to find shares granting 'Everyone' or 'Domain Users' write/full control
* Review GPO delegation: who can edit, link, or create GPOs in each OU

```powershell
# PowerView check ACLs on AdminSDHolder and Tier-0 OUs:
Get-ObjectAcl -ADSPath 'DC=corp,DC=local' -ResolveGUIDs | ?{$_.ActiveDirectoryRights -match 'GenericAll|WriteDACL|WriteOwner'}
# Find accounts with DCSync rights:
Get-ObjectAcl -DistinguishedName 'DC=corp,DC=local' -ResolveGUIDs | ?{$_.ObjectAceType -match 'Replication'}
# Enumerate shares with broad access:
Find-DomainShare -CheckShareAccess | ?{$_.ShareAccess -match 'Everyone|Domain Users'}
```

3) LAPS deployed but not monitored
Local Administrator Password Solution (LAPS) randomizes and manages the local administrator password on domain-joined machines, storing it in a confidential AD attribute. However, deploying LAPS without monitoring creates a false sense of security. Machines that fall out of compliance revert to static passwords, and stale LAPS passwords may indicate the LAPS agent is broken or the machine is offline and unmanaged.

* Identify domain-joined computers that do NOT have a LAPS password stored in AD (ms-Mcs-AdmPwd attribute is null).
* Check LAPS password age across all computers: any password older than your defined rotation interval (recommended: 30 days) should be investigated
* Verify the LAPS GPO is applied to all target OUs and that the CSE (Client-Side Extension) is installed on endpoints
* Audit who has read access to the ms-Mcs-AdmPwd attribute — only specific admin roles should be able to retrieve passwords
* Identify machines where the LAPS agent is installed but the password has not updated (stuck agent or connectivity issue)

```powershell
# Find computers with NO LAPS password (non-compliant):
Get-ADComputer -Filter * -Properties ms-Mcs-AdmPwd | ?{!$_.'ms-Mcs-AdmPwd'} | Select Name
# Find computers where LAPS password is older than 30 days:
Get-ADComputer -Filter * -Properties ms-Mcs-AdmPwdExpirationTime | ?{$_.'ms-Mcs-AdmPwdExpirationTime' -lt (Get-Date)} | Select Name,ms-Mcs-AdmPwdExpirationTime
```

4) Deploying Active Directory Certificate Services but never checking for misconfigs
Active Directory Certificate Services (AD CS) is one of the most under-audited attack surfaces in enterprise environments. Misconfigurations in certificate templates can allow an attacker to enroll a certificate on behalf of any user enabling persistent, password-independent authentication.

* Run Locksmith or Certify to enumerate all certificate templates and identify ESC1–ESC8 vulnerabilities
-  ESC1: Templates that allow the enrollee to specify a Subject Alternative Name (SAN) and permit enrollment by low-privileged users
-  ESC2: Templates with the 'Any Purpose' EKU or no EKU, allowing unrestricted certificate usage
-  ESC3: Templates with the Certificate Request Agent EKU, enabling enrollment on behalf of others
-  ESC4: Templates where low-privileged users have write permissions on the template object itself
-  ESC6: CA configured with EDITF_ATTRIBUTESUBJECTALTNAME2 flag, allowing SAN specification on any request
-  ESC8: HTTP enrollment endpoints (CES/CEP) susceptible to NTLM relay attacks
* Check who has enrollment permissions on each template — look for 'Domain Users', 'Authenticated Users', or 'Everyone'

```powershell
# Run Locksmith (recommended — provides fix guidance automatically):
Invoke-Locksmith -Mode 4   # Mode 4 = audit + output remediation steps
# Or use Certify to find vulnerable templates:
.\Certify.exe find /vulnerable
# List all certificate templates and their ACLs:
Get-ADObject -SearchBase 'CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=corp,DC=local' -Filter * -Properties * | Select Name,nTSecurityDescriptor
```

5) Allowing regular users to have local admin rights
Granting standard users local administrator rights dramatically expands the attack surface. Local admins can install software, disable security tools, extract credentials from LSASS, and persist malware.

* Enumerate all members of the local Administrators group on domain-joined machines using LAPS reporting, CME, or a script via GPO startup script
* Check for users added to local Administrators via GPO Restricted Groups or Local Users and Groups preferences
* Identify machines where standard domain user accounts appear in the local Admins group
* Check Endpoint Detection & Response (EDR) telemetry for processes run with elevated privileges by standard user accounts

```powershell
# Enumerate local admins remotely (requires admin rights on target):
Invoke-Command -ComputerName WORKSTATION01 -ScriptBlock {Get-LocalGroupMember -Group 'Administrators'}
# Find GPO Restricted Groups policies that grant local admin:
Get-GPO -All | Get-GPOReport -ReportType Xml | Select-String -Pattern 'Administrators'
```

6) Including daily use accounts in privileged groups, like Domain Admins
Using Domain Admin or other Tier-0 accounts for day-to-day work is one of the most dangerous practices in AD security. These accounts are exposed to phishing, browser exploits, and credential theft every time they browse the web, read email, or open a document.

* Enumerate all members of Domain Admins, Enterprise Admins, Schema Admins, Administrators, and other Tier-0 groups
* For each member, check if the account is also used for interactive logons on workstations (Event ID 4624, Logon Type 2 or 10)
* Look for Tier-0 accounts in the same OU as standard user accounts (they should be in a protected Tier-0 OU)
* Check if any Tier-0 accounts have SPN set

```powershell
# List Domain Admins:
Get-ADGroupMember -Identity 'Domain Admins' -Recursive | Get-ADUser -Properties LastLogonDate,Enabled | Select Name,SamAccountName,LastLogonDate,Enabled
# Find DA accounts with SPNs set (Kerberoastable):
Get-ADUser -Filter {adminCount -eq 1} -Properties ServicePrincipalName | ?{$_.ServicePrincipalName} | Select Name,ServicePrincipalName
```

7) Logging into untrusted hosts with Domain Admin accounts
When a Domain Admin account authenticates to a workstation or member server, Windows caches credentials in LSASS memory. If that system is compromised an attacker can extract NTLM hashes or Kerberos tickets using Mimikatz or similar tools and reuse them to authenticate as Domain Admin elsewhere. This is why the source of a logon matters as much as the account itself.

* Review Windows Security Event Logs (Event ID 4624, 4648) across all non-DC systems for logons by accounts in Domain Admins, Enterprise Admins, or other Tier-0 groups
* Use EDR telemetry to identify Domain Admin interactive sessions on workstations
* Check if any Domain Admin accounts have 'Allow logon locally' or 'Allow logon through Remote Desktop' rights on non-DC systems via GPO

```powershell
# Check GPO for DA logon restrictions:
Get-GPOReport -All -ReportType Xml | Select-String -Pattern 'DenyInteractiveLogon|DenyRemoteInteractiveLogon'
# PowerShell to find DA accounts with interactive logons (requires event log access):
Get-EventLog -LogName Security -InstanceId 4624 -ComputerName WORKSTATION01 | ?{$_.ReplacementStrings[5] -in (Get-ADGroupMember 'Domain Admins').SamAccountName}
```

8) Not using Protected Users group
The Protected Users security group (introduced in Windows Server 2012 R2) applies a set of non-overridable credential protections to its members. These protections prevent NTLM authentication, block credential caching, require AES Kerberos encryption, and force Kerberos TGT lifetime limits without requiring individual account configuration. It is one of the simplest and most effective controls available for protecting privileged accounts.

* Check current membership of the Protected Users group. It should include all Domain Admins, Enterprise Admins, Schema Admins, and other Tier-0 accounts
* Identify privileged accounts NOT in Protected Users and assess why
* Test for services or applications authenticating via NTLM using Protected Users accounts. These will break and must be identified before rollout
* Check Kerberos pre-authentication enforcement. Protected Users accounts always require pre-auth, confirm no accounts have 'Do not require Kerberos preauthentication' set

```powershell
# Check Protected Users membership:
Get-ADGroupMember -Identity 'Protected Users' | Select Name,SamAccountName,ObjectClass
# Find privileged accounts NOT in Protected Users:
$puMembers = Get-ADGroupMember 'Protected Users' | Select -ExpandProperty SamAccountName
$daMembers = Get-ADGroupMember 'Domain Admins' -Recursive | Select -ExpandProperty SamAccountName
$daMembers | ?{$_ -notin $puMembers}
```

9) Weak LM/NTLM domain settings
LAN Manager (LM) and NTLM authentication protocols are decades-old and carry significant security weaknesses. LM hashes are trivially cracked. NTLMv1 is susceptible to relay and cracking attacks. Even NTLMv2, while stronger, is vulnerable to relay attacks when SMB signing is not enforced and can be captured and cracked offline. The goal is to eliminate older NTLM versions, enforce NTLMv2 at minimum, and ultimately reduce NTLM dependency across the environment in favour of Kerberos.

* Audit current LM Compatibility Level across all Domain Controllers and member machines via GPO settings or registry
* Use a network capture or SIEM to identify systems still sending LM or NTLMv1 authentication requests
* Check whether SMB signing is enforced on Domain Controllers and all member servers (unsigned SMB enables NTLM relay attacks)
* Check Event ID 4776 (NTLM authentication attempt) on DCs to identify which accounts and machines are using NTLM vs Kerberos
* Identify any applications, services, or third-party systems explicitly configured to use NTLMv1 or LM authentication

```powershell
# Check LM Compatibility Level on a DC:
Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa' -Name LmCompatibilityLevel
# Value meanings: 0=LM+NTLM, 1=LM+NTLM(NTLMv2 if requested), 3=NTLMv2 only(send), 5=NTLMv2 only(send+accept)
# Check SMB signing on DCs:
Get-SmbServerConfiguration | Select RequireSecuritySignature,EnableSecuritySignature
# Audit NTLM usage via event logs:
Get-WinEvent -ComputerName DC01 -FilterHashtable @{LogName='Security';Id=4776} | Select -First 50 | Format-List
```

## Community Project

[Privileged App Path Auditor](https://github.com/nicolasblank/privileged-app-path-auditor) by Nicolas Blank

A PowerShell tool that maps privilege escalation attack paths through Entra ID application ownership. If a regular user owns an app registration that has `RoleManagement.ReadWrite.Directory`, `AppRoleAssignment.ReadWrite.All`, or another Global Admin-equivalent permission, that user can add a secret to the app, authenticate as it, and **silently become a Global Administrator** — no alerts, no approval, no MFA. This tool finds every one of those paths in your tenant.

It also detects **role-based SP control** (Application Administrators who can add secrets to *any* SP with privileged roles — the [most common real-world escalation to Global Admin](https://posts.specterops.io/azure-privilege-escalation-via-service-principal-abuse-210ae2be2a5)), **SP-level credentials** hidden from the Entra portal, **unowned privileged apps** with no accountability, **app instance property lock** status, shadow admins, stale high-privilege apps, credential hygiene issues, and consent policy weaknesses — all in a single script with zero cost and no dependencies beyond the Microsoft Graph PowerShell SDK.
