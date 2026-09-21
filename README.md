# Serveur web Debian 13 : nginx, MariaDB, PHP et pare-feu UFW

Mise en place d'une pile web **LEMP** (Linux, nginx, MariaDB, PHP-FPM) sur **Debian 13**, administrée à distance en SSH, puis premier durcissement réseau avec le pare-feu **UFW** : tout le trafic entrant est refusé, sauf le SSH.

## Ce qui a été réalisé

| Étape | Ce que j'ai fait | Capture |
|---|---|---|
| 1 | Connexion SSH depuis Windows PowerShell, puis passage en `root` avec `su -` | [01](images/01-connexion-ssh-et-installation-ufw.png) |
| 2 | Installation de la pile web : nginx, MariaDB, PHP 8.4-FPM et son module MySQL | [02](images/02-installation-nginx-mariadb-php.png) |
| 3 | Installation d'UFW, puis politique « tout refuser en entrée, sauf SSH » | [01](images/01-connexion-ssh-et-installation-ufw.png) et [03](images/03-ufw-ssh-uniquement.png) |

Système : Debian GNU/Linux 13, noyau 6.12.

## Les commandes

**Connexion et passage administrateur**

```bash
ssh utilisateur@adresse-du-serveur     # depuis PowerShell
su -                                   # devenir root
```

**Pile web**

```bash
apt install nginx mariadb-server php-fpm php-mysql -y
```

- `nginx` : serveur web ;
- `mariadb-server` : base de données ;
- `php-fpm` : exécution de PHP, appelée par nginx ;
- `php-mysql` : connexion de PHP à MariaDB.

**Pare-feu UFW**

```bash
apt install ufw -y
ufw default deny incoming     # tout refuser en entrée
ufw default allow outgoing    # tout autoriser en sortie
ufw allow ssh                 # sauf le SSH (22/tcp, IPv4 et IPv6)
ufw enable                    # activer, y compris au démarrage
ufw status verbose            # vérifier
```

## Vérification

`ufw status verbose` doit afficher :

```
Status: active
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

Le principe : **moindre privilège réseau**. Un service n'est joignable que si une règle l'autorise explicitement.

## À savoir

- Avec cette politique, **nginx n'est pas encore joignable de l'extérieur** : les ports 80 et 443 sont fermés. Il faut ajouter `ufw allow 'Nginx Full'` (ou ouvrir 80 et 443) le jour où le site doit être servi.
- Activer UFW peut couper une session SSH existante : c'est pourquoi la règle SSH est ajoutée **avant** `ufw enable`.

## Pistes d'amélioration (non réalisées ici)

- Sécuriser MariaDB (`mariadb-secure-installation`) et créer un utilisateur dédié par application.
- Passer le SSH en authentification par clé, interdire le mot de passe et la connexion directe de `root`.
- Bloquer les tentatives répétées avec `fail2ban`.
- Configurer un bloc serveur nginx et le HTTPS (Let's Encrypt).
- Activer les mises à jour de sécurité automatiques.

## Compétences illustrées

Administration Linux (Debian), accès distant SSH, installation de services avec `apt`, pare-feu UFW, réflexion sur la surface d'exposition d'un serveur.
