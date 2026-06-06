---
layout: post
title:  "02x05-maester_of_puppets.ps1"
date:   2026-06-06 10:00:00 +0200
categories: episodes
cover: 'assets/images/02x05/02x05-cover.jpg'
navigation: True
subclass: 'post'
logo:
---
In this episode Live and face-to-face again, this time recorded from Experts Live Netherlands. Chris previews his EL:NL session on practical SecOps for Microsoft 365: how to automate the boring-but-critical parts of M365 security operations, continuously assess tenant posture, detect configuration drift, and respond to common issues using native Microsoft tools you already own.
And Koos recently went on a hunt for a compact FIDO2 security key and discovered there are a lot more options out there than you'd think. He'll walk through what he found, what the trade-offs are between form factor, protocols, and price, and which keys ended up making the shortlist.

<iframe src="https://player.rss.com/df3ndr/2891875?theme=dark&v=2" width="100%" height="202px" title="02x05-maester_of_puppets.ps1" frameBorder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen scrolling="no"><a href="https://rss.com/podcasts/df3ndr/2891875">02x05-maester_of_puppets.ps1 | RSS.com</a></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/eqL40C513Z4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Automate the Boring Stuff: Practical SecOps for Microsoft 365

> Microsoft 365 security is often treated as a collection of one-time configuration decisions, but in reality it's a constantly moving target. New features, policy changes, and administrative actions can quickly introduce risk, and this is true especially when security is managed manually and reactively. In this session, we'll explore how to apply practical SecOps principles to Microsoft 365 by automating the boring but critical parts of security operations. Rather than assuming unlimited budget or a fully staffed SOC, this session focuses on starting small and evolving over time. You'll learn how to continuously assess your tenant's security posture, detect configuration drift, and respond to common security issues using native Microsoft tools and automation capabilities you already own.

### What is SecOps and why do we care?
The brutal truth is that most M365 security operations are still manual and inconsistent. The work that matters most tends to be the work that's easiest to skip, and the gap usually isn't knowledge or tooling — it's time. SecOps is the operational side of security: detection, monitoring, alerting, and response. Specifically, things that run on a schedule, watch for a condition, and tell a human only when needed. 

*SecOps = People + Process + Technology*

Automating it buys you consistency, speed to detection, and leverage — but with a key warning: *automation done badly is worse than no automation.*

### Foundation and concepts
Every automation job has the same anatomy: run on a schedule, look at state or events, tell a human only when a human is needed, and record what you saw. 

The single most important concept is don't re-alert on what you've already seen. Two patterns deliver that: pollers, which watch an event stream and advance a time watermark, and scanners, which inspect current config state and diff against a fingerprint.

You keep two distinct records with different jobs: the audit trail (every finding, every run, forever) and decision state (the small, fast, consistent memory you read each run to decide "is this new?"). Alerting comes down to confidence and routing — escalate on correlated signals rather than single events, and route by severity since not everything deserves the same channel.

Identity and secrets are the foundation under all of it. Wherever possible use workload identity/federation so there's no stored secret at all — the workload proves who it is. Where a real credential is unavoidable, contain it in a vault.

### Tips and traps
Do: 
* Bind to immutable IDs (GUIDs)
* Vault secrets or federate and store none
* Make every run safe to repeat
* Fail loud and force errors
* Think least privilege. 

Don't: 
* Key off UPNs or display names
* Hardcode credentials in scripts or config
* Re-alert on findings you've already reported
* Blindly trust a green "success,"
* Grant the runbook identity more than it needs.

### Building blocks
An Azure Automation runbook orchestrates all five stages, executing in-process and delegating "alert" and "remember" to a service. The supporting pieces: identity (no stored secret), alert channels (Email, Teams, Phone), storage for the audit trail, a Variable for state, Key Vault for a contained credential when managed identity or federation isn't possible, and community tools and scripts like Maester and MaesterDiff.

### Use-cases
The session we talk through six practical scenarios: 
* Break-glass and privileged account monitoring
* Guest and external collaboration hygiene
* BEC mailbox monitoring
* Conditional Access review and change tracking
* App and OAuth consent governance
* Configuration baseline drift detection

Demo code for some of these scenarios can be downloaded [here](https://github.com/cgoosen/ELDemo26/)

## FIDO2 Keys: the hunt for a second and compact model

### Why a *second* key?

There's this saying: *"two is one, one is none"* I learned that lesson the hard way over the past few months. I kept forgetting my FIDO2 key at home or at the office, and the moment a tenant enforced phishing-resistant MFA, I was locked out. The only way back in was burning a Temporary Access Pass (TAP), which is fine once or twice but not as a routine workflow. So I went on the hunt for a second key, specifically one I could leave permanently plugged into my laptop as a backup.

### Why FIDO2 over the Microsoft Authenticator app?

I've also been gradually phasing out the Authenticator app for daily auth. No fishing my phone out of my pocket, no number-matching prompt, and no QR-code scanning step that you still hit with passkey flows on the desktop. A FIDO2 key (especially a biometric one that skips the PIN prompt) is just *quicker*.

There's a bigger long-term reason too: **what's on your phone is tied to your phone**. Lose the device, drop it in the canal, or just upgrade to a new model, and your registered MFA methods go with it. Anyone who's ever had to re-register the Authenticator app across a long list of customer tenants knows exactly how much of a chore that is. A FIDO2 key is completely independent from your phone. Swapping handsets has zero impact on your ability to sign in anywhere.

And you're not limited to a single key per account. You can register multiple FIDO2 keys against the same identity, which is exactly the point of getting a second one in the first place: primary key in the laptop, backup key on the keychain, both enrolled, neither dependent on the other. Modern keys also have plenty of on-device storage for resident credentials (passkeys), so you're unlikely to bump into the slot limit in any realistic scenario.

### Why biometric specifically?

Speed, mostly. Tap and done. A non-biometric key still requires you to touch the key *and* type your PIN. That's perfectly fine if you sign in once a day. It is not fine if you're switching between half a dozen customer tenants on the same day and re-authenticating constantly.

### What about syncable passkeys?

We've covered syncable passkeys in earlier episodes, and on paper they eliminate the need for a physical key altogether. They still don't sit right with me from a security standpoint though. The moment a credential syncs to a cloud account, the security of that credential is effectively bounded by the security of that cloud account. For my own primary identities I'd much rather stay hardware-bound.

### The shortlist

There genuinely aren't *that* many options that are both compact AND biometric. The [YubiKey Nano](https://www.yubico.com/nl/product/yubikey-5c-nano/) is probably the most compact key on the market, but its sensor is purely a capacitive touch trigger for the PIN, not a fingerprint reader.
I also looked at USB-C only. Since I have this in my iPhone and iPad as well, and USB-A is less and less implemented, I won't need anything else. FIDO2 keys with NFC could be nice, but they appear to never have biometric support because of the lack of a power-source during NFC-usage. So NFC-capable devices are always PIN-only.

Three keys made the final shortlist:

* **[YubiKey Bio USB-C](https://www.yubico.com/nl/product/yubikey-c-bio/)**: this is the one I was already using as my primary. Biometrics are dead reliable. But it sticks out of the laptop's USB-C port quite a bit. I was always worried to break it when walking around with my laptop while the key is till sticking out.
* **[Kensington VeriMark Guard USB-C](https://www.kensington.com/p/products/data-protection/fingerprint-security-keys/verimark-guard-usb-c-fingerprint-security-key-fido2-webauthnctap2-fido-u2f-cross-platform/)**: properly compact. Barely protrudes from the USB-C port at all, which means it can literally live in the laptop full-time, even in the bag. Still real multi-factor: something you physically own and carry with you, combined with something you know (PIN) or something you are (biometric). The only thing I can "fault" it is that it's unclear how many passkeys can be stored on the device (currently have 8 stored) and the management app vor macOS is still running x86 code. And Rosetta x86 emulation will be discontinued in the next major release next year according to Apple. So I hope Kensington will re-compile this tool before then.
* **[AuthenTrend ATKey.Pro](https://authentrend.com/atkeypro)**: somewhere in between size-wise, but it wins in the design department. Cool LED lighting on activation and a zinc-alloy exterior, which incidentally makes it great to customise with my laser engraver. 😎

## Community Project: [DeClutter](https://github.com/TheCloudScout/declauttr)

*"Because not every session deserves a comeback"*

It's been a while since `ArchivR`, `R0t8R`, `$pl1tR` and `0ptimiz€r` were unleashed on the world, and apparently Koos couldn't help himself, there's a new R-suffixed tool in the family.

This small #PowerShell utility cleans up the 🪦 graveyard of Claude Code sessions clogging your `claude --resume` picker. **`𝗗𝗲𝗖𝗹𝗮𝘂𝘁𝘁𝗥`** lists every session with its title, size, and prompt contents, pre-checks the ones that look disposable, and lets you bulk-delete TUI-style. Custom-titled sessions get highlighted so you don't accidentally nuke the keepers.

And there is also a **search** function to finally find back that particular session where you had Claude help you solve that nasty problem!

_Cross-platform: works on macOS, Linux, and Windows under PowerShell 7+ (pwsh)._

### Features

* Walks every project directory under `~/.claude/projects/` and lists each session with:
  * timestamp
  * UUID (the filename — same one `claude --resume` shows internally)
  * size on disk
  * session **title** (custom title set via `/title`, or the AI-generated one)
  * first real user prompt, word-wrapped to the terminal width
* **Auto-recommends** likely-disposable sessions and pre-checks them, including:
  * sessions with no real user message (empty/aborted starts)
  * sessions under 20 KB
  * one-word prompts like `resume`, `config`, `exit`, `help`, `init`
  * prompts shorter than 15 characters
* **Highlights keepers**: sessions with a user-assigned custom title
  (set via `/title`) are rendered in cyan and marked with a leading `!` to
  flag the ones you most likely want to hold on to.
* **Rename from the preview**: press **R** inside a session preview to set a custom title without leaving DeClauttR, useful for promoting an AI-generated title to a custom one (so the row becomes a `!`-flagged
keeper) or for relabelling a session into something you'll recognise later.
* Safe by default: nothing is deleted without an explicit confirmation.

### The R-suffixed back catalogue

* [**ArchivR**: Unlimited Advanced Hunting for Microsoft 365 Defender with Azure Data Explorer](https://koosg.medium.com/unlimited-advanced-hunting-for-microsoft-365-defender-with-azure-data-explorer-646b08307b75)
* [**R0t8R**: Secure Logstash to Microsoft Sentinel and Azure Log Analytics connections](https://koosg.medium.com/secure-logstash-to-microsoft-sentinel-and-azure-log-analytics-connections-a63e8c9cb9b8)
* [**$pl1tR**: Split up your logs with pl1tR](https://koosg.medium.com/split-up-your-logs-with-pl1tr-3ab3c76e3125)
* [**0ptimiz€r**: Auto-scale your Sentinel pricing tiers](https://koosg.medium.com/auto-scale-your-sentinel-pricing-tiers-3d1f46b4c6ce)
