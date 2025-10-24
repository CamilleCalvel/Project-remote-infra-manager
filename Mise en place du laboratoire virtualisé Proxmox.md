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
<img src="https://github.com/user-attachments/assets/3e76b6ce-6d7c-4f5e-9ab3-b440d136eed9" alt="Pictures" width="850" >
</p>

---  

### Étape 1 : Création du bridge

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

---  

### Étape 2 : Configuration du bridge

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

---

### Étape 3 : Associer le bridge à une VM dans Proxmox

<p align="center">
<img src="https://github.com/user-attachments/assets/035b9329-775d-462d-859d-6a2347fc04b7" alt="Pictures" width="800" >
</p>

1. **Accéder à la configuration de la VM**  
   - Dans l’arborescence de Proxmox, sélectionnez la machine virtuelle concernée (exemple : `Virtual Machine 1001 (stormshield-site1)`) puis rendez-vous sur l’onglet **`Hardware`**.

2. **Ajouter ou modifier une interface réseau**  
   - Vous pouvez soit **ajouter une nouvelle interface réseau** (`Add` → `Network Device`) soit **éditer une interface existante** en la sélectionnant puis en cliquant sur **`Edit`**.

3. **Sélectionner le bridge**  
   - Dans la fenêtre de configuration, sélectionnez le **bridge réseau** voulu dans le champ **`Bridge`** (exemple : `vmbr10`).

4. **Valider l’ajout ou la modification**  
   - Cliquez sur **`Create`** ou **`OK`** pour valider l’association du bridge à l’interface réseau de la VM.

</details>
</details>
<details><summary><h2>Stormshield</h2></summary>
  
### Mise en place de Stormshield



### 🔧 Configuration des interfaces réseau — Stormshield

#### ✅ 1. Prérequis et droits d’accès

Avant toute modification via l’interface web Stormshield :

- Assurez-vous que votre compte dispose des **droits en écriture**.
- Si besoin, modifiez les permissions en cliquant sur votre **profil administrateur (en haut à droite)** de l’interface web.

---

#### 🌐 2. Accéder à la gestion des interfaces

1. Connectez-vous à l’interface web du pare-feu Stormshield.  
2. Accédez au menu : **Configuration → Network → Interfaces**

#### ⚙️ 3. Paramétrage des interfaces

**Statut de l’interface**  
| Option | Description |
|--------|-------------|
| **ON** | Active l'interface et permet l’acheminement du trafic. |
| **OFF** | Désactive l’interface — aucun trafic ne passe. |

💡 *N’activez que les interfaces réellement utilisées.*

---

**Nom de l’interface**    
- Indiquez un nom **clair et explicite**.
- Exemple : `LAN-SERVERS`, `DMZ-WEB`, `WAN-FIBER`.

---

**Type d’interface**  
| Type | Usage | Description |
|------|-------|-------------|
| **Interne (protégée)** | Réseau local sécurisé | LAN, serveurs internes, VLAN. |
| **Externe (publique)** | Réseau non maîtrisé / Internet | WAN, lien vers FAI. |

---

**Mode d’attribution IP**  
| Mode | Description | Recommandé pour |
|------|-------------|------------------|
| **Statique (IP fixe)** | Adresse IP manuelle, ne change pas | Serveurs, pare-feu, routeurs |
| **DHCP (dynamique)** | Adresse IP attribuée automatiquement par un serveur DHCP | Postes utilisateurs, appareils mobiles, équipements temporaires |

---

**Adresse IP et masque**  
- Configurez l’adresse IPv4 et son masque (ex : `192.168.10.10 / 255.255.255.0`).
- Respectez le **plan d’adressage IP de l’infrastructure**.

---

**Commentaires**  
- Ajoutez des informations utiles : rôle, VLAN, plage réseau, remarques techniques.
- Utile pour la **maintenance, le support et les audits**.

<p align="center">
<img src="https://github.com/user-attachments/assets/ae453763-cdc0-4dc0-bcad-805180eee7ac" alt="Pictures" width="800" >
</p>

### Mise en place du Nat
### VPN site à site
### Dhcp server
### Port forwarding

<p align="center">
<img src="" alt="Pictures" width="800" >
</p>

</details>
