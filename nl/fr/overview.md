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

# Qu'est-ce que le concept Cloud Native ?
{: #overview}

Les environnements Cloud Computing sont dynamiques, ils offrent une libération et une allocation à la demande des ressources d'un pool partagé virtualisé. Ces environnements élastiques proposent des options d'évolution plus flexibles par rapport à l'allocation de ressources initiale généralement utilisée dans les centres de données sur site standard.
{:shortdesc}

Selon la fondation [Cloud Native Computing Foundation](https://github.com/cncf/foundation/blob/master/charter.md){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), les systèmes Cloud native ont les caractéristiques suivantes :

- Les applications ou les processus s'exécutent dans des conteneurs logiciels en tant qu'unités isolées.
- Les processus sont gérés par des processus d'orchestration centraux afin d'améliorer l'utilisation des ressources et de réduire les coûts de maintenance.
- Les applications ou les services (microservices) sont faiblement associés à des dépendances décrites explicitement.

Ces attributs décrivent un système hautement dynamique composé de processus indépendants fonctionnant ensemble pour offrir une valeur métier, qui prend la forme d'un système distribué.

L'informatique en réseau est un concept qui est apparu il y plusieurs décennies. Le document [Fallacies of Distributed Computing
Explained](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") analyse les hypothèses suivantes, faites par des architectes et des concepteurs de systèmes distribués, qui se sont révélées fausses au final. 

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

La méthodologie d'[application à douze facteurs](https://12factor.net){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") a été ébauchée par des développeurs sur la plateforme Heroku. Les caractéristiques qui sont mentionnées dans les douze facteurs ne sont pas spécifiques à un langage, une plateforme ou à un fournisseur de cloud. Ces facteurs représentent un ensemble d'instructions ou de meilleures pratiques pour les applications résilientes et portables qui se développent dans des environnements de cloud (comme celui de l'application SaaS - Software as a Service, plus spécifiquement). Les douze facteurs sont présentés dans la liste suivante :

1. Il existe une association élément par élément entre un codebase versionné (un référentiel Git, par exemple) et un service déployé. Le même codebase est utilisé pour un grand nombre de déploiements.
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

Vous n'avez pas besoin de suivre strictement ces facteurs pour obtenir un environnement de microservice de qualité mais sachez toutefois qu'ils permettent de générer et de gérer des applications ou des services portables dans des environnements de distribution continue.

## Applications
{: #apps-intro}

Une application se compose de code, de données, de services et de chaînes d'outils. Ainsi, l'application mobile {{site.data.keyword.cloud_notm}}, qui contient le code de l'appareil ainsi qu'une logique de back end, le stockage de données, les services d'analyse et de sécurité, est configurée pour la distribution continue.

![Réutiliser](images/garage_reuse2.png "Avec Developer Experience, vous pouvez réutiliser l'existant")

Vous pouvez créer et gérer une application en utilisant un portail de développeur {{site.data.keyword.cloud_notm}} ou le plug-in {{site.data.keyword.dev_cli_notm}}.

Vous pouvez créer des applications simples et vides directement ou créer des applications plus complexes en utilisant les kits de démarrage. Si vous choisissez de créer des applications vides sans avoir recours à un kit de démarrage, vous pouvez utiliser le tableau de bord [{{site.data.keyword.cloud_notm}} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://{DomainName}){: new_window} sans accès à un portail.

Vous pouvez vous servir d'un modèle de code pour créer rapidement votre application et la déployer dans {{site.data.keyword.cloud_notm}}. Sur le site Web [IBM Developer ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")](https://developer.ibm.com/patterns/){:new_window}, choisissez un modèle de code. Vous pouvez afficher le code dans GitHub ou créer et générer une application sur {{site.data.keyword.cloud_notm}}, où vous pouvez utiliser une chaîne d'outils DevOps pour déployer automatiquement votre application.

## Microservices
{: #microservices}

Un *microservice* est un ensemble de petits composants architecturaux indépendants, qui communiquent via une API légère courante. Chaque microservice de l'exemple simple suivant est une application à douze facteurs qui utilise des services de sauvegarde remplaçables pour stocker des données et transmettre des messages :

![Applications de microservices](images/microservice.png "Application de microservices")

Les microservices sont indépendants. L'agilité est l'un des avantages des architectures de microservice mais elle n'existe que lorsque les services peuvent être réécrits sans perturber d'autres services. 

Des limites d'API claires offrent aux équipes qui utilisent un service une plus grande flexibilité pour faire évoluer l'implémentation. Cette caractéristique permet la persistance et la programmation polyglotte.

Les microservices sont résilients. La stabilité de l'application dépend de la résistance aux échecs des microservices individuels. Il s'agit d'une différence significative par rapport aux architectures standard dans lesquelles l'infrastructure de prise en charge gère les échecs à votre place. Chaque service doit appliquer des modèles d'isolation, tels que des disjoncteurs et des cloisons, afin de répondre aux échecs et de définir des comportements de rétromigration pour protéger les services en amont.

Les microservices, qui sont sans état, sont stockés dans des services de sauvegarde de cloud externes, tels que Redis. De plus, le comportement de démarrage rapide et d'arrêt approprié permet aux microservices de fonctionner correctement dans des environnements automatisés qui créent et suppriment des instances en réponse à la charge ou pour gérer la santé du système. 

### Signification de "petit"
{: #small-microsvc}

L'utilisation du terme "petit", quand il est appliqué à un microservice, signifie que le microservice poursuit un objectif spécifique. Un grand nombre de descriptions effectuent un parallèle entre les rôles des microservices individuels et les commandes en chaîne sur la ligne de commande Unix :

```
ls | grep 'service' | sort -r
```
{:pre}

Chacune de ces commandes UNIX effectue séparément des tâches différentes et vous pouvez les enchaîner, quel que soit le langage de programmation ou la quantité de code.

## Applications polyglottes : choix de l'outil approprié pour le travail
{: #polyglot-apps}

L'aspect polyglotte constitue un avantage fréquemment cité des architectures reposant sur des microservices. Utilisez ce bénéfice de façon équilibrée en créant une liste des technologies prises en charge avec possibilité d'en choisir une, associée à une règle autorisant l'élargissement de cette liste par l'ajout de nouvelles technologies au fil du temps. Vérifiez que les exigences non fonctionnelles ou réglementaires, comme la facilité de maintenance, l'auditabilité et la sécurité des données, peuvent être respectées, tout en conservant l'agilité et en prenant en charge l'innovation via l'expérimentation.

...   

## REST et JSON
{: #rest-json}

Les applications polyglottes sont possibles uniquement avec des protocoles indépendants du langage. Les modèles d'architecture REST définissent des instructions de création d'interfaces uniformes qui séparent la représentation de données sur le fil de l'implémentation du service.

JSON, dans les architectures de microservices, est le format WF (Wire Format) de prédilection pour les données à base de texte, supplantant le format XML du fait de sa simplicité et de sa concision. L'exemple suivant est un enregistrement de base qui contient des données sur un employé en notation JSON :

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
