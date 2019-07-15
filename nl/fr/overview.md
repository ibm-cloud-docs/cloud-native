---

copyright:
  years: 2019
lastupdated: "2019-05-20"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Qu'est-ce que le concept Cloud Native ?
{: #overview}

Les environnements Cloud Computing sont dynamiques, ils offrent une libération et une allocation à la demande des ressources d'un pool partagé virtualisé. Ces environnements élastiques proposent des options d'évolution plus flexibles par rapport à l'allocation de ressources initiale généralement utilisée dans les centres de données sur site standard.
{:shortdesc}

Selon la fondation [Cloud Native Computing Foundation](https://github.com/cncf/foundation/blob/master/charter.md){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), les systèmes Cloud native ont les caractéristiques suivantes :

- Les applications ou les processus s'exécutent dans des conteneurs logiciels en tant qu'unités isolées.
- Les processus sont gérés par des processus d'orchestration centraux afin d'améliorer l'utilisation des ressources et de réduire les coûts de maintenance.
- Les applications ou les services (microservices) sont faiblement associés à des dépendances décrites explicitement.

Ces attributs décrivent un système hautement dynamique composé de processus indépendants, qui associés offrent une valeur métier : un système distribué.

L'informatique en réseau est un concept datant de plusieurs décennies. Le document [Fallacies of Distributed Computing](https://www.simpleorientedarchitecture.com/8-fallacies-of-distributed-systems/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") rassemble les suppositions suivantes des architectes et des concepteurs de systèmes distribués qui se sont révélées fausses à long terme. 

* Le réseau est fiable.
* Le réseau est sécurisé.
* Le réseau est homogène.
* Le temps d'attente est égal à zéro.
* La bande passante est illimitée.
* La topologie ne change pas.
* Il existe un seul administrateur.
* Les coûts de transport sont égaux à zéro.

Les technologies cloud, comme Kubernetes et Istio, ont pour but de traiter ces problèmes dans l'infrastructure elle-même.

## Douze facteurs
{: #twelve-factors}

La méthodologie d'[application à douze facteurs](https://12factor.net){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") a été ébauchée par des développeurs sur la plateforme Heroku. Les caractéristiques mentionnées dans les douze facteurs ne sont pas spécifiques à un langage, une plateforme ou à un fournisseur de cloud. Ces facteurs représentent un ensemble d'instructions ou de meilleures pratiques pour les applications résilientes et portables qui se développent dans des environnements de cloud (spécifiquement des applications SaaS, Software as a Service). Les douze facteurs sont présentés dans la liste suivante :

1. Il existe une association individuelle entre un codebase versionné, par exemple, un référentiel Git et un service déployé. Le même codebase est utilisé pour un grand nombre de déploiements.
2. Les services déclarent explicitement toutes les dépendances, et ne s'appuient pas sur la présence de bibliothèques ou d'outils de niveau système.
3. La configuration qui varie entre les différents environnements de déploiement est stockée dans l'environnement, plus particulièrement dans les variables d'environnement.
4. Tous les services de sauvegarde sont traités comme des ressources associées, qui sont gérées (associées et dissociées) par l'environnement d'exécution.
5. Le pipeline de distribution inclut des étapes strictement séparées : génération, publication, exécution.
6. Les applications sont déployées en tant qu'un ou plusieurs processus sans état. Plus spécifiquement, les processus transitoires sont sans état et ne partagent rien. Les données persistantes sont stockées dans un service de sauvegarde approprié.
7. Les services autonomes se rendent disponibles pour les autres services en écoutant sur un port spécifié.
8. L'accès concurrent est obtenu en faisant évoluer des processus individuels (mise à l'échelle horizontale).
9. Les processus sont remplaçables : des comportement de démarrage rapide et d'arrêt approprié génèrent un système plus robuste et résilient.
10. Tous les environnements, du développement local à la production, sont le plus similaires possible.
11. Les applications génèrent des journaux sous forme de flux d'événements (par exemple, écriture dans `stdout` et `stderr`) et laissent à l'environnement d'exécution la tâche d'agréger les flux.
12. Si des tâches d'administration ponctuelles sont requises, elles sont conservées dans le contrôle des sources et conditionnées avec l'application afin de garantir qu'elles s'exécutent avec le même environnement que cette dernière.

Il n'est pas nécessaire de suivre strictement ces facteurs pour obtenir un environnement de microservice de qualité. Cependant, gardez-les à l'esprit afin de générer et gérer des services ou des applications portables dans des environnements de distribution continue.

## Microservices
{: #microservices}

Un *microservice* est un ensemble de petits composants architecturaux indépendants, chacun ayant un but spécifique, qui établit une communication via une API légère standard. Chaque microservice de l'exemple simple suivant est une application à douze facteurs qui utilise des services de sauvegarde remplaçables pour stocker des données et transmettre des messages :

![Applications de microservices](images/microservice.png "Application de microservices")

Les microservices sont indépendants. L'agilité constitue un des avantages des architectures de microservice, mais cette fonction est disponible uniquement lorsque les services peuvent être complètement réécrits sans affecter les autres services. Cette situation se produit rarement mais cela explique cette exigence. Des limites d'API précises offrent à l'équipe utilisant un service une forte flexibilité pour faire évoluer l'implémentation. Cette caractéristique permet la persistance et la programmation polyglotte.

Les microservices sont résilients. La stabilité de l'application dépend de la résistance aux échecs des microservices individuels. Cela constitue une différence majeure par rapport aux architectures standard où l'infrastructure de prise en charge gère pour vous les échecs. Chaque service doit appliquer des modèles d'isolation, tels que des disjoncteurs et des cloisons, afin de répondre aux échecs et de définir des comportements de rétromigration pour protéger les services en amont.

Les microservices sont des processus sans état transitoires. Cela ne revient pas à dire que les microservices ne peuvent pas avoir d'état. Cela signifie que l'état doit être stocké dans des services de cloud de sauvegarde externes, tels Redis, et non en mémoire. De plus, le comportement de démarrage rapide et d'arrêt approprié permet aux microservices de fonctionner correctement dans des environnements automatisés qui créent et suppriment des instances en réponse au chargement ou pour gérer la santé du système.

### Signification de "petit"
{: #small-microsvc}

L'utilisation du mot "petit", lorsqu'il est appliqué à un microservice, signifie principalement que le microservice a un objectif : il doit faire quelque chose et le faire bien. Un grand nombre de descriptions effectuent un parallèle entre les rôles des microservices individuels et les commandes en chaîne sur la ligne de commande Unix :

```
ls | grep 'service' | sort -r
```
{:pre}

Chacune de ces commandes Unix effectue séparément des tâches différentes et vous pouvez les enchaîner, quel que soit le langage de programmation ou la quantité de code.

## Applications polyglottes : choix de l'outil approprié pour le travail
{: #polyglot-apps}

L'aspect polyglotte constitue un avantage fréquemment cité des architectures reposant sur des microservices. D'une part, la possibilité de choisir le magasin de données ou le langage approprié pour la fonction mise à disposition par un service peut être très puissante et peut apporter une efficacité optimale. D'autre part, l'utilisation de technologies méconnues peut compliquer la maintenance à long terme et empêcher les développeurs de passer d'une équipe à une autre. 

Etablissez un équilibre entre ces deux points en créant une liste de technologies prises en charge dans laquelle effectuer une sélection initiale, avec une règle définie permettant d'élargir la liste avec de nouvelles technologies au fil du temps. Vérifiez que les exigences non fonctionnelles ou réglementaires, comme la facilité de maintenance, l'auditabilité et la sécurité des données, peuvent être respectées, tout en conservant l'agilité et en prenant en charge l'innovation via l'expérimentation.

## REST et JSON
{: #rest-json}

Les applications polyglottes sont possibles uniquement avec des protocoles indépendants du langage. Les modèles d'architecture REST définissent des instructions de création d'interfaces uniformes qui séparent la représentation de données sur le fil de l'implémentation du service.

JSON a émergé dans des architectures de microservices en tant que format WF de prédilection pour les données de type texte, remplaçant XML avec sa concision et sa simplicité comparatives. L'exemple suivant est un enregistrement de base qui contient des données sur un employé en notation JSON :

```json
{
  "name": "Marley Cassin",
  "serial": 228264,
  "title": "Senior Software Engineer",
  "address": {
    "office": "501-B101",
    "street": "3858 Kuvalis Pass",
    "city": "East Craig",
    "state": "NC",
    "zip": "64519-8934"
  }
}
```
{: codeblock}

Et l'exemple suivant correspond au même enregistrement d'employé au format XML :

```xml
<person>
  <name>Marley Cassin</name>
  <serial>228264</serial>
  <title>Senior Software Engineer</title>
  <address>
    <office>501-B101</office>
    <street>3858 Kuvalis Pass</street>
    <city>East Craig</city>
    <state>NC</state>
    <zip>64519-8934</zip>
  </address>
</person>
```
{: codeblock}

JSON utilise des paires attribut/valeur pour représenter des objets de données dans une syntaxe concise qui conserve les informations de certains types de base, comme des nombres, des chaînes, des grappes et des objets. JSON et XML représentent dans les exemples précédents l'objet address imbriqué, mais vous avez besoin du schéma XML associé pour déterminer le type de l'élément `serial`. En notation JSON, la syntaxe indique clairement que la valeur de `serial` est un nombre et non une chaîne.
