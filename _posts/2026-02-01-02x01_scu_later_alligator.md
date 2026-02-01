---
layout: post
title:  "02x01_scu_later_alligator.json"
date:   2026-02-01 19:25:00 +1100
categories: episodes
image:
    path: assets/images/02x01.jpg
    width: 1500
    height: 1500
cover: 'assets/images/cover/cover-02x01.jpg'
navigation: True
subclass: 'post'
logo:
---
In this episode, Chris explores Agent 365 while Koos takes another look at Security Copilot. Since our last episode, several new announcements have dropped, making this a great time to dive in and see how these tools can help streamline the work of security teams.

<iframe src="https://player.rss.com/df3ndr/2510613?theme=dark&v=2" width="100%" height="202px" title="02x01_scu_later_alligator.json" frameBorder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen scrolling="no"><a href="https://rss.com/podcasts/df3ndr/2510613/">02x01_scu_later_alligator.json | RSS.com</a></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/rwLKE0YPvjM?si=z4uqqZ0MWOO0v7To" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Agent 365

If 2024 was the year of LLMs, and 2025 was the year of AI Agents then I'm really hoping 2026 will be the year of AI Governance! Microsoft frames Agent 365 as the foundation for the emerging “agentic era,” where autonomous agents are not just assistants but participants in business processes.

### What is Agent 365?

Agent 365 is Microsoft’s new “control plane” for AI agents across Microsoft 365, Teams, Dynamics, Power Platform, and third‑party systems.

It provides a central place to register, manage, monitor, and govern agents, similar to how organizations manage human users today.

Agents get a unique Entra Agent ID, which allows them to authenticate, follow least‑privilege policies, and operate like secure digital workers.

### But why?

Agent sprawl is real - Organizations start experimenting with copilot, custom agents, and workflow bots, and quickly lose track of where agents exist and what they’re doing. Agent 365 counters this with a centralized registry and inventory, providing full visibility.

AI agents need different governance than human users, think about it - agents:

* Act autonomously
* Trigger workflows
* Access sensitive data
* Act continuously and at scale

This creates new compliance, security, and operational risks. Agent 365 provides logging, access controls, threat detection, and lifecycle management specifically designed around agent behavior. Agent 365 standardizes the governance approach regardless if agents come from Microsoft, third parties like ServiceNow, or your internal Copilot Studio builds.

### Agent 365 features

Agent 365 unlocks five capabilities that make enterprise-scale AI possible:

* Registry – provides a single inventory of all AI agents so organisations can see what exists, who owns them, and how they behave.
* Access Control – uses Entra Agent IDs and least‑privilege permissions to ensure every agent only accesses the data and systems it truly needs.
* Visualization – offers dashboards and telemetry that help IT and security teams monitor agent performance, risk signals, and ROI.
* Interoperability – connects agents with Microsoft 365 apps, organisational data, and third‑party platforms so they can operate across business workflows.
* Security – extends Microsoft Defender protections to agents, detecting misconfigurations, vulnerabilities, and threats like prompt‑injection or risky data access.

### What about licensing?

As always with licensing - it's *complicated*. Agent 365 introduces a new per‑agent A365 SKU that licenses AI agents as digital workers inside Entra ID, while keeping all existing Microsoft 365, Copilot, and Azure OpenAI licensing in place — resulting in clarity of governance but complexity in cost and deployment.

* Agents require their own licence (A365 SKU) - Agent 365 uses a dedicated A365 licence that must be assigned to each AI agent instance. These licences are not for humans
* Agents appear in Entra ID like digital employees - Licensed agents receive an Entra Agent ID and can show up in org charts, have permissions, and interact with systems like a user account.
* Agent 365 does not replace existing Microsoft licences - Human users still need their usual Microsoft 365, Dynamics 365, and security/compliance licences.
* A365 covers governance, not model execution - The A365 licence pays for the control plane not AI inference itself.

You need to be part of the [Frontier preview program](https://adoption.microsoft.com/copilot/frontier-program/) to get early access to Microsoft Agent 365.

## Security Copilot

I mentioned AI news but before we dive into that subject there's one other news item I had to share.

### Sentinel data lake now supports Defender XDR logs

In episode 10, we brought in September of last year, I talked about Sentinel data lake. And one of the major shortcomings back then is now resolved.

You can now extend Defender XDR data into data lake natively without any additional complex tooling and solutions!

Just to into your tables overview, select `DeviceNetworkEvent` for example, and extend its retention into data lake by increasing it beyond the default 30 days. Storage costs are roughly $ 0.025 per GB/month without query costs.

### Microsoft brings Security Copilot for free for E5 customers, FOR FREE!

Yes, that right folks.

_Customers with Microsoft 365 E5 will have 400 Security Compute Units (SCU) each month for every 1,000 paid user license, up to 10,000 SCUs each month at no additional cost._

* Example 1: An organization with 400 user licenses gets 160 SCUs/month.
* Example 2: An organization with 4,000 user licenses gets 1,600 SCUs/month.

At first I was like; _"ah there's the catch!"_ With 1600 SCU's per month is only little over 2 SCU's per hour. And I remember how quickly I burned through those in no time a couple of months ago.

First, it's good to understand that is is not converted into an hourly rate. You actually have tho'se SCU's to spend within a month. You can spend them over the course of 30 days, or all in a single day. That's up to you.

Which brings me to the seconds points: agents!

I would advise starting by only using the SCUs for running Security Copilot Agents and using Microsoft Sentinel MCP Server for LLM-based contextual questions.

Microsoft provides several Agents from their "Security Store" like the "Phishing Triage Agent" which will automatically perform otherwise manual tasks like its name suggest.

You can also find a lot of third-party agents in this store or even build your own.

<img src="/assets/images/02x01/02x01-security-copilot-agents.png" alt="Security Copilot Agents" width="100%">

**PRO TIP** set your incident summary generation to _“Generate on Demand”_ (`Settings` —> `Copilot in Defender` —> `Preferences`)

This will save you A LOT of unnecessary SCU usage!

<img src="/assets/images/02x01/02x01-security-copilot-incident-summary-generation.png" alt="Security Copilot Incident Summary Generation" width="100%">

The bad news unfortunately is that Microsoft is still rolling out to customers. It's probably a capacity problem. If you're eligible you'll receive a notification 30-days in advance before they activate it for you.

### All those new shiny AI tools also come with new attack vectors

#### Entra ID Identity Protection for Agents is available in _Preview_

Because agents can operate autonomously and on behalf of a user, they can display unique sign-in behavior. Agents can take initiative, interact with sensitive data, and operate at scale. Microsoft Entra ID Protection for Agents is designed to identify and mitigate risks associated with these capabilities.

And once Identity Protection flags a user as risky, you can prevent them from accessing certain resource with Conditional Access by blocking certain Risk Levels.

#### Creating your own Agents within Copilot Studio also comes with some risks

Because Prompt Injection Attacks exist, data can potentially be exfiltrated through conversations and AI permissions can be abused thru Privilege Escalation.

Fellow [Security MVP Raymond Roethof](https://www.linkedin.com/in/thalpius/) (he's Dutch of course 😉) wrote an excellent blog about how you can protect Copilot Studio with the help of Microsoft Defender for Cloud Apps. Definitely check this out.

#### YELLOWHAT Capture-the-Flag was hacked with AI

During **YELLOWHAT** on January 13th, people on-site were able to compete in a [Capture-the-Flag we got provided through Blu Raven](https://academy.bluraven.io/course/threat-hunting-incident-response-case-001).

Nicola Suter decided to see what's possible with AI. He used Azure Fabric Real-time Intelligence MCP Server to connect to the lab environment and let AI assist him during his hunt.

He wrote down his experience and lessons learned in a blog you should also check out.

### Links

* [Security Copilot inclusion in Microsoft 365 E5 subscriptions](https://learn.microsoft.com/en-us/copilot/security/security-copilot-inclusion)
* [Microsoft Security Copilot Agents overview](https://learn.microsoft.com/en-us/copilot/security/agents-overview)
* [Entra ID Identity Protection for Agents](https://learn.microsoft.com/en-us/entra/id-protection/concept-risky-agents)
* [Configure and enable risk policies in Conditional Access](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-configure-risk-policies)
* [Microsoft Copilot Studio: Real-time protection for AI Agents with Microsoft Defender for Cloud Apps](https://thalpius.com/2026/01/17/microsoft-copilot-studio-real-time-protection-for-ai-agents/)
* [AI just solved a CTF for me!](https://tech.nicolonsky.ch/ai-solved-ctf/)
* [Generative AI training for Beginners](https://microsoft.github.io/generative-ai-for-beginners/#/)
* [Enhance security operations by using Microsoft Security Copilot](https://learn.microsoft.com/en-us/training/paths/security-copilot-and-ai/)
* [Elevate Your Cyber Defense Skills with Blu Raven](https://academy.bluraven.io)
* [Agent Identity Security](https://derkvanderwoude.medium.com/agent-identity-security-8138fb43780d)

## Community Project

[Azure Sentinel Solutions Analyzer](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Solutions%20Analyzer) by Ofer Shezaf - The SIEM Guy @ Microsoft

People in the community probably know his as the father of Sentinel. 😉

Het worked on ArcSight @ HP and came to Microsoft in 2019 to help create Microsoft Sentinel.

_I get asked all the time which tables each Sentinel connector writes to. Surprisingly, the answer isn’t straightforward: many connectors share tables, others write to multiple tables, and—until now—there hasn’t been a single, complete list._
_So…I built one. 🚀 (Okay, a GitHub Copilot agent with Claude Sonnet 4.5 did most of the heavy lifting 😅)_
