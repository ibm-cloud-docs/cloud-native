---

copyright:
  years: 2019
lastupdated: "2019-04-09"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Observabilité, télémétrie et surveillance
{: #observability-cn}

L'arrivée du concept Cloud Native a entraîné une modification de la surveillance. Bien qu'il soit attendu que les applications des environnements sur site et des environnements Cloud native soient hautement disponibles et résistantes aux défaillances, les méthodes utilisées pour atteindre ces objectifs sont très différentes. Par conséquent, l'objectif de la surveillance a changé : au lieu d'effectuer la surveillance pour éviter des défaillances, effectuez la surveillance pour gérer les défaillances. 
{:shortdesc}

Dans des environnements sur site, l'infrastructure et les logiciels intermédiaires sont mis à disposition en fonction de la capacité planifiée et des modèle de haute disponibilité, par exemple active-active ou active-passive. Des défaillances inattendues peuvent être complexes dans cet environnement, ce qui exige des efforts importants pour la détermination des problèmes et leur résolution. La surveillance externe est effectuée par des agents qui examinent l'utilisation des ressources afin d'éviter des classes connues de défaillances. Pensez à optimiser la taille de segment de mémoire, les délais d'attente et les règles de récupération de place pour les applications Java.

Une application Cloud native est composée de microservices indépendants et de services de sauvegarde requis. Même si une application Cloud native dans son ensemble doit rester disponible et continuer de fonctionner, des instances de service individuelles seront démarrées ou arrêtées lorsque cela est nécessaire en fonction des exigences de capacité ou pour la restauration après une défaillance. 

## Observabilité
{: #observability}

La surveillance de ce système fluide exige que chaque participant puisse *être observé*. Chaque entité doit générer des données appropriées pour la prise en charge de la détection de problème et de la génération d'alerte automatisées, du débogage manuel lorsque cela est nécessaire et de l'analyse de l'intégrité du système (analyses et tendances historiques).

Quels types de données doit générer un service pour pouvoir être observé ?

* Les **diagnostics d'intégrité** (souvent des noeuds finaux HTTP personnalisés) permettent aux orchestrateurs, comme Kubernetes ou Cloud Foundry, d'effectuer des actions automatisées pour gérer l'état général du système.
* Les **métriques** constituent une représentation numérique des données collectées à des intervalles définis dans une série temporelle. Les données de séries temporelles numériques sont faciles à stocker et à interroger, ce qui vous permet de rechercher plus facilement des tendances historiques. Pendant une longue période, les données numériques peuvent être compressées dans des agrégats plus granulaires, tous les jours ou toutes les semaines, par exemple.
* Les **entrées de journal** représentent des événements discrets ayant eu lieu. Ces éléments sont essentiels pour le débogage car ils incluent souvent des traces de pile et d'autres informations contextuelles qui peuvent vous aider à identifier la cause principale des défaillances observées.
* Le **traçage distribué, de demande ou de bout en bout** capture le flux de bout en bout d'une demande dans le système. Le traçage capture principalement les relations entre les services (services affectés par la demande) et la structure des travaux transitant via le système (traitement synchrone ou asynchrone, relations d'enfants ou d'éléments suivants).

## Télémétrie
{: #telemetry}

Les applications Cloud native doivent s'appuyer sur l'environnement pour la *télémétrie*, qui correspond à la collecte et à la transmission automatiques de données à des emplacements centralisés pour une analyse ultérieure. Cette fonction est accentuée par un des douze facteurs qui définissent les journaux en tant que flux d'événements. De plus, elle est étendue à toutes les données générées par un microservice afin de garantir que l'observation est possible.

Kubernetes inclut des fonctions de télémétrie intégrées, comme Heapster, mais il est plus probable que la télémétrie soit fournie par d'autres systèmes du plan régulateur Kubernetes. Par exemple, deux des composants d'Istio, Mixer et Envoy fonctionnent ensemble pour [collecter de manière transparente la télémétrie à partir d'applications déployées](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

Les défaillances ne sont plus des occurrences rares et perturbatrices. Le fractionnement d'une application monolithique dans des microservices permet de transférer une plus grande partie du chemin principal dans le réseau, ce qui augmente l'impact du temps d'attente et d'autres problèmes réseau. Les demandes atteignent également des processus qui ne sont pas prêts à fonctionner. Les services sont automatiquement redémarrés si les ressources sont insuffisantes. De plus, les stratégies de tolérance aux pannes permettent au système dans son ensemble de continuer de fonctionner. Une intervention manuelle pour les défaillances individuelles n'est pas particulièrement utile ou possible dans ce type d'environnement.

## Surveillance
{: #monitoring}

Les concepts d'observabilité et de télémétrie permettent de mettre en exergue certaines différences significatives en matière de surveillance des applications Cloud native dans des systèmes distribués à grande échelle. Gardez à l'esprit que les processus des environnements Cloud native sont transitoires. Trois des douze facteurs (processus, accès concurrent et disponibilité) mettent l'accès sur cette spécificité. Les processus préalloués, à exécution longue et monolithiques sont remplacés ou entourés par un grand nombre de processus éphémères démarrés et arrêtés en réponse au chargement pour la mise à l'échelle horizontale ou s'ils ne fonctionnent pas correctement. La télémétrie est un processus critique lorsque les données doivent être collectées et conservées à un autre emplacement afin d'empêcher qu'elles ne soient perdues lorsque les processus (conteneurs) sont créés et supprimés. La télémétrie est également souvent requise pour des raisons de conformité. 

Pour toutes les raisons décrites précédemment, la surveillance change de cible : au lien de surveiller le comportement et l'état des ressources (processus individuels ou machines individuelles), l'état du système dans son ensemble est surveillé. Chaque service individuel génère des données qui sont placées dans cette vue agrégée.

