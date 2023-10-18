---
title: Autoloader
---

<a name="introduction"></a>
## Introduction

Chaque application se compose d'un grand nombre de classes dans de nombreux endroits différents. BlitzPHP fournit des classes pour les fonctionnalités de base. Votre application aura un certain nombre de bibliothèques, de modèles et d'autres entités pour la faire fonctionner. Votre projet utilise peut-être des classes tierces. Garder une trace de l'emplacement de chaque fichier et coder en dur cet emplacement dans vos fichiers en une série de `require` est un énorme casse-tête et très sujet aux erreurs. C'est là que les chargeurs automatiques entrent en jeu

En plus de l'autoloader de [Composer](https://getcomposer.org), BlitzPHP fournit un autoloader très flexible qui peut être utilisé avec très peu de configuration. Il peut localiser des classes individuelles sans namespace, des classes avec des namespace qui adhèrent aux structures de répertoires d'autoload [PSR4](https://www.php-fig.org/psr/psr-4/) , et tentera même de localiser des classes dans des répertoires communs (comme les contrôleurs, les modèles, etc.).

> **Attention**  
> Vous devez toujours faire attention à la casse des noms de fichiers. Beaucoup les développeurs développent sur des systèmes de fichiers insensibles à la casse sous Windows ou macOS. Toutefois, la plupart des environnements serveur utilisent des systèmes de fichiers sensibles à la casse. Si la casse du nom de fichier est incorrecte, les autoloaders (celui de Composer et BlitzPHP) ne pourront pas trouver le fichier sur le serveur.

<a name="configuration"></a>
## Configuration

La configuration initiale se fait dans `app/Config/autoload.php`. Ce fichier contient deux principaux tableaux : un pour la carte de classe et un pour les namespaces compatibles PSR-4.

<a name="namespaces"></a>
### Namespaces

La méthode recommandée pour organiser vos classes consiste à créer un ou plusieurs namespaces pour vos fichiers de l’application. Ceci est particulièrement important pour toutes les classes liées à la logique métier, les classes d’entité, etc. Le tableau `psr4` dans le fichier de configuration vous permet de mapper un namespace à un répertoire donné.

```php 
return [
    // ...
    'psr4' => [
        APP_NAMESPACE => APP_PATH, // Pour le namespace d'application personnalisé
        
        // Vous pourriez ajouter autant de namespace
        // 'App\Admin' => ROOTPATH . 'modules/Admin', 
    ],
    // ...
];

```

La clé de chaque ligne est le namespace lui-même. Cela n’a pas besoin d’un backslash de fin. La valeur est l’emplacement du répertoire dans lequel les classes peuvent être trouvées.

> **Note**  
> Vous pouvez vérifier la configuration des namespaces avec la commande Klinge `namespaces`  
> ```shell
> php klinge namespaces
> ```

Par défaut, le dossier `/app` est mappé au namespace `App`. Vous devez donc nommer les contrôleurs, les entités, ou modèles du dossier `app`, et ils seront trouvés sous le namespace `App`.

Vous pouvez modifier ce namespace en modifiant (ou en créant s'il n'existe pas) le fichier `app/Config/constants.php` et en définissant une valeur à la constante `APP_NAMESPACE`

```php
defined('APP_NAMESPACE') || define('APP_NAMESPACE', 'App');
```

Vous devrez modifier tous les fichiers existants qui font référence au namespace actuel.

<a name="carte-de-classes"></a>
### Carte de classes (classmap)

La carte de classes est largement utilisée par pour tirer les dernières onces de performance du système en n’atteignant pas le système de fichiers avec des appels supplémentaires de la fonction `is_file`. Vous pouvez utiliser la carte de classe pour créer un lien vers Bibliothèques tierces qui n'ont pas de namespace

```php 
return [
    // ...
    'classmap' => [
        'PDF' => 'path/to/library/Pdf.php', 
    ],
    // ...
];

```

La clé de chaque ligne est le nom de la classe que vous souhaitez localiser. La valeur est le chemin d’accès pour le localiser.

<a name="prise-en-charge-de-composer"></a>
## Prise en charge de Composer

La prise en charge de Composer est automatiquement initialisée par défaut. Par défaut, il recherche le fichier de chargement automatique de Composer à l’emplacement `/vendor/autoload.php`. Si vous devez modifier l’emplacement de ce fichier pour une raison quelconque, vous pouvez modifier la valeur définie dans `/app/Config/path.php`.

> **Note**  
> Si le même namespace est défini dans le fichier `app/Config/autoload.php` et Composer, l'autoloader de Composer sera le premier à avoir une chance de localiser le fichier.