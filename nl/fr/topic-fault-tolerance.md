---

copyright:
  years: 2019
lastupdated: "2019-02-11"

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

La tolérance aux pannes est une fonction du système permettant à ce dernier de fonctionner même après une défaillance partielle. La création d'un système résilient impose des exigences pour tous les services de ce dernier. La nature dynamique des environnements de cloud exige que les services soient écrits de manière à s'attendre et à répondre à des événements inattendus, comme la réception de données incorrectes, l'impossibilité d'atteindre un service de sauvegarde requis ou le traitement de conflits dus à des mises à jour simultanées dans un système distribué. 

Les solutions de tolérance aux pannes se concentrent principalement sur les délais d'attente, les rétromigrations, les cloisons et les disjoncteurs.

Dans certains environnements, des mécanismes de tolérance aux pannes peuvent être fournis pas des composants d'infrastructure, comme Istio. Que l'infrastructure soit utile ou non, un service doit supposer que l'appel distant peut échouer et qu'il doit être préparé avec des actions de rétromigration appropriées.

## Délais d'attente

La première ligne de défense face à une défaillance partielle consiste à utiliser des délais d'attente. Ces derniers permettent de garantir que votre application reçoit une erreur lorsqu'un service de sauvegarde ne répond pas. Ainsi, il est possible de prendre en charge le problème en adoptant un comportement de rétromigration approprié. Cela ne signifie pas nécessairement que l'opération demandée a échoué. Les délais d'attente changent la durée pendant laquelle un client créant une demande attend une réponse. Ils n'ont aucune conséquence sur le comportement de traitement du service cible.

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

Avec cette configuration, les demandes effectuées auprès du service de cours des actions via le proxy de composant sidecar ou la passerelle Ingress attend 30 secondes avant que la demande n'échoue et non 15 secondes qui est la valeur par défaut.

L'avantage de définir les délais d'attente en procédant ainsi réside dans le fait que le délai d'attente est appliqué à toutes les demandes utilisant cette route. Il s'agit d'une couche de base de sécurité fournie par Istio. Même si la bibliothèque ou l'infrastructure d'application n'impose pas de délai d'attente, elle n'attendra pas de manière illimitée car un délai est imposé par Istio. Cependant, les délais d'attente de niveau d'application s'appliquent toujours. Dans l'exemple ci-dessus, le délai d'attente de niveau d'infrastructure pour le service de cours des actions a été étendu à 30 secondes. Si la bibliothèque d'application définit un délai d'attente de 5 secondes, la demande de l'application échoue toujours. 

## Rétromigrations

Une application doit définir ce qui se passe lorsqu'une demande auprès d'un service de sauvegarde échoue. Il existe plusieurs options mais l'objectif est d'effectuer une dégradation lorsque ces services ne répondent pas dans le délai imparti. Lorsqu'un service distant échoue, vous pouvez émettre à nouveau la demande, effectuer une nouvelle demande ou renvoyer les données mises en cache à la place.

A premier abord, émettre à nouveau la demande semble être le mécanisme de rétromigration le plus simple. Cependant, on ne pense pas immédiatement que le fait d'émettre à nouveau les demandes peut générer des erreurs système en cascade ("rafale de tentatives", variation du problème présenté sur la page [Thundering herd problem](https://en.wikipedia.org/wiki/Thundering_herd_problem)). Le code de niveau d'application ne prend pas suffisamment en compte l'état du système ou du réseau. De plus, il est difficile d'obtenir les algorithmes d'incrément exponentiel corrects.

Istio peut effectuer de nouvelles tentatives de manière beaucoup plus efficace. Il est directement impliqué dans le routage de demande et il fournit une implémentation cohérente ne prenant pas en charge le langage pour les règles de nouvelle tentative. Par exemple, il est possible de définir une règle d'administration, semblable à l'élément suivant, pour notre service de cours d'action :

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

Avec cette configuration simple, les demandes effectuées auprès du service de cours des actions via un proxy de composant sidecar Istio ou la passerelle Ingress sont soumises à nouveau jusqu'à 3 fois, avec un délai de 5 secondes entre chaque tentative. Les [règles de correspondance de route supplémentaires](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPMatchRequest) peuvent restreindre cette règle de nouvelle tentative aux demandes `GET`, par exemple.

Il existe une nuance qu'il est néanmoins facile de ne pas remarquer : vous ne spécifiez pas d'intervalle entre les nouvelles tentatives. Le composant sidecar détermine cet intervalle et définit de manière délibérée une période d'instabilité entre les différentes tentatives afin d'éviter de bombarder les services surchargés.

## Cloisons

Dans le domaine du transport, une cloison est une partition qui empêche une fuite dans un compartiment de se répandre dans l'intégralité du chargement. Le modèle est appliqué de différentes manières aux environnements de cloud afin d'atteindre un objectif similaire.

Pour les langages multiprocessus, tels Java, il est possible d'utiliser des cloisons internes pour contrôler ou restreindre comment les ressources sont utilisées pour la communication avec les ressources distantes, en utilisant des mécanismes de file d'attente ou de sémaphore :

- Pour utiliser une file d'attente, le service associe un nombre spécifique d'unités d'exécution à une file d'attente spécifique. Une erreur survient pour toute demande effectuée une fois que la file d'attente est saturée. Dans Java, il peut s'agir d'une erreur `ThreadPoolExecutor` associée à un élément `BlockingQueue`, en tant qu'exemple.
- Le mécanisme de sémaphore fonctionne en ayant un nombre défini d'autorisations. Une demande sortante exige une autorisation. Une fois qu'une demande est terminée, une autorisation d'utilisation d'une autre demande est générée.

Vous pouvez également définir des cloisons entre services en utilisant une règle Istio DestinationRule afin de contraindre le pool de connexions pour un service en amont, en utilisant une commande similaire à l'exemple suivant :

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

Cette configuration limite à 10 le nombre maximal de connexions simultanées à chaque instance du service de cours des actions. Les serveurs ne pouvant pas établir de connexion dans les 30 secondes obtiennent une réponse `503 -- Service non disponible`. Ce type de cloison peut empêcher un service de calcul de recevoir plus de demandes qu'il ne peut traiter, par exemple.

## Disjoncteurs

Les disjoncteurs permettent d'optimiser le comportement des demandes sortantes lorsque des défaillances surviennent. Au lieu de continuer à effectuer des demandes auprès d'un service qui ne répond pas, un disjoncteur observe le nombre de défaillances ayant eu lieu pendent une période définie. Si le taux d'erreur dépasse un seuil défini, le disjoncteur ouvre le circuit et fait échouer la demande ainsi que toutes les demandes suivantes jusqu'à ce que le circuit soit fermé à nouveau.

Les disjoncteurs sont également définis en utilisant un élément Istio DestinationRule :

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

Cette configuration crée des contraintes pour les demandes effectuées par d'autres services auprès du service de cours des actions. La règle de trafic `outlierDetection` spécifiée est appliquée à chaque instance individuelle. Il est possible reformuler la configuration précédente en utilisant une phrase du type suivant : "Ejectez toute instance de service de cours des actions qui échoue 3 fois en 5 secondes pendant au moins 5 minutes ; au delà, toutes les instances peuvent être éjectées". Le dernier paramètre `maxEjectionPercent` est lié à l'équilibrage de charge. Istio gère un pool d'équilibrage de charge et éjecte les instances en situation d'échec de ce pool. Par défaut, il éjecte au maximum 10 % des instances disponibles du pool d'équilibrage de charge.

Si vous maîtrisez d'autres mécanismes de disjonction, sachez qu'Istio n'a pas d'état entrouvert. Il applique à la place un calcul simple, une instance reste éjectée du pool pendant `baseInjectionTime * <number of times it has been ejected>`. Ainsi, il est possible d'utiliser à nouveau les instances après des échecs transitoires tout en conservant hors du pool les instances en échec permanent. 

