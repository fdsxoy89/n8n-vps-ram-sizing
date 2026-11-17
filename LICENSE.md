# VPS for n8n: How Much RAM Do You Actually Need, What Does It Cost per Month, and Which Plan Should You Buy? (Specs, Renewal Prices, Setup Notes)

n8n will run on 1 vCPU and 2 GB of RAM, and that is the floor rather than a recommendation. Once a webhook fires often or a workflow moves real data, the target moves to 2–4 vCPU and 4–8 GB of RAM on Ubuntu 24.04 LTS. RackNerd's annual KVM specials sit on both sides of that line: $35.99/year clears the minimum, $59.99/year buys the 4 GB plan most people running production automations should pick, and both renew at the rate you originally paid.

Below: the full plan table, what each tier actually handles in n8n, the preinstall-versus-Docker decision, and the renewal math that promo pricing usually hides.

> **Quick pick.** Take the **4 GB KVM special at $59.99/year** (2 vCPU, 4 GB RAM, 60 GB SSD, 7 TB monthly transfer) if n8n is doing real work for you. It meets the 2 vCPU / 4 GB production line for about $5/month. Testing flows or running two or three low-traffic automations? The **2 GB plan at $35.99/year** is enough. Either way, skip the $21.99 1 GB tier for n8n: RackNerd's own one-click app notes put n8n's comfortable floor at 2 GB.
>
> 👉 [See RackNerd's current KVM VPS specials](https://bit.ly/RacKnerd)

## All five RackNerd KVM VPS specials

| Plan | vCPU / RAM | SSD | Monthly transfer | Price (billed annually) | Order |
| --- | --- | --- | --- | --- | --- |
| 1 GB KVM | 1 vCore / 1 GB | 20 GB | 3,000 GB @ 1 Gbps | $21.99/year (≈$1.83/mo) | [Order 1 GB](https://bit.ly/RacKnerd) |
| 2 GB KVM | 2 vCore / 2 GB | 35 GB | 5,000 GB @ 1 Gbps | $35.99/year (≈$3.00/mo) | [Order 2 GB](https://bit.ly/RacKnerd) |
| 4 GB KVM | 2 vCore / 4 GB | 60 GB | 7,000 GB @ 1 Gbps | $59.99/year (≈$5.00/mo) | [Order 4 GB](https://bit.ly/RacKnerd) |
| 6 GB KVM | 6 vCore / 6 GB | 100 GB | 12,000 GB @ 1 Gbps | $89.99/year (≈$7.50/mo) | [Order 6 GB](https://bit.ly/RacKnerd) |
| 8 GB KVM | 7 vCore / 8 GB | 150 GB | 20,000 GB @ 1 Gbps | $119.99/year (≈$10.00/mo) | [Order 8 GB](https://bit.ly/RacKnerd) |

Every plan is full KVM virtualisation with root SSH access, a dedicated IPv4, a 1 Gbps port, and OS choice at order time across CentOS, Rocky Linux, AlmaLinux, Fedora, Debian and Ubuntu. Custom ISO installs go through a support ticket. Deployment is close to instant: your IP and root password arrive by email minutes after payment.

RackNerd also lists Windows VPS options ($69.00/year for 2.5 GB RAM, $115.00/year for 4 GB, $289.00/year for 12 GB). n8n is a Linux application, so those are irrelevant here.

## Memory is the constraint, not CPU

Memory is usually the more critical resource for n8n, and that is straight from n8n's hosting documentation, which every provider guide quotes. The reason shows up fast in practice: n8n is a Node.js process, each running execution holds its working data in memory, and the execution history accumulates in the database. A 1 GB box will start, but you can hit memory errors on workflows that look trivial. That is why 2 GB is the honest minimum and 4 GB is where things stop being fragile.

Two numbers to hold onto:

- **Chronological or scheduled workflows** (hourly sync, nightly report) have predictable needs. The minimum configuration carries them.
- **Webhook-triggered workflows** fire in bursts. Concurrency needs CPU cores for parallel executions and RAM for the active processes. If you expect meaningful traffic, start at 4 vCPU.

That second rule is where RackNerd's lineup has an oddity worth knowing. The 4 GB plan ships 2 vCores. The 6 GB plan ships **6 vCores for $30/year more**. If your bottleneck is concurrent webhook executions rather than raw memory, the 6 GB tier is the better $89.99 you'll spend, because cores are exactly what a burst of simultaneous triggers consumes.

## What each RackNerd tier means for n8n

**1 GB, $21.99/year.** A lightweight site, a proxy, a VPN endpoint, a Linux playground. Not n8n. RackNerd's own one-click app panel states that n8n (like Coolify) is comfortable from 2 GB, and the panel will let you install it on 1 GB anyway and let you find out why that matters.

**2 GB, $35.99/year.** The minimum viable n8n box: one instance, a handful of workflows, low trigger volume, no queue mode. Fine for a solo operator automating their own admin work.

**4 GB, $59.99/year.** The default recommendation. Matches the 2 vCPU / 4 GB production line, leaves headroom for larger payloads and more complex logic, and the 60 GB SSD holds a couple of years of execution history before you need to prune.

**6 GB, $89.99/year.** The concurrency pick. Six cores handle parallel webhook executions that would queue up on a 2-vCore box.

**8 GB, $119.99/year.** Queue mode with separate worker processes, multiple n8n instances, or n8n sharing the box with Postgres and something else. If you also want to run local models through Ollama alongside n8n, this is the tier to look at; Ollama alone wants at least 4 GB, and RackNerd's panel notes as much.

For AI nodes that call OpenAI, Anthropic, Gemini or Hugging Face over HTTPS, none of this matters much. Those calls are network-bound, and even the 2 GB plan handles them.

## Preinstalled, Docker, or npm?

RackNerd ships a control panel called ctrl with every VPS special, and it includes a one-click app library with n8n in it. Pick the app, wait for the install, and the credentials appear in the panel. Docker with Portainer is also a one-click install if you'd rather manage containers yourself.

The three install paths and what they mean for maintenance:

1. **One-click from ctrl.** Fastest route to a working instance. Updates still land on you, since the plan is unmanaged.
2. **Docker by hand.** The sane long-term choice. Updating n8n is a `docker pull` of the new image and a container restart, and the whole stack moves to another server by copying the volume and the compose file.
3. **npm.** Works, and it's what the official docs describe for bare-metal installs, but you own Node version drift and dependency conflicts on top of everything else.

Two things to get right in the first hour, regardless of install method:

- **HTTPS is not optional.** n8n stores API keys, passwords and personal data as credentials. Running it over plain HTTP on port 5678 means those travel in clear text. Point a domain at the VPS, put Caddy or Nginx in front, terminate TLS there, and close 5678 to the outside once the proxy works. Port 5678 is n8n's default and needs to be reachable during first-time setup, not afterwards.
- **The firewall is your job.** ctrl has a network firewall for inbound rules. RackNerd's support covers hardware, power, network and service availability. It does not cover your OS updates, your firewall rules, your n8n version or your workflow logic, and no VPS provider's does.

## The renewal price is the part worth checking

Budget VPS pricing usually works like this: a steep first-term discount, then a renewal at two to three times the promotional rate. A concrete example from a mainstream competitor's self-hosted n8n page: its 2 vCPU / 8 GB / 100 GB NVMe / 8 TB plan is advertised at **$12.99/month on a 24-month term ($311.76 paid today)** and renews at **$28.99/month**. Over two years that's $311.76, and from month 25 you're paying $347.88/year.

RackNerd's specials work differently. The specials page states these plans can be renewed at the same rate for as long as you keep the service. Run the two-year numbers on comparable RAM:

|  | RackNerd 8 GB special | Competitor 8 GB plan (24-month promo) |
| --- | --- | --- |
| vCPU / RAM | 7 vCore / 8 GB | 2 vCPU / 8 GB |
| Storage | 150 GB SSD | 100 GB NVMe |
| Transfer | 20 TB/month | 8 TB/month |
| First 2 years | $239.98 | $311.76 |
| Year 3 onward | $119.99/year | $347.88/year at the renewal rate |

The competitor isn't charging that for nothing. You get NVMe instead of SSD, a hosting control panel, and a 30-day money-back guarantee that RackNerd's specials don't advertise. If a refund window is what you need to feel safe, that's a real difference and worth the premium.

If you just want to test RackNerd's network before committing to a year, the standard monthly-billed KVM line exists for exactly that; its list prices run far above the amortised annual specials (the 1 GB monthly tier has been listed around $17.99/month). Paying one month to check latency from your location, then switching to the annual special, costs a few dollars and removes the guesswork.

👉 [Compare the annual KVM specials before you commit](https://bit.ly/RacKnerd)

## Who shouldn't buy a self-hosted VPS for n8n

Self-hosting removes per-execution billing and user caps, but the trade is operational work, and it's not the right call for everyone:

- **You don't want to touch Linux.** A VPS gives you a root shell and a bill. Everything above that is yours.
- **You need SSO/SAML, audit logs, or encryption of credentials at rest.** Those are n8n Enterprise features, not something a VPS includes. No amount of RAM on a $60/year box turns on SAML.
- **You need someone to fix a broken workflow.** Support at every provider in this category stops at the infrastructure line.
- **You need backups handled for you.** RackNerd's specials don't advertise automatic backups or snapshots. Build your own routine on day one: a nightly `pg_dump` plus a workflow export pushed to object storage somewhere other than the VPS. That's fifteen minutes of setup and it's the difference between an outage and a rebuild.

## Before you order: four things to check

1. **Pick the datacenter near your users.** Latency matters most when n8n calls APIs your team waits on. RackNerd lets you choose a location per plan, and if your audience is in Asia, the Los Angeles nodes are the usual pick.
2. **Ask for IPv6 if you want it.** Los Angeles and France locations allow up to 100 free IPv6 addresses via a support ticket after ordering. One dedicated IPv4 is included everywhere.
3. **Don't hunt for a coupon.** The specials price is the price; codes circulating on coupon sites for "30% off VPS" are expired or bait. RackNerd's legitimate official codes apply to dedicated server lines, not VPS.
4. **Don't buy 1 GB for n8n to save $14.** The gap between the 1 GB and 2 GB plans is $14/year. The gap in usable memory against n8n's floor is the difference between a working instance and a support thread about heap errors.

## Which plan fits your case

Running n8n for yourself, a few scheduled workflows, no public webhooks: **2 GB KVM, $35.99/year**. Business automations with webhook triggers and a Postgres backend on the same box: **4 GB KVM, $59.99/year**. Webhook-heavy workloads where executions pile up concurrently, or queue mode with workers: **6 GB KVM at $89.99/year** for the six cores, or **8 GB at $119.99/year** if you're also running Ollama or a second service alongside.

One last check before you pay: confirm the plan you're buying still shows the annual price at checkout and that you've selected the datacenter you actually want, since the plan list rotates and locations differ per special.

👉 [Pick a RackNerd KVM plan and deploy n8n from ctrl](https://bit.ly/RacKnerd)
