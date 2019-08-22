---

copyright:
  years: 2019
lastupdated: "2019-07-16"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Création de microservices RESTful
{: #rest-api}

Les applications natives pour le cloud produisent et utilisent des API, que ce soit dans une architecture de microservices ou non. Certaines API sont considérées comme internes, ou privées et d'autres sont considérées comme externes. 
{:shortdesc}

Les API internes sont utilisées uniquement dans un environnement de pare-feu afin que les services de back end puissent communiquer les uns avec les autres. Les API externes, qui offrent un point d'entrée unifié pour les consommateurs, sont souvent **gérées**  par des outils, tels qu'{{site.data.keyword.apiconnect_long}}, qui peuvent imposer des limites de débit ou d'autres contraintes d'utilisation. L'API [GitHub Developer](https://developer.github.com/v3/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est un exemple de ce type d'API. Cette API unifiée permet d'utiliser de façon cohérente les codes retour et les instructions HTTP ainsi que le comportement de pagination sans afficher les détails d'implémentation internes. Elle peut être soutenue par une application de grande taille ou par une collection de microservices. Ces détails n'étant pas révélés au consommateur, GitHub peut faire évoluer ses systèmes internes si nécessaire.

## Meilleures pratiques pour les API RESTful
{: #bps-apis}

Les API REST utilisent des instructions HTTP standard pour les opérations CRUD (Create Retrieve Update Delete), en vérifiant particulièrement que chaque opération est idempotente (possibilité d'effectuer de nouvelles tentatives en toute sécurité). 

* Les opérations POST peuvent être utilisées pour créer ou mettre à jour des ressources. Les opérations POST ne peuvent pas être appelées de manière répétée. Par exemple, si une demande POST est utilisée pour créer des ressources et qu'elle est appelée plusieurs fois , une nouvelle ressource unique est créée suite à chaque appel.
* Les opérations GET doivent pouvoir être appelées de manière répétée et ne doivent pas provoquer d'effet secondaire. Elles sont destinées à extraire des informations. Les demandes GET avec des paramètres de requête ne doivent pas être utilisées pour modifier ou mettre à jour des informations. Utilisez à la place les opérations POST, PUT ou PATCH.
* Les opérations PUT peuvent être utilisées pour mettre à jour des ressources. Elles incluent généralement une copie complète de la ressource à mettre à jour, ce qui permet d'appeler l'opération plusieurs fois.
* Les opérations PATCH permettent une mise à jour partielle des ressources. Elles peuvent être appelées de manière répétée en fonction du mode de spécification du delta et d'application de ce dernier à la ressource. Par exemple, si une opération PATCH indique qu'une valeur doit être changée de A en B, elle peut être appelée de manière répétée. Il n'y aucune conséquence si elle est appelée plusieurs fois et que la valeur est déjà B.
* Les opérations DELETE peuvent être appelées plusieurs fois alors qu'une ressource ne peut être supprimée qu'une seule fois. Toutefois, le code retour varie, lorsque la première opération aboutit (`200` ou `204`) alors que les appels suivants ne trouvent pas la ressource (`404` ou `410`).

### Résultats descriptifs et conviviaux
{: #rest-results}

Comme les API sont appelées par un logiciel et non par un être humain, prenez soin de communiquer des informations à l'appelant de la manière la plus efficace et productive possible.

Utilisez des codes d'état HTTP appropriés et utiles, dont les descriptions sont présentées dans le tableau suivant : 

| Code d'erreur HTTP | Conseils d'utilisation |
|-----------------|----------------|
| `200 (OK)` | A utiliser quand la situation est normale et qu'il existe des données à renvoyer|
| `204 (NO CONTENT)` | A utiliser quand la situation est normale mais qu'il n'y a pas de données de réponse|
| `201 (CREATED)` | A utiliser pour les demandes POST qui génèrent la création d'une ressource, qu'il existe un corps de réponse ou non|
| `409 (CONFLICT)` | A utiliser quand des modifications simultanées provoquent un conflit |
| `400 (BAD REQUEST)` | A utiliser lorsque les paramètres sont incorrectement formés|

Pour plus d'informations, voir [Response status codes](https://tools.ietf.org/html/rfc7231#section-6){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"). 

Pour une communication optimale, pensez à déterminer les données à renvoyer dans vos réponses. Ainsi, si une ressource est créée avec une demande POST, la réponse doit inclure l'emplacement de la ressource nouvellement créée dans un en-tête Location. La ressource créée est aussi souvent incluse dans le corps de réponse, afin d'éliminer la demande GET supplémentaire d'extraction de la ressource créée. Cela s'applique également aux demandes PUT et PATCH.

### URI de ressource RESTful
{: #rest-uris}

Il existe différentes opinions sur certains aspects des URI de ressource RESTful. Généralement, il est convenu que les ressources doivent être des noms et non des verbes et que les noeuds finaux doivent être au pluriel. Ainsi, vous obtenez une structure bien définie pour les opérations CRUD :

* `POST /accounts` : Créer un compte.
* `GET /accounts` : Extraire une liste de comptes.
* `GET /accounts/16` : Extraire un compte spécifique.
* `PUT /accounts/16` : Mettre à jour un compte spécifique.
* `PATCH /accounts/16` : Mettre à jour un compte spécifique.
* `DELETE /accounts/16` : Supprimer un compte spécifique.

Les relations sont modélisées en utilisant des URI hiérarchiques, ` /accounts/16/credentials`, par exemple, pour la gestion des données d'identification associées à un compte.

Il n'existe pas de moyen unique de gérer les opérations avec une ressource qui ne s'inscrit pas dans une structure typique. Optez pour les moyens qui donneront les meilleurs résultats pour le consommateur de l'API.

<!-- Operation example -->

### Robustesse et API RESTful
{: #robust-api}

La section [1.2.2  Robustness Principle](https://tools.ietf.org/html/rfc1122#page-12){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") inclut un conseil très avisé : "Be liberal in what you accept, and conservative in what you send". Supposez que les API vont évoluer au fil du temps et soyez tolérant en ce qui concerne les données que vous ne comprenez pas.

#### Génération d'API
{: #robust-producer}

Quand vous mettez une API à disposition de clients externes, vous devez effectuer deux actions lors de l'acceptation des demandes et de l'envoi des réponses : 

* Acceptez les attributs inconnus comme partie de la demande.
    > Si un service appelle votre API avec des attributs non nécessaires, rejetez ces valeurs. Le renvoi d'une erreur dans ce scénario peut entraîner des échecs inutiles, ce qui a un impact négatif sur l'utilisateur.
* Renvoyez les attributs requis par vos consommateurs
    > Evitez d'exposer les détails de service interne. Ne dévoilez que les attributs dont les consommateurs ont besoin dans l'API.

#### Consommation des API
{: #robust-consumer}

Lors de la consommation d'API :

* Validez la demande en fonction des variables ou des attributs dont vous avez besoin.
    > N'effectuez pas de validation en fonction de variables uniquement parce qu'elles sont disponibles. Si vous ne les utilisez pas dans la demande, ne comptez pas sur le fait qu'elles soient présentes.

* Acceptez les attributs inconnus comme partie de la réponse.
    > Ne générez pas d'exception si vous recevez une variable inattendue. Si la réponse contient les informations dont vous avez besoin, le reste des données n'a pas d'importance.

Ces instructions sont particulièrement adaptées aux langages fortement typés comme Java, où la sérialisation et la désérialisation JSON se produisent souvent indirectement (bibliothèques Jackson ou JSON-P/JSON-B, par exemple). Recherchez les mécanismes de langage qui permettent de spécifier un comportement plus généreux, comme le fait d'ignorer des attributs inconnus ou de définir ou filtrer les attributs à sérialiser.

### Gestion des versions des API RESTful
{: #version-api}

Un des avantages principaux des microservices réside dans le fait que les services ont la possibilité d'évoluer indépendamment. Comme les microservices appellent d'autres services, cette indépendance implique une restriction importante : vous ne pouvez pas provoquer de modification problématique dans votre API.

Si le principe de robustesse est suivi, le délai peut être assez long avant qu'une modification problématique ne soit requise. Quand cette modification problématique survient, vous pouvez choisir de générer un service complètement différent puis d'arrêter progressivement d'utiliser celui d'origine.

Si vous avez effectivement besoin d'effectuer des modifications d'API problématiques pour un service, décidez comment gérer ces modifications. Le service va-t-il traiter toutes les versions de l'API, allez-vous gérer des versions indépendantes du service pour prendre en charge chaque version de l'API ou votre service va-t-il prendre en charge uniquement la dernière version de l'API et compter sur d'autres couches adaptatives pour la conversion de et vers l'ancienne API ?

Une fois que vous avez déterminé comment gérer les modifications, vous devez alors définir comment refléter la version dans votre API. Il existe généralement trois méthodes permettant de versionner une ressource REST :

* Inclure la version dans le chemin d'URI.
* Inclure la version dans l'en-tête Accept HTTP et s'appuyer sur la négociation de contenu.
* Utiliser un en-tête de demande personnalisé.

#### Inclusion de la version dans le chemin d'URI
{: #version-path}

La méthode la plus simple de spécification de version consiste à l'inclure dans le chemin de l'URI. Cette approche offre plusieurs avantages : il est plus facile d'obtenir le résultat souhaité lors de la génération des services dans votre application et cette approche est compatible avec des outils de navigation, tels Swagger, et des outils de ligne de commande, tels `curl`, etc.

Si vous incluez la version dans le chemin de l'URI, elle doit s'appliquer à votre application dans son ensemble, par exemple `/api/v1/accounts` et non `/api/accounts/v1`. HATEOS (Hypermedia as the Engine of Application State) constitue une des manières de mettre à disposition des URI auprès des consommateurs d'API. Ainsi, ces derniers n'ont pas la charge de construire les URI eux-mêmes. GitHub, par exemple, met à disposition des [URL hypermédia](https://developer.github.com/v3/#hypermedia){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") dans les réponses pour cette raison. Il devient difficile, voire impossible d'utiliser HATEOAS si différents services de back end peuvent avoir des versions variant de manière indépendante dans leurs URI.

#### Modification de l'en-tête Accept afin d'inclure la version
{: #version-accept}

L'en-tête Accept constitue l'emplacement désigné pour définir une version mais il s'agit également de l'élément le plus difficile à tester. De plus, cet en-tête est un emplacement fréquent d'arrivée pour l'activation/la désactivation des fonctionnalités. Si des en-têtes HTTP sont spécifiés, des appels d'API plus détaillés sont requis.

#### Ajout d'un en-tête de demande personnalisé
{: #version-custom}

Vous pouvez ajouter un en-tête de demande personnalisé pour indiquer la version d'API. Tout comme pour l'en-tête Accept, vous pouvez également utiliser des en-têtes personnalisés pour acheminer le trafic vers des instances de back end spécifiques. Avec cette méthode, vous rencontrez les mêmes problèmes de facilité d'utilisation que ceux rencontrés avec la méthode d'en-tête Accept avec l'exigence supplémentaire impliquant que les consommateurs doivent approfondir leurs connaissances sur cet en-tête.

Pour plus d'informations, voir la page [Your API versioning is wrong, which is why I decided to do it 3 different wrong ways](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Création et génération d'API
{: #create-api}

[OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") constitue la spécification officielle pour les services RESTful. Cette spécification est régie par l'élément [OpenAPI Initiative](https://www.openapis.org/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), association d'entreprises sous la fondation Linux.

Vous pouvez procéder d'une des manières suivantes pour créer une API :

  * Commencer avec une définition OpenAPI (méthode descendante) : Dans cette approche, vous commencez par créer une définition OpenAPI dans un format indépendant du langage (généralement YAML). Vous utilisez ensuite un générateur de code pour créer un squelette puis générez à partir de ce dernier votre implémentation de service. Cette méthode est généralement adoptée par les entreprises ayant une équipe de conception d'API centrale. De plus, elle permet une progression parallèle du développement et du test.
  * Commencer avec le code (méthode ascendante) : Votre code constitue la source de votre définition d'API. Cette approche est particulièrement adaptée pour les nouvelles applications avec un aspect expérimental, car votre définition d'API évolue au fur et à mesure que vous comprenez mieux ce que votre service a besoin de faire. Elle fonctionne mieux dans certains langages que dans d'autres, car elle repose sur des outils générant une définition OpenAPI à partir de votre code. Java, par exemple, offre un excellent support pour la génération de documents OpenAPI à partir de structures REST utilisant des annotations.

Dans tous les cas, l'utilisation d'une définition OpenAPI peut vous aider à identifier les zones dans lesquelles l'API est incohérente ou difficile à comprendre du point de vue d'un consommateur. Les définitions OpenAPI contrôlées par version ou publiées peuvent également être utilisées par des outils de génération afin de marquer les modifications problématiques ayant un impact potentiel sur les consommateurs.

### Création d'une API à partir d'une définition OpenAPI
{: #openapi-first}

Vous pouvez créer votre fichier YAML OpenAPI en utilisant l'outil de votre choix. Toutefois, l'utilisation d'un éditeur de texte en clair peut générer des erreurs. Certains éditeurs ont un support de base pour YAML et certains peuvent avoir des extensions supplémentaires pour la prise en charge des définitions OpenAPI. Par exemple, vous pouvez utiliser des extensions Visual Studio Code, telles [Swagger Viewer](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") ou [OpenAPI Preview](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour valider votre définition OpenAPI par rapport à une version de spécification indiquée et afficher une vue Web dans le panneau de prévisualisation :

![Aperçu OpenAPI](images/create-api-image1.png "Aperçu OpenAPI") 

Il existe également plusieurs éditeurs d'analyse syntaxique basés sur navigateur que vous pouvez utiliser en ligne ou localement. Voici quelques exemples :

* Le [projet OpenAPI-GUI](https://github.com/Mermade/openapi-gui){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") prend en charge les versions v2 et v3 de la spécification OpenAPI et peut migrer une définition OpenAPI v2 vers la version v3 pour vous.
* L'[éditeur Swagger de SmartBear](https://editor.swagger.io){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") prend également en charge les versions v2 et v3 d'OpenAPI.
* [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") inclut un ensemble d'éditeurs et d'outils pour la création et la modélisation d'API.

### Génération de l'implémentation d'API
{: #code-first}

Vous pouvez utiliser le [générateur OpenAPI](https://github.com/OpenAPITools/openapi-generator){: new_window} open source ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour créer un projet de squelette pour votre implémentation de service à partir d'une définition OpenAPI. Vous pouvez spécifier le langage ou la structure pour le squelette à partir de la ligne de commande. Par exemple, pour créer un projet Java pour l'API PetStore exemple qui utilise des annotations de méthode JAX-RS génériques, indiquez la commande suivante :

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

