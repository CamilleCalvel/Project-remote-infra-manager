# Contexte du projet

Dans le cadre de l’amélioration de la gestion et de la supervision de son infrastructure répartie sur plusieurs sites, **Biodélisse** met en place une solution centralisée de supervision.  
Pour atteindre cet objectif, l’entreprise prévoit l’installation de **Zabbix** sur un serveur dédié.  

Cette étape constitue le **socle technique indispensable** à la supervision globale de l’ensemble des sites, permettant de centraliser les alertes, le suivi des performances et la maintenance proactive des équipements.

---

# 1. Installation de Zabbix

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
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -u zabbix -p
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
