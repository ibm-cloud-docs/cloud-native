---

copyright:
  years: 2019
lastupdated: "2019-07-18"

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

L'arrivée du concept Cloud Native a entraîné une modification de la surveillance. Bien qu'il soit admis que les applications des environnements sur site comme celles des environnements natifs pour le cloud sont hautement disponibles et résistantes aux défaillances, les méthodes utilisées pour atteindre ces objectifs sont très différentes. Par conséquent, l'objectif de la surveillance a changé : au lieu d'effectuer la surveillance pour éviter des défaillances, effectuez la surveillance pour gérer les défaillances. 
{:shortdesc}

Dans des environnements sur site, l'infrastructure et les logiciels middleware sont mis à disposition en fonction d'une capacité planifiée et de modèles haute disponibilité (de type active-active ou active-passive, par exemple). Des défaillances inattendues peuvent être complexes dans cet environnement, ce qui exige des efforts importants pour la détermination des problèmes et leur résolution. La surveillance externe est effectuée par des agents qui examinent l'utilisation des ressources afin d'éviter des classes connues de défaillances. Pensez à optimiser la taille de segment de mémoire, les délais d'attente et les règles de récupération de place pour les applications Java.

Une application native pour le cloud est composée de microservices indépendants et de services de sauvegarde requis. Même si une application de ce type, dans son ensemble, doit rester disponible et continuer à fonctionner, des instances de service individuelles démarreront ou s'arrêteront si nécessaire afin de s'adapter aux exigences de capacité ou procéder à une récupération après un incident. 

## Observabilité
{: #observability}

La surveillance de ce système fluide exige que chaque participant puisse *être observé*. Chaque entité doit générer des données appropriées pour la prise en charge de la détection de problème et de la génération d'alerte automatisées, du débogage manuel lorsque cela est nécessaire et de l'analyse de l'intégrité du système (analyses et tendances historiques).

Quels types de données doit générer un service pour pouvoir être observé ?

* Les **diagnostics d'intégrité** (souvent des noeuds finaux HTTP personnalisés) permettent aux orchestrateurs, comme Kubernetes ou Cloud Foundry, d'effectuer des actions automatisées pour gérer l'état général du système.
* Les **métriques** sont une représentation numérique des données qui sont collectées à des intervalles définis dans une série temporelle. Les données de séries temporelles numériques sont faciles à stocker et à interroger, ce qui vous permet de rechercher plus facilement des tendances historiques. Sur une plus longue période, les données numériques peuvent être compressées en des agrégats moins granulaires (tous les jours ou toutes les semaines, par exemple).
* Les **entrées de journal** représentent des événements discrets. Ces éléments sont essentiels pour le débogage car ils incluent souvent des traces de pile et d'autres informations contextuelles qui peuvent vous aider à identifier la cause principale des défaillances observées.
* Le **traçage distribué, de demande ou de bout en bout** capture le flux de bout en bout d'une demande dans le système. Le traçage capture principalement les relations entre les services (affectés par la demande) et la structure des travaux transitant via le système (traitement synchrone ou asynchrone, relations enfant ou élément résultant).

## Télémétrie
{: #telemetry}

Les applications natives pour le cloud s'appuient sur un environnement de *télémétrie*, qui permet de collecter et de transmettre automatiquement les données vers des emplacements centralisés pour une analyse ultérieure. Cette fonction est accentuée par un des douze facteurs qui définissent les journaux en tant que flux d'événements. De plus, elle est étendue à toutes les données générées par un microservice afin de garantir que l'observation est possible.

Kubernetes inclut des fonctions de télémétrie intégrées, comme Heapster, mais il est plus probable que la télémétrie soit fournie par d'autres systèmes du plan régulateur Kubernetes. Par exemple, deux des composants d'Istio, Mixer et Envoy fonctionnent ensemble pour [collecter de manière transparente la télémétrie à partir d'applications déployées](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

Les défaillances ne sont plus des occurrences rares et perturbatrices. Le fractionnement d'une application monolithique dans des microservices permet de transférer une plus grande partie du chemin principal dans le réseau, ce qui augmente l'impact du temps d'attente et d'autres problèmes réseau. Les demandes atteignent également des processus qui ne sont pas prêts à fonctionner. Les services sont automatiquement redémarrés si les ressources sont insuffisantes. De plus, les stratégies de tolérance aux pannes permettent au système dans son ensemble de continuer de fonctionner. Une intervention manuelle pour les défaillances individuelles n'est pas utile ou possible dans ce type d'environnement.

## Surveillance
{: #monitoring}

Les concepts d'observabilité et de télémétrie permettent de mettre en exergue certaines différences significatives en matière de surveillance des applications Cloud native dans des systèmes distribués à grande échelle. Gardez à l'esprit que les processus des environnements Cloud native sont transitoires. Trois des douze facteurs (processus, accès concurrent et disponibilité) mettent l'accès sur cette spécificité. Les processus préalloués, à exécution longue et monolithiques sont remplacés ou entourés par un grand nombre de processus éphémères démarrés et arrêtés en réponse au chargement pour la mise à l'échelle horizontale ou s'ils ne fonctionnent pas correctement. La télémétrie est un processus critique lorsque les données doivent être collectées et conservées à un autre emplacement afin d'empêcher qu'elles ne soient perdues lorsque les processus (conteneurs) sont créés et supprimés. La télémétrie est également souvent requise pour des raisons de conformité. 

Pour toutes les raisons décrites précédemment, la surveillance change de cible : au lien de surveiller le comportement et l'état des ressources (processus individuels ou machines individuelles), l'état du système dans son ensemble est surveillé. Chaque service individuel génère des données qui sont placées dans cette vue agrégée.

## Traçage, journalisation et métriques
{: #trace-log-metrics}

 

La journalisation est utilisée quand le développeur veut explicitement générer un message pour que quelqu'un d'autre le voit. Ce message, qui est codé directement dans la classe Java, inclut le transfert de valeurs des variables pertinentes. Quand des problèmes surviennent, les journaux sont utiles pour le débogage, car ils indiquent l'endroit où l'erreur s'est produite, en fournissant par exemple une trace de pile pour une exception qui a été émise. Vous utilisez *Kibana* pour afficher une vue fédérée de ces journaux sur les pods/microservices.

Le traçage se produisant automatiquement, le développeur n'effectue aucune action à ce niveau. Ainsi, vous pouvez configurer Liberty pour envoyer des enregistrements de trace vers un serveur de trace compatible Open Tracing dès qu'une méthode annotée JAX-RS est invoquée, ce qui vous permet de disposer d'un enregistrement d'audit de ce tout ce qui a été appelé, quand, par qui et pendant combien de temps. Vous pouvez aussi étendre cette trace, de façon à inclure par exemple des informations sur les méthodes privées qui ont été appelées dans votre code, en ajoutant des annotations Open Tracing aux méthodes qui sont tracées. 

Vous pouvez utiliser un outil tel que *Zipkin* ou *Jaeger* pour afficher une vue fédérée des traces sur les pods/microservices. Un *maillage de service* peut également fournir un traçage automatique des appels qui sont effectués via un composant sidecar pour votre conteneur.  

Les métriques sont utilisées pour suivre les valeurs agrégées. Ainsi, au lieu de passer par une trace ou un journal pour voir à quelle fréquence un utilisateur crée des portefeuilles, vous pouvez inclure cette opération dans une métrique de compteur personnalisé. Vous pouvez libeller votre déploiement de façon à ce qu'il soit "repéré" par *Prometheus*, qui agira en conséquence (appel périodique de l'URI /metrics sur vos pods, par exemple). Vous pouvez vous servir ensuite d'un outil tel que *Grafana* pour obtenir une vue fédérée des métriques sur les pods/microservices.

Le traçage vous permet de savoir qu'une méthode spécifique a été appelée. La journalisation vous indique ce qui s'est passé dans cette méthode quand elle a été appelée. C'est en consultant les métriques que vous saurez combien de fois elle a été appelée. Le traçage, qui est généralement implicite, se produit automatiquement sans intervention du développeur. Par contre, la journalisation, qui est explicite, nécessite le codage et l'envoi par le développeur d'informations qui peuvent être utiles lors de l'analyse a posteriori d'un problème. Les métriques sont elles aussi explicites, en ce sens que vous devez ajouter une annotation à une méthode donnée de votre code (même s'il existe souvent des métriques par défaut, de bas niveau, utilisables par le programmeur sans intervention de sa part, dont le rôle se limite à surveiller l'utilisation de la mémoire et de l'unité centrale ou à dénombrer les unités d'exécution, par exemple).

Généralement, les métriques sont utiles pour l'analyse alors que la journalisation est plutôt réservée à l'identification des problèmes et le traçage à la compréhension du flux de contrôle circulant d'un microservice à l'autre.
