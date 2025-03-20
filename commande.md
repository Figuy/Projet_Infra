# 1. Installation de Nextcloud sur Rocky Linux

## Commande installer : 

- sudo dnf update -y
- sude dnf tar


## install ip fixe

```powershell
sudo nmtui
```

## Changement de port 

```powershell
firewall-cmd --list-all
sudo firewall-cmd --permanent --add-port=774/tcp
sudo firewall-cmd --reload
systemctl restart sshd
```


## Installation des dépendances

```powershell
sudo dnf install -y httpd mariadb-server php php-mysqlnd php-xml php-mbstring php-json php-gd php-curl
sudo systemctl enable --now httpd mariadb
wget https://download.nextcloud.com/server/releases/nextcloud-31.0.0.zip
```


## Configuration de MariaDB

```powershell
sudo mysql_secure_installation
```

Connexion à MariaDB et création de la base de données :
```powershell
sudo mysql -u root -p
```
```sql
CREATE DATABASE nextcloud;
CREATE USER 'nextcloud_user'@'localhost' IDENTIFIED BY 'mon_mot_de_passe';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## Téléchargement et installation de Nextcloud

```bash

cd /var/www/

wget https://download.nextcloud.com/server/releases/latest.tar.bz2
```

Extraction et déplacement des fichiers :
```bash
tar -xjf latest.tar.bz2
sudo mv nextcloud /var/www/html/
```

## Configuration des permissions

```bash
sudo chown -R apache:apache /var/www/html/nextcloud
sudo chmod -R 755 /var/www/html/nextcloud
```

## Configuration d'Apache

Création d'un fichier de configuration :
```bash
sudo nano /etc/httpd/conf.d/nextcloud.conf
```

Ajout du contenu suivant :
```apache
<VirtualHost *:80>
    ServerName yourdomain.com
    DocumentRoot /var/www/nextcloud

    <Directory /var/www/nextcloud/>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/nextcloud_error.log
    CustomLog /var/log/httpd/nextcloud_access.log combined
</VirtualHost>
```

Redémarrer Apache :
```bash
sudo systemctl restart httpd
```

## Finalisation de l'installation

Accéder à l'interface web de Nextcloud :
```
http://<IP_DE_TON_SERVEUR>
```

Créer un compte administrateur et configurer la base de données avec :
- Nom de la base : `nextcloud`
- Nom d'utilisateur : `nextcloud_user`
- Mot de passe : `mon_mot_de_passe`
- Hôte : `localhost`


## Vérification

Tester l'accès à Nextcloud :
```bash
systemctl status httpd mariadb
```


## Sécurisation

### Firewall

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

### Fail2Ban 
``` powershell
sudo dnf install fail2ban -y

sudo nano /etc/fail2ban/jail.local
```

Ajouter ceci :

```powershell
[apache-auth]
enabled = true
filter = apache-auth
action = iptables-multiport[name=Apache, port="http,https"]
logpath = /var/log/httpd/*error.log
maxretry = 3
bantime = 3600
```
On redémarre fail2ban 

```bash
sudo systemctl enable --now fail2ban
```
### MAJ autonome

```powershell
sudo dnf install -y dnf-automatic
sudo nano /etc/dnf/automatic.conf
sudo systemctl enable --now dnf-automatic-install.timer
systemctl list-timers --all | grep dnf
```

### Problème de version de php

```powershell
sudo dnf install dnf-utils -y
sudo dnf install http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
 dnf module list php
sudo dnf module reset php -y
sudo dnf module enable php:8.4 -y
php -v
sudo dnf module enable php:remi-8.4 -y
sudo dnf install php php-cli php-common php-mysqlnd php-fpm php-json php-curl php-mbstring php-zip php-xml php-bcmath -y
php -v
sudo systemctl restart httpd
```

### Augmentation de la mémoire php 

```powershell
php -i | grep memory_limit
sudo nano /etc/php-fpm.d/www.conf
sudo systemctl restart httpd
sudo systemctl restart php-fpm
```

# Installation et Configuration de Prometheus

## 1. Installation de Prometheus

```bash
sudo wget https://github.com/prometheus/prometheus/releases/download/v3.2.1/prometheus-3.2.1.linux-amd64.tar.gz
sudo tar -xvf prometheus-*.linux-amd64.tar.gz
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
```

## 2. Configuration de Prometheus

### Création du dossier de configuration
```bash
sudo mkdir -p /etc/prometheus
```

### Édition du fichier de configuration
```bash
sudo nano /etc/prometheus/prometheus.yml
```

Ajout du contenu suivant :
```powershell
global:
  scrape_interval: 15s  # Fréquence de collecte des données

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

---

## 3. Configuration du Service Systemd

### Création du fichier de service
```bash
sudo nano /etc/systemd/system/prometheus.service
```

Ajout du contenu suivant :
```powershell
[Unit]
Description=Prometheus
After=network.target

[Service]
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus
Restart=always
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target
```

### Activation et Démarrage du Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
sudo systemctl status prometheus
```

---

# Modification du fichier de configuration de Prometheus

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Ajout du contenu suivant :
```powershell
global:
  scrape_interval: 15s  # Fréquence de collecte des données

scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'prometheus'
    static_configs:
      - targets: ['193.250.5.69:9091']
```

---

# Installation et Configuration de Node Exporter

## 1. Installation de Node Exporter
```bash
sudo dnf install -y node_exporter
cd node_exporter-1.9.0.linux-amd64/
sudo mv node_exporter /usr/local/bin/
```

## 2. Création du fichier de service
```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Ajout du contenu suivant :
```powershell
[Unit]
Description=Node Exporter
After=network.target

[Service]
ExecStart=/usr/local/bin/node_exporter
Restart=always
User=prometheus
Group=prometheus

[Install]
WantedBy=multi-user.target
```

## 3. Activation et Démarrage de Node Exporter
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter
```

---

# Installation et Configuration de Loki et Promtail

## 1. Installation de Loki et Promtail
```bash
sudo dnf install loki promtail
```

## 2. Configuration de Promtail

### Modification du fichier de configuration
```bash
sudo mv /etc/promtail-config.yml /etc/promtail/promtail.yml
sudo nano /etc/promtail/promtail.yml
```

Ajout du contenu suivant :
```powershell
server:
  http_listen_port: 9080
  grpc_listen_port: 9095

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push  

scrape_configs:
  - job_name: system-logs
    static_configs:
      - targets: ['localhost']
        labels:
          job: varlogs
          path: /var/log/*log
```

---

# Création du fichier de service pour Promtail

```bash
sudo nano /etc/systemd/system/promtail.service
```

Ajout du contenu suivant :
```powershell
[Unit]
Description=Promtail service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=promtail
ExecStart=/usr/bin/promtail -config.file /etc/promtail/promtail.yml

TimeoutSec=60
Restart=on-failure
RestartSec=2

[Install]
WantedBy=multi-user.target
```

---

# Permissions et Démarrage du Service

Ajout des permissions nécessaires :
```powershell
sudo usermod -aG root promtail
sudo chmod 644 /var/log/messages
sudo systemctl daemon-reload
sudo systemctl enable --now promtail
sudo systemctl status promtail
```
