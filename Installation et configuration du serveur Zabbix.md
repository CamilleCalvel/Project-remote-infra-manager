# Contexte du projet

Dans le cadre de l’amélioration de la gestion et de la supervision de son infrastructure répartie sur plusieurs sites, **Biodélisse** met en place une solution centralisée de supervision.  
Pour atteindre cet objectif, l’entreprise prévoit l’installation de **Zabbix** sur un serveur dédié.  

Cette étape constitue le **socle technique indispensable** à la supervision globale de l’ensemble des sites, permettant de centraliser les alertes, le suivi des performances et la maintenance proactive des équipements.

---
<details><summary><h1>1. Installation de Zabbix</h1></summary>  

## 1.1 Préparation et téléchargement

Pour commencer, rendez-vous sur le site officiel de Zabbix, dans la section **Téléchargements**, et sélectionnez les paramètres adaptés à votre environnement :

- Version de Zabbix
- Système d’exploitation (OS)
- Version de l’OS
- Composants Zabbix à installer
- Serveur de base de données
- Serveur web

<p align="center">
<img src="https://github.com/user-attachments/assets/cf8cc24c-65b2-46af-8490-450463b00451" alt="Téléchargements Zabbix" width="1100">
</p>

> Dans ce document, la version **7.0 LTS** a été installée sur une machine **Debian 13** avec le serveur web **Nginx**.  

---

### 1.1.1 Ajout du dépôt Zabbix aux dépôts APT du serveur

Téléchargez et installez le paquet `zabbix-release` pour Debian 11, qui ajoute le dépôt officiel de Zabbix à votre système. Puis mettez à jour la liste des paquets :

```bash
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian13_all.deb
export PATH=$PATH:/usr/local/sbin:/usr/sbin:/sbin
dpkg -i zabbix-release_latest_7.0+debian13_all.deb
apt update
```
---

### 1.1.2 Installation des paquets Zabbix

Installez les paquets Zabbix :

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

#### Description des paquets :

- **zabbix-server-mysql** : serveur Zabbix avec prise en charge de MySQL  
- **zabbix-frontend-php** : interface Web pour Zabbix basée sur PHP  
- **zabbix-nginx-conf** : configuration Nginx pour Zabbix  
- **zabbix-sql-scripts** : scripts SQL pour la création et mise à jour de la base de données Zabbix  
- **zabbix-agent** : agent Zabbix pour surveiller les hôtes distants

---

### 1.1.3 Configuration de la base de données du serveur

#### Création de la base et de l'utilisateur Zabbix

```sql
mysql -u root -p
```

Dans le prompt MariaDB :

```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
flush privileges;
quit;
```

---

### 1.1.4 Importation de la structure de base de données initiale

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -u zabbix -p
```

> Entrez le mot de passe créé précédemment.

---

### 1.1.5 Désactivation de l’option dangereuse

```sql
mysql -u root -p
set global log_bin_trust_function_creators = 0;
quit;
```

---

### 1.1.6 Configuration du serveur de traitement Zabbix

Ouvrez le fichier de configuration de Zabbix Server :

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

- Modifiez `DBPassword` pour inclure le mot de passe de la base de données Zabbix  
- Modifiez `DBName` si vous avez changé le nom par défaut

---

### 1.1.7 Configuration du serveur web de Zabbix

Si Apache est installé, arrêtez-le et désactivez-le :

```bash
systemctl stop apache2
systemctl disable apache2
```

Éditez le fichier Nginx :

```bash
sudo nano /etc/zabbix/nginx.conf
```

- Décommentez les 2 premières lignes :  

```nginx
listen 8080;
server_name example.com;
```

Redémarrez les services et activez-les au démarrage :

```bash
systemctl restart zabbix-server zabbix-agent nginx php8.4-fpm
systemctl enable zabbix-server zabbix-agent nginx php8.4-fpm
```

---

### 1.1.8 Installation de l’interface de gestion Zabbix

Ouvrez votre navigateur sur :

```
http://<IP-de-votre-serveur>:8080
```

ou

```
http://<nom-de-votre-serveur>:8080
```

1. Choisissez la langue puis cliquez sur **Prochaine étape**  
2. Vérifiez les prérequis puis cliquez à nouveau sur **Prochaine étape**  
3. Configurez la connexion à la base de données :  
   - Hôte : `localhost`  
   - Port : valeur par défaut  
   - Nom de la base : `zabbix`  
   - Utilisateur : `zabbix`  
   - Mot de passe : `password`  
4. Indiquez le nom du serveur Zabbix (identique au `hostname`)  
5. Cliquez sur **Prochaine étape** jusqu'à arriver sur la page de résumé  
6. Cliquez sur **Prochaine étape** pour finaliser l’installation

> Identifiants par défaut :  
> - Utilisateur : `Admin` (A majuscule)  
> - Mot de passe : `zabbix`

</details>

<details><summary><h1>2. Installation du Proxy Zabbix</h1></summary>  

Dans le cadre de la supervision de notre **site distant (Site 2)**, cette procédure décrit l’installation et la configuration d’un **proxy Zabbix**.  
Ce proxy permet de remonter les informations de supervision vers le serveur Zabbix principal, situé sur notre infrastructure centrale.

Le serveur proxy utilisé pour cette installation est une machine **Ubuntu 24.04**, intégrée au **LAN serveurs** de notre architecture réseau.

📎 [**Schéma réseau associé**](https://github.com/CamilleCalvel/Project-remote-infra-manager?tab=readme-ov-file#-sch%C3%A9ma-dinfrastructure-r%C3%A9seau)

## 2.1 Devenir utilisateur root

Démarrer une session shell avec les privilèges root :

``` bash
sudo -s
```

## 2.2 Installer le dépôt Zabbix

<p align="center">
<img src="https://github.com/user-attachments/assets/5d0177c1-afc4-43e8-b487-968bd2fed7bf" alt="Téléchargements Zabbix" width="1100">
</p>

> Dans ce document, la version **7.0 LTS** a été installée sur une machine **Ubuntu 24.04**.   

Télécharger le paquet du dépôt officiel Zabbix :

``` bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
apt update
```

## 2.3 Installer Zabbix Proxy (avec support MySQL/MariaDB)

``` bash
apt install zabbix-proxy-mysql zabbix-sql-scripts
```

## 2.4 Créer la base de données initiale pour le Proxy

⚠️ Assurez-vous qu'un serveur MySQL/MariaDB est installé et fonctionnel.
> Dans cette procédure, la base de données **MariaDB** a été installée

### a. Se connecter à MySQL

``` bash
mysql -u root -p
```

### b. Créer la base et l'utilisateur

``` sql
create database zabbix_proxy character set utf8mb4 collate utf8mb4_bin;
create user 'zabbix'@'localhost' identified by 'password';
grant all privileges on zabbix_proxy.* to 'zabbix'@'localhost';
set global log_bin_trust_function_creators = 1;
flush privileges;
quit;
```

## 2.5 Importer le schéma de base de données

``` bash
cat /usr/share/zabbix-sql-scripts/mysql/proxy.sql | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix_proxy
```

## 2.6 Désactiver l'option `log_bin_trust_function_creators`

``` bash
mysql -u root -p
```

``` sql
set global log_bin_trust_function_creators = 0;
quit;
```

## 2.7 Configuration du Proxy Zabbix 

Cette étape consiste à configurer le fichier `zabbix_proxy.conf` afin de permettre au proxy d’établir la communication avec le serveur Zabbix principal et la base de données locale.  

```bash
nano /etc/zabbix/zabbix_proxy.conf
```

### 📌 Paramètres essentiels à modifier

```ini
############################
# Mode de fonctionnement du Proxy
############################
# 0 = Proxy actif (envoie les données de lui-même au serveur Zabbix)
# 1 = Proxy passif (attend que le serveur Zabbix vienne récupérer les données)
ProxyMode=1                         # Mode passif

############################
# Serveur Zabbix principal
############################
Server=192.168.10.101               # Adresse IP du serveur Zabbix principal
#Server=192.168.10.101:10051        # Optionnel : préciser le port si différent du port par défaut (10051/TCP)

############################
# Identification du Proxy
############################
Hostname=Zabbix-proxy-site2         # Nom déclaré dans l'interface du serveur Zabbix

############################
# Fichiers de journalisation
############################
LogFile=/var/log/zabbix/zabbix_proxy.log   # Fichier de logs du proxy
LogFileSize=0                               # 0 = taille illimitée

# Niveau de journalisation :
# 0 = désactivé / 1 = critique / 2 = erreur / 3 = avertissement
# 4 = informations détaillées / 5 = mode debug maximum
DebugLevel=4

############################
# Processus et sockets
############################
PidFile=/run/zabbix/zabbix_proxy.pid
SocketDir=/run/zabbix

############################
# Base de données utilisée par le Proxy
############################
DBHost=127.0.0.1                    # Adresse du serveur MariaDB/MySQL
DBName=zabbix_proxy                # Nom de la base créée précédemment
DBUser=zabbix                     # Utilisateur de la base
DBPassword=password               # Mot de passe associé

############################
# Supervision SNMP (optionnel)
############################
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log

############################
# Paramètres réseau et commandes externes
############################
Timeout=4                         # Temps d’attente (en secondes) pour une réponse d’un agent
FpingLocation=/usr/bin/fping     # Chemin vers la commande fping (ICMP)
Fping6Location=/usr/bin/fping6   # Pour les requêtes ICMPv6

############################
# Requêtes lentes et accès aux statistiques
############################
LogSlowQueries=3000               # Enregistre les requêtes SQL > 3000 ms
StatsAllowedIP=127.0.0.1          # Adresse(s) IP autorisée(s) à accéder aux statistiques du proxy
```

## 2.8 Redémarrer et activer le service Proxy Zabbix 

``` bash
systemctl restart zabbix-proxy
systemctl enable zabbix-proxy
```
## 2.9 Ajouter le Proxy dans le serveur Zabbix

Une fois le service **Zabbix Proxy** installé, configuré et démarré, il doit être déclaré dans l’interface du serveur Zabbix principal pour que la communication soit possible.

### 🖥️ Étapes dans l’interface Zabbix

1. Connectez-vous à l’interface web du serveur Zabbix (avec un compte administrateur).
2. Accédez au menu :  
   **Administration → Proxys → Create proxy**
3. Renseignez les champs suivants :

| Champ              | Description |
|--------------------|-------------|
| **Proxy name**     | Nom du proxy (doit correspondre exactement à la valeur `Hostname=` définie dans `zabbix_proxy.conf`, ex : `Zabbix-proxy-site2`) |
| **Proxy mode**     | Mode de fonctionnement du proxy : <br>• **Active** : le proxy envoie les données vers le serveur Zabbix.<br>• **Passive** : le serveur Zabbix vient collecter les données (mode configuré ici : `ProxyMode=1`). |
| **Interface**      | Adresse IP ou nom d’hôte du proxy. Le port par défaut utilisé est **10051/TCP**. |

<p align="center">
<img src="https://github.com/user-attachments/assets/ec654e60-7bf2-45f8-a885-d8d8758da062" alt="Création Proxy Zabbix" width="700">
</p>

- Une fois ajouté, le proxy apparaîtra avec le statut **"En attente"** jusqu’à ce qu’il envoie ses premières données au serveur.

<p align="center">
<img src="https://github.com/user-attachments/assets/de809ca8-87a0-4a45-a425-57ff24aad89e" alt="Téléchargements Zabbix" width="800">
</p>

</details>

<details><summary><h1>3. Activer le chiffrement du trafic entre le serveur Zabbix et son Proxy</h1></summary>

Par défaut, les communications entre le serveur Zabbix et ses agents ou proxys ne sont pas chiffrées.  
Cette section décrit comment activer un **chiffrement symétrique via une clé PSK (Pre-Shared Key)**.

## 3.1 Génération de la clé partagée PSK

1. Générer la clé sur le proxy et l'enregistrer dans un fichier sécurisé :

```bash
openssl rand -hex 128 > /usr/local/etc/zabbix_proxy.psk
```

2. Restreindre les droits d'accès au fichier :

```bash
chown zabbix.zabbix /usr/local/etc/zabbix_proxy.psk
chmod 400 /usr/local/etc/zabbix_proxy.psk
```

- `chown zabbix.zabbix` : attribue le fichier à l'utilisateur et au groupe `zabbix`.  
- `chmod 400` : seul le propriétaire peut lire le fichier.

## 3.2 Configuration du Proxy Zabbix

Ajouter les paramètres suivants dans `/etc/zabbix/zabbix_proxy.conf` :

```ini
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=proxyzabbix
TLSPSKFile=/usr/local/etc/zabbix_proxy.psk
```

- **TLSConnect** : active TLS avec méthode PSK pour la connexion sortante vers le serveur.  
- **TLSAccept** : active TLS sur les connexions entrantes.  
- **TLSPSKIdentity** : nom du proxy utilisé pour l’authentification TLS.  
- **TLSPSKFile** : chemin vers le fichier clé PSK.

Redémarrer le service proxy :

```bash
systemctl restart zabbix-proxy
```

Pour vérifier la clé générée :

```bash
cat /usr/local/etc/zabbix_proxy.psk
```

## 3.3 Configuration dans l'interface web Zabbix

1. Aller dans **Administration → Proxys**.  
2. Sélectionner le proxy souhaité et ouvrir l’onglet **Chiffrement**.

<p align="center">
<img src="https://github.com/user-attachments/assets/5ff7bf6b-f236-4f32-b144-3cc80fed18ca" alt="Chiffrement Proxy Zabbix" width="700">
</p>

3. Activer le chiffrement **PSK**.  
4. Dans **Identité PSK**, entrer le même nom que `TLSPSKIdentity` du fichier proxy (ex : `proxyzabbix`).  
5. Coller la clé partagée dans le champ **PSK**.

<p align="center">
<img src="https://github.com/user-attachments/assets/2bd6b4e5-79c7-4bdb-a6f2-503b0659fae2" alt="Configuration PSK" width="650">
</p>

## 3.4 Vérification du chiffrement

### Vérification côté serveur

```bash
grep -ni "TLS" /var/log/zabbix/zabbix_server.log
```

Les logs doivent indiquer une connexion TLS établie, par exemple :

```
End of zbx_tls_connect(): SUCCEED (established TLSv1.3 ...)
```

<p align="center">
<img src="https://github.com/user-attachments/assets/ebde56f8-bc0a-45b6-a428-b0fcf70b1060" alt="Vérification TLS Proxy" width="850">
</p>

### Vérification côté proxy

```bash
grep -ni "TLS" /var/log/zabbix/zabbix_proxy.log
```

Les résultats doivent également montrer l’établissement de la connexion TLS.

</details>

<details><summary><h1>4. Sécurisation de l'accès à l'interface web en HTTPS</h1></summary>
Pour sécuriser l'accès à l'interface web de Zabbix, il est recommandé d'activer **HTTPS (SSL/TLS)** via Nginx.

## 4.1 Génération de la clé de chiffrement et des certificats

1. Générez une clé privée RSA 2048 bits pour le serveur Web Nginx avec OpenSSL :

```bash
openssl genpkey -algorithm RSA -out private.key -pkeyopt rsa_keygen_bits:2048
```

2. Créez une CSR (demande de signature de certificat) en configurant les informations de votre organisme :

```bash
openssl req -new -key private.key -out certificate.csr
```

> **Astuce :** le "Common Name" (CN) doit correspondre au nom d'hôte utilisé pour accéder au serveur web.

3. Générez un certificat auto-signé valide 365 jours :

```bash
openssl x509 -req -in certificate.csr -signkey private.key -out certificate.crt -days 365
```

4. Déplacez les fichiers vers des emplacements adaptés :

```bash
mv certificate.crt /etc/ssl/certs/certificate_zabbix_server.crt
mv certificate.csr /etc/ssl/certs/certificate_zabbix_server.csr
mv private.key /etc/ssl/private/private_zabbix_server.key
```

---

## 4.2 Configuration de Nginx

1. Ouvrez le fichier de configuration du site par défaut :

```bash
nano /etc/nginx/sites-available/default
```

2. Localisez le bloc `server` et ajoutez/modifiez les directives SSL :

```nginx
ssl_certificate /etc/ssl/certs/certificate_zabbix_server.crt;
ssl_certificate_key /etc/ssl/private/private_zabbix_server.key;
listen 443 ssl;
```

<p align="center">
<img src="https://github.com/user-attachments/assets/7f23cd6b-4960-4eda-bacd-feaeb10512ca" alt="Vérification TLS Proxy" width="850">
</p>

4. Testez la configuration Nginx :

```bash
nginx -t
# ou
/usr/sbin/nginx -t
```

5. Rechargez Nginx pour appliquer la configuration :

```bash
systemctl reload nginx
```

---

## 4.3 Configuration de Zabbix sous Nginx

1. Modifiez le fichier de configuration de Zabbix (souvent `/etc/nginx/conf.d/zabbix.conf`).

2. Ajoutez un bloc serveur pour servir Zabbix via HTTPS :

```nginx
server {
    listen 443 ssl;
    server_name fqdn_du_server;
    ssl_certificate /etc/ssl/certs/certificate_zabbix_server.crt;
    ssl_certificate_key /etc/ssl/private/private_zabbix_server.key;

    location / {
        proxy_pass http://127.0.0.1:8080; # redirection vers le serveur Zabbix sur le port 8080
        include /etc/nginx/proxy_params;
    }
}
```

> Si vous n'avez pas de résolution DNS, remplacez `server_name` par l'adresse IP du serveur.

<p align="center">
<img src="https://github.com/user-attachments/assets/e7b9da8e-1774-45a8-a343-296d5cd594fd" alt="Vérification TLS Proxy" width="850">
</p>

3. Vérifiez la syntaxe :

```bash
nginx -t
# ou
/usr/sbin/nginx -t
```

4. Redémarrez Nginx pour appliquer les modifications :

```bash
systemctl restart nginx
```

---

## 4.4 Connexion en HTTPS

- Accédez à l'interface web de Zabbix via :

```text
https://debian-server-11.lab.lan
```

ou

```text
https://192.168.19.11
```

> Le port 8080 n'est plus nécessaire.  
> Le navigateur affichera un avertissement car le certificat est auto-signé, poursuivez la connexion.

- La connexion sera sécurisée via HTTPS, et vous pourrez vérifier le certificat directement dans le navigateur.
  
--- 

✅ L’accès web de Zabbix est désormais chiffré, assurant la confidentialité des informations échangées.
