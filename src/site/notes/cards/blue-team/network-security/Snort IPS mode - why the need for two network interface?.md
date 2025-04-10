---
{"dg-publish":true,"permalink":"/cards/blue-team/network-security/snort-ips-mode-why-the-need-for-two-network-interface/"}
---

[[cards/blue-team/network-security/Snort\|Snort]]

To use Snort in **inline mode** (as an IPS), you need to put the Snort system **in the path of the traffic**. This means traffic must pass _through_ Snort for inspection before reaching its destination.

How Do We Place Snort Inline?

1. Imagine you have a PC (or device running Snort) with **two network interfaces** (e.g., `enp0s3` and `enp0s8`).
2. You connect:

    - `enp0s3` to the **router**.
    - `enp0s8` to your **PC (or switch that connects other devices)**.

1. Now, traffic flows:
    - From the router → Snort (via `enp0s3`) → Your PC or other devices (via `enp0s8`).
    - From your PC → Snort (via `enp0s8`) → Router (via `enp0s3`).

This setup allows Snort to inspect every packet passing between your PC and the internet.

**Analogy**
- **Without bridging**: There’s no gate between your router and PC. The security guard (Snort) has no traffic to inspect.
- **With bridging**: A gate is set up, and the security guard (Snort) stands there, checking every packet (person) trying to pass.