---
title: Configuration
---

<a name="introduction"></a>
## Introduction

Tous les fichiers de configuration du framework BlitzPHP sont stockés dans le répertoire `app/Config`. Chaque option est documentée, alors n’hésitez pas à parcourir les fichiers et à vous familiariser avec les options qui s’offrent à vous.

Ces fichiers de configuration vous permettent de configurer des éléments tels que vos informations de connexion à la base de données, les informations de votre serveur de messagerie, ainsi que diverses autres valeurs de configuration de base telles que le fuseau horaire et la clé de chiffrement de votre application.

<a name="vue-d-ensemble-de-l-application"></a>
#### Vue d’ensemble de l’application

Pressé ? Vous pouvez obtenir un aperçu rapide de la configuration, des pilotes et de l’environnement de votre application via la commande Klinge `about`:

```bash
php klinge about
```

Si vous n’êtes intéressé que par une section particulière de la sortie de la vue d’ensemble de l’application, vous pouvez filtrer cette section à l’aide de l’option `--only`:

```bash
php artisan about --only=production
```

Ou, pour explorer en détail les valeurs d’un fichier de configuration spécifique, vous pouvez utiliser la commande `config:check`:

```bash
php klinge config:check database
```

<a name="configuration-d-environnement"></a>
## Configurations basées sur l'environnement

L’une des meilleures pratiques actuelles pour la configuration des applications consiste à utiliser des variables d’environnement. L’une des raisons en est que les variables d’environnement sont faciles à modifier entre les déploiements sans modifier le code. La configuration peut changer beaucoup d’un déploiement à l’autre, mais pas le code. Par exemple, plusieurs environnements, tels que l’ordinateur local du développeur et le serveur de production, nécessitent généralement des valeurs de configuration différentes pour chaque configuration particulière.

Les variables d’environnement doivent également être utilisées pour tout ce qui est privé, comme les mots de passe, les clés API ou d’autres données sensibles.

<a name="le-fichier-dotenv"></a>
### Le fichier `dotenv`

BlitzPHP simplifie et facilite la définition des variables d’environnement à l’aide d’un fichier « dotenv ». Le terme vient du nom du fichier, qui commence par un point avant le texte « env ».

<a name="creation-d-un-fichier-dotenv"></a>
#### Création d'un fichier `dotenv`

BlitzPHP s’attend à ce que le fichier `.env` soit à la racine de votre projet à côté des répertoires d’application. Il existe un fichier de modèle distribué avec BlitzPHP qui est situé à la racine du projet nommé `.env.example`.

Il contient une grande collection de variables que votre projet pourrait utiliser et qui ont été attribuées valeurs vides, factices ou par défaut. Vous pouvez utiliser ce fichier comme point de départ pour votre application soit en le renommant en `.env`, soit en faisant une copie de celui-ci nommée `.env.`

```bash
cp .env.example .env
```

> **Note**  
> Assurez-vous que le fichier `.env` n’est PAS suivi par votre système de contrôle de version. Pour git, cela signifie l’ajouter au fichier `.gitignore`. Ne pas le faire pourrait entraîner l’exposition au public de valeurs sensibles telles que le clé d'API ou les paramètres d'accès à la base de données. <br> Le fichier `.env.example` par contre doit être suivi par votre VCS. En effet, si vous développez avec une équipe ou si c'est un projet open source, vous pouvez mettre des clés vrais et des valeurs fausses dans ce fichier de configuration d'exemple pour que d'autres développeurs puissent clairement voir quelles variables d'environnement sont nécessaires pour exécuter votre application.

<a name="definition-des-variables"></a>
#### Définition des variables

Les paramètres sont stockés dans des fichiers .env comme une simple collection de paires nom/valeur séparées par un signe égal.

```ini
S3_BUCKET = dotenv
SECRET_KEY = super_secret_key
ENVIRONMENT = development
```

Lors de l’exécution de votre application, `.env` sera chargé automatiquement et les variables mises dans l’environnement. Si une variable existe déjà dans l’environnement, elle ne le sera PAS écrasée.

<a name="recuperation-des-variables-d-environnement"></a>
#### Récupération des variables d'environnement

Toutes les variables répertoriées dans le fichier `.env` seront chargées dans les variables super-globales PHP `$_ENV` et `$_SERVER`. Toutefois, vous pouvez utiliser la fonction `env()` pour récupérer des valeurs de ces variables dans vos fichiers de configuration. En fait, si vous examinez les fichiers de configuration BlitzPHP, vous remarquerez que la plupart des options sont déjà définies en utilisant cette fonction:

```php
return [
    'base_url' => env('app.baseURL', 'http://site.com'),  
];
```

La deuxième valeur passée à la fonction est la « valeur par défaut ». Cette valeur sera renvoyée si aucune variable d'environnement n'existe pour la clé donnée.

<a name="imbrication-de-variables"></a>
#### Imbrication de variables

Pour économiser lors de la saisie, vous pouvez réutiliser les variables que vous avez déjà spécifiées dans le fichier en encapsulant le nom de la variable dans `${...}`

```ini
BASE_DIR = "/var/webroot/project-root"
CACHE_DIR = "${BASE_DIR}/cache"
TMP_DIR = "${BASE_DIR}/tmp"
```

<a name="variables-a-namespace"></a>
#### Variables à namespace

Il y aura des moments où vous aurez plusieurs variables avec le même nom. Le système a besoin d’un moyen de savoir quel devrait être le réglage correct. Ce problème est résolu en « namepacing » les variables.

Les variables à namespace utilisent une notation à points pour qualifier les noms de variables afin qu’elles soient uniques lorsqu’il est incorporé dans l’environnement. Cela se fait en incluant un préfixe suivi d’un point (.), puis du nom de la variable elle-même.

```ini
// variables sans namespace
name = "John"
db = my_db

// variables avec namespace
address.city = "Yaoundé"
address.country = "Cameroun"
frontend.db = sales
backend.db = admin
```

Certains environnements, par exemple Docker, CloudFormation, n’autorisent pas le nom de variable avec des points (`.`). Dans ce cas, vous pouvez également utiliser des traits de soulignement (`_`) comme séparateur.

```ini
frontend_db = sales
backend_db = admin
```

<a name="fichiers-d-environnement-supplementaires"></a>
#### Fichiers d'environnement supplémentaires 

Avant de charger les variables d'environnement de votre application, BlitzPHP détermine si une variable d'environnement `APP_ENV` a été fournie à l'extérieur ou si l'argument `--env` de CLI est spécifié. Si c'est le cas, BlitzPHP tentera de charger un fichier `.env.[APP_ENV]`, s'il existe. S'il n'existe pas, le fichier par défaut `.env` est chargé.

<a name="types-des-variables-d-environnement"></a>
#### Types des variables d'environnement

Toutes les variables de vos fichiers de `.env` sont généralement analysées en tant que chaînes, de sorte que certaines valeurs réservées ont été créées pour vous permettre de retourner une plus large gamme de types à partir de la fonction `env()`,

| Valeur `.env`| Valeur `env()`|
|--------------|---------------|
| true         | (bool) true   |
| (true)       | (bool) true   |
| false        | (bool) false  |
| (false)      | (bool) false  |
| empty        | (string) ''   |
| (empty)      | (string) ''   |
| null         | (null) null   |
| (null)       | (null) null   |

Si vous devez définir une variable d'environnement avec une valeur qui contient des espaces, vous pouvez le faire en mettant la valeur en guillemets doubles :

```ini
app.name = "My Application"
```

<a name="determiner-l-environnement-actuel"></a>
### Déterminer l'environnement actuel

L'environnement d'application actuel est déterminé par l'intermédiaire de la variable `ENVIRONMENT` de votre fichier `.env`. Vous pouvez accéder à cette valeur via la fonction `environnement`:

```php
$environment = environment();
```

Vous pouvez également passer des arguments à la fonction `environment` pour déterminer si l'environnement correspond à une valeur donnée. Elle retournera `true` si l'environnement correspond à une des valeurs données :

```php
if (environment('local')) {
    // L'environnement est local
}

if (environment(['local', 'staging'])) {
    // L'environnement est soit local, soit staging...
}
```

> **Note**  
> La détection d'environnement d'application actuelle peut être substituée en définissant une variable d'environnement `ENVIRONMENT` au niveau du serveur.

<a name="acceder-aux-valeurs-de-configuration"></a>
## Accéder aux valeurs de configuration

Vous pouvez facilement accéder à vos valeurs de configuration à l'aide de la fonction globale `config` n'importe où dans votre application. Les valeurs de configuration peuvent être consultées à l'aide de la syntaxe "dot" qui inclut le nom du fichier et de l'option à laquelle vous souhaitez accéder. Une valeur par défaut peut également être spécifiée et sera renvoyée si l'option de configuration n'existe pas :

```php
$value = config('app.timezone');

// Récupérer une valeur par défaut si la valeur de configuration n'existe pas...
$value = config('app.timezone', 'Africa/Douala');
```

Pour définir des valeurs de configuration au moment de l'exécution, passez un tableau à la fonction `config` :

```php 
config(['app.timezone' => 'America/Chicago']);
```

<a name="mode-debug"></a>
## Mode Debug

L'option de débogage dans votre fichier de configuration `app/Config/app.php` détermine la quantité d'informations affichées sur une erreur à l'utilisateur. Par défaut, cette option est définie pour respecter la valeur de la variable d'environnement `app.debug`, qui est stockée dans votre fichier de `.env`. 

Pour le développement local, vous devez définir la variable d'environnement `app.debug` à `true`. **Dans votre environnement de production, cette valeur doit toujours être de `false`. Si la variable est définie sur `true` en production, vous risquez d'exposer les valeurs de configuration sensibles aux utilisateurs finaux de votre application.** 

> **Note**  
> Vous pouvez également la définie à `auto`. Dans ce cas, BlitzPHP la modifiera automatiquement en `true` ou `false` lors de l'exécution en fonction de l'environnement dans lequel votre application est lancée.
