---
title: Chiffrement des données
---

<a name="introduction"></a>
## Introduction

Le service de chiffrement de BlitzPHP fournit un chiffrement symétrique bidirectionnel (clé secrète) des données. Toutes les valeurs chiffrées par BlitzPHP sont signées à l'aide d'un code d'authentification de message (MAC) afin que leur valeur sous-jacente ne puisse pas être modifiée ou altérée une fois qu'elle est chiffrée.

<a name="configuration"></a>
## Configuration

Avant d'utiliser le service de chiffrement de BlitzPHP, vous devez définir l'option de configuration de la clé dans votre fichier de configuration `app/Config/encryption.php`. Cette valeur de configuration est pilotée par la variable d'environnement `encryption.key`. Vous pouvez utiliser la commande `php klinge key:generate` pour générer la valeur de cette variable car la commande `key:generate` utilisera le générateur d'octets aléatoires sécurisé de PHP pour construire une clé cryptographiquement sécurisée pour votre application. Généralement, la valeur de la variable d'environnement `encryption.key` est générée lors de [l'installation de BlitzPHP](/docs/{version}/installation).

Le tableau ci-dessous vous présente les paramètres de configuration qui se trouvent dans `app/Config/encryption.php`.

<div class="overflow-auto">

Option  | Valeurs possibles (valeur par défaut entre parenthèses)
------------- | -------------
`key`  |  Clé de chiffrement (`env('encryption.key')`)
`driver`  |  Gestionnaire utilisé pour le chiffrement des données, par ex, OpenSSL ou Sodium (`OpenSSL`)
`digest`  |  Algorithme d'analyse des données (`SHA512`)
`block_size`  |  [**SodiumHandler** uniquement] Longueur de remplissage en octets (`16`)
`cipher`  |  [**OpenSSLHandler** uniquement] Chiffrement à utiliser (`AES-256-CTR`)
`encrypt_key_info`  |  [**OpenSSLHandler** uniquement] Informations sur la clé de chiffrement (`''`)
`auth_key_info`  |  [**OpenSSLHandler** uniquement] Informations sur la clé d'authentification (`''`)
`raw_data`  |  [**OpenSSLHandler** uniquement] Si le texte chiffré doit être brut (`true`)

</div>

<a name="utilisation-du-chiffreur"></a>
## Utilisation du chiffreur

Comme tous les services de BlitzPHP, le chiffreur peut être chargé via la fonction `service` : 

```php
<?php 

$encrypter = service('encrypter'); 
```

En supposant que vous ayez généré votre clé de chiffrement, le chiffrement et le déchiffrement des données est simple - passez la chaîne appropriée aux méthodes `encrypt` et/ou `decrypt`.

<a name="chiffrer-une-valeur"></a>
### Chiffrer une valeur

Vous pouvez chiffrer une valeur à l'aide de la méthode `encrypt` fournie par le service `encrypter`. Toutes les valeurs seront chiffrées en fonction de vos configurations (OpenSSL ou Sodium, AES-256-CTR ou AES-256-CBC, etc.). En outre, toutes les valeurs chiffrées sont signées par un code d'authentification de message (MAC). Le code d'authentification des messages intégré empêchera le déchiffrement de toutes les valeurs qui ont été altérées par des utilisateurs malveillants :

```php
<?php
 
namespace App\Controllers;
 
class TokenController extends AppController
{
    /**
     * Stocke le token d'API de l'utilisateur.
     */
    public function store()
    {
        $token = $this->request->token;
        
        auth()->user()->fill([
            'token' => service('encrypter')->encrypt($token),
        ])->save();
 
        return redirect()->to('/secrets');
    }
}
```

<a name="dechiffrer-une-valeur"></a>
### Déchiffrer une valeur

Vous pouvez déchiffrer les valeurs à l'aide de la méthode `decrypt` fournie par le service `encrypter`. Si la valeur ne peut pas être correctement déchiffrée, par exemple lorsque le code d'authentification du message n'est pas valide, une exception `BlitzPHP\Exceptions\EncryptionException` sera levée :

```php
use BlitzPHP\Exceptions\EncryptionException;
 
try {
    $decrypted = service('encrypter')->decrypt($encryptedValue);
} catch (EncryptionException $e) {
    // ...
}
```

<a name="comportement-par-defaut"></a>
## Comportement par défaut

Par défaut, le service de chiffrement utilise le gestionnaire `OpenSSL`. Ce gestionnaire chiffre les données en utilisant l'algorithme `AES-256-CTR`, la clé configurée et l'authentification HMAC `SHA512`.

<a name="definition-de-la-cle-de-chiffrement"></a>
## Définition de la clé de chiffrement

Votre clé de chiffrement **doit** être aussi longue que le permet l'algorithme de chiffrement utilisé, soit 256 bits ou 32 octets (caractères). Pour l'AES-256, il s'agit de 256 bits ou 32 octets (caractères). 

La clé doit être aussi aléatoire que possible et ne doit pas être une chaîne de texte ordinaire, ni la sortie d'une fonction de hachage, etc. Pour créer une clé appropriée, vous pouvez le faire manuellement en utilisant la méthode `createKey` de la classe `Encryption` ou alors utilisant la commande Klinge `key:generate` comme [décrit plus haut](#configuration).

<a name="definition-manuelle"></a>
### Définition manuelle

```php
<?php

// Une clé aléatoire de 32 octets (256 bits) est attribuée à $key.
$key = \BlitzPHP\Encryption\Encryption::createKey();

// pour le SodiumHandler, vous pouvez utiliser l'un ou l'autre :
$key = sodium_crypto_secretbox_keygen();
$key = \BlitzPHP\Encryption\Encryption::createKey(SODIUM_CRYPTO_SECRETBOX_KEYBYTES);
```

Une fois la clé générée, elle peut être stockée dans le fichier `app/Config/encryption.php`, ou vous pouvez concevoir votre propre mécanisme de stockage et transmettre la clé dynamiquement lors du chiffrement/déchiffrement. 

Pour enregistrer votre clé dans `app/Config/encryption.php`, ouvrez le fichier et définissez le paramètre de configuration `key`:

```php
<?php 
return [
    // ---
    'key' => 'ENTREZ_VOTRE_CLE_GENEREE_ICI',
    // ---
];
```

<a name="encodage-des-cles"></a>
#### Encodage des clés

Vous remarquerez que la méthode `createKey` produit des données binaires, qui sont difficiles à traiter (un copier-coller peut les endommager), vous pouvez donc utiliser les fonctions `bin2hex`, ou `base64_encode` pour travailler avec la clé d'une manière plus conviviale. Par exemple :

```php
<?php

// Obtenir une représentation hexagonale de la clé :
$encoded = bin2hex(\BlitzPHP\Encryption\Encryption::createKey(32));

//  Mettre la même valeur avec hex2bin(), 
// pour qu'elle soit toujours transmise en binaire à la bibliothèque :
$key = hex2bin('votre-clé-encodée');
```

<a name="utilisation-de-prefixes-dans-l-enregistrement-des-cles"></a>
#### Utilisation de préfixes dans l'enregistrement des clés

Vous pouvez utiliser deux préfixes spéciaux pour stocker vos clés de chiffrement : `hex2bin:` et `base64:`. Lorsque ces préfixes précèdent immédiatement la valeur de votre clé, Le service de chiffrement analysera intelligemment la clé et transmettra toujours une chaîne binaire à la bibliothèque.

```php
<?php 
return [
    // ---
    
    // Pour le paramètre `key`, vous pouvez utiliser
    'key' => 'hex2bin:<VOTRE_CLE_ENCODEE_EN_HEX>',
    // ou
    'key' => 'base64:<VOTRE_CLE_ENCODEE_EN_BASE64>',
    
    // ---
];
```

<a name="definition-automatique"></a>
### Définition automatique

Comme nous l'avons dit plut haut, vous pouvez utiliser la commande `key:generate` pour générer une clé de chiffrement pour votre application. Le commande générera une clé sécurisée et l'attribuera automatiquement à la variable `encryption.key` de votre fichier `.env`.

Il faudra alors, câbler cette variable d'environnement au paramètre `key` de votre fichier de configuration `app/Config/encryption.php`

```php
<?php 
return [
    // ---
    'key' => env('encryption.key'),
    // ---
];
```

Vous pouvez ajouter des options supplémentaires à la commande `key:generate` pour personnaliser la génération de votre clé. Les options suivantes sont autorisés :

* `--prefix`: Le préfixe associé à la clé généré. Les valeurs admissibles sont `base64`, `hex2bin` (valeur par défaut).
* `--length`: La longueur de la chaîne aléatoire qui doit être retournée en bytes. par défaut, elle vaut `32`.

Il est possible de générer une clé de chiffrement sans mettre à jour votre fichier d'environnement. Pour cela rajouter l'option `--show` dans votre commande :

```bash
php klinge key:generate --show
```

<a name="rembourrage"></a>
## Rembourrage

Parfois, la longueur d'un message peut fournir beaucoup d'informations sur sa nature. Si un message se compose de "oui", "non" et "peut-être", le chiffrement du message n'est d'aucune utilité : il suffit de connaître la longueur du message pour savoir de quoi il s'agit.

Le padding est une technique qui permet d'atténuer ce phénomène en faisant de la longueur un multiple de la taille d'un bloc donné. 

Le padding est mis en œuvre dans SodiumHandler à l'aide des fonctions natives `sodium_pad` et `sodium_unpad` de la librairie Sodium. Cela nécessite l'utilisation d'une longueur de remplissage (en octets) qui est ajoutée au message en clair avant le chiffrement, et supprimée après le déchiffrement. La longueur de remplissage est configurable via la clé `block_Size` du fichier `app/Config/encryption.php`. Cette valeur doit être supérieure à zéro.

> **Attention**  
> Il est conseillé de ne pas concevoir sa propre implémentation du rembourrage. Vous devez toujours utiliser l'implémentation la plus sûre d'une bibliothèque. En outre, les mots de passe ne doivent pas être complétés. Il n'est pas recommandé d'utiliser le remplissage pour masquer la longueur d'un mot de passe. Un client souhaitant envoyer un mot de passe à un serveur devrait plutôt le hacher (même avec une seule itération de la fonction de hachage). Cela garantit que la longueur des données transmises est constante et que le serveur ne peut pas obtenir facilement une copie du mot de passe.

<a name="notes-sur-le-gestionnaire-de-chiffrement"></a>
## Notes sur le gestionnaire de chiffrement

<a name="notes-sur-open-ssl"></a>
### Notes sur OpenSSL

L'extension <a href="https://www.php.net/openssl" target="_blank">OpenSSL</a> fait partie intégrante de PHP depuis longtemps. 

Le gestionnaire OpenSSL de BlitzPHP utilise le chiffrement AES-256-CTR. 

La clé fournie par votre configuration est utilisée pour dériver deux autres clés, l'une pour le chiffrement et l'autre pour l'authentification. Cette opération est réalisée au moyen d'une technique connue sous le nom de <a href="https://en.wikipedia.org/wiki/HKDF" target="_blank">HMAC-based Key Derivation Function</a> (HKDF) (fonction de dérivation de clé basée sur HMAC).

<a name="notes-sur-sodium"></a>
### Notes sur Sodium

L'extension <a href="https://www.php.net/manual/fr/book.sodium.php" target="_blank">Sodium</a> est intégrée par défaut dans PHP depuis PHP 7.2.0.

Sodium utilise les algorithmes XSalsa20 pour le chiffrement, Poly1305 pour le MAC et XS25519 pour l'échange de clés lors de l'envoi de messages secrets dans un scénario de bout en bout. Pour chiffrer et/ou authentifier une chaîne à l'aide d'une clé partagée, comme le chiffrement symétrique, Sodium utilise l'algorithme XSalsa20 pour le chiffrement et HMAC-SHA512 pour l'authentification.

> **Note**  
> Le gestionnaire `SodiumHandler` de BlitzPHP utilise `sodium_memzero` dans chaque session de chiffrement ou de déchiffrement. Après chaque session, le message (en clair ou chiffré) et la clé de chiffrement sont effacés des tampons. Il se peut que vous deviez fournir à nouveau la clé avant de commencer une nouvelle session.

<a name="longueur-du-message"></a>
## Longueur du message

Une chaîne chiffrée est généralement plus longue que la chaîne originale en texte clair (en fonction du système de chiffrement).

Ceci est influencé par l'algorithme de chiffrement lui-même, le vecteur d'initialisation (IV) ajouté au texte chiffré et le message d'authentification HMAC qui est également ajouté. En outre, le message crypté est également codé en Base64 afin qu'il puisse être stocké et transmis en toute sécurité, quel que soit le jeu de caractères utilisé. 

Gardez ces informations à l'esprit lorsque vous choisissez votre mécanisme de stockage de données. Les cookies, par exemple, ne peuvent contenir que **4K** d'informations.