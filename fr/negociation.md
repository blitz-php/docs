---
title: Négociation de contenu
---

<a name="qu-est-ce-que-la-negociation-de-contenu"></a>
## Qu'est-ce que la négociation de contenu ?

La négociation de contenu est un moyen de déterminer le type de contenu à renvoyer au client en fonction de ce que le client peut gérer et de ce que le serveur peut gérer. Elle peut être utilisée pour déterminer si le client souhaite recevoir du HTML ou du JSON, si l'image doit être renvoyée au format JPEG ou PNG, quel type de compression est pris en charge, et bien d'autres choses encore. Pour ce faire, quatre en-têtes différents sont analysés. Chacun d'entre eux peut prendre en charge plusieurs options de valeur, chacune ayant sa propre priorité.

Essayer de faire correspondre ces éléments manuellement peut s'avérer assez difficile. BlitzPHP fournit la classe `Negotiator` qui peut gérer cela pour vous.

Au fond, la négociation de contenu est simplement une partie de la spécification HTTP qui permet à une ressource unique de servir plus d'un type de contenu, ce qui permet aux clients de demander le type de données qui leur convient le mieux.

Un exemple classique est celui d'un navigateur qui ne peut pas afficher de fichiers PNG et qui ne peut demander que des images GIF ou JPEG. Lorsque le serveur reçoit la requête, il examine les types de fichiers disponibles demandés par le client et sélectionne la meilleure correspondance parmi les formats d'image qu'il prend en charge, dans ce cas, il est probable qu'il renvoie une image JPEG.

Cette même négociation peut avoir lieu avec quatre types de données : 
* **Media/Document Type** - il peut s'agir d'un format d'image, ou HTML vs. XML ou JSON. 
* **Character Set** - Le jeu de caractères dans lequel le document retourné doit être défini. Il s'agit généralement d'UTF-8. 
* **Encodage du document** - Il s'agit généralement du type de compression utilisé pour les résultats. 
* **Langue du document** - Pour les sites qui prennent en charge plusieurs langues, cela permet de déterminer celle qui doit être retournée. 

<a name="chargement-de-la-classe"></a>
## Chargement de la classe

Vous pouvez charger manuellement une instance de la classe par l'intermédiaire du fournisseur de service :

```php
$negotiate = service('negotiator');
```

Cela permet de récupérer l'instance de requête actuelle et de l'injecter automatiquement dans la classe du négociateur.

Cette classe n'a pas besoin d'être chargée seule. Au lieu de cela, elle peut être accédée à travers l'instance `ServerRequest` de la requête actuelle. Bien que vous ne puissiez pas y accéder directement de cette manière, vous pouvez facilement accéder à toutes les méthodes par le biais de la méthode `negotiate()` :

```php
$request->negotiate('media', ['foo', 'bar']);
```

Lorsqu'on y accède de cette manière, le premier paramètre est le type de contenu pour lequel vous essayez de trouver une correspondance, tandis que le second est un tableau de valeurs prises en charge.

<a name="negocier"></a>
## Négocier

Dans cette section, nous examinerons les quatre types de contenu qui peuvent être négociés et nous montrerons comment cela se passerait en utilisant les deux méthodes décrites ci-dessus pour accéder au négociateur.

<a name="media"></a>
### Media

Le premier aspect à examiner est la gestion des négociations relatives aux "médias". Celles-ci sont fournies par l'en-tête `Accept` et constituent l'un des en-têtes les plus complexes qui soient. Un exemple courant est celui du client qui indique au serveur le format dans lequel il souhaite recevoir les données. Cet exemple est particulièrement courant dans les API. Par exemple, un client peut demander des données au format JSON à un point de terminaison de l'API :

```http
GET /foo HTTP/1.1
Accept: application/json
```

Le serveur doit maintenant fournir une liste des types de contenu qu'il peut fournir. Dans cet exemple, l'API pourrait renvoyer des données sous forme de HTML brut, de JSON ou de XML. Cette liste doit être fournie par ordre de préférence :

```php  
$supported = [
    'application/json',
    'text/html',
    'application/xml',
];

$format = $request->negotiate('media', $supported);
// ou
$format = $negotiate->media($supported);
```

Dans ce cas, le client et le serveur peuvent se mettre d'accord sur le formatage des données en JSON, de sorte que la méthode negotiate renvoie "json". Par défaut, si aucune correspondance n'est trouvée, le premier élément du tableau `$supported` est renvoyé. Dans certains cas, cependant, vous pouvez avoir besoin d'imposer une correspondance stricte du format. Si vous passez `true` comme valeur finale, la chaîne retournée sera vide si aucune correspondance n'est trouvée :

```php
$format = $request->negotiate('media', $supported, true);
// ou
$format = $negotiate->media($supported, true);
```

<a name="langue"></a>
### Langue

Une autre utilisation courante est de déterminer la langue dans laquelle le contenu doit être servi. Si vous n'exploitez qu'un site en une seule langue, cela ne fera évidemment pas une grande différence, mais tout site qui peut proposer plusieurs traductions de son contenu trouvera cette fonction utile, puisque le navigateur enverra généralement la langue préférée dans l'en-tête `Accept-Language` :

```http
GET /foo HTTP/1.1
Accept-Language: fr; q=1.0, en; q=0.5
```

Dans cet exemple, le navigateur préférerait le français, avec un deuxième choix d'anglais. Si votre site web prend en charge l'anglais et l'allemand, vous devez procéder de la manière suivante :

```php
$supported = [
    'en',
    'de',
];

$lang = $request->negotiate('language', $supported);
// ou
$lang = $negotiate->language($supported);
```

Dans cet exemple, 'en' sera retourné comme étant la langue courante. Si aucune correspondance n'est trouvée, le système renvoie le premier élément du tableau `$supported`, qui devrait donc toujours être la langue préférée.

<a name="encodage"></a>
### Encodage

L'en-tête `Accept-Encoding` contient les jeux de caractères que le client préfère recevoir et est utilisé pour spécifier le type de compression que le client prend en charge :

```http
GET /foo HTTP/1.1
Accept-Encoding: compress, gzip
```

Votre serveur web définit les types de compression que vous pouvez utiliser. Certains, comme Apache, ne prennent en charge que gzip :

```php
$type = $request->negotiate('encoding', ['gzip']);
// ou
$type = $negotiate->encoding(['gzip']);
```

Plus d'informations sur <a href="https://en.wikipedia.org/wiki/HTTP_compression" target="_blank">Wikipédia</a>.

<a name="jeu-de-caracteres"></a>
### Jeu de caractères

Le jeu de caractères souhaité est transmis par le biais de l'en-tête `Accept-Charset` :

```http
GET /foo HTTP/1.1
Accept-Charset: utf-16, utf-8
```

Par défaut, si aucune correspondance n'est trouvée, `utf-8` sera renvoyé :

```php
$charset = $request->negotiate('charset', ['utf-8']);
// ou
$charset = $negotiate->charset(['utf-8']);
```
