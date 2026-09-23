# cybersec-lab

Personal cybersecurity lab built on VirtualBox to practise network configuration, reconnaissance, exploitation and mitigation in a fully isolated environment.

## Lab overview

| Machine         | Role       | Internal IP   |
|-----------------|------------|---------------|
| Kali Linux      | Attacker   | 192.168.50.10 |
| Metasploitable2 | Target     | 192.168.50.11 |
| Ubuntu Server   | Web target | 192.168.50.12 |

All machines share a VirtualBox internal network (`cyberlab`, `192.168.50.0/24`). Metasploitable2, which is deliberately vulnerable, has no internet access. Kali and Ubuntu Server have a separate NAT adapter used only for updates.

## Repository structure

- [`setup/network-config.md`](setup/network-config.md): network architecture, static IP configuration per machine, a troubleshooting case from a cloned VM, and connectivity/isolation checks.

## Status

- [x] Three VMs installed and connected on an isolated internal network
- [x] Persistent static IPs configured on each machine
- [x] Connectivity and isolation verified
- [ ] Planned: first exercise against Metasploitable2 (Nmap reconnaissance, exploitation and mitigation)
- [ ] Planned: deploy vulnerable web applications (DVWA, OWASP Juice Shop) on Ubuntu Server with Docker

## Tools

VirtualBox · Kali Linux · Metasploitable2 · Ubuntu Server · netplan · NetworkManager · Git / GitHub

## Disclaimer

All testing is performed in an isolated virtual network, against machines I own, for educational purposes only.

