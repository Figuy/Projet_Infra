# Projet_Infra

Groupe : Teddy Le Moal, Hugo Salinas--Marty, Victoria Pasquaud

Projet : Hébergement Collaboratif – Serveur Web & Cloud

Créer une solution d’hébergement combinant un serveur web et un cloud privé, permettant le stockage, le partage de fichiers et la gestion des utilisateurs.


1- Installation d’Apache ou Nginx pour héberger l’application
Configuration de PHP et de la base de données
Mise en place d’un proxy inverse (optionnel(Nginx/Traefik)) pour la gestion des accès
Installation et configuration de Nextcloud

2 -Installation via paquets ou docker-compose
Configuration des utilisateurs et permissions
Intégration avec la base de données
Sécurisation du serveur

3 -Activation du pare-feu (UFW/IPTables)
Installation de Fail2ban pour protéger contre les attaques
Mise en place d’un certificat SSL avec Let's Encrypt
Optimisation et accessibilité

4 -Ajout du support WebDAV pour un accès distant aux fichiers
Configuration d’un système de sauvegarde automatique
Possibilité d’ajouter une authentification multi-facteurs (2FA)