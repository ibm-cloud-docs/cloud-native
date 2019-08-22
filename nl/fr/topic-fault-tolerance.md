---

copyright:
  years: 2019
lastupdated: "2019-07-19"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Tolérance aux pannes
{: #fault-tolerance}

La tolérance aux pannes est la capacité d'un système de continuer à fonctionner durant une défaillance partielle. La création d'un système résilient impose des exigences pour tous les services de ce dernier. La nature dynamique des environnements de cloud exige que les services soient écrits de façon à pouvoir être confrontés à des événements imprévus et à y faire face de façon appropriée. 

Les solutions de tolérance aux pannes se concentrent principalement sur les délais d'attente, les procédures de secours, les cloisonnements et les disjoncteurs.

Dans certains environnements, des mécanismes de tolérance aux pannes peuvent être fournis pas des composants d'infrastructure, comme Istio. Que l'infrastructure apporte son aide ou pas, un service doit supposer que l'appel distant peut échouer et être prêt à réagir en disposant d'une panoplie d'actions de secours appropriées.

## Délais d'attente

La première ligne de défense face à une défaillance partielle consiste à utiliser des délais d'attente. Ces derniers permettent de garantir que votre application reçoit une erreur quand un service de sauvegarde ne répond pas, rendant ainsi possible la prise en charge du problème en adoptant un comportement de secours approprié, ce qui n'entraîne pas l'échec de l'opération demandée. Les délais d'attente changent la durée pendant laquelle un client créant une demande attend une réponse. Ils n'ont aucun impact sur le comportement de traitement du service cible.

Un grand nombre de bibliothèques de langages utilisent un délai d'attente par défaut pour les demandes, tout comme Istio. Par défaut, le proxy du composant sidecar échoue s'il ne reçoit pas de réponse dans un délai de 15 secondes. Cette valeur peut être changée en définissant une règle d'administration de délai d'attente pour la route dans la définition VirtualService (similaire à l'exemple suivant) utilisant un service qui renvoie le cours des actions en tant qu'exemple :

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    timeout: 30s
```
{: codeblock}

Les demandes effectuées auprès du service boursier via le proxy de composant sidecar ou la passerelle Ingress bénéficient d'un délai de 30 secondes avant d'échouer, plutôt que les 15 secondes par défaut.

En réglant le délai d'attente de cette façon, toutes les demandes qui utilisent la route se voient appliquer cette valeur. Cette fonctionnalité est fournie par une couche de sécurité de base Istio. Même si la bibliothèque ou l'infrastructure d'application n'impose pas de délai d'attente, elle n'attendra pas indéfiniment, car un délai est imposé par Istio. Toutefois, les délais d'attente au niveau de l'application s'appliquent toujours. Le délai d'attente au niveau de l'infrastructure pour le service boursier a été étendu à 30 secondes. Si la bibliothèque d'application définit un délai d'attente de 5 secondes, la demande de l'application échoue toujours.

## Procédures de secours

Une application définit ce qui se passe en cas d'échec d'une demande effectuée auprès d'un service de sauvegarde. Il existe plusieurs options mais l'objectif est d'effectuer une dégradation lorsque ces services ne répondent pas dans le délai imparti. Lorsqu'un service distant échoue, vous pouvez émettre à nouveau la demande, effectuer une nouvelle demande ou renvoyer les données mises en cache à la place.

Au premier abord, une nouvelle émission de la demande semble être le mécanisme de secours le plus simple. Cependant, on ne pense pas immédiatement que le fait d'émettre à nouveau les demandes peut générer des erreurs système en cascade ("rafale de tentatives", variation du problème présenté sur la page [Thundering herd problem](https://en.wikipedia.org/wiki/Thundering_herd_problem)). Le code de niveau d'application ne prend pas suffisamment en compte l'état du système ou du réseau. De plus, il est difficile d'obtenir les algorithmes d'incrément exponentiel corrects.

Istio peut effectuer de nouvelles tentatives de manière beaucoup plus efficace. Il est directement impliqué dans le routage de demande et il fournit une implémentation cohérente ne prenant pas en charge le langage pour les règles de nouvelle tentative. Ainsi, vous pouvez définir une règle similaire à la suivante pour le service boursier : 

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    retries:
      attempts: 3
      perTryTimeout: 5s
```
{: codeblock}

Avec cette configuration simple, les demandes effectuées auprès du service boursier via un proxy de composant sidecar Istio ou une passerelle Ingress sont renouvelées trois fois, avec un délai d'attente de cinq secondes entre chaque tentative. Des [règles de correspondance de route supplémentaires](https://istio.io/docs/reference/config/networking/#HTTPMatchRequest) peuvent limiter cette possibilité de renouvellement des demandes aux seules demandes `GET`, par exemple.

Il existe une nuance qu'il est néanmoins facile de ne pas remarquer : vous ne spécifiez pas d'intervalle entre les nouvelles tentatives. Le composant sidecar détermine cet intervalle et définit de manière délibérée une période d'instabilité entre les différentes tentatives afin d'éviter de bombarder les services surchargés.

<!-- Notes about other approaches here: -->

## Cloisonnement
{: #bulkheads-fault-tolerance}

Dans le domaine du transport, une cloison est une partition qui empêche une fuite dans un compartiment de se répandre dans l'intégralité du chargement. Le modèle est appliqué de différentes manières aux environnements de cloud afin d'atteindre un objectif similaire.

Pour les langages multiprocessus, tels Java, il est possible d'utiliser des cloisonnements internes pour restreindre ou contrôler la façon dont les ressources sont utilisées pour communiquer avec les ressources distantes, en utilisant des mécanismes de file d'attente ou de sémaphore : 

- Pour utiliser une file d'attente, le service associe un nombre spécifique d'unités d'exécution à une file d'attente spécifique. Une erreur survient pour toute demande effectuée une fois que la file d'attente est saturée. Dans Java, il peut s'agir d'une erreur `ThreadPoolExecutor` associée à un élément `BlockingQueue`, par exemple.
- Le mécanisme de sémaphore fonctionne en ayant un nombre défini d'autorisations. Une demande sortante exige une autorisation. Une fois qu'une demande est terminée, une autorisation d'utilisation d'une autre demande est générée.

Vous pouvez également définir des cloisonnements inter-service en utilisant une règle Istio DestinationRule afin d'appliquer des limites au pool de connexions pour le service sortant, comme illustré dans l'exemple suivant :

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
        connectTimeout: 30s
```
{: codeblock}

Cette configuration limite à 10 le nombre maximal de connexions simultanées à chaque instance du service boursier. Les services qui ne parviennent pas à établir une connexion dans les 30 secondes obtiennent une réponse `503 -- Service Unavailable`. Ce type de cloisonnement peut empêcher un service de calcul de recevoir plus de demandes qu'il ne peut traiter, par exemple.

## Disjoncteurs

Les disjoncteurs permettent d'optimiser le comportement des demandes sortantes lorsque des défaillances surviennent. Au lieu de faire des demandes répétées auprès d'un service qui ne répond pas, un disjoncteur observe le nombre d'échecs qui se produisent au cours d'une période donnée. Si le taux d'erreur dépasse un seuil défini, le disjoncteur ouvre le circuit et fait échouer la demande ainsi que toutes les demandes suivantes jusqu'à ce que le circuit soit fermé à nouveau.

Les disjoncteurs sont eux aussi définis en utilisant une règle Istio DestinationRule :

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
      interval: 5s
      baseEjectionTime: 5m
      maxEjectionPercent: 100
```
{: codeblock}

Cette configuration crée des contraintes pour les demandes effectuées par d'autres services auprès du service boursier. La règle de trafic `outlierDetection` spécifiée est appliquée à chaque instance individuelle. Il est possible de reformuler la configuration précédente en deux phrases : "Ejectez toutes les instances du service boursier qui échouent trois fois en cinq secondes pendant au moins 5 minutes. Toutes les instances sont concernées par cette règle". Le dernier paramètre `maxEjectionPercent` est lié à l'équilibrage de charge. Istio gère un pool d'équilibrage de charge et éjecte les instances en situation d'échec de ce pool. Par défaut, il éjecte au maximum 10 %  de toutes les instances disponibles depuis le pool d'équilibrage de charge.

Pour ceux qui maîtrisent d'autres mécanismes de disjonction, notez qu'Istio ne dispose pas d'un état à moitié ouvert. Il applique un calcul simple. Une instance reste éjectée du pool pendant `baseInjectionTime * <number of times it has been ejected>`, ce qui permet d'utiliser à nouveau les instances après des échecs transitoires tout en conservant hors du pool les instances en échec permanent.

Avec cette configuration, le nombre maximal d'appels simultanés au service boursier est limité à 10. Au-delà, une réponse `503 -- Service Unavailable` est envoyée.

### Istio -- Limites de débit
{: #istio-rate-limits}

Vous pouvez aussi définir des **limites de débit** avec Istio. Similaires aux cloisonnements, elles sont définies sur une fenêtre de temps spécifique, plutôt que de se contenter de limiter les appels simultanés qui sont traités en même temps.

Vous pouvez, par exemple, ne pas vouloir autoriser plus d'un tweet par seconde, situation relativement courante dans la vie de tous les jours. Autre exemple de même type : Twitter verrouillait automatiquement, dans le passé, tous les comptes `@IBMStockTrader` quand des tests de contrainte étaient exécutés par un opérateur boursier.

<!-->
*<Exemple requis.>*


## Test : injection d’erreurs avec Istio

Vous pouvez utiliser Istio pour provoquer délibérément des erreurs afin de voir comment votre code réagit.
-->
