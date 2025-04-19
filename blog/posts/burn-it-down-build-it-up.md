---
title: "Burn It Down, Build It Up"
slug: burn-it-down-build-it-up
date: 2025-04-19
tags: [homelab, reboot, blog]
description: "Why I decided to tear down my original homelab and rebuild it with purpose."
type: post
draft: false
---

[News](https://stackedinfraverse.com/tag/news/)

Why I'm Rebuilding My Homelab from Scratch

[![Shawn Hank](https://stackedinfraverse.com/content/images/size/w160/2025/04/IMG_1815.png)](https://stackedinfraverse.com/author/shawn/)

![Burn It Down, Built it Up](https://images.unsplash.com/photo-1551964508-20b2b2e81667?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=M3wxMTc3M3wwfDF8c2VhcmNofDMwfHxidXJuJTIwfGVufDB8fHx8MTc0NTA4MzI4OXww&ixlib=rb-4.0.3&q=80&w=1200)

Photo by [Florian Olivo](https://unsplash.com/@florianolv?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit) / [Unsplash](https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit)

### Why I'm Rebuilding My Homelab from Scratch

If I were starting over today, this is how I'd do it—and why.

___

## 🤨 Let’s Get Real

My homelab was a mess.

Not because I didn’t know what I was doing, though that was part of it. It just wasn't the complete story.

I’ve been in the technology industry for nearly 30 years. I’ve soldered analog phone circuits, buried outside plant (copper and fiber optic cables), built VoIP trunking from scratch, setup C-band video transmission systems to film and record missile tests from helicopters, configured legacy firewall systems and ridden the wave through almost every generation of "this is the new thing" in computing that happened over the last 3 decades.

But all that experience isn't enough when your lab becomes a house of cards held together with hope, string and some duct tape.

**My previous homelab was a tech junkyard:**

-   Raspberry Pis scattered everywhere, performing different functions
-   SuperMicro mini ITX mini servers running Proxmox and VMware infrastructure for Virtual Machines and Containers
-   Core functions like DHCP, DNS, NTP, Domain Controller and Certificate Authorities managed across different systems and hardware
-   Random scripts on random systems
-   Manual hardening (sometimes), no real automation
-   Zero documentation

It started as a playground to learn technologies related to my role as a presales engineer. Over time it became a less than mission-critical kludge. Everything was reactive, experimental, or temporary—which became permanent by accident. It got to the point where I couldn’t trust it to scale, evolve, or even reboot cleanly. It was an embarrassment, and I was too ashamed to show and tell.

> "Some things work sometimes. Other things have never worked."

So I did the only sane thing one could do in this situation: I stopped everything. Cold turkey. No more string. No more duct tape. No more trying to change engine and wings of an airplane in mid-flight.

I decided to **burn it down** and start over.

___

## 📃 Lessons from the Failure

Here’s what failed and why:

| Symptom | Root Problem |
| --- | --- |
| Inconsistent DNS and hostnames | No internal authority; Pi-hole and router DNS fought each other, causing poor connectivity and a low WAF (Wife Approval Factor). |
| TLS warnings everywhere | No internal CA, no public Let's Encrypt fallback. Everything was insecure and undisciplined. |
| Untrusted services | Self-signed certs, expired certs, broken ACME chains. |
| No backups, no redeploys | No Git, no Ansible, no Terraform. Manual everything. What a time suck. |
| IoT + Everything on one VLAN | No segmentation, isolation, or firewall visibility. At least the management traffic was separated. |
| Zero observability | No metrics, logs, heartbeat checks, or dashboards. Just the family saying, “Hey, the Internet is down again.” Livin’ the Dream. |
| Insecure defaults | Default users, password SSH logins, no firewalls. Everything open. |
| Service sprawl | Containers running random things on random hardware. No unified stack. No harmony. |

The deeper truth: I half-assed my homelab. It didn’t reflect how I think, what I value, or how I really feel about my passion, my career, or my life. It needed to change.

___

## 💪 What I'm (re) Building Instead

Yes, on the surface this is a rebuild, but it's also much more than that. It’s a refactor of everything from self-hosting, home lab infrastructure, and, to be honest, my life and how I choose to show up in it each and every day.

### Guiding Principles:

-   **Automated**: Every device is provisioned from a known-good script or playbook. Things will be more consistent, reliable, scalable and I'll save a ton of time.
-   **Documented**: All content, configs, decisions, and diagrams live in Git and synced to GitHub (and maybe GitLab) for public viewing, consumption and feedback.
-   **Segmented**: VLANs, firewall rules, and device trust boundaries are defined and enforced
-   **Secure**: SSH keys only, hardened OS's, local firewalling, root logins disabled
-   **Observable**: Metrics, logs, health checks, and uptime monitoring built-in - complete with visuals and interactive real time dashboards
-   **Modular**: DNS, certs, services, monitoring—all separate, swappable, and upgradable.

___

## 🔧 First Things First: DNS, Identity, and Trust

The first services I’m rebuilding are the ones everything else depends on:

### 1\. **DNS Infrastructure**

I’m building a small test cluster with 2 Pi-hole nodes and 2 AdGuard Home nodes, all backed by **Unbound** for recursive resolution. These will sit behind a load balancer/reverse proxy so I can test, benchmark, and observe each solution side-by-side before choosing a primary.

All of this will be internally resolvable via a `thisdemo.rocks` domain namespace (e.g., `truenas.thisdemo.rocks`, `ca.thisdemo.rocks`, `minipc2.thisdemo.rocks`). The Unifi Dream Machine Pro Max will remain the central DHCP authority, and will forward DNS requests to a load balanced cluster of Raspberry Pi’s running PiHole, AdGuardHome and Unbound for recursive DNS.

Unbound will handle DNS resolution for everything outside the homelab (e.g., google.com, digitalocean.com, etc.)

I'm deploying **Smallstep CA** on a dedicated Raspberry Pi. It will issue internal TLS certificates internally to allow me to secure:

-   IP-based services (like TrueNAS or IPMI interfaces)
-   Hostname-based services on private DNS

No more self-signed cert hell. No more TLS warnings. And no more relying on Let's Encrypt for internal infrastructure. I’ll still use Let’s Encrypt for externally exposed services via ACME DNS validation or Cloudflare SSL Origin Certs with their Tunnels solution.

### 3\. **Network Segmentation Probes**

I'll be dropping a few different physical Pi SCBs into different network segments and using tools like:

-   `iperf3` for bandwidth testing
-   `tcpdump` for packet captures
-   `ping` and `mtr` for reachability

These will verify that VLANs are working as intended, and that firewall rules aren’t just assumed correct.

___

## 🤖 Infrastructure as Code (Git Repos FTW)

Everything in the new homelab gets committed to Git:

-   `content/` — notes, diagrams, blog posts, documentation
-   `infra/` — provisioning, hardening scripts, templates
-   `services/` — Docker stacks, Compose files, reverse proxies

No more undocumented changes. No more tribal knowledge. Everything gets tracked. Everything gets shared via GitHub public repos.

The repos are mostly empty at the moment because I'm just starting the rebuild process, but the framework and structure for the repos are completed as best as they can be for now.

Feel free to check them out:

> [https://github.com/stackedinfraverse/content](https://github.com/stackedinfraverse/content?ref=stackedinfraverse.com)  
> [https://github.com/stackedinfraverse/infra](https://github.com/stackedinfraverse/infra?ref=stackedinfraverse.com)  
> [https://github.com/stackedinfraverse/services](https://github.com/stackedinfraverse/services?ref=stackedinfraverse.com)

___

## 🌟 Closing Thoughts

This entire project is about rethinking things and doing everything right. "Right" means implementing best practices so everything is secure, repeatable, observable, and versioned. It’s a platform I can learn from, publish on, and ultimately... trust.

___

## ⏭️ Future Posts

-   The secure Raspberry Pi bootstrap process (`pi-secure.sh`)
-   Automating device setup with raspi-config
-   Building admin tools with `pi-admin-setup.sh`
-   Hardening x86 systems with `minipc2-secure-v5.sh`

This isn’t a weekend project. This is the new foundation.

LFG
