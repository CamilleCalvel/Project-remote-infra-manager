# Supervision avec l'agent zabbix natif

### Windows

### Linux

# SNMPv2

### Windows sans utiliser la communauté

### Linux en utilisant la communauté

# SNMPv3


## Vérification de la supervision

Pour vérifier que SNMPv3 fonctionne correctement :

1. Dans **Collecte de données → Hôtes → Éléments**, sélectionnez un item (comme *Uptime*).
2. Clique sur **Exécuter maintenant**.



3. Le message `Requête envoyée avec succès` doit apparaître.  


4. Allez dans **Surveillance → Dernières données**, filtrez les informations affichées en tapant le nom de votre hôte dans le champ Hôtes et le mot « uptime » dans le champ Nom.
5. Sur l’un des noms Uptime et choisissez d’afficher les 500 dernières valeurs


Si les valeurs les plus récentes affichées sont postérieures à la configuration de l’hôte en SNMPv3 alors c’est que la surveillance via SNMPv3 s’exécute correctement actuellement
<p align="center">
<img src="https://github.com/user-attachments/assets/f3e4e79d-77a3-4af1-9759-0adb4dbca7b3" alt="Téléchargements Zabbix" width="1500">
</p

# SNMPv3 – Mise en œuvre sécurisée

## 📌 Introduction

SNMPv3 (**Simple Network Management Protocol version 3**) est la version sécurisée du protocole SNMP. Elle apporte trois atouts majeurs :

- 🔐 **Authentification sécurisée** (SHA, MD5…)
- 🔒 **Chiffrement des échanges** (AES, DES…)
- 👤 **Contrôle d’accès par utilisateur**

Contrairement à SNMPv2, les échanges ne sont plus transmis en texte clair, garantissant **confidentialité**, **intégrité** et **authenticité** des données.

---

## ✅ Prérequis

1. Un hôte fonctionnant sous **Ubuntu avec SNMPv2** déjà opérationnel.
2. Installez les outils nécessaires :

```bash
apt install libsnmp-dev
```

3. Arrêtez le service SNMP avant toute modification :

```bash
service snmpd stop
```

---

## 👤 Création d’un utilisateur SNMPv3

Sur l’hôte supervisé, exécutez :

```bash
net-snmp-config --create-snmpv3-user -ro -a SHA-512 -A mon-super-mot-de-passe -x AES -X ma-super-phrase-privee authPrivUser
```

<p align="center">
<img src="https://github.com/user-attachments/assets/f76f2440-9d0d-4644-855a-a0b50bf287d3" alt="Téléchargements Zabbix" width="1200">
</p>

### ✔️ Explication rapide des options

| Option | Rôle |
| :-- | :-- |
| `-ro` | Accès lecture seule |
| `-a SHA-512` | Algorithme d’authentification |
| `-A` | Mot de passe d’authentification |
| `-x AES` | Algorithme de chiffrement |
| `-X` | Phrase secrète de chiffrement |
| `authPrivUser` | Nom de l’utilisateur |

🛠️ **Vérifiez la création de l’utilisateur dans les fichiers suivants :**

- `/etc/snmp/snmpd.conf`
  
 <p align="center">
<img src="https://github.com/user-attachments/assets/5b1d0384-cf3f-4a0f-a8ba-cde899d0a331" alt="Téléchargements Zabbix" width="400">
</p>

- `/var/lib/snmp/snmpd.conf`

<p align="center">
<img src="https://github.com/user-attachments/assets/4f487d30-8bb9-4cac-9b0f-9804564ce0cb" alt="Téléchargements Zabbix" width="800">
</p>

Puis redémarrez le service SNMP :

```bash
service snmpd start
service snmpd status
```

<p align="center">
<img src="https://github.com/user-attachments/assets/4d9b8d89-d8ff-4515-95e4-50720d88085d" alt="Téléchargements Zabbix" width="1100">
</p>

---

## 🧪 Test local de SNMPv3

Vérifiez le bon fonctionnement en local :

```bash
snmpget -v 3 -u authPrivUser -l AuthPriv -a SHA-512 -A mon-super-mot-de-passe -x AES -X ma-super-phrase-privee 127.0.0.1 1.3.6.1.2.1.1.1.0
```

Le résultat doit afficher des informations sur le système (ex : nom de l’hôte).

<p align="center">
<img src="https://github.com/user-attachments/assets/73767ee3-8cfe-4de4-9ee0-15e52460f895" alt="Téléchargements Zabbix" width="1500">
</p>

---

## 🌐 Test réseau avec adresse IP

Testez depuis l’extérieur :

```bash
snmpget -v 3 -u authPrivUser -l AuthPriv -a SHA-512 -A mon-super-mot-de-passe -x AES -X ma-super-phrase-privee 192.168.11.101 1.3.6.1.2.1.1.1.0
```

✅ Si la commande renvoie les informations système, **SNMPv3 fonctionne entre l’hôte et le serveur Zabbix**.

<p align="center">
<img src="https://github.com/user-attachments/assets/af83fc3d-0476-4f67-bc59-b9b691604330" alt="Téléchargements Zabbix" width="1500">
</p>

---

## 🖥️ Configuration de SNMPv3 dans Zabbix

1. Accédez à l’interface Zabbix.
2. `Configuration → Hôtes → Votre hôte`.
3. Sélectionnez :
   - **Version SNMP :** SNMPv3
   - **Utilisateur :** `authPrivUser`
   - **Sécurité :** `authPriv`
   - **Authentification :** SHA-512
   - **Mot de passe & chiffrement :** ceux définis précédemment
4. Enregistrez puis faites **Actualiser**.

<p align="center">
<img src="https://github.com/user-attachments/assets/b568b028-ee67-4681-b746-8e0d1fc63c26" alt="Téléchargements Zabbix" width="900">
</p>
---

## 🔍 Vérification dans Zabbix

### Méthode n°1 – Test d’un Item

1. `Collecte de données → Hôtes → Items`
2. Sélectionnez un item, ex : **Uptime**
3. Cliquez sur **Exécuter maintenant**
<p align="center">
<img src="https://github.com/user-attachments/assets/5c89ee4b-c2ff-49d1-8b56-7c7ea0b914cc" alt="Téléchargements Zabbix" width="1200">
</p>
4. ✅ Message attendu : `Requête envoyée avec succès`

<p align="center">
<img src="https://github.com/user-attachments/assets/0e6ebfbe-8306-41de-a257-aa5ebd8937e7" alt="Téléchargements Zabbix" width="1200">
</p>

### Méthode n°2 – Dernières données

1. `Surveillance → Dernières données`
2. Filtrez avec votre **hôte** et recherchez **uptime**
3. <p align="center">
<img src="https://github.com/user-attachments/assets/b61552a6-ce46-4d90-b897-af2fa21574d2" alt="Téléchargements Zabbix" width="1200">
</p>
4. Vérifiez que les données sont récentes ➜ **SNMPv3 fonctionne**

---

## 📁 Optionnel – Vérifier dans les logs Zabbix

```bash
grep -i snmpv3 /var/log/zabbix/zabbix_server.log
```

<p align="center">
<img src="https://github.com/user-attachments/assets/e4c18462-ec24-4366-918c-ccb4f1e54aff" alt="Téléchargements Zabbix" width="1700">
</p>

Vous pouvez y trouver :
- `snmpv3securityname`
- `snmpv3authpassphrase`
- `snmpv3privpassphrase`

---

## 🧭 Résumé rapide

| Étape | Commande / Action |
| :-- | :-- |
| Installation | `apt install libsnmp-dev` |
| Stop service | `service snmpd stop` |
| Création utilisateur | `net-snmp-config --create-snmpv3-user …` |
| Start service | `service snmpd start` |
| Test local | `snmpget … 127.0.0.1` |
| Test réseau | `snmpget … 192.168.X.X` |
| Paramétrage Zabbix | Interface web → SNMPv3 |
| Vérification | Uptime ou logs Zabbix |

---

✔️ **SNMPv3 est maintenant configuré de manière sécurisée avec chiffrement, authentification et supervision dans Zabbix.**

