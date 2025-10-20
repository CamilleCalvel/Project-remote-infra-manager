# Mise en place du laboratoire virtualisé Proxmox
<p align="center">
<img src="https://github.com/user-attachments/assets/b1110e3a-a99e-46e3-85e4-9b8ba662131b" alt="Pictures" width="2500" >
</p>

<details><summary><h2>Configuration réseau sur Proxmox</h2></summary>  

<details><summary><h3>Plan d'adressage</h3></summary>  

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

</details>

<details><summary><h3>Mise en place du bridge réseau</h3></summary>

**🎯 Objectif** : Configurer les **bridges réseau** sur le nœud Proxmox **SRV-AIS2** afin de permettre aux machines virtuelles ou conteneurs d'accéder au réseau physique.

<p align="center">
<img src="https://github.com/user-attachments/assets/7ba4b82b-bdb6-4b42-93aa-0b9689ec9d96" alt="Pictures" width="700" >
</p>

#### Étape 1 : Création du bridge

<p align="center">
<img src="https://github.com/user-attachments/assets/8e0fc111-0090-444d-acbc-2cd5a0be7069" alt="Pictures" width="500" >
</p>

1. **Accéder à l’interface Proxmox en mode “Server View”**
   - Dans le menu déroulant situé en haut à gauche de l’interface, sélectionner **`Server View`** pour afficher l’ensemble des datacenters et des nœuds.

2. **Sélectionner le nœud concerné**
   - Dans l’arborescence à gauche, cliquer sur le nœud souhaité (exemple : **`SRV-AIS2`**) pour y appliquer la configuration réseau.

3. **Ouvrir la section “Network”**
   - Dans le menu latéral du nœud, cliquez sur l’onglet "Network" (icône d’interconnexion).

4. **Créer une nouvelle interface réseau**
   - En haut à droite, cliquer sur le bouton **`Create`**.
   - Une liste s’affiche avec les types d’interfaces disponibles.

5. **Choisir “Linux Bridge”**
   - Dans la liste déroulante, sélectionner **`Linux Bridge`** pour commencer la création du pont réseau permettant aux VM/LXC de communiquer avec le réseau physique.


#### Étape 2 : Configuration du bridge

<p align="center">
<img src="https://github.com/user-attachments/assets/7ce5359f-caf7-455e-84d7-5a721b14b750" alt="Pictures" width="700" >
</p>

1. **Nommer le bridge**  
   - Saisissez un nom explicite dans le champ **`Name`** (exemple : `vmbr10`).

2. **Définir le réseau IPv4**  
   - Renseignez l’adresse du réseau dans le champ **`IPv4/CIDR`** (exemple : `192.168.10.0/24`).

3. **Définir les options de démarrage**  
   - Cochez **`Autostart`** pour que le bridge démarre automatiquement avec le système.

4. **Ajouter un commentaire**  
   - Précisez l’usage dans le champ **`Comment`** (exemple : `Réseau LAN Serveurs`).

5. **Valider la création**  
   - Une fois tous les champs remplis, cliquez sur le bouton **`Create`** pour finaliser la création du bridge.
     
6. **Activer la configuration**
   - Cliquez sur **`Apply Configuration`** pour activer le bridge.


</details>
</details>
<details><summary><h2>Stormshield</h2></summary>
  
### Mise en place de Stormshield
### Paramétrage des interfaces réseaux
### Mise en place du Nat
### VPN site à site
### Dhcp server
### Port forwarding

</details>
