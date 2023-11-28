---
title: Installation
---

<a name="decouvrez-blitzphp"></a>
## Découvrez BlitzPHP

Par définition, un framework Web fournit une structure et un point de départ pour la création de votre application, vous permettant de vous concentrer sur la création de quelque chose d’incroyable pendant que nous transpirons les détails. BlitzPHP en est un, ayant une syntaxe expressive et élégante pour une expérience unique. 

BlitzPHP s'inspire des meilleurs framework existant pour fournir aux développeurs une expérience incroyable tout en fournissant des fonctionnalités puissantes telles que l’injection de dépendances, une couche d’abstraction de base de données expressive, des mécanismes de validations de données, etc.

Que vous soyez nouveau dans les frameworks Web PHP ou que vous ayez des années d’expérience, BlitzPHP a été conçu pour évoluer avec vous. Nous vous aiderons à faire vos premiers pas en tant que développeur Web ou vous donnerons un coup de pouce alors que vous faites passer votre expertise au niveau supérieur.

<a name="pourquoi-blitzphp"></a>
### Pourquoi BlitzPHP?

Il existe une variété d’outils et de frameworks à votre disposition lors de la création d’une application Web. Néanmoins, nous pensons que BlitzPHP peut être un choix judicieux pour créer des applications Web modernes et complètes. En effet, BlitzPHP fédère les meilleurs pratiques et fonctionnalités de ses pères pour vous fournir un cadre encore plus éficace.

## Votre premier projet BlitzPHP

Avant de créer votre premier projet BlitzPHP, vous devez vous assurer que votre machine locale dispose de PHP et [Composer](https://getcomposer.org) installé. De plus, bien qu'étant facultatif, nous recommandons d'[installer de Node et NPM](https://nodejs.org).

Après avoir installé PHP et Composer, vous pouvez créer un nouveau projet BlitzPHP via la commande Composer `create-project` :

```shell
composer create-project blitz-php/app-skeleton example-app
```

Vous pouvez également créer de nouveaux projets BlitzPHP en installant globalement le programme d’installation BlitzPHP via Composer. 

```shell
composer global require blitz-php/installer

blitz new example-app
```

Une fois le projet créé, démarrez le serveur de développement local de BlitzPH à l’aide de la commande Klinge `serve`:

```shell
cd example-app

php klinge serve
```

Une fois que vous avez démarré le serveur de développement BlitzPHP, votre application sera accessible dans votre navigateur Web à l’adresse `http://localhost:3300`. Ensuite, vous êtes prêt à [commencez à franchir les prochaines étapes dans notre écosystème](#next-steps).

## Configuration initiale

Tous les fichiers de configuration du framework sont stockés dans le répertoire `app/Config`. Chaque option est documentée, alors n’hésitez pas à parcourir les fichiers et à vous familiariser avec les options qui s’offrent à vous.

BlitzPHP n’a presque pas besoin de configuration supplémentaire prête à l’emploi. Vous êtes libre de commencer à développer! Toutefois, vous voudrez peut-être examiner le fichier `app/Config/app.php` et sa documentation. Il contient plusieurs options telles que `timezone` et `language` que vous voudrez peut-être modifier en fonction de votre application.

<a name="configuration-basee-sur-l-environnement"></a>
### Configuration basée sur l’environnement

Étant donné que de nombreuses valeurs d’options de configuration de BlitzPHP peuvent varier selon que votre application s’exécute sur votre ordinateur local ou sur un serveur Web de production, de nombreuses valeurs de configuration importantes sont définies à l’aide du fichier `.env` qui existe à la racine de votre application.

Votre fichier `.env` ne doit pas être validé dans le versioning de votre application, car chaque développeur/serveur utilisant votre application peut nécessiter une configuration d’environnement différente. En outre, il s’agirait d’un risque de sécurité dans le cas où un intrus accèderait à votre dépôt de git, car toutes les informations d’identification sensibles seraient exposées.

> **Note**  
> Pour plus d'informations à propos du fichier `.env` et les configurations basées sur l'environnement, veuillez consulter notre [documentation de configuration](/docs/{version}/configuration#configuration-d-environnement).

### Configuration du dossier de base

BlitzPHP doit toujours être servi à partir de la racine du « dossier public » configuré pour votre serveur Web. Vous ne devez pas tenter de servir une application BlitzPHP à partir d’un sous-dossier du « dossier public ». Si vous tentez de le faire, vous risquez d’exposer des fichiers sensibles présents dans votre application.

<a name="next-steps"></a>
## Prochaines étapes

Maintenant que vous avez créé votre projet BlitzPHP, vous vous demandez peut-être quoi apprendre ensuite. Tout d’abord, nous vous recommandons fortement de vous familiariser avec le fonctionnement de BlitzPHP en lisant la documentation suivante :

<div class="content-list" markdown="1">

- [Cycle de vie](/docs/{version}/cycle-de-vie)
- [Configuration](/docs/{version}/configuration)
- [Structure des dossiers](/docs/{version}/structure)
- [Frontend](/docs/{version}/frontend)
- [Prestation de services](/docs/{version}/services)
- [Facades](/docs/{version}/facades)

</div>

La façon dont vous souhaitez utiliser BlitzPHP dictera également les prochaines étapes de votre voyage. Il existe différentes façons d’utiliser BlitzPHP, et nous explorerons deux cas d’utilisation principaux pour le framework ci-dessous.

<a name="un-framework-fullstack"></a>
### Un framework Full Stack

BlitzPHP peut servir de framework full stack c'est-à-dire que vous allez utiliser BlitzPHP pour acheminer les requêtes vers votre application et rendre votre frontend via un des nombreux moteur de template pris en charge par BlitzPHP ou une technologie hybride SPA comme [Inertia](https://inertiajs.com). C’est la façon la plus courante d’utiliser BlitzPHP.

Si c’est ainsi que vous prévoyez d’utiliser BlitzPHP, vous pouvez consulter notre documentation sur [le développement frontend](/docs/{version}/frontend), [routage](/docs/{version}/routage), [vues](/docs/{version}/vues).

<a name="les-api-backend"></a>
### Les API Backend

BlitzPHP peut également servir de backend API à une application JavaScript SPA ou à une application mobile. Par exemple, vous pouvez utiliser BlitzPHP comme backend d’API pour votre application Nuxt.js. Dans ce contexte, vous pouvez utiliser BlitzPHP pour fournir [l'authentification](/docs/{version}/schild) et le stockage / récupération de données pour votre application, tout en tirant parti des puissants services de BlitzPHP tels que les e-mails, les notifications, etc.