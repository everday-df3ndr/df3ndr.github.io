---
layout: post
title:  "02x10-same_soc_new_socks.zip"
date:   2026-11-01 09:45:00 +1000
categories: episodes
cover: 'assets/images/02x10/02x10-cover.jpg'
navigation: True
subclass: 'post'
logo:
---
Koos couldn’t help himself this month, so he’s brought two topics. First up is Microsoft’s ISOC announcement. It sounds like a brand-new product, but is it? He unpacks what it actually is, why everyone’s confused, and what it means for you. Then there’s a story from the field. A customer had to hand law enforcement “a copy” of their Sentinel workspace, and there’s no clone button in Azure. Koos walks through how he did it with Azure Monitor export jobs (currently in public preview), the gotchas he hit along the way, and the script he wrote to make it repeatable. Chris rounds things out with the EWS deprecation in Exchange Online: it didn’t just switch off on 1 October, and it’s worth paying attention to this one.

<iframe src="https://player.rss.com/df3ndr/3283605?theme=dark&v=2" width="100%" height="202px" title="02x10-same_soc_new_socks.zip" frameBorder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen scrolling="no"><a href="https://rss.com/podcasts/df3ndr/3283605/">02x10-same_soc_new_socks.zip | RSS.com</a></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/4i8gt15q8OM?si=mEeSww0xybUg8BBb" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## ISOC: a new product, or a new sticker?

On September 23rd Microsoft announced the public preview of the **Integrated Security Operations Center**, or ISOC, in Microsoft Defender. Rob Lefferts called it "a foundation built for agentic security that brings leading solutions for SIEM and threat protection together". The Tech Community post says it brings XDR, SIEM, threat intelligence, automation and AI together in the Defender portal.

Sounds big. And judging by my LinkedIn feed and the questions I got from customers in the weeks after, a lot of people had a hard time understanding what ISOC really is. So let's go through it.

### What it is, in essence

The most useful sentence is in Microsoft's own FAQ: _"ISOC is not a new standalone product. It is a benefit that expands the value of your existing investments in Microsoft Defender Suite, Microsoft 365 E5 and E7."_

In plain English: if you have one of those licenses, you get a chunk of Microsoft Sentinel's SecOps features inside the Defender portal, without setting up a Sentinel workspace first. There are two layers:

1. **Out of the box, no Sentinel workspace needed.** Case management (the new 'Cases' experience), workbooks, natural-language playbook generation and 'enhanced' automation rules. These just appear in the Defender navigation for eligible tenants.
2. **With an 'ISOC workspace'.** you add the features we already know from Sentinel. like:

  * UEBA
  * Content hub connectors
  * Repositories (CI/CD)
  * Threat intelligence
  * and the ability to ingest Azure and any third-party data in general.

  You create that workspace from the Defender portal: Settings --> Microsoft Sentinel --> SIEM workspaces --> Create workspace. You need an Azure subscription and either Owner, or User Access Administrator plus Microsoft Sentinel Contributor on it.
  Yes, it literally lives under a settings page called 'Microsoft Sentinel'.

> This now also shows us how we will use "Sentinel" in the future, once it will be removed from the Azure Portal on March 31st, 2027.

Who gets it during the preview: Defender Suite, Microsoft 365 E5 or E7 customers **without an active Sentinel workspace**. No minimum seat count. Microsoft explicitly says: don't disconnect a production Sentinel workspace just to qualify. For existing Sentinel customers nothing changes for now; from **November 15th, 2026** they can choose to move to ISOC if their licensing qualifies.

### Why everyone is confused

* **The name.** "Integrated Security Operations Center" sounds like a product. Or a portal. Or a managed service. Microsoft's own FAQ had to open with "Is ISOC a new product? No."
* **The benefit is aimed at people who don't have Sentinel yet.** If you already run a Sentinel workspace you had cases, workbooks and playbook generation in the Defender portal already. But the ISOC-specific extras (the included 90 days of Defender retention, the new ingestion meter, the ISOC workspace; more on these later) are tied to ISOC eligibility, and during this preview phase that explicitly excludes tenants with an active Sentinel workspace. From November 15th you can choose to move. So the people who follow the Sentinel blogs the closest are exactly the ones who have to wait.
* **"ISOC workspace".** A new term for something that, as far as the docs show, is a Sentinel workspace provisioned from the Defender portal. Same Azure subscription, same Azure roles, same settings page. Microsoft never says "it's a Log Analytics workspace", but all the signs point that way.
* **"Is Sentinel going away?"** No. The Sentinel product stays, and the FAQ says the current offering "will continue to exist as is". What does go away is Sentinel in the Azure portal, on March 31st, 2027, but that was announced long ago. What confuses people is that three things landed in the same week with 'ISOC' on the banner: (1) the ISOC benefit itself, (2) the new Cases experience with 'incident cases', and (3) a reshuffle of the Sentinel data lake: integrated onboarding and KQL on the lake straight from Advanced Hunting (see below).
* **Pricing.** A new $2.40 per GB meter for "non-Microsoft data", "regional pricing may vary", 30 days of retention now and 90 "planned", and nobody knows yet what happens to the E5 benefit of 5 MB per user per day once a workspace moves to ISOC.

### My take

Most of this we have seen before. Case management has been in public preview since January 2025 for Sentinel customers in the Defender portal. Editing workbooks in the Defender portal went into preview in August 2025. AI-generated playbooks went GA in May 2026. UEBA, Content hub, repositories, threat intelligence and automation rules are Sentinel features that have been around for years. So yes, it's mostly a collection of existing (preview) features with a new sticker on the box.

But a few things are genuinely new, and some of them are real money:

* **No workspace needed for the basics.** Sentinel never had a license fee; you pay per GB, and incidents, alerts, Azure Activity and the Office 365 audit logs were free to ingest anyway. What you did need was a Sentinel workspace: an Azure subscription, a Log Analytics workspace, Azure RBAC and an Azure bill, before you could touch cases, workbooks or automation rules in the Defender portal. For those out-of-the-box features that requirement is gone. For the workspace-dependent ones you still create a workspace, just from the Defender portal.
* **Incident cases.** The new flavour of cases (preview since September 23rd, MC1477993): one case per incident, with the attack story, alerts, evidence and response actions inside the case. Microsoft calls it the recommended way to work incidents from now on; the legacy incident page stays during the preview.
* **Included retention goes from 30 to 90 days.** More on that below.
* **A cheaper ingestion meter.** Also below.
* **The framing.** 'Perception agents' and the 'agentic SOC'. Project Perception was introduced in July (I covered it in 02x08). Pricing is still unknown; hopefully we'll know more after Ignite (November 17th to 20th).

### Retention: 30 days today, 90 days with ISOC

Today Defender XDR keeps advanced hunting data for 30 days. Incidents and alerts stay visible for 180 days, but the raw tables (`DeviceEvents`, `EmailEvents`, `IdentityLogonEvents`, the lot) are gone after 30. If you wanted 90 days you had to extend the tables into Sentinel, and that meant paying Sentinel ingestion on some of the biggest tables you own (storage up to 90 days was free, ingestion wasn't).

With ISOC, eligible customers get **30 days of included retention for Defender data during the preview, and that becomes 90 days starting November 15th, 2026**. The Tech Community post says the 90 days apply "across all Microsoft Defender data, Azure Activity, and Office 365 Activity". Defender data here means Defender for Endpoint, Office 365, Identity, Cloud Apps and Cloud, plus Entra ID Protection logs. Azure Activity and Office 365 Activity come in through a connector.

Caveats: the post says "eligible customers", the FAQ says "planned for November 15", and what existing Sentinel customers get once they move over isn't spelled out. But three times the hunting window, included in the license? That alone is worth the announcement.

### The price: a new meter at $2.40 per GB (on ISOC workspaces)

From **October 1st, 2026**, eligible ISOC customers can ingest non-Microsoft data through the 500+ connectors on a pay-as-you-go meter of **$2.40 per GB**. Regional pricing and other terms may vary.

To be clear: this is not a new table tier or plan. It's a meter that comes with the ISOC benefit, and ingesting that data requires an ISOC workspace. An existing Sentinel workspace stays on the Sentinel meters. If you're a Sentinel customer today, the route to this price is 'move to ISOC' from November 15th, and how that move works (same workspace or a new one, and what happens to the E5 benefit of 5 MB per user per day) isn't documented yet.

For context, from the Azure retail price list on the day of writing:

| Meter | Price |
| --- | --- |
| Sentinel pay-as-you-go, analytics tier (East US) | $ 4.30 per GB |
| Sentinel pay-as-you-go, analytics tier (West Europe) | € 4.919 per GB |
| Sentinel 2,000 GB per day commitment tier (East US) | $ 4,800 per day, which is exactly $ 2.40 per GB |

So the ISOC meter gives you the price of the 2,000 GB per day commitment tier, without committing to 2,000 GB per day. Not bad. The meter isn't in the retail price API yet, so I can't tell you the euro price.

Does it replace the commitment tiers? Not that I can find. The FAQ says the current Sentinel offering "will continue to exist as is", the pricing page still lists every commitment tier from 50 GB to 50,000 GB per day (plus the pre-purchase plan), and the price API still carries all of them. Whether an ISOC workspace can even be put on a commitment tier isn't documented. My expectation, and this is unconfirmed, is that Microsoft will eventually have everyone move from Sentinel workspaces to ISOC workspaces; I don't see a long-term plan to keep two kinds of workspace around. If that happens, this is a huge price drop for smaller customers on pay-as-you-go, and hardly a drop at all for large enterprises that already sit on a big commitment tier. We'll see if ISOC gets commitment tiers of its own.

Community reporting (Truls Dahlsveen on infernux.no) lists a few more 'later phase' figures: an optional 180-day extension for Defender data at $1.20 per GB, interactive retention up to two years at $0.10 per GB per month, lake ingestion at $0.15 per GB and lake storage at $0.026 per GB per month. I couldn't find those on any official page, so treat them as "subject to change" until Microsoft publishes them.

### Under the radar: the data lake tier for (almost) all tables

We talked about Sentinel data lake in 01x10 (September 2025) and back then the lake tier still smelled a lot like Auxiliary logs: no XDR tables, and not every built-in table either. In 02x01 (February 2026) the XDR tables got fixed. What quietly happened since is that the built-in Azure tables got covered too.

Per the Azure Monitor table reference on the day of writing: 941 built-in Azure tables, of which 489 support the 'Auxiliary / Lake' plan and 452 don't. And the rule in the docs is simple: **all DCR-based custom tables support all plans; for built-in Azure tables it varies per table.** The Sentinel table management doc adds that tier changes aren't available for every table, because "some XDR and Microsoft Sentinel solution tables must remain in the Analytics tier".

The good news is that everything you'd actually want in the lake is now on the 'yes' list:

* `SigninLogs`, `AADNonInteractiveUserSignInLogs`, `AuditLogs`, `MicrosoftGraphActivityLogs`
* `SecurityEvent`, `WindowsEvent`, `Syslog`, `CommonSecurityLog`, `OfficeActivity`
* `SecurityAlert` and `SecurityIncident`
* Every XDR hunting table: `Device*`, `Email*`, `Identity*`, `CloudAppEvents`, `AlertInfo`, `AlertEvidence`

Still on the 'no' list: `AzureActivity`, `AzureDiagnostics` (switch to resource-specific tables, like we said in 02x06), `AzureMetrics`, `Heartbeat`, `Usage`, `Perf`, the threat intelligence tables, `Watchlist`, the UEBA tables (`BehaviorAnalytics`, `SentinelBehaviorInfo`), and any legacy custom log table that was created with the old HTTP Data Collector API instead of a DCR. In the Sentinel connector reference that's the pattern too: of 677 table rows, 398 say "lake-only ingestion supported", and the 279 that don't are almost all old-style `_CL` connectors plus those three `Azure*` tables.

So not "all". But "all the big ones". That's the flexibility we were looking forward to for years now. 🌊

### Sentinel reshaped, once more

Hidden in the Sentinel 'What's new' for September 2026 (same week as ISOC):

* **The data lake is now part of Sentinel onboarding.** Customers onboarding after September 23rd don't go through a separate data lake onboarding and billing setup anymore. Onboard Sentinel, connect it to the Defender portal, and switch the lake tier on per table in Table management. Existing Sentinel customers who never onboarded to the lake: your 'total retention' "evolves from archive-based retention to data lake retention".
* **Interactive KQL on the lake from Advanced Hunting.** A table on the Data Lake tier can now be queried straight from Advanced Hunting, no detour via 'Data lake exploration' and no restore first; it shows up with a gold table icon. What stays split is an Analytics table whose *total* retention extends into the lake: Advanced Hunting sees the Analytics window (say 90 days), and the lake tail beyond that still needs a search job or a KQL job. One footnote: the known-issues list words this as 'new Microsoft Sentinel customers', so if you onboarded the lake before September 23rd, check whether you see the gold icon.
* **Expose Sentinel tables in Microsoft Fabric.** Not to be confused with the lake tier itself: the Sentinel data lake already mirrors your Analytics tables into the lake, that's not new. This is one step further out. A 'Mirrored Azure Monitor' item in Fabric points at the tables in your Sentinel workspace and makes them show up in OneLake as shortcuts, without copying the data and without an extra Sentinel charge (you do need Fabric capacity). From there you query them with KQL in an Eventhouse, Spark notebooks, pipelines or Power BI, and join them with whatever else lives in OneLake: S3 or ADLS via shortcuts, Databricks via federation. In preview it only picks up new data (no backfill) and one mirrored item takes about 500 tables.
* And the reminder: Sentinel in the Azure portal is gone after March 31st, 2027.

### Links

* [Integrated Security Operations Center in Microsoft Defender (Tech Community, September 23rd, 2026, with the FAQ)](https://techcommunity.microsoft.com/blog/microsoftthreatprotectionblog/integrated-security-operations-center-in-microsoft-defender/4559097)
* [Reimagining the SOC for the agentic era in Microsoft Defender (Microsoft Security Blog, Rob Lefferts)](https://www.microsoft.com/en-us/security/blog/2026/09/23/reimagining-the-soc-for-the-agentic-era-in-microsoft-defender/)
* [Case management in the Microsoft Defender portal](https://learn.microsoft.com/defender-xdr/siem-defender-case-management)
* [What's new in Microsoft Sentinel: September 2026](https://learn.microsoft.com/azure/sentinel/whats-new#september-2026)
* [What is ISOC for Microsoft Defender XDR? (Truls Dahlsveen, infernux.no)](https://infernux.no/blog/isoc/)
* [ISOC in Microsoft Defender: deep dive (Charbel Nemnom)](https://charbelnemnom.com/isoc-microsoft-defender-workspace-guide/)

## Backup your data in Sentinel / Log Analytics

This one started with a customer request. The customer got involved in a forensics investigation through law enforcement, and they were asked to provide "a copy" of their Microsoft Sentinel workspace and their Defender XDR data. Three months: May 23rd to August 23rd, 2026, about 61 GB across 26 tables.

There's no 'clone workspace' button in Azure. And investigators don't want a login to your SOC tooling; they want files, with hashes, that nobody can tamper with afterwards.

Luckily Microsoft announced **Export job (preview)** for Azure Monitor back in July, and it did exactly what we needed.

### What is an export job?

One table, a time range and (optionally) a KQL filter, and the records land as gzip-compressed Parquet in hourly folders in a storage account. It's asynchronous: you submit it, get a job id back, and poll for status. Under the hood the job is split into parallel 'bins' with a built-in retry; a manual retry only redoes the missing bins, so no duplicates. It runs on separate compute, so your analysts won't notice.

It fills a gap that has annoyed me for years. Data export rules only forward *new* data as it arrives. An export job can reach back into everything that's still in the workspace. Think legal hold and audit requests, migrating to another SIEM with your history, or cheap long-term copies in cool or archive storage (or a data lake, ADX, some ML experiments).

Limits worth knowing: REST API only (no portal), Analytics and Basic tables only (no Auxiliary), Parquet only, no Private Link or network security perimeter, no workspace replication, 5 concurrent jobs per workspace, 1 year per job, a 7 day job timeout and 5 manual retries per job. A cancelled job can't be retried at all. And if there's an ABAC condition or a protected table you can't read, the export fails for the whole workspace with a 403 and no message. Keep that one in mind.

### What else I considered

| Option | Verdict |
| --- | --- |
| Export job (preview) | Winner. Native, Parquet, no query limits, logged in `LAJobLogs`. |
| Query API script (PowerShell / Python) | Works, but 500k rows / 64 MB / 10 minutes per call, so you end up slicing per hour. A fallback. |
| Azure Data Factory / Logic Apps | Same Query API limits with more plumbing (Logic Apps: 16.7 MB per call). |
| Data export rules, XDR Streaming API | Forward only, no backfill. Do switch them on for the period after. |
| Search job / restore | Results stay inside the workspace. Not supported over Lighthouse either. |
| Just extend retention | Cheapest if preserving in place is acceptable. Not when someone wants files. |

### What bit me

It's a preview, and it showed:

**1. The export permission is a 'data action'.** `Microsoft.OperationalInsights/workspaces/jobs/export/action` is registered as a data action, and Contributor and Owner don't carry any data actions at all. So as a subscription Contributor you get an HTTP 403 with an **empty body**, on submit and on status. Log Analytics Reader does include it, so assign that on the workspace. That one took me half a day. 😅

**2. Check your retention before anything else.** 844 out of 845 tables were on 365 days. `AADNonInteractiveUserSignInLogs` was on 90, had already lost 7.8 GB of the requested window, and was losing another 0.23 GB every day while we were preparing. Raising total retention applies to whatever hasn't been purged yet, so do that *first*.

**3. The storage firewall blocks everything.** `AuthorizationFailure: This request is not authorized to perform this operation` looks like RBAC, but it's the firewall. Cloud Shell and the export service both come in over public Azure IPs. Keep the storage account on "Enabled from all networks" during the export and close it again afterwards; the legal hold protects the content either way. Premium storage isn't supported as a destination, by the way. Use StorageV2 with Standard_LRS or GRS.

**4. The workspace identity needs two roles on the storage account.** Enable the system-assigned managed identity on the workspace, then give it 'Storage Blob Data Contributor' to write, and 'Reader' (or Log Analytics Reader) to resolve the endpoint. Role assignments and locks need Owner or User Access Administrator, so you might need to borrow someone for five minutes.

**5. Status needs some interpretation.** Jobs sit in `Queued` for a while before `InProgress`. `progressPercentage` is the share of hourly bins that are done, not the share of data. The status API calls a finished job `Completed`, while `LAJobLogs` (enable the 'Jobs' diagnostic category) calls it `Succeeded`; my script checked the wrong one. And one of my 27 jobs, `SigninLogs`, sat in `Queued` at 0% long after all its siblings were done, while `LAJobLogs` claimed "Job InProgress". So which one do you believe? I cancelled it and submitted a fresh job: new job id, same container, a few cents. Keep the cancelled id, it's part of your acquisition story. My advice: submit a tiny table like `SecurityIncident` as a canary first.

Two smaller ones: Conditional Access blocked `az login` on my unregistered Mac (error 53003), while Cloud Shell worked fine because it gets its token from the browser session. And nothing about the export job needs Azure CLI; it's a REST POST to `https://api.loganalytics.io` with a bearer token.

### And what about Defender XDR?

This is the part that's less fun to explain to an investigator. Defender XDR isn't a workspace, and the export job doesn't apply to it. What you can still get:

* **Raw advanced hunting tables** are kept for **30 days**. Unless you extended a table into Sentinel, anything older is gone, and Microsoft can't get it back either. (With ISOC that becomes 90 days, see above.)
* **Alerts and incidents** are kept for **180 days**. Export them with the Graph Security API (`security/incidents` and `security/alerts_v2`). `SecurityAlert` and `SecurityIncident` in Sentinel hold a copy for as long as your workspace retention.
* **The device timeline** goes back 90 days and can be exported per device, 7 days at a time, as CSV.
* **For the future:** the Streaming API to a storage account, or extend the XDR tables into Sentinel. Neither one backfills, so switch them on *before* you need them.

### What does it cost?

Two Azure Monitor meters plus your own storage:

| Meter (West Europe, list prices September 2026) | Price |
| --- | --- |
| Search Jobs Scanned (what the export job scans) | € 0,0056 per GB |
| Log Analytics Data Export (GB landed in storage) | € 0,1116 per GB |
| Blob LRS stored: Hot / Cool / Cold / Archive | € 0,0168 / 0,0086 / 0,0039 / 0,0015 per GB per month |

I estimated about 7 euros for our 61 GB. But the 'Data Export' meter bills the GB that *land* in storage, and Parquet compresses really well: the 26 finished jobs scanned about 58 GB and landed 4.6 GB (roughly 12:1, anywhere from 7:1 to 39:1 per table). So the bill ends up under one euro one-off, plus a few cents a month. The meter is shared with data export rules; tell them apart in Cost analysis via 'Additional info': `ExportType:Export job` vs. `ExportType:Data export rule`.

### The script

I didn't want to click through all of this again for the next customer, so I wrapped it in a PowerShell script: `Export-LogAnalyticsHistory.ps1`, on GitHub as a gist (link below). It's built for Azure Cloud Shell. Nothing blocks or sleeps, every phase finishes in seconds and you just re-run it, so the Cloud Shell idle timeout doesn't matter. The job manifest lives both in `$HOME` and in the storage account (container `export-work`), so a disposable session can't lose your job ids.

Phases: **Check** (token, API, identity, storage, RBAC, diagnostics) --> **Size** (what's there, what retention already took, what it costs) --> **Prepare** (storage account, identity, roles, diagnostic setting, lock; it *prints* the commands an Owner has to run for whatever your role can't) --> **Submit** (5 at a time, re-run until "All tables submitted") --> **Status** / **Retry** / **Cancel** --> **Verify** (row counts against the export, SHA-256 of every blob, manifests into an `evidence-manifest` container) --> **Hold** (legal hold on every evidence container, only after Verify).

```powershell
$p = @{ WorkspaceName = 'law-prod'; WorkspaceResourceGroup = 'rg-security'; StorageAccountName = 'stlawevidence01'
        StartTime = '2026-05-23'; EndTime = '2026-08-24' }

./Export-LogAnalyticsHistory.ps1 @p -Phase Check
./Export-LogAnalyticsHistory.ps1 @p -Phase Size
./Export-LogAnalyticsHistory.ps1 @p -Phase Prepare     # hand the printed lines to an Owner, wait ~5 minutes
./Export-LogAnalyticsHistory.ps1 @p -Phase Submit -Tables SecurityIncident   # canary
./Export-LogAnalyticsHistory.ps1 @p -Phase Submit      # repeat until "All tables submitted"
./Export-LogAnalyticsHistory.ps1 @p -Phase Status
./Export-LogAnalyticsHistory.ps1 @p -Phase Verify
./Export-LogAnalyticsHistory.ps1 @p -Phase Hold
```

Without `-Tables` it picks every table that ingested data in the window (according to `Usage`). Without `-StartTime` / `-EndTime` it takes the last 365 days. Afterwards you can spot-check the Parquet files straight from Log Analytics with `externaldata(...) with (format="parquet")`.

### Making it hold up as evidence

A copy is only useful to investigators if you can show nobody touched it:

1. Raise retention on any table shorter than the window. Put a `CanNotDelete` lock on the workspace (never `ReadOnly`).
2. Export, then verify: row count per table in the workspace against the job's result count. Be ready to explain differences (late-arriving data raises the workspace count, purges lower it).
3. Hash every blob (SHA-256), hash the hash list, store both in the `evidence-manifest` container and in the case file. Keep `LAJobLogs`, `LAQueryLogs` and the Activity Log entries as your acquisition record.
4. Apply the legal hold **last**. Retries need write access to the containers.
5. For the period after, switch on a data export rule and, if XDR raw data matters, the Streaming API.

Is it perfect? No, it's a preview, and I hope the 'empty 403' gets a proper error message soon. But for less than a euro (and half a day lost on gotcha 1) we had a verified, hashed, immutable copy of a workspace. I'll take it!

### Links

* [Export-LogAnalyticsHistory.ps1 (GitHub gist)](https://gist.github.com/TheCloudScout/ab4aae2ca77f61a963403c7301cbbbde)
* [Export historical data from Log Analytics workspace with Export Job (preview) (Tech Community, July 2026)](https://techcommunity.microsoft.com/blog/azureobservabilityblog/export-historical-data-from-log-analytics-workspace-with-export-job-preview/4532425)
* [Legal holds for immutable blob data](https://learn.microsoft.com/en-us/azure/storage/blobs/immutable-legal-hold-overview)

## EWS Is Dead, Long Live the Allow List..

In case you missed it, the deprecation of EWS in Exchange Online started on 1 October 2026. Microsoft and other folks in the community have been warning us about this for years, but EWS in Exchange Online didn't just switch off on 1 October - It turned into an AppID allow list, often auto-populated by Microsoft.

Let’s take a look at what actually changed. On 1 October, EWS in Exchange Online didn’t simply switch off. It became an allow list of approved app IDs, and for a lot of tenants Microsoft quietly built that list for them based on 60 days of usage. That kept things running, but it also means anything that runs less often, like a quarter-end script, a year-end export or an occasional backup job, may have been left off without anyone noticing. Then there’s the bit most admins haven’t thought about yet: that list is effectively an inventory of every app with API access to your mailboxes. So the real question isn’t just “what broke?” It’s “who owns each of these apps, do we still need them, and why do they have that much access?”

### Key facts and dates

The hard stop is 1 April 2027, everything before that is a transition period.

| Date | What happens |
| --- | --- |
| 1 Apr 2027 | EWS permanently removed from Exchange Online. No exceptions, no admin control. |
| From 11 Oct 2026 | Any admin setting EWSEnabled to True must populate EWSAllowedAppIDs themselves. |
| 10 to 11 Oct 2026 | Microsoft populated allow lists for tenants with EWSEnabled True and no list, from 60 days of observed usage. |
| 1 Oct 2026 | EWS disabled by default in Exchange Online for tenants that hadn't configured it. |
| End Aug 2026 | Deadline to set EWSEnabled True plus an allow list to avoid the automatic block. |
| Sep 2023 | Retirement announced (MC676299). |
| 2018 | Microsoft stops adding features to EWS and points developers to Graph. |

**Settings at a glance**

- **EWSEnabled = False:** all EWS blocked.
- **EWSEnabled = True, list populated:** only listed AppIDs allowed.
- **EWSEnabled = True, list empty:** all EWS blocked.
- **Null and Null:** 7-day Message Center warning, then Microsoft builds the list and sets EWSEnabled to False.
- **Mailbox level:** a mailbox can only use EWS if both the org and mailbox settings allow it.

**Known Graph gaps to look at:** archive mailbox access, Exchange Online management, folder-associated information and user configuration, and third-party public folder access.

**Not affected:** on-premises Exchange Server, and Microsoft's own apps such as Teams. Classic Outlook still needs the Microsoft Office AppID on the list.

### PowerShell snippets

How to check the state, see the list, add to it safely.

**1. Check tenant's EWS state and allow list?**

```powershell
Get-OrganizationConfig -RetrieveEwsOperationAccessPolicy |
    Format-List EwsEnabled, EwsAllowedAppIDs
```

**2. Add an AppID without wiping the existing list**

```powershell
$current = (Get-OrganizationConfig -RetrieveEwsOperationAccessPolicy).EwsAllowedAppIDs
$updated = @($current) + '<app-id-guid>' | Select-Object -Unique
Set-OrganizationConfig -EwsEnabled $true -EwsAllowedAppIDs ($updated -join ',')
```

Important: Setting the list replaces it. Forget to include what's already there and you block it.

**3. Which mailboxes are individually blocked?**

```powershell
Get-CASMailbox -ResultSize Unlimited |
    Where-Object { $_.EwsEnabled -eq $false } |
    Select-Object PrimarySmtpAddress, EwsEnabled
```

Changes to EWSEnabled usually apply in under an hour but can take up to four. Allow-list changes can take up to 24 hours. For finding the apps themselves, start with the EWS usage report in the Microsoft 365 admin center.

### Quick Checklist

1. Check your tenant's EWSEnabled value and allow list today.
2. If Microsoft built the list for you, review every AppID on it - don't assume it's right!
3. Match each AppID to an owner and an app registration. No owner, no access!
4. Look for jobs that run less often than every 60 days, such as quarter-end or year-end scripts, and make sure to plan for them.
5. Confirm your backup vendor's AppIDs are on the list and that backups have actually run since October 1.
6. Check for Apple Mail and legacy Outlook for Mac users, and plan their move.
7. Review the permissions on each EWS app registration and plan a least-privilege Graph replacement.
8. Put 1 April 2027 in the risk register. The allow list buys time; it isn't a strategy.

### Links

This has been well covered by the community, here are some links to more detailed info for those interested:

* [Microsoft: EWS Deprecation Is Here](https://techcommunity.microsoft.com/blog/exchange/ews-deprecation-is-here-%e2%80%93-what-this-means-to-you/4561431)
* [MC1485116](https://mc.merill.net/message/MC1485116)
* [Microsoft: Exchange Online EWS, Your Time is Almost Up](https://techcommunity.microsoft.com/blog/exchange/exchange-online-ews-your-time-is-almost-up/4492361)
* [Tony Redmond: The Final Countdown](https://office365itpros.com/2026/02/06/ews-retirement-may-2027/)
* [Tony Redmond: EWSEnabled tenant setting](https://office365itpros.com/2025/02/21/ewsenabled-tenant-setting/amp/)
* [Glen Scales on M365.FM](https://www.m365.fm/the-end-of-ews/)
* [LazyAdmin: Find every app still using EWS](https://lazyadmin.nl/office-365/ews-is-being-blocked-in-exchange-online-how-to-find-every-app-still-using-it/)
* [Petri: Countdown to the end of EWS](https://petri.com/microsoft-counts-down-to-end-of-exchange-web-services/)

## Community Project

### EntraTrace

Microsoft Security MVP Bert-Jan Pals has released [EntraTrace](https://github.com/Bert-JanP/EntraTrace) which is a defensive security research tool for documenting and identifying the observable behaviour of offensive tooling targeting Microsoft Entra ID. The project builds a automatic knowledge base around tools such as AzureHound, AADInternals, O365Enum, PingCastle, and others, with a focus on the UserAgent and API artefacts they generate.

Offensive security tools are frequently used to assess and attack identity environments. Understanding how those tools interact with Entra ID can help defenders identify their use, investigate suspicious activity, and improve detection coverage.
EntraTrace aims to bridge the gap between offensive tooling research and defensive security operations.

#### Use Cases

* Build detections for known offensive security tools
* Develop Microsoft Sentinel / SIEM hunting queries
* Investigate suspicious Entra ID and Microsoft Graph activity
* Identify tooling during incident response
* Research the behaviour of offensive identity tooling
* Improve defensive visibility into identity attack techniques
