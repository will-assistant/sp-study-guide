Service Provider Study Guide. Introduction.

Welcome to the Service Provider Study Guide — an operations-first training curriculum for engineers building and running service provider backbone networks. This guide contains over 220,000 words across 12 modules, covering everything from IS-IS adjacency formation to SRv6 network programming to coherent optics at 800 gig.

Who is this for?

This guide is designed for three audiences. First, enterprise engineers transitioning into service provider environments. Second, Internet Expert — Service Provider and IE-SP candidates who need structured, end-to-end topic coverage. And third, practitioners who already know MPLS basics and want modern context on Segment Routing, EVPN, optical transport, and automation.

What is Internet Expert — Service Provider?

The Cisco Certified Internetwork Expert, Service Provider track, is widely considered the hardest networking certification in existence. It covers the full stack of technologies that service providers use to build and operate backbone networks.

This includes: Interior Gateway Protocols at continental scale — IS-IS and OSPF. BGP as both the internet routing protocol and the VPN control plane. MPLS forwarding, label distribution, and traffic engineering. Segment Routing — both SR-MPLS and SRv6 — the modern replacement for LDP and RSVP-TE. L3VPN and L2VPN EVPN services — the revenue-generating products that pay for the backbone. Optical transport including DWDM, OTN, and coherent optics. 5G transport and network slicing. And automation via YANG, NETCONF, gNMI, and streaming telemetry.

The Juniper IE-SP covers comparable depth from the Junos perspective. This guide covers both platforms side by side.

Design Philosophy.

This guide follows five principles.

Operations first. Every section starts with "why does this exist and when does it break" — not with an RFC number. Theory serves operations, not the other way around.

Dual vendor. IOS-XR and Junos configurations are presented in parallel. Most service provider networks run both platforms. You should be fluent in both.

War stories. Each section includes at least one real-world incident — a misconfiguration that took down a metro ring, a label leak that propagated across continents, a DIS election that created phantom blackholes. Learning from failures is faster than learning from theory.

Progressive depth. Modules build on each other deliberately. IGP, then BGP, then MPLS, then traffic engineering, then Segment Routing, then VPN services, then transport, then slicing, then automation, then design case studies. The dependency chain is intentional.

And finally: exam-ready but not exam-limited. This guide will prepare you for IE-SP and IE-SP lab exams. But the real goal isn't to pass a test — it's to be the engineer who gets called at 2 AM when a core router is dropping traffic and nobody else can figure out why.

The Curriculum.

The guide is organized into 12 modules with 51 total sections.

Module 1 covers Foundations — the service provider architecture overview.

Module 2 covers IGP at Scale — IS-IS deep dive, OSPF in SP networks, convergence tuning, and the IS-IS versus OSPF decision framework.

Module 3 covers BGP at SP Scale — fundamentals, iBGP design with route reflectors and confederations, eBGP peering at internet exchanges, and BGP policy and traffic engineering.

Module 4 covers MPLS Core Operations — LDP and label distribution, RSVP-TE, label operations including push swap and pop, and MPLS OAM and troubleshooting.

Module 5 covers Traffic Engineering — fundamentals, advanced RSVP-TE, Segment Routing TE, and TE deployment and design.

Module 6 covers Segment Routing in six sections — SR-MPLS fundamentals, SR-TE policies, TI-LFA for sub-50-millisecond convergence, SRv6 fundamentals, SRv6 network programming, and SR migration strategies.

Module 7 covers L3VPN Services in five sections — architecture, MP-BGP for VPNv4 and VPNv6, Inter-AS L3VPN, extranet and shared services, and L3VPN scale and convergence.

Module 8 covers L2VPN and EVPN in five sections — legacy L2VPN with VPWS and VPLS, EVPN fundamentals, EVPN-MPLS versus EVPN-VXLAN, EVPN multi-homing, and EVPN for data center interconnect.

Module 9 covers Transport and Optical Networks in five sections — the transport hierarchy, DWDM fundamentals, OTN, packet-optical integration, and coherent optics at 400 gig and beyond.

Module 10 covers Network Slicing and 5G Transport — slicing concepts, Flexible Ethernet, and 5G crosshaul requirements.

Module 11 covers Automation and Telemetry — model-driven networking, streaming telemetry, SR-TE controller integration, CI/CD for network config, and a hands-on lab with gNMI and SR-TE policy automation.

Module 12 ties it all together with Design Case Studies — ISP backbone design, DCI with EVPN, mobile backhaul and 5G transport, internet exchange point design, and EVPN L2VPN migration.

The total estimated audio for all modules is approximately 22 hours.

Let's begin with Module 1: Foundations.