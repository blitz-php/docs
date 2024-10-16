---
title: Stockage de fichiers
---

<a name="introduction"></a>
## Introduction

BlitzPHP fournit une abstraction puissante du système de fichiers grâce au package <a href="https://github.com/thephpleague/flysystem" target="_blank">league/flysystem</a>. BlitzPHP possède des pilotes simples pour travailler avec des systèmes de fichiers locaux, SFTP, et Amazon S3. Mieux encore, il est incroyablement simple de passer d'une option de stockage à l'autre entre votre machine de développement locale et votre serveur de production, car l'API reste la même pour chaque système.

<a name="configuration"></a>
## Configuration

Le fichier de configuration du système de fichiers de BlitzPHP est situé dans `app/Config/filesystems.php`. Dans ce fichier, vous pouvez configurer tous les "disques" de votre système de fichiers. Chaque disque représente un pilote de stockage particulier et un emplacement de stockage. Des exemples de configuration pour chaque pilote pris en charge sont inclus dans le fichier de configuration afin que vous puissiez modifier la configuration pour refléter vos préférences de stockage et vos informations d'identification. 

Le pilote `local` interagit avec les fichiers stockés localement sur le serveur exécutant l'application BlitzPHP, tandis que le pilote `s3` est utilisé pour écrire sur le service de stockage cloud S3 d'Amazon.

> **Note**  
> Vous pouvez configurer autant de disques que vous le souhaitez et même avoir plusieurs disques qui utilisent le même pilote.

<a name="le-pilote-local"></a>
### Le pilote `local`

Lorsque vous utilisez le pilote `local`, toutes les opérations sur les fichiers sont relatives au répertoire "root" défini dans votre fichier de configuration des systèmes de fichiers. Par défaut, cette valeur est fixée au répertoire `storage/app`. Par conséquent, la méthode suivante écrira dans `storage/app/example.txt` :

```php
service('storage')->disk('local')->put('example.txt', 'Contents');
```

Vous pouvez également utiliser la façade `Storage`

```php
use BlitzPHP\Facades\Storage;
 
Storage::disk('local')->put('example.txt', 'Contents');
```

<a name="le-disque-public"></a>
### Le disque `public`

Le disque `public` inclus dans le fichier de configuration des systèmes de fichiers de votre application (`app/Config/flysystems.php`) est destiné aux fichiers qui seront accessibles au public. Par défaut, le disque `public` utilise le pilote `local` et stocke ses fichiers dans `storage/app/public`. 

Pour rendre ces fichiers accessibles depuis le web, vous devez créer un lien symbolique de `public/storage` vers `storage/app/public`. L'utilisation de cette convention de dossier permettra de conserver vos fichiers accessibles au public dans un seul répertoire qui peut être facilement partagé entre les déploiements. 

Pour créer le lien symbolique, vous pouvez utiliser la commande Klinge `storage:link` :

```shell
php klinge storage:link
```

Une fois qu'un fichier a été stocké et que le lien symbolique a été créé, vous pouvez créer une URL vers les fichiers à l'aide de la fonction `url()` :

```php
echo asset('storage/file.txt');
```

Vous pouvez configurer des liens symboliques supplémentaires dans votre fichier de configuration des systèmes de fichiers. Chacun des liens configurés sera créé lorsque vous exécuterez la commande `storage:link` :

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```

La commande `storage:unlink` peut être utilisée pour détruire les liens symboliques configurés :

```shell
php klinge storage:unlink
```

<a name="conditions-prealables-pour-le-pilote"></a>
### Conditions préalables pour le pilote

<a name="configuration-du-pilote-s3"></a>
#### Configuration du pilote S3

Avant d'utiliser le pilote S3, vous devez installer le paquetage S3 de Flysystem via le gestionnaire de paquetage Composer :

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

Les informations relatives à la configuration du pilote S3 se trouvent dans le fichier de configuration `app/Config/filesystems.php`. Ce fichier contient un exemple de tableau de configuration pour un pilote S3. Vous êtes libre de modifier ce tableau avec votre propre configuration S3 et vos identifiants. Par souci de commodité, ces variables d'environnement correspondent à la convention d'appellation utilisée par l'interface de commande AWS.

<a name="configuration-du-pilote-ftp"></a>
#### Configuration du pilote FTP

Avant d'utiliser le pilote FTP, vous devrez installer le package FTP Flysystem via le gestionnaire de packages Composer :

```shell
composer require league/flysystem-ftp "^3.0"
```

Les intégrations Flysystem de BlitzPHP fonctionnent très bien avec FTP ; cependant, un exemple de configuration n'est pas inclus dans le fichier de configuration `filesystems.php` par défaut du framework. Si vous avez besoin de configurer un système de fichiers FTP, vous pouvez utiliser l'exemple de configuration ci-dessous :

```php
'ftp' => [
    'driver' => 'ftp',
    'host' => env('FTP_HOST'),
    'username' => env('FTP_USERNAME'),
    'password' => env('FTP_PASSWORD'),
 
    // Configuration FTP optionnelle...
    // 'port' => env('FTP_PORT', 21),
    // 'root' => env('FTP_ROOT'),
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],
```

<a name="configuration-du-pilote-sftp"></a>
#### Configuration du pilote SFTP

Avant d'utiliser le pilote SFTP, vous devrez installer le package SFTP Flysystem via le gestionnaire de packages Composer :

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Les intégrations Flysystem de BlitzPHP fonctionnent très bien avec SFTP ; cependant, un exemple de configuration n'est pas inclus dans le fichier de configuration `filesystems.php` par défaut du framework. Si vous avez besoin de configurer un système de fichiers SFTP, vous pouvez utiliser l'exemple de configuration ci-dessous :

```php
'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'),
 
    // Paramètres pour l'authentification de base...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),
 
    // Paramètres pour l'authentification basée sur une clé SSH avec mot de passe de cryptage...
    'privateKey' => env('SFTP_PRIVATE_KEY'),
    'passphrase' => env('SFTP_PASSPHRASE'),
 
    // Paramètres pour les autorisations de fichiers et de répertoires...
    'visibility' => 'private', // `private` = 0600, `public` = 0644
    'directory_visibility' => 'private', // `private` = 0700, `public` = 0755
 
    // Paramètres SFTP optionnels...
    // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
    // 'maxTries' => 4,
    // 'passphrase' => env('SFTP_PASSPHRASE'),
    // 'port' => env('SFTP_PORT', 22),
    // 'root' => env('SFTP_ROOT', ''),
    // 'timeout' => 30,
    // 'useAgent' => true,
],
```

<a name="systemes-de-fichiers-en-lecture-seule-et-a-scopes"></a>
#### Systèmes de fichiers en lecture seule et scopés

Les disques scopés vous permettent de définir un système de fichiers où tous les chemins sont automatiquement préfixés avec un préfixe de chemin donné. Avant de créer un disque à système de fichiers délimité, vous devez installer un paquetage Flysystem supplémentaire via le gestionnaire de paquetages Composer :

```php
composer require league/flysystem-path-prefixing "^3.0"
```

Vous pouvez créer une instance à portée de chemin de n'importe quel disque de système de fichiers existant en définissant un disque qui utilise le pilote `à portée de chemin`. Par exemple, vous pouvez créer un disque qui définit le champ d'application de votre disque `s3` existant en fonction d'un préfixe de chemin spécifique, et chaque opération de fichier utilisant votre disque défini par le champ d'application utilisera le préfixe spécifié :

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

Les disques "en lecture seule" vous permettent de créer des disques de système de fichiers qui n'autorisent pas les opérations d'écriture. Avant d'utiliser l'option de configuration `read-only`, vous devez installer un paquetage Flysystem supplémentaire via le gestionnaire de paquetages Composer :

```shell
composer require league/flysystem-read-only "^3.0"
```

Ensuite, vous pouvez inclure l'option de configuration `read-only` dans un ou plusieurs tableaux de configuration de votre disque :

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

<a name="obtention-d-instances-de-disque"></a>
## Obtention d'instances de disque

Le service `storage` peut être utilisé pour interagir avec n'importe lequel des disques configurés. Par exemple, vous pouvez utiliser la méthode `put` pour stocker un avatar sur le disque par défaut. Si vous appelez des méthodes du service `storage` sans appeler au préalable la méthode `disk`, la méthode sera automatiquement transmise au disque par défaut :

```php
service('storage')->put('avatars/1', $content);

// Utilisation de la façade
//
// use BlitzPHP\Facades\Storage;
// Storage::put('avatars/1', $content);
```

Si votre application interagit avec plusieurs disques, vous pouvez utiliser la méthode `disk` pour travailler avec des fichiers sur un disque particulier :

```php
service('storage')->disk('s3')->put('avatars/1', $content);

// Utilisation de la façade
//
// use BlitzPHP\Facades\Storage;
// Storage::disk('s3')->put('avatars/1', $content);
```

<a name="disques-a-la-demande"></a>
### Disques à la demande

Il peut arriver que vous souhaitiez créer un disque au moment de l'exécution en utilisant une configuration donnée sans que cette configuration soit présente dans votre fichier de configuration `app/Config/filesystems.php`. Pour ce faire, vous pouvez passer un tableau de configuration à la méthode de `build` du service `storage` :

```php
$disk = service('storage')->build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);
 
$disk->put('image.jpg', $content);
```

<a name="recuperation-de-fichiers"></a>
## Récupération de fichiers

La méthode `get` peut être utilisée pour récupérer le contenu d'un fichier. Elle renvoie la chaîne de caractères brute du fichier. N'oubliez pas que tous les chemins d'accès aux fichiers doivent être spécifiés par rapport à l'emplacement "root" du disque :

```php
$contents = service('storage')->get('file.jpg');
```

Si le fichier que vous récupérez contient du JSON, vous pouvez utiliser la méthode `json` pour récupérer le fichier et décoder son contenu :

```php
$orders = service('storage')->json('orders.json');
```

La méthode `exists` peut être utilisée pour déterminer si un fichier existe sur le disque :

```php
if (service('storage')->disk('s3')->exists('file.jpg')) {
    // ...
}
```

La méthode `missing` peut être utilisée pour déterminer si un fichier est manquant sur le disque :

```php
if (service('storage')->disk('s3')->missing('file.jpg')) {
    // ...
}
```

<a name="telechargement-de-fichiers"></a>
### Téléchargement de fichiers

La méthode `download` peut être utilisée pour générer une réponse qui force le navigateur de l'utilisateur à télécharger le fichier au chemin donné. La méthode `download` accepte un nom de fichier comme deuxième argument de la méthode, qui déterminera le nom de fichier vu par l'utilisateur qui télécharge le fichier. Enfin, vous pouvez passer un tableau d'en-têtes HTTP comme troisième argument de la méthode :

```php
return service('storage')->download('file.jpg');
 
return service('storage')->download('file.jpg', $name, $headers);
```

<a name="url-de-fichiers"></a>
### URL de fichiers

Vous pouvez utiliser la méthode `url` pour obtenir l'URL d'un fichier donné. Si vous utilisez le pilote local, cette méthode ajoutera simplement `/storage` au chemin donné et renverra une URL relative au fichier. Si vous utilisez le pilote `s3`, l'URL distante entièrement qualifiée sera renvoyée :

```php
$url = service('storage')->url('file.jpg');
```

> **Attention**  
> Lors de l'utilisation du pilote local, la valeur de retour de url n'est pas encodée. Pour cette raison, nous vous recommandons de toujours stocker vos fichiers en utilisant des noms qui créeront des URL valides.

<a name="personnalisation-de-l-url-de-l-hote"></a>
#### Personnalisation de l'URL de l'hôte

Si vous souhaitez prédéfinir l'hôte pour les URL générées à l'aide du service de stockage, vous pouvez ajouter une option `url` au tableau de configuration du disque :

```php
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => config('app.baseUrl').'/file',
    'visibility' => 'public',
],
```

<a name="metadonnees-de-fichier"></a>
### Métadonnées de fichier

Outre la lecture et l'écriture de fichiers, BlitzPHP peut également fournir des informations sur les fichiers eux-mêmes. Par exemple, la méthode `size` peut être utilisée pour obtenir la taille d'un fichier en octets :

```php
$size = service('storage')->size('file.jpg');
```

La méthode `lastModified` renvoie l'horodatage UNIX de la dernière modification du fichier :

```php
$time = service('storage')->lastModified('file.jpg');
```

Le type MIME d'un fichier donné peut être obtenu via la méthode `mimeType` :

```php
$mime = service('storage')->mimeType('file.jpg');
```

<a name="chemins-de-fichiers"></a>
#### Chemins de fichiers

Vous pouvez utiliser la méthode `path` pour obtenir le chemin d'accès à un fichier donné. Si vous utilisez le pilote `local`, cette méthode renverra le chemin absolu vers le fichier. Si vous utilisez le pilote `s3`, cette méthode renvoie le chemin relatif vers le fichier dans le seau S3 :

```php
$path = service('storage')->path('file.jpg');
```

<a name="stockage-de-fichiers"></a>
## Stockage de fichiers 

La méthode `put` peut être utilisée pour stocker le contenu d'un fichier sur un disque. Vous pouvez également passer une ressource PHP à cette méthode, qui utilisera le support de flux sous-jacent de `Flysystem`. N'oubliez pas que tous les chemins d'accès aux fichiers doivent être spécifiés par rapport à l'emplacement "root" configuré pour le disque :

```php
service('storage')->put('file.jpg', $contents);
 
service('storage')->put('file.jpg', $resource);
```

#### Échec des écritures

Si la méthode `put` (ou d'autres opérations "d'écritures") est incapable d'écrire le fichier sur le disque, `false` sera renvoyé :

```php
if (! service('storage')->put('file.jpg', $contents)) {
    // Le fichier ne peut pas être écrit sur le disque...
}
```

Si vous le souhaitez, vous pouvez définir l'option throw dans le tableau de configuration de votre disque de système de fichiers. Lorsque cette option est définie comme vraie, les méthodes d'"écriture" telles que `put` lancent une instance de `League\Flysystem\UnableToWriteFile` lorsque les opérations d'écriture échouent :

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

<a name="predosage-et-ajout-de-fichiers"></a>
### Prédosage et ajout de fichiers

Les méthodes `prepend` et `append` permettent d'écrire au début ou à la fin d'un fichier :

```php
service('storage')->prepend('file.log', 'Texte ajouté au début');
 
service('storage')->append('file.log', 'Texte ajouté à la fin');
```

<a name="copier-et-deplacer-des-fichiers"></a>
### Copier et déplacer des fichiers

La méthode de `copy` peut être utilisée pour copier un fichier existant vers un nouvel emplacement sur le disque, tandis que la méthode `move` peut être utilisée pour renommer ou déplacer un fichier existant vers un nouvel emplacement :

```php
service('storage')->copy('old/file.jpg', 'new/file.jpg');
 
service('storage')->move('old/file.jpg', 'new/file.jpg');
```

<a name="streaming-automatique"></a>
### Streaming automatique

La diffusion en continu de fichiers vers un espace de stockage permet de réduire considérablement l'utilisation de la mémoire. Si vous souhaitez que BlitzPHP gère automatiquement la diffusion en continu d'un fichier donné vers votre emplacement de stockage, vous pouvez utiliser la méthode `putFile` ou `putFileAs`. Cette méthode accepte une instance `BlitzPHP\Filesystem\Files\File` ou `BlitzPHP\Filesystem\Files\UploadedFile` et transmet automatiquement le fichier à l'emplacement souhaité :

```php
use BlitzPHP\Filesystem\Files\File;
 
$storage = service('storage');

// Générer automatiquement un identifiant unique pour le nom de fichier...
$path = $storage->putFile('photos', new File('/path/to/photo'));
 
// Spécifier manuellement un nom de fichier...
$path = $storage->putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

Il y a quelques points importants à noter à propos de la méthode `putFile`. Notez que nous n'avons spécifié qu'un nom de répertoire et non un nom de fichier. Par défaut, la méthode `putFile` génère un identifiant unique qui sert de nom de fichier. L'extension du fichier sera déterminée en examinant le type MIME du fichier. Le chemin d'accès au fichier sera renvoyé par la méthode `putFile` afin que vous puissiez stocker le chemin d'accès, y compris le nom de fichier généré, dans votre base de données.

Les méthodes `putFileAs` et `putFileAs` acceptent également un argument pour spécifier la "visibilité" du fichier stocké. Ceci est particulièrement utile si vous stockez le fichier sur un disque en nuage tel qu'Amazon S3 et que vous souhaitez que le fichier soit accessible au public via des URL générées :

```php
use BlitzPHP\Filesystem\Files\File;

service('storage')->putFile('photos', new File('/path/to/photo'), 'public');
```

<a name="upload-de-fichiers"></a>
### Upload de fichiers

Dans les applications web, l'un des cas d'utilisation les plus courants pour le stockage de fichiers est le stockage de fichiers téléchargés par l'utilisateur, tels que des photos et des documents. BlitzPHP facilite le stockage des fichiers téléchargés en utilisant la méthode `store` sur une instance de fichier téléchargé. Appelez la méthode `store` avec le chemin où vous souhaitez stocker le fichier téléchargé :

```php
<?php
 
namespace App\Controllers;
 
use BlitzPHP\Controllers\ApplicationController;
 
class UserAvatarController extends ApplicationController
{
    /**
     * Modifier l'avatar de l'utilisateur
     */
    public function update(): string
    {
        $path = $this->request->file('avatar')->store('avatars');
 
        return $path;
    }
}
```

Il y a quelques points importants à noter dans cet exemple. Nous n'avons spécifié qu'un nom de répertoire, et non un nom de fichier. Par défaut, la méthode `store` génère un identifiant unique qui sert de nom de fichier. L'extension du fichier sera déterminée en examinant le type MIME du fichier. Le chemin d'accès au fichier sera renvoyé par la méthode `store` afin que vous puissiez stocker le chemin d'accès, y compris le nom de fichier généré, dans votre base de données. 

Vous pouvez également appeler la méthode `putFile` du service `storage` pour effectuer la même opération de stockage de fichiers que dans l'exemple ci-dessus :

```php
$path = service('storage')->putFile('avatars', $this->request->file('avatar'));
```

<a name="specification-d-un-nom-de-fichier"></a>
#### Spécification d'un nom de fichier

Si vous ne souhaitez pas qu'un nom de fichier soit automatiquement attribué à votre fichier stocké, vous pouvez utiliser la méthode `storeAs`, qui reçoit le chemin d'accès, le nom de fichier et le disque (facultatif) comme arguments :

```php
$path = $this->request->file('avatar')->storeAs('avatars', user_id());
```

Vous pouvez également utiliser la méthode `putFileAs` du service `storage`, qui effectuera la même opération de stockage de fichiers que dans l'exemple ci-dessus :

```php
$path = service('storage')->putFileAs(
    'avatars',
    $this->request->file('avatar'), 
    user_id()
);
```

> **Attention**  
> Les caractères unicode non imprimables et non valides seront automatiquement supprimés des chemins d'accès aux fichiers. Par conséquent, vous pouvez souhaiter assainir vos chemins de fichiers avant de les transmettre aux méthodes de stockage de fichiers. Les chemins de fichiers sont normalisés à l'aide de la méthode `League\Flysystem\WhitespacePathNormalizer::normalizePath`.

<a name="specification-d-un-disque"></a>
#### Spécification d'un disque

Par défaut, la méthode `store` utilisera votre disque par défaut. Si vous souhaitez spécifier un autre disque, indiquez le nom du disque en tant que deuxième argument de la méthode de stockage :

```php
$path = $this->request->file('avatar')->store(
    'avatars/' . user_id(), 
    's3'
);
```

Si vous utilisez la méthode `storeAs`, vous pouvez transmettre le nom du disque en tant que troisième argument de la méthode :

```php
$path = $this->request->file('avatar')->storeAs(
    'avatars',
    user_id(), 
    's3'
);
```

<a name="autres-informations-sur-les-fichiers-telecharges"></a>
#### Autres informations sur les fichiers téléchargés

Si vous souhaitez obtenir le nom et l'extension d'origine du fichier téléchargé, vous pouvez le faire en utilisant les méthodes `getClientFilename` et `clientExtension` :

```php
$file = $this->request->file('avatar');
 
$name = $file->getClientFilename();
$extension = $file->clientExtension();
```

Cependant, n'oubliez pas que les méthodes `getClientFilename` et `clientExtension` sont considérées comme peu sûres, car le nom et l'extension du fichier peuvent être modifiés par un utilisateur malveillant. C'est pourquoi il est préférable d'utiliser les méthodes `hashName` et `extension` pour obtenir un nom et une extension pour le téléchargement d'un fichier donné :

```php
$file = $this->request->file('avatar'); 

$name = $file->hashName(); // Génère un nom unique et aléatoire... 
$extension = $file->extension(); // Détermine l'extension du fichier en fonction de son type MIME...
```

<a name="visibilite-des-fichiers"></a>
### Visibilité des fichiers

Dans l'intégration Flysystem de BlitzPHP, la "visibilité" est une abstraction des permissions de fichiers sur plusieurs plateformes. Les fichiers peuvent être déclarés `publics` ou `privés`. Lorsqu'un fichier est déclaré `public`, vous indiquez qu'il doit être généralement accessible à d'autres personnes. Par exemple, lorsque vous utilisez le pilote `S3`, vous pouvez récupérer les URL des fichiers `publics`.

Vous pouvez définir la visibilité lors de l'écriture du fichier via la méthode `put` :

```php
service('storage')->put('file.jpg', $contents, 'public');
```

Si le fichier a déjà été stocké, sa visibilité peut être récupérée et définie via les méthodes `getVisibility` et `setVisibility` :

```php
$visibility = service('storage')->getVisibility('file.jpg');
 
service('storage')->setVisibility('file.jpg', 'public');
```

Lorsque vous interagissez avec des fichiers téléchargés, vous pouvez utiliser les méthodes `storePublicly` et `storePubliclyAs` pour stocker le fichier téléchargé avec une visibilité `publique` :

```php
$path = $this->request->file('avatar')->storePublicly('avatars', 's3');
 
$path = $this->request->file('avatar')->storePubliclyAs(
    'avatars',
    user_id(),
    's3'
);
```

<a name="fichiers-locaux-et-visibilite"></a>
#### Fichiers locaux et visibilité

Lorsque vous utilisez le pilote local, [la visibilité](#visibilite-des-fichiers) publique se traduit par des autorisations `0755` pour les répertoires et `0644` pour les fichiers. Vous pouvez modifier les permissions dans le fichier de configuration du système de fichiers de votre application :

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app'),
    'permissions' => [
        'file' => [
            'public' => 0644,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0755,
            'private' => 0700,
        ],
    ],
],
```

<a name="suppression-de-fichiers"></a>
## Suppression de fichiers

La méthode `delete` accepte un nom de fichier unique ou un tableau de fichiers à supprimer :

```php
$storage = service('storage');
 
$storage->delete('file.jpg');
 
$storage->delete(['file.jpg', 'file2.jpg']);
```

Si nécessaire, vous pouvez spécifier le disque à partir duquel le fichier doit être supprimé :

```php
service('storage')->disk('s3')->delete('path/file.jpg');
```

<a name="repertoires"></a>
## Répertoires

<a name="obtenir-tous-les-fichiers-d-un-repertoire"></a>
### Obtenir tous les fichiers d'un répertoire

La méthode `files` renvoie un tableau de tous les fichiers d'un répertoire donné. Si vous souhaitez obtenir une liste de tous les fichiers d'un répertoire donné, y compris tous les sous-répertoires, vous pouvez utiliser la méthode `allFiles` :

```php
$storage = service('storage');
 
$files = $storage->files($directory);
 
$files = $storage->allFiles($directory);
```

<a name="obtenir-tous-les-repertoires-d-un-repertoire"></a>
### Obtenir tous les répertoires d'un répertoire

La méthode `directories` renvoie un tableau de tous les répertoires d'un répertoire donné. En outre, vous pouvez utiliser la méthode `allDirectories` pour obtenir une liste de tous les répertoires d'un répertoire donné et de tous ses sous-répertoires :

```php
$storage = service('storage');
 
$files = $storage->directories($directory);
 
$files = $storage->allDirectories($directory);
```

<a name="creer-un-repertoire"></a>
### Créer un répertoire 

La méthode `makeDirectory` crée le répertoire donné, y compris les sous-répertoires nécessaires :

```php
service('storage')->makeDirectory($directory);
```

<a name="supprimer-un-repertoire"></a>
### Supprimer un répertoire 

Enfin, la méthode `deleteDirectory` peut être utilisée pour supprimer un répertoire et tous ses fichiers :

```php
service('storage')->deleteDirectory($directory);
```