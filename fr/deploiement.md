---
title: Déploiement
---

<a name="introduction"></a>
## Introduction

Une application BlitzPHP peut être exécutée de différentes manières: hébergée sur un serveur Web, en utilisant la virtualisation, ou en utilisant l’outil de ligne de commande Klinge pour les tests. Cette section explique comment utiliser chaque technique et explique certains des avantages et des inconvénients d’entre elles.

> **Attention**  
> Vous devez toujours faire attention à la **casse** des noms de fichiers. Beaucoup les développeurs développent sur des systèmes de fichiers insensibles à la casse sous Windows ou macOS. Toutefois, la plupart des environnements serveur utilisent des systèmes de fichiers sensibles à la casse. Si la casse du nom de fichier est incorrecte, le code qui fonctionne en locale ne fonctionnera pas sur le serveur.

<a name="exigeances-du-serveur"></a>
## Exigéances du serveur

Le framework BlitzPHP a quelques exigences système. Vous devez vous assurer que votre serveur Web dispose de la version et des extensions PHP minimales suivantes:

<div class="content-list" markdown="1">

- PHP `>= 8.1`
- L'extension [`mbstring`](https://www.php.net/manual/en/mbstring.requirements.php)
- L'extension [`intl`](https://www.php.net/manual/en/intl.requirements.php)
- L'extension [`json`](https://www.php.net/manual/en/json.requirements.php)
- L'extension [`json`](https://www.php.net/manual/en/json.requirements.php)
- L'extension [`cURL`](https://www.php.net/manual/en/curl.requirements.php) si vous devez utiliser le [client http](/docs/{version}/client-http)
- L'extension [`simplexml`](https://www.php.net/manual/en/simplexml.requirements.php) si vous voulez utiliser le formattage xml
- L'extension [`memcache`](https://www.php.net/manual/en/memcache.requirements.php) si vous utilisez le gestionnaire `Memcached` pour [gérer votre cache](/docs/{version}/cache)
- L'extension [`redis`](https://github.com/phpredis/phpredis) si vous utilisez le gestionnaire `Redis` pour [gérer votre cache](/docs/{version}/cache)
- L'extension `PDO` si vous prévoyez [interagir avec une base de données](/docs/{version}/base-de-donnees)

</div>

<a name="configuration-initiale"></a>
## Configuration initiale

1. Ouvrez le fichier `app/Config/app.php` avec un éditeur de texte et définissez votre URL de base sur `base_url`. Si vous avez besoin de plus de flexibilité, l’URL de base peut être défini dans le fichier `.env` de cette manière: `app.baseURL = 'http://example.com/'`. (Utilisez toujours une barre oblique de fin sur votre URL de base!)

> **Note**  
> Si vous définissez mal votre URL de base, en mode développement, la barre de débogage peut ne pas se charger correctement et les pages Web peuvent prendre considérablement plus long à afficher.

2. Si vous avez l’intention d’utiliser une base de données, ouvrez le fichier `app/Config/database.php` avec un éditeur de texte et définissez votre paramètres de base de données. Alternativement, ceux-ci peuvent être définis dans votre fichier `.env`.

3. Si votre application ne se trouve pas sur le serveur de production, définissez la valeur de la variable `ENVIRONMENT` à `development` dans le fichier `.env` pour tirer parti des outils de débogage fournis. 

> **Attention**  
> Dans les environnements de production, vous devez désactiver l’affichage des erreurs et toute autre fonctionnalité de développement. Dans BlitzPHP, cela peut être fait en définissant l’environnement sur « `production` ».

> **Note**  
> Si vous utilisez votre site à l’aide d’un serveur Web (par exemple, Apache ou Nginx), Vous devrez modifier les autorisations pour le dossier `storage` à l’intérieur votre projet, afin qu’il soit accessible en écriture par l’utilisateur ou le compte utilisé par votre serveur web.

<a name="nginx"></a>
## Serveur de développement local

BlitzPHP est livré avec un serveur de développement local, tirant parti du serveur Web intégré de PHP avec le routage BlitzPHP. Vous pouvez le lancer, avec la ligne de commande suivante à partir du dossier principal :

```shell
php klinge serve
```

Cela lancera le serveur et vous pouvez maintenant afficher votre application dans votre navigateur à l'adresse `http://localhost:3300`.

> **Note**  
> Le serveur de développement intégré ne doit être utilisé que sur les ordinateurs de développement locaux. Il ne devrait JAMAIS être utilisé sur un serveur de production.

Le serveur de développement local peut être personnalisé avec trois options de ligne de commande:

* Vous pouvez utiliser l'option `--host` pour spécifier un hôte différent pour exécuter l'application:

```shell
php klinge serve --host=example.dev
```

* Par défaut, le serveur s'exécute sur le port **3300**, mais vous pouvez avoir plusieurs sites en cours d'exécution ou avoir déjà une autre application utilisant ce port. Vous pouvez utiliser l'option `--port` pour en spécifier un autre:

```shell
php klinge serve --port=8000
```

* Vous pouvez également spécifier une version spécifique de PHP à utiliser, avec l'option `--php`, dont la valeur définie sur le chemin de l'exécutable PHP que vous souhaitez utiliser:

```shell
php klinge serve --php=/usr/bin/php8.7.6.5
```

> **Note**  
> Si vous devez exécuter le site sur un hôte autre que localhost (en utilisant l'option `--host`), vous devez d'abord ajouter l'hôte souhaité à votre fichier `hosts`. L'emplacement exact de ce fichier varie dans chacun des principaux systèmes d'exploitation, bien que tous les systèmes de type Unix (y compris OS X) conservent généralement le fichier dans `/etc/hosts`. sur Windows ce fichier se trouve à `C:\WINDOWS\system32\drivers\etc\hosts`.

<a name="hebergement-avec-apache"></a>
## Hébergement avec Apache

Une application BlitzPHP est normalement hébergée sur un serveur web. Apache HTTP Server est la plate-forme « standard » et elle est reprise dans une grande partie de notre documentation.

<a name="configurer-le-fichier-de-configuration-principal"></a>
### Configurer le fichier de configuration principal

<a name="activation-de-mod-rewrite"></a>
#### Activation de mod_rewrite

Avant toute chose, assurez-vous que le module de réécriture est activé (sans commentaire) dans le Fichier de configuration, par exemple, `apache2/conf/httpd.conf`

```apacheconf
LoadModule rewrite_module modules/mod_rewrite.so
```

<a name="definition-du-dossier-racine"></a>
#### Définition du dossier racine

Assurez-vous également que l'élément `<Directory>` de la racine du document par défaut le permet également, dans le paramètre `AllowOverride` :

```apacheconf
<Directory "/opt/lamp/apache2/htdocs">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

<a name="hebergement-avec-les-hotes-virtuels"></a>
### Hébergement avec les hôtes virtuels

Nous vous recommandons d'utiliser « l'hébergement virtuel » pour exécuter vos applications. Vous pouvez configurer différents alias pour chacune des applications sur lesquelles vous travaillez.

<a name="activation-du-module-d-alias-des-vhost"></a>
#### Activation du module d'alias des vhost

 Assurez-vous que le module d'hébergement virtuel est activé (non commenté) dans le fichier de configuration principal, par exemple `apache2/conf/httpd.conf` :

```apacheconf
LoadModule vhost_alias_module modules/mod_vhost_alias.so
```

<a name="ajout-d-un-alias-d-hote"></a>
#### Ajout d'un alias d'hôte

Ajoutez un alias d'hôte dans votre fichier « hosts », généralement situé à `/etc/hosts` sur les plates-formes de type Unix, ou `C:\WINDOWS\system32\drivers\etc\hosts` sous Windows. Ajoutez une ligne au fichier. Cela peut être `monprojet.local` ou `monprojet.test`, par exemple :

```ini
127.0.0.1 monprojet.local
```

> **Attention**  
Rassurez vous d'avoir mis un domaine fictif (qui n'existe pas réellement) sinon vous ne pourriez plus accéder au "vrai" site lorsque vous seriez connecté à internet.

<a name="configuration-de-l-hote-virtuel"></a>
#### Configuration de l'hôte virtuel

Ajoutez un élément `<VirtualHost>` pour votre application Web dans le fichier de configuration d'hébergement virtuel, par exemple `apache2/conf/extra/httpd-vhost.conf` :

```apacheconf
<VirtualHost *:80>
    DocumentRoot "/opt/lamp/apache2/monprojet/public"
    ServerName   monprojet.local
    ErrorLog     "logs/monprojet-error_log"
    CustomLog    "logs/monprojet-access_log" common

    <Directory "/opt/lamp/apache2/monprojet/public">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

La configuration ci-dessus suppose que le dossier du projet se trouve comme suit :

apache2/  
   ├── monproject/      (Dossier du projet)  
   │      └── public/  (DocumentRoot pour   monprojet.local)  
   └── htdocs/

<a name="test-de-fonctionnement"></a>
#### Test de fonctionnement

Avec la configuration ci-dessus, votre application Web sera accessible avec l'URL `http://myproject.local/` dans votre navigateur.   
Apache doit être redémarré chaque fois que vous modifiez sa configuration.

<a name="hebergement-avec-sous-dossier"></a>
### Hébergement avec sous-dossier

Si vous voulez une URL de base comme `http://localhost/monprojet/` avec un sous-dossier, il existe trois manières.

<a name="creation-d-un-lien-symbolique"></a>
#### Création d'un lien symbolique 

Placez votre dossier de projet comme suit, où `htdocs` est la racine du document Apache :

```
├── monprojet/ (dossier du projet)  
│      └── public/  
└── htdocs/
```

Accédez au dossier `htdocs` et créez un lien symbolique comme suit :

```shell
cd htdocs/
ln -s ../monprojet/public/ monprojet
```

<a name="utilisation-d-alias"></a>
#### Utilisation d'alias

Placez votre dossier de projet comme suit, où `htdocs` est la racine du document Apache :

```
├── monprojet/ (dossier du projet)  
│      └── public/  
└── htdocs/
```

Ajoutez ce qui suit dans le fichier de configuration principal, par exemple `apache2/conf/httpd.conf` :

```apacheconf
Alias /monprojet /opt/lamp/apache2/monprojet/public
<Directory "/opt/lamp/apache2/monprojet/public">
    AllowOverride All
    Require all granted
</Directory>
```

Redémarrez Apache.

<a name="ajout-de-htaccess"></a>
#### Ajout de .htaccess

Le dernier recours consiste à ajouter `.htaccess` à la racine du projet.

Il n'est pas recommandé de placer le dossier du projet à la racine du document. Cependant, si vous n'avez pas d'autre choix, comme sur un serveur partagé, vous pouvez l'utiliser.

Placez votre dossier de projet comme suit, où `htdocs` est la racine du document Apache, et créez le fichier `.htaccess` :

```
└── htdocs/    
    └── myproject/ (dossier du projet)  
        ├── .htaccess  
        └── public/
```
Et modifiez `.htaccess` comme suit :

```apacheconf
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule ^(.*)$ public/$1 [L]
</IfModule>

<FilesMatch "^\.">
    Require all denied
    Satisfy All
</FilesMatch>
```

<a name="hebergement-avec-nginx"></a>
## Hébergement avec Nginx

Nginx est le deuxième serveur HTTP le plus utilisé pour l'hébergement Web. Vous trouverez ici un exemple de configuration utilisant PHP 8.1 FPM (sockets Unix) sous Ubuntu Server.


```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name example.com;

    root  /var/www/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    error_page 404 /index.php;

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="optimisation"></a>
## Optimisation

<a name="optimisation-de-l-autoloader"></a>
### Optimisation de l'Autoloader 

Lors du déploiement en production, assurez-vous d'optimiser la carte des classes chargées par Composer afin qu'il puisse trouver rapidement le fichier approprié à charger pour une classe donnée :

```shell
composer install --optimize-autoloader --no-dev
```

> **Note**  
> En plus d'optimiser le chargeur automatique, vous devez toujours vous assurer d'inclure un fichier `composer.lock` dans le référentiel de contrôle de code source de votre projet. Les dépendances de votre projet peuvent être installées beaucoup plus rapidement lorsqu'un fichier `composer.lock` est présent.
