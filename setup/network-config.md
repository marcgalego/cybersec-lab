# Lab Network Configuration

## Architecture

All lab machines are connected to a VirtualBox internal network named `cyberlab`. An "Internal Network" has no access to the host's LAN or the internet: it only allows the virtual machines attached to it to communicate with each other.

Kali and the Ubuntu Server also have a second adapter in NAT mode, used only to download packages and tool updates. Metasploitable2 intentionally has no NAT adapter, so it stays fully isolated from the internet. This matters because it is deliberately vulnerable.

## Machines and IP addresses

| Machine         | Internal interface | Internal IP     | Netmask         | NAT adapter             | Role       |
|-----------------|--------------------|-----------------|-----------------|-------------------------|------------|
| Kali Linux      | eth0               | 192.168.50.10   | 255.255.255.0   | Yes (`eth1`, DHCP) | Attacker   |
| Metasploitable2 | eth0               | 192.168.50.11   | 255.255.255.0   | No                      | Target     |
| Ubuntu Server   | enp0s3             | 192.168.50.12   | 255.255.255.0   | Yes (`enp0s8`, DHCP)    | Web target |

## Static IP configuration

VirtualBox does not attach a DHCP server to an Internal Network by default, so nothing assigns IP addresses automatically: each machine's IP address has to be set manually. Fixed addresses are also convenient in a lab, since scans, notes and exploits always point to the same target.

A temporary address (e.g. `ip addr add`) is lost on reboot, so the configuration was written to each system's network configuration to make it persistent, so the addresses don't have to be reassigned every time a VM boots. Each OS uses a different network manager, so the method differs between machines.

### Kali (NetworkManager)

```bash
sudo nmcli connection modify "eth0" ipv4.addresses 192.168.50.10/24 ipv4.method manual
sudo nmcli connection up "eth0"
```

- `connection modify "eth0"` edits the NetworkManager connection profile named `eth0`.
- `ipv4.method manual` disables DHCP on this profile and uses the given address.
- The profile is stored on disk (`/etc/NetworkManager/system-connections/`), so it persists across reboots.
- `connection up` re-activates the profile to apply the change immediately.

No gateway or DNS is set, since the internal network has no route to the outside.

### Metasploitable2 (`/etc/network/interfaces`)

Metasploitable2 is based on Ubuntu 8.04, it uses `ifupdown` configuration instead of netplan or NetworkManager.

```bash
sudo nano /etc/network/interfaces
```

The default `iface eth0 inet dhcp` line was replaced with a static configuration:

```text
auto eth0
iface eth0 inet static
    address 192.168.50.11
    netmask 255.255.255.0
```

Apply and verify:

```bash
sudo /etc/init.d/networking restart
ifconfig eth0
```

### Ubuntu Server (netplan)

The Ubuntu Server uses netplan with `systemd-networkd`. The configuration file `/etc/netplan/70-cyberlab-interna.yaml` defines both interfaces:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.50.12/24
    enp0s8:
      dhcp4: true
```

Apply and verify:

```bash
sudo netplan apply
ip -4 addr show enp0s3
```

## Troubleshooting: IP conflict on the cloned Ubuntu Server VM

The Ubuntu Server VM was cloned from an existing university lab VM. The clone carried over a leftover netplan file (`60-red-interna.yaml`) that statically assigned `192.168.50.1/24` to `enp0s8`, the NAT adapter. That address was in the same subnet as enp0s3, so two interfaces claimed the same network and traffic to the lab could be routed out through the NAT adapter.

**Fix:**
1. Renamed the conflicting file to `.bak` so netplan no longer reads it (kept for reference instead of deleting it).
2. Created `70-cyberlab-interna.yaml` (shown above), explicitly defining both interfaces.

**Lesson:** cloning a VM saves setup time, but it can silently carry over network configuration that conflicts with the new environment. On any cloned Linux VM, check `/etc/netplan/` before assuming a clean network state.

## Connectivity verification

|      From       |          To          |          Command          |            Result            |
|-----------------|----------------------|---------------------------|------------------------------|
| Kali            | Metasploitable2      | `ping -c 4 192.168.50.11` | 4/4 received, 0% packet loss |
| Kali            | Ubuntu Server        | `ping -c 4 192.168.50.12` | 4/4 received, 0% packet loss |
| Metasploitable2 | Ubuntu Server        | `ping -c 4 192.168.50.12` | 4/4 received, 0% packet loss |
| Metasploitable2 | Internet (`8.8.8.8`) | `ping -c 2 8.8.8.8`       | Network is unreachable       |

All machines reach each other inside `192.168.50.0/24`, and Metasploitable2 has no path to the internet, which confirms the isolation.