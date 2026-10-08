#Home Lab Network Segmentation Validation

## Overview

This project assessed network segmentation within a personal cybersecurity home lab. The aim was to verify that the various systems were correctly positioned within the network and that their access to other areas of the network was appropriately configured.
Particularly, since security testing uses deliberately vulnerable machines such as Metasploitable 2, network isolation of such elements is critical to reduce the risk of exposing trusted systems to threats originating within the lab.
During the assessment, we tested connectivity between the various machines, the trusted network and the public internet, and examined how that connectivity related to the firewall rules configured within OPNsense.

## Technologies Used

- **Proxmox VE:** Virtualisation platform hosting the lab infrastructure.

- **OPNsense:** Open-source firewall and routing platform providing network segmentation, traffic filtering and DHCP services.

- **Kali Linux:** A Debian-based distribution designed for penetration testing and security auditing.

- **Ubuntu Server:** A Debian-based distribution designed for server environments.

- **Metasploitable 2:** An intentionally vulnerable virtual machine running an older Ubuntu Linux distribution.

## Outcome

The assessment identified unintended outbound internet access from a deliberately vulnerable system. Firewall policy was subsequently modified, and connectivity testing verified that the intended restrictions were effective.

## Documentation

[Read the full technical assessment](Report/homelab-network-segmentation-validation.md)
