# 🌐 Project-remote-infra-manager

## 🎯 Objectif du projet  
Mettre en place une infrastructure informatique complète et sécurisée ainsi qu’une solution de monitoring avancée pour l’entreprise **Biodélisse**. 

## 🏢 Présentation de Biodélisse 
**Biodélisse** est un groupe agro-alimentaire innovant, implanté dans le sud-ouest de la France, avec des sites de production à **Toulouse** et **Aire-sur-l'Adour**. Spécialistes dans la production de produits alimentaires biologiques, nous mettons un point d'honneur à allier **qualité**, **durabilité** et **respect de l’environnement**.  

<p align="center">
<img src="https://github.com/user-attachments/assets/582f7ecb-05e8-456d-be63-a275fe37691c" alt="Pictures" width="350" >
</p>

## 🖧 Infrastructure du Système d’Information

L’entreprise dispose d’un système d’information centralisé réparti entre ses deux sites :

- Nom de domaine interne : **Biodélisse.local**
- Annuaire Active Directory pour la gestion des utilisateurs et des permissions
- Services réseau (imprimantes, partages de fichiers, serveurs applicatifs)
- Pare-feu **Stormshield** certifiés **ANSSI** pour assurer filtrage, IDS/IPS et haute disponibilité
- Interconnexion sécurisée des deux sites via **VPN site-à-site Stormshield**

  
## 📡 Schéma d’Infrastructure Réseau

<p align="center">
<img src="https://github.com/user-attachments/assets/7e62e254-81af-43ae-8ddf-ea8ee55d6b1d" alt="Pictures" width="1500" >
</p>

## ⚙️ Avancement du Projet

✅ **Mise en place du laboratoire virtualisé Proxmox**  
✅ **Installation et configuration d’un serveur de supervision Zabbix sur Debian**  
✅ **Supervision de serveurs Linux et Windows :**
- Agent **Zabbix** natif
- **SNMP** 

## 🛠️ Technologies et Outils Utilisés

| Domaine | Outils / Technologies |
|---------|------------------------|
| Virtualisation | Proxmox |
| Supervision | Zabbix Server, Zabbix Agent, SNMP |
| Sécurité | Pare-feu Stormshield, VPN site-to-site |
| Systèmes | Debian, Windows Server |
