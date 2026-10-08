# Project 001: Network Segmentation Validation

**Analyst:** Orlo
**Date:** 7 October 2026
**Status:** Final

## 1. Executive Summary

We assessed the company's internal security testing environment to verify its isolation from the company's trusted network and the public internet.

In doing so, we discovered an intentionally vulnerable machine with unrestricted access to the public internet.

The key risk is that an attacker gaining access to the testing environment would immediately find a series of intentional vulnerabilities available for exploitation. 

However, since there was no access to the trusted network, we classified this risk as **Medium level**.

As remediation, we applied internal firewall rules to restore the intended isolation of the vulnerable machine.

Testing then validated that the at-risk machine was isolated as per security policy and the risk was mitigated.

## 2. Objective

The objective of this assessment is to verify that the organisation's testing environment is secure and suitably isolated from the trusted network and the public internet.

## 3. Scope

The assessment examined the virtual infrastructure, network configuration, firewall configuration and connectivity of the systems within the security testing environment.

## 4. Environment

The assessment environment is hosted on a Proxmox VE hypervisor named, Alfred.

The environment currently contains the following virtual machines:

- VM 100 Ubuntu Server
- VM 101 Kali Linux
- VM 102 OPNsense
- VM 103 Metasploitable 2

Initial infrastructure review indicates that the lab systems are connected to a dedicated virtual bridge separate from the trusted network.

The Proxmox host uses two active Linux bridges:

- `vmbr0` connects to the trusted network through physical interface `nic0`.
- `vmbr1` is a separate virtual bridge labelled `Basement Lab`.
- `vmbr1` has no physical interface, IP address, or gateway configured on the Proxmox host.

OPNsense provides the boundary between the trusted network and the Basement Lab.

It is configured with two virtual network interfaces:

- One interface connected to `vmbr0` for the trusted/upstream network.
- One interface connected to `vmbr1` for the Basement Lab network.

Its virtual network interfaces are configured as follows:

- `net0` is connected to `vmbr0` and operates as the WAN interface with address `192.168.68.68/22`.
- `net1` is connected to `vmbr1` and operates as the LAN interface with address `10.20.0.1/24`.

Kali Linux is connected to the Basement Lab network with address `10.20.0.121/24`.

It uses `10.20.0.1` as its default gateway, corresponding to the OPNsense LAN interface.

Metasploitable 2 is connected to the Basement Lab network with address `10.20.0.181/24`.

## 5. Methodology

The assessment began with a review of the Proxmox network configuration to identify virtual bridges and their relationship to the trusted network and Basement lab.

The OPNsense virtual network interfaces and IP addressing were then reviewed to identify the network boundary between the trusted network and the Basement lab.

Kali Linux network addressing and routing were inspected to confirm Kali's placement within the Basement lab and identify its default gateway.

Connectivity testing was then performed from Kali Linux to verify access to the OPNsense LAN interface and test whether systems in the Basement lab could reach infrastructure on the trusted network.

Internet connectivity was tested from Kali Linux by sending ICMP traffic to the public IP address `1.1.1.1`.

Internet connectivity was tested from Metasploitable 2 by sending ICMP traffic to the public IP address `1.1.1.1`.

## 6. Findings

### Finding 1: Metasploitable 2 Had Internet Access

Metasploitable 2, a deliberately vulnerable system within the Basement Lab, was able to reach the public IP address `1.1.1.1`.

Because this system is intentionally insecure, outbound internet access increases the potential impact of a compromise and indicates that the intended network isolation controls require review.

**Severity: Medium**


## 7. Remediation

Review of the OPNsense firewall configuration identified a broad LAN rule permitting traffic from the entire Basement Lab network to any destination.

Kali Linux was assigned a DHCP reservation of `10.20.0.121` based on its MAC address to provide a stable source address for firewall policy.

The broad LAN allow rule was then modified so that outbound access was permitted only from Kali Linux at `10.20.0.121`.

This change removed general outbound internet access from other systems within the Basement Lab while retaining internet connectivity for Kali Linux.

## 8. Validation

Following remediation, connectivity testing was repeated to verify the effectiveness of the firewall change.

Kali Linux retained internet connectivity and was able to reach the public IP address `1.1.1.1`.

Kali Linux also retained connectivity to Metasploitable 2 at `10.20.0.181`, confirming that communication within the Basement Lab remained available.

Metasploitable 2 was unable to reach the public IP address `1.1.1.1`, with 100% packet loss observed.

These results confirm that outbound internet access from Metasploitable 2 has been blocked while the required connectivity for Kali Linux has been preserved.

## 9. Technical Appendix

### Evidence 1: Proxmox Network Configuration
Screenshot captured of Proxmox network configuration.
File: 'evidence-01-proxmox-network.png'

### Evidence 2: OPNsense Network Interfaces
Screenshot captured showing both OPNsense virtual network interfaces and their associated Proxmox bridges.
File: 'evidence-02-opnsense-nics.png'

### Evidence 3: OPNsense Interface Addresses
Screenshot captured from the OPNsense console showing the configured LAN and WAN addresses.
File: 'evidence-03-opnsense-addresses.png'

### Evidence 4: Kali Network Configuration
The Kali Linux system was observed with interface 'eth0' configured as '10.20.0.121/24'.
File: 'evidence-04-kali-ip-address.png'

### Evidence 5: Kali Routing Table
The Kali Linux routing table shows a default route via '10.20.0.1' on interface 'eth0'.
File: 'evidence-05-kali-routing.png'

### Evidence 6: Kali Connectivity to OPNsense
Kali successfully reaches the OPNsense LAN interface at '10.20.0.1' using ICMP.
Four packets were transmitted and four were received with 0% packet loss.
File: 'evidence-06-kali-to-opnsense-ping.png'

### Evidence 7: Kali Connectivity to Trusted Network Gateway

Kali was unable to reach the trusted-network gateway at `192.168.68.1` using ICMP.

Four packets were transmitted and no responses were received.

File: `evidence-07-kali-to-trusted-gateway.png`

### Evidence 8: Kali Internet Connectivity

Kali successfully reached the public IP address `1.1.1.1` using ICMP.

Four packets were transmitted and four were received with 0% packet loss.

File: `evidence-08-kali-internet-connectivity.png`

### Evidence 9: Metasploitable 2 Internet Connectivity

Metasploitable 2 successfully reached the public IP address `1.1.1.1` using ICMP.

This demonstrates that outbound connectivity from the vulnerable host to an external network is currently possible.

File: `evidence-09-meta-internet-connectivity.png`

### Evidence 10: Metasploitable 2 Network Configuration

Metasploitable 2 was observed with interface `eth0` configured as `10.20.0.181/24`.

File: `evidence-10-meta-ip-address.png`

### Evidence 11: OPNsense Firewall Rule Remediation

The OPNsense LAN firewall configuration was modified so that outbound access to any destination is permitted only from Kali Linux at `10.20.0.121`.

The existing rule blocking access from the Basement Lab to the trusted network `192.168.68.0/22` remains in place.

File: `evidence-11-opnsense-firewall-remediation.png`

### Evidence 12: Post-Remediation Kali Connectivity to Metasploitable 2

Following the firewall change, Kali Linux successfully reached Metasploitable 2 at `10.20.0.181` using ICMP.

Four packets were transmitted and four were received with 0% packet loss, confirming that connectivity within the Basement Lab remained available.

File: `evidence-12-kali-to-meta-validation.png`
### Evidence 13: Post-Remediation Metasploitable 2 Internet Connectivity

Following the firewall change, Metasploitable 2 was unable to reach the public IP address `1.1.1.1` using ICMP.

No responses were received and 100% packet loss was observed, confirming that outbound Internet connectivity from Metasploitable 2 had been blocked.

File: `evidence-13-meta-internet-blocked.png`







