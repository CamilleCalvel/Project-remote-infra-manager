# Supervision avec l'agent zabbix natif

### Windows

### Linux

# SNMPv2

### Windows sans utiliser la communauté

### Linux en utilisant la communauté

# SNMPv3

## Introduction à SNMPv3

SNMPv3 est une version sécurisée du protocole **Simple Network Management Protocol**, apportant :

- Authentification renforcée (SHA, MD5, etc.)
- Chiffrement des échanges (AES, DES…)
- Gestion d’accès granulaire par utilisateur

Contrairement à SNMPv2, cette version empêche la lecture des données en texte clair et garantit la confidentialité et l’intégrité des échanges.

***

## Prérequis

1. Avoir un hôte **Ubuntu supervisé via SNMPv2**.
2. Installer le paquet suivant :
```bash
apt install libsnmp-dev
```

3. Stopper le service SNMP avant de créer l’utilisateur :
```bash
service snmpd stop
```


***

## Création de l’utilisateur SNMPv3

Sur l’hôte supervisé, exécute la commande :

```bash
net-snmp-config --create-snmpv3-user -ro -a SHA-512 -A mon-super-mot-de-passe -x AES -X ma-super-phrase-privee authPrivUser
```
<p align="center">
<img src="https://github.com/user-attachments/assets/f76f2440-9d0d-4644-855a-a0b50bf287d3" alt="Téléchargements Zabbix" width="1200">
</p>

**Explication des options :**


| Option | Description |
| :-- | :-- |
| `-ro` | Accès en lecture seule |
| `-a SHA-512` | Algorithme de hachage pour l’authentification |
| `-A mon-super-mot-de-passe` | Mot de passe d’authentification |
| `-x AES` | Algorithme de chiffrement |
| `-X ma-super-phrase-privee` | Phrase secrète pour le chiffrement |
| `authPrivUser` | Nom de l’utilisateur SNMPv3 |

Vérifie la création de l'utilisateur dans le fichier  `/etc/snmp/snmpd.conf`.

<p align="center">
<img src="https://github.com/user-attachments/assets/5b1d0384-cf3f-4a0f-a8ba-cde899d0a331" alt="Téléchargements Zabbix" width="400">
</p>

Vérifie la création de l'utilisateur dans le fichier `/var/lib/snmp/snmpd.conf`.

<p align="center">
<img src="https://github.com/user-attachments/assets/4f487d30-8bb9-4cac-9b0f-9804564ce0cb" alt="Téléchargements Zabbix" width="800">
</p>

Relance le service :

```bash
service snmpd start
service snmpd status
```
Après le redémarrage, modification dans le fichier `/var/lib/snmp/snmpd.conf`.

<p align="center">
<img src="https://github.com/user-attachments/assets/4d9b8d89-d8ff-4515-95e4-50720d88085d" alt="Téléchargements Zabbix" width="1100">
</p>

***

## Test du fonctionnement local SNMPv3

Teste localement avec :

```bash
snmpget -v 3 -u authPrivUser -l AuthPriv -a SHA-512 -A mon-super-mot-de-passe -x AES -X ma-super-phrase-privee 127.0.0.1 1.3.6.1.2.1.1.1.0
```

Le résultat doit afficher le nom du système supervisé.

<p align="center">
<img src="https://github.com/user-attachments/assets/73767ee3-8cfe-4de4-9ee0-15e52460f895" alt="Téléchargements Zabbix" width="1500">
</p>


***

## Test réseau avec l’adresse IP

Remplace `127.0.0.1` par l’adresse réseau de la machine supervisée :

```bash
snmpget -v 3 -u authPrivUser -l AuthPriv -a SHA-512 -A mon-super-mot-de-passe -x AES -X ma-super-phrase-privee 192.168.19.100 1.3.6.1.2.1.1.1.0
```

Si la commande retourne le nom du système, la communication SNMPv3 est fonctionnelle entre l’hôte et le serveur Zabbix.

<p align="center">
<img src="https://github.com/user-attachments/assets/af83fc3d-0476-4f67-bc59-b9b691604330" alt="Téléchargements Zabbix" width="1500">
</p>

***

## Configuration SNMPv3 dans Zabbix

1. Accède à l’interface web de Zabbix.
2. Ouvre **Configuration → Hôtes**.
3. Sélectionne ton hôte, puis choisis :
    - **Version SNMP :** SNMPv3.
4. Renseigne les champs demandés :
    - Nom d’utilisateur : `authPrivUser`
    - Niveau de sécurité : `authPriv`
    - Authentification : `SHA-512`
    - Phrase d’authentification et de chiffrement selon tes paramètres.
5. Clique sur **Actualiser**.

<p align="center">
<img src="https://github.com/user-attachments/assets/b568b028-ee67-4681-b746-8e0d1fc63c26" alt="Téléchargements Zabbix" width="900">
</p>

***

## Vérification de la supervision

Pour vérifier que SNMPv3 fonctionne correctement :

1. Dans **Collecte de données → Hôtes → Éléments**, sélectionnez un item (comme *Uptime*).
2. Clique sur **Exécuter maintenant**.

<p align="center">
<img src="https://github.com/user-attachments/assets/5c89ee4b-c2ff-49d1-8b56-7c7ea0b914cc" alt="Téléchargements Zabbix" width="1200">
</p>

3. Le message `Requête envoyée avec succès` doit apparaître.  
<p align="center">
<img src="https://github.com/user-attachments/assets/0e6ebfbe-8306-41de-a257-aa5ebd8937e7" alt="Téléchargements Zabbix" width="1200">
</p>

4. Allez dans **Surveillance → Dernières données**, filtrez les informations affichées en tapant le nom de votre hôte dans le champ Hôtes et le mot « uptime » dans le champ Nom.
5. Sur l’un des noms Uptime et choisissez d’afficher les 500 dernières valeurs
<p align="center">
<img src="https://github.com/user-attachments/assets/b61552a6-ce46-4d90-b897-af2fa21574d2" alt="Téléchargements Zabbix" width="1200">
</p>

Si les valeurs les plus récentes affichées sont postérieures à la configuration de l’hôte en SNMPv3 alors c’est que la surveillance via SNMPv3 s’exécute correctement actuellement
<p align="center">
<img src="https://github.com/user-attachments/assets/f3e4e79d-77a3-4af1-9759-0adb4dbca7b3" alt="Téléchargements Zabbix" width="1500">
</p

***

## Vérification via les logs (optionnel)

Tu peux aussi rechercher les interactions SNMPv3 dans les logs de Zabbix :

```bash
grep -i snmpv3 /var/log/zabbix/zabbix_server.log
```

Si des lignes indiquent `snmpv3securityname`, `snmpv3authpassphrase`, etc., la configuration est bien en place.

<p align="center">
<img src="https://github.com/user-attachments/assets/e4c18462-ec24-4366-918c-ccb4f1e54aff" alt="Téléchargements Zabbix" width="1700">
</p

***

## Résumé rapide

| Étape | Commande ou action |
| :-- | :-- |
| Installation | `apt install libsnmp-dev` |
| Stop service | `service snmpd stop` |
| Création utilisateur | `net-snmp-config --create-snmpv3-user …` |
| Démarrage service | `service snmpd start` |
| Test local | `snmpget … 127.0.0.1 …` |
| Test réseau | `snmpget … 192.168.X.X …` |
| Configuration Zabbix | Interface web → SNMPv3 |
| Vérification | Uptime ou logs SNMPv3 |
