---

copyright:
  years: 2019
lastupdated: "2019-03-26"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Métriques
{: #metrics}

Les métriques sont des mesures numériques simples capturées sous la forme clé/valeur. Certaines métriques incrémentent des compteurs, d'autres effectuent des agrégations, comme la somme de toutes les valeurs collectées au cours de la dernière minute ou le temps moyen écoulé au cours de la dernière minute. Certaines métriques sont uniquement des jauges qui renvoient la dernière valeur observée. La capture et le traitement des métriques peuvent vous aider à identifier et à répondre aux problèmes potentiels avant que la situation n'empire et que des problèmes plus graves n'aient lieu.
{:shortdesc}

Trois facteurs généraux s'appliquent aux métriques d'un système distribué : producteurs, agrégateurs et processeurs. Il existe plusieurs combinaisons assez répandues de ces facteurs, comme l'utilisation de Prometheus lorsque l'agrégateur avec traitement Grafana collecte des métriques pour affichage dans les tableaux de bord graphiques ou l'utilisation de StatsD avec graphite.

![Trois facteurs des métriques de système distribué](images/metrics-systems.png "Trois facteurs des métriques de système distribué"){: caption="Figure 1. Trois facteurs des métriques de système distribué" caption-side="bottom"}

Le producteur est, bien évidemment, l'application elle-même. Dans certains cas, l'application est directement impliquée dans les métriques de production. Dans d'autres cas, les agents ou d'autres infrastructures observent passivement ou instrument activement l'application afin de produire des métriques en son nom. La suite des événements dépend de l'agrégateur.

Les métriques sont transférées du producteur à l'agrégateur via des mécanismes "push" ou "pull". Certains agrégateurs, tels StatsD, s'attendent à ce que l'application (ou un agent au nom de l'application) se connecte à l'agrégateur pour transmettre des données. Pour cela, les informations de connexion de l'agrégateur doivent être distribuées à tous les processus d'application à mesurer. Les autres agrégateurs, comme Prometheus, se connectent régulièrement à un noeud final connu afin de rassembler des données de métriques. Pour cela, le producteur doit définir et fournir un noeud final pouvant être amassé et l'agrégateur doit indiquer où se trouvent les noeuds finaux. En cas d'utilisation avec Kubernetes, Prometheus peut détecter des noeuds finaux en fonction des annotations de service.

Pour finir, le processeur est un élément utilisant l'ensemble des données agrégées. Comme mentionné précédemment, les services tels Grafana traitent les métriques agrégées pour la visualisation en utilisant des tableaux de bord. Grafana prend également en charge les alertes par règles stockées et évaluées indépendamment du tableau de bord.

## Reconnaissance automatique des noeuds finaux Prometheus dans Kubernetes
{: #prometheus-kubernetes}

Le modèle basé sur la demande a créé son propre écosystème. Certains agrégateurs, tels Sysdig, peuvent également amasser des données de métriques provenant de noeud finaux Prometheus. Autrement dit, certains systèmes utilisent des métriques Prometheus sans utiliser le serveur Prometheus.

Dans les environnements Kubernetes, des annotations sont utilisées pour la reconnaissance de noeud final Prometheus. Par exemple, un service qui fournit un noeud final `/metrics` via HTTP sur le port 8080 ajoute les annotations suivantes à la définition de service :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /metrics
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

Tout agrégateur compatible avec Prometheus peut ensuite reconnaître ces noeuds finaux via l'API Kubernetes en filtrant par annotation.

## Métriques d'application/métriques de plateforme
{: #app-platform-metrics}

L'opération de collecte de métriques exige de la réflexion. Quels points de données devez-vous rassembler ? Dans le cas d'une application cloud native, vous pouvez répartir les métriques dans deux catégories, application et plateforme :

* Les métriques d'application se concentrent principalement sur les entités de domaine d'application. Par exemple, quel est le nombre de connexions ayant abouti au cours des cinq dernières minutes ? Combien d'utilisateurs sont actuellement connectés ? Combien de transactions ont été effectuées au cours de la dernière seconde ? Le rassemblement de métriques spécifiques à l'application exige un code personnalisé pour collecter et publier les informations. La plupart des langages ont des bibliothèques de métriques permettant de simplifier l'ajout de métriques personnalisées.
* En revanche, les métriques de plateforme hébergent des entités de domaine. Par exemple, quelle est la durée de l'appel d'un service ? Combien de temps dure cette requête de base de données ? Elles sont alignées avec le mode de transit via le système du trafic et des travaux. De plus, il est souvent possible de les mesurer sans aucune modification de l'application. Certaines infrastructures d'application fournissent également un support intégré pour la mesure de ces données, comme les durées d'exécution des requêtes.

Ces catégories seules ne sont pas suffisantes pour réellement définir les éléments à mesurer. N'oubliez pas que dans un système distribué, un grand nombre d'opérations génèrent des métriques. Il existe plusieurs méthodes bien connues qui tentent de restreindre le vaste pool de métriques à celles qui sont essentielles et qui doivent être surveillées :

* Les [quatre signaux principaux](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") constituent les principales métriques identifiées par l'équipe SRE (Site Reliability Engineering) de Google pour l'observabilité au niveau du service lors de la surveillance d'un système distribué. Autrement dit, "Si vous ne pouvez mesurer que quatre métriques de votre système utilisateur, concentrez-vous sur ces métriques." Il s'agit du temps d'attente, du trafic, des erreurs et de la saturation.
* La [méthode USE (Utilization Saturation and Errors)](http://www.brendangregg.com/usemethod.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est conçue en tant que liste de contrôle d'urgence permettant d'analyser les performances d'un système. Cette méthode peut être résumée en une seule phrase, "Pour chaque ressource, vérifiez l'utilisation, la saturation et les erreurs." Une ressource (dans ce cas, ressources logiques ou physiques avec des limites absolues), comme des unités centrales ou des disques. Pour chaque ressource limitée, vous mesurez l'utilisation, la saturation et les erreurs. 
* La [méthode RED](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est une mnémonique dérivée des quatre principaux signaux qui définit trois métriques clés que vous devez mesurer pour chaque microservice de votre architecture. Cette méthode est principalement centrée sur les demandes (plus particulièrement en comparaison avec la méthode USE). De plus, elle est adaptée à la conception et à l'architecture d'un grand nombre d'applications Cloud native. Les métriques sont les suivantes : débit, erreurs et durée. 

Il existe plusieurs éléments communs dans ces méthodes. Elles effectuent toutes le suivi du taux d'erreur, par exemple. Cependant, il existe également des différences notables. La méthode USE se concentre sur les métriques d'infrastructure (utilisation/saturation). Les environnements Cloud native sont conçus pour mieux utiliser le matériel physique (ou virtuel). Ces mesures d'infrastructure vérifient si votre système traite correctement la charge. La méthode RED est entièrement tournée vers les métriques de demande (temps d'attente/durée, trafic/débit), qui peuvent indiquer des problèmes liés à l'infrastructure (problèmes réseau) ou à vos applications (configurations incorrectes, blocages, défaillances répétées du service, etc.), plus particulièrement lorsque ces données sont associées à des taux d'erreur observés. Les principaux signaux s'appliquent à ces éléments, plaçant les métriques d'infrastructure et de demande dans une vue holistique.

## Définition de métriques
{: #defining-metrics}

La surveillance des métriques rassemblées à partir d'un service unique peut vous donner une idée de son utilisation des ressources mais s'il existe plusieurs instances du service (en raison de la mise à l'échelle horizontale ou de déploiements multi-régions), vous devez les distinguer afin d'isoler les problèmes parmi la masse de données similaires entrantes. Cela revient au processus d'affectation de nom. Dans certains cas, le système de métriques que vous utilisez peut imposer une structure à vos métriques. Prometheus, par exemple, recommande un élément tel `namespace_subsystem_name`, d'autres recommandent `namespace.subsystem.targetNoun.actioned`.

Par exemple, si vous souhaitiez effectuer le suivi du nombre de "transactions" d'une application d'opérations sur action, vous pouvez capturer cette dernière dans une propriété appelée `stock.trades`. Pour effectuer une distinction entre les différentes instances, vous pouvez ajouter un préfixe à la propriété en indiquant l'ID d'instance : `<instanceid>.stock.trades`. Cela prend en charge la collecte de valeurs d'instance individuelles ainsi que l'agrégation de données en utilisant `*.stock.trades`. Cependant, que se passe-t-il lorsque vous déployez les différents centres de données et que vous souhaitez analyser les métriques avec cette méthode ? Vous pouvez mettre à jour votre nom en indiquant `<datacenter>.<instanceid>.stock.trades`, mais cela interrompt toute production de rapports qui utilise le caractère générique précédent de `*.stock.trades`. Il est nécessaire d'utiliser `*.*.stock.trades` à la place. 

Une utilisation sans aucune vigilance des propriétés nommées hiérarchiquement peut générer des masques de caractère générique faibles liés à la structure arbitraire de votre stratégie d'attribution de nom qui ne vous aident pas à observer les informations dont vous avez besoin afin de vous assurer que votre application fonctionne correctement.

Les systèmes de métriques qui prennent en charge les données dimensionnelles vous permettent d'associer des libellés d'identification supplémentaires (ou étiquettes) à des données de métriques. Vous pouvez ensuite filtrer, grouper ou analyser les métriques collectées en utilisant ces dimensions supplémentaires sans avoir recours aux caractères génériques et aux conventions d'attribution de nom. Les libellés standard incluent un nom de noeud final ou de service, un centre de données, un code réponse, un environnement d'hébergement (prod/transfert/dev/test) ou des identifications d'exécution (informations de serveur d'application, de version Java).

Lors de l'utilisation du même exemple d'opérations d'actions, la métrique `stock.trades` est associée à plusieurs libellés : `{ instanceid=..., datacenter=... }`, qui permet le filtrage ou le regroupement de la valeur agrégée par `instanceid` ou `datacenter` sans avoir recours aux caractères génériques. Il existe un équilibre entre la métrique nommée (`stock.trades`) et les libellés associés (comparez cela à l'exemple hiérarchique de `<datacenter>.<instanceid>.stock.trades`) : chaque métrique doit capturer les données porteuses de sens, avec des libellés évitant toute ambiguïté lorsque cela est approprié.

De plus, soyez vigilant lors de la définition des libellés. Dans Prometheus, par exemple, chaque combinaison unique de paires clé/valeur est traitée comme une série temporelle distincte. Pour garantir un comportement de requête approprié et une collection de données liées, il est recommandé d'utiliser des libellés avec un nombre limité de valeurs admises. Par exemple, si vous utilisez une métrique qui compte le nombre d'erreurs, vous pouvez utiliser le code retour en tant que libellé lorsque les valeurs se trouvent dans un ensemble raisonnable (401, 404, 409, 500, ... ) mais vous ne devez pas utiliser de libellé pour l'URL ayant échoué, car il s'agit d'un ensemble non lié (toute URL de requête ayant échoué, par exemple pour raison de non validité).

Pour plus d'informations sur les meilleures pratiques en matière d'attribution de nom aux métriques et aux libellés, voir [Metric and Label Naming](https://prometheus.io/docs/practices/naming/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Remarques supplémentaires
{: #metrics-considerations}

Lors de la collecte de vos métriques, n'oubliez pas qu'un chemin d'échec est souvent très différent d'un chemin de succès. Par exemple, une réponse d'erreur sur une ressource HTTP peut durer beaucoup plus longtemps qu'une réponse de succès si l'erreur implique des délais d'attente et la collecte de la trace de pile. Comptez et traitez les chemins d'erreur séparément des demandes ayant abouti.

Certaines mesures d'un système distribué incluent des variations naturelles. Des erreurs occasionnelles sont normales, car les demandes peuvent être dirigées vers des processus au milieu de l'opération de démarrage ou d'arrêt. Filtrez les données brutes à intercepter lorsque cette variation naturelle commence à dépasser une plage valide. Par exemple, répartissez les métriques dans des compartiments. Placez la durée des demandes dans des catégories telles 'plus petit/plus rapide', 'moyen/normal' et 'plus long/plus grand', selon les observations dans une fenêtre de durée. Si les durées des demandes sont toujours classées dans le compartiment "plus long/plus grand", vous pouvez identifier un problème. Les métriques d'histogramme ou récapitulatives sont généralement utilisées pour ce type de données. Pour plus d'informations, voir [Histograms and Summaries](https://prometheus.io/docs/practices/histograms/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

En tant que développeur d'application, vérifiez que vos applications ou vos services émettent des métriques avec des noms et des libellés respectant les conventions de l'organisation afin de prendre en charge les différents efforts de surveillance ciblés sur les principales activités de bout en bout de votre entreprise. Pour plus d'informations, voir [Monitoring distributed systems](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").
