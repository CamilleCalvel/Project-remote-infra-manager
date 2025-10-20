# Mise en place du laboratoire virtualisé Proxmox
<p align="center">
<img src="https://github.com/user-attachments/assets/b1110e3a-a99e-46e3-85e4-9b8ba662131b" alt="Pictures" width="2500" >
</p>

## Plan d'adressage

| Nom machine                | Adresse IP            | Passerelle       | Commentaire      |
|----------------------------|-----------------------|------------------|------------------|
| **Stormshield-site1**      | 192.168.0.10/24       | 192.168.0.1      | WAN              |
|                            | 192.168.10.10/24      |                  | LAN-SERVEURS     |
|                            | 192.168.11.10/24      |                  | DMZ              |
|                            | 192.168.12.10/24      |                  | LAN-CLIENTS      |
| **Stormshield-site2**      | 192.168.0.11/24       | 192.168.0.1      | WAN              |
|                            | 192.168.13.10/24      |                  | LAN-SERVEURS     |
|                            | 192.168.14.10/24      |                  | DMZ              |
|                            | 192.168.15.10/24      |                  | LAN-CLIENTS      |
| **deb13-server-site1**     | 192.168.10.101/24     | 192.168.10.10    | LAN-SERVEURS     |
| **WinServer-site1**        | 192.168.10.102/24     | 192.168.10.10    | LAN-SERVEURS     |
| **ubuntu-serverDMZ-site1** | 192.168.11.101/24     | 192.168.11.10    | DMZ              |
| **ubuntu-server-site2**    | 192.168.13.101/24     | 192.168.13.10    | LAN-SERVEURS     |
| **ubuntu-client-site1**    | DHCP                  | 192.168.12.10    | LAN-CLIENTS      |



## Stormshield
### Mise en place de Stormshield
### Paramétrage des interfaces réseaux
### Mise en place du Nat
### VPN site à site
### Dhcp server
### Port forwarding
