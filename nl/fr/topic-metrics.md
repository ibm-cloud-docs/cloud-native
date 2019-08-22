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

# Métriques
{: #metrics}

Les métriques sont des mesures numériques simples qui sont capturées en tant que paires clé-valeur. Certaines métriques incrémentent des compteurs et d'autres effectuent des agrégations (somme de toutes les valeurs collectées au cours de la dernière minute ou temps écoulé moyen, par exemple). D'autres métriques sont de simples jauges qui renvoient la dernière valeur observée. La capture et le traitement des métriques peuvent vous aider à identifier et à répondre aux problèmes potentiels avant que la situation n'empire et que des problèmes plus graves n'aient lieu.
{:shortdesc}

Trois facteurs généraux s'appliquent aux métriques d'un système distribué : producteurs, agrégateurs et processeurs. Il existe plusieurs combinaisons courantes de ces facteurs, telles que l'utilisation de Prometheus comme agrégateur avec un traitement Grafana qui collecte des métriques pour affichage dans les tableaux de bord graphiques, ou l'utilisation de StatsD avec Graphite.

![Trois facteurs des métriques de système distribué](images/metrics-systems.png "Trois facteurs des métriques de système distribué")

Le producteur est l'application. Dans certains cas, l'application est directement impliquée dans les métriques de production. Dans d'autres cas, les agents ou d'autres infrastructures observent passivement ou instrumentent activement l'application afin de produire des métriques en son nom. La suite des événements dépend de l'agrégateur.

Les métriques sont transférées du producteur à l'agrégateur via des mécanismes "push" ou "pull". Certains agrégateurs, comme StatsD, attendent de l'application qu'elle se connecte à l'agrégateur pour transmettre des données. Les informations de connexion de l'agrégateur doivent être distribuées à tous les processus d'application qui sont mesurés. Les autres agrégateurs, comme Prometheus, se connectent périodiquement à un noeud final connu afin de collecter des données de métriques. Pour cela, le producteur doit définir et fournir un noeud final pouvant être amassé et l'agrégateur doit indiquer où se trouvent les noeuds finaux. En cas d'utilisation avec Kubernetes, Prometheus peut détecter des noeuds finaux en fonction des annotations de service.

Enfin, le processeur utilise toutes les données agrégées. Des services tels que Grafana traitent les métriques agrégées pour la visualisation en utilisant des tableaux de bord. Grafana prend également en charge les alertes par règles stockées et évaluées indépendamment du tableau de bord.

## Reconnaissance automatique des noeuds finaux Prometheus dans Kubernetes
{: #prometheus-kubernetes}

Le modèle basé sur l'extraction crée son propre écosystème. Certains agrégateurs, tels Sysdig, peuvent également amasser des données de métriques provenant de noeud finaux Prometheus. Autrement dit, certains systèmes utilisent des métriques Prometheus sans utiliser le serveur Prometheus.

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

Quels sont les points de données qui doivent être collectés ? Avec une application native pour le cloud, vous pouvez diviser les métriques en deux catégories, application et plateforme :

* Les métriques d'application se concentrent principalement sur les entités de domaine d'application. Par exemple, quel est le nombre de connexions ayant abouti au cours des cinq dernières minutes ? Combien d'utilisateurs sont actuellement connectés ? Combien de transactions ont été effectuées au cours de la dernière seconde ? Le rassemblement de métriques spécifiques à l'application exige un code personnalisé pour collecter et publier les informations. La plupart des langages ont des bibliothèques de métriques permettant de simplifier l'ajout de métriques personnalisées.
* En revanche, les métriques de plateforme hébergent des entités de domaine. Par exemple, quelle est la durée de l'appel d'un service ? Combien de temps dure cette requête de base de données ? Ces métriques, qui sont alignées sur la façon dont le trafic et le travail transitent via le système, peuvent souvent être mesurées sans modification de l'application. Certaines infrastructures d'application fournissent également un support intégré pour la mesure de ces données, comme les durées d'exécution des requêtes.

Ces catégories seules ne sont pas suffisantes pour réellement définir les éléments à mesurer. Dans un système distribué, un grand nombre d'opérations génèrent des métriques. Il existe plusieurs méthodes bien connues, dont l'un des rôles est de limiter le vaste pool des métriques disponibles aux métriques principales qui nécessitent une surveillance :

* Connues sous le nom de [four golden signals of monitoring](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), elles constituent les métriques essentielles pour la surveillance d'un système. Elles sont identifiées par l'équipe SRE (Site Reliability Engineering) pour leur observabilité au niveau du service lors de la surveillance. Autrement dit, "Si vous ne pouvez mesurer que quatre métriques de votre système utilisateur, concentrez-vous sur ces métriques." Il s'agit du temps d'attente, du trafic, des erreurs et de la saturation.
* La [méthode USE (Utilization Saturation and Errors)](http://www.brendangregg.com/usemethod.html){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est conçue en tant que liste de contrôle d'urgence permettant d'analyser les performances d'un système. Cette méthode peut être résumée en une seule phrase, "Pour chaque ressource, vérifiez l'utilisation, la saturation et les erreurs." Une ressource (dans ce cas, ressources logiques ou physiques avec des limites absolues), comme des unités centrales ou des disques. Pour chaque ressource limitée, vous mesurez l'utilisation, la saturation et les erreurs. 
* La [méthode RED](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") est une mnémonique dérivée des quatre principaux signaux évoqués précédemment (voir "four golden signals of monitoring", ci-dessus) qui définit trois métriques clé que vous mesurez pour chaque microservice de votre architecture. Cette méthode, principalement centrée sur les demandes (plus particulièrement si vous la comparez à la méthode USE), est bien adaptée à la conception et à l'architecture d'un grand nombre d'applications natives pour le cloud. Les métriques sont les suivantes : débit, erreurs et durée. 

La méthode USE se concentre sur les métriques d'infrastructure. Les environnements natifs pour le cloud sont conçus pour mieux utiliser le matériel physique ou virtuel. Ces mesures d'infrastructure vérifient si votre système traite correctement la charge. La méthode RED porte entièrement sur les métriques de demande qui indiquent les problèmes liés à l'infrastructure et aux applications. Les principaux signaux s'appliquent à ces éléments, plaçant les métriques d'infrastructure et de demande dans une vue holistique.

## Définition de métriques
{: #defining-metrics}

La surveillance des métriques à partir d'un service unique peut vous donner une idée de l'utilisation des ressources. Toutefois, s'il existe plusieurs instances du service, vous devez les distinguer pour isoler les problèmes avec des données entrantes similaires. Cela revient au processus d'affectation de nom. Dans certains cas, le système de métriques que vous utilisez peut imposer une structure à vos métriques. Prometheus recommande un élément tel `namespace_subsystem_name`, d'autres systèmes conseillent `namespace.subsystem.targetNoun.actioned`.

Par exemple, si vous souhaitiez effectuer le suivi du nombre de "transactions" d'une application d'opérations sur action, vous pouvez capturer cette dernière dans une propriété appelée `stock.trades`. Pour effectuer une distinction entre les différentes instances, vous pouvez ajouter un préfixe à la propriété en indiquant l'ID d'instance : `<instanceid>.stock.trades`. Cette commande prend en charge la collecte de valeurs d'instance individuelles et agrège les données en utilisant `*.stock.trades`. Cependant, que se passe-t-il lorsque vous déployez les différents centres de données et que vous souhaitez analyser les métriques avec cette méthode ? Vous pouvez mettre à jour votre nom en indiquant `<datacenter>.<instanceid>.stock.trades`, mais cela interrompt toute production de rapports qui utilise le caractère générique précédent de `*.stock.trades`. Il est nécessaire d'utiliser `*.*.stock.trades` à la place. 

Une utilisation sans précaution des seules propriétés nommées hiérarchiquement peut conduire à des modèles de caractères génériques fragiles liés à la structure arbitraire de votre stratégie d'attribution de nom, ce qui ne facilite pas l'analyse des informations dont vous avez besoin pour vous assurer que votre application fonctionne correctement.

Avec les systèmes de métriques qui prennent en charge les données dimensionnelles, vous pouvez associer des libellés d'identification. Les libellés standard incluent un nom de noeud final ou de service, un centre de données, un code réponse, un environnement d'hébergement (prod/transfert/dev/test) ou des identifications d'exécution (informations de serveur d'application, de version Java).

En utilisant le même exemple boursier, la métrique `stock.trades`, associée à plusieurs libellés tels que `{ instanceid=..., datacenter=... }`, permet le filtrage ou le regroupement de la valeur agrégée par `instanceid` ou `datacenter` sans avoir recours aux caractères génériques. Un équilibre est établi entre la métrique nommée, `stock.trades`, et les libellés associés. Chaque métrique capture des données significatives, associées à des libellés pour éviter toute ambiguïté.

Définissez les libellés avec précaution. Dans Prometheus, chaque combinaison unique de paires clé/valeur est traitée comme une série temporelle distincte. Pour garantir un comportement de requête approprié et une collecte efficace des données liées, il est recommandé d'utiliser des libellés avec un nombre limité de valeurs admises.  

Si vous vous servez d'une métrique qui compte le nombre d'erreurs, vous pouvez utiliser le code retour en tant que libellé, dans lequel les valeurs sont incluses dans un ensemble raisonnable. N'attribuez pas de libellé à l'URL en échec. Il s'agit d'un ensemble non lié.

Pour plus d'informations sur les meilleures pratiques en matière d'attribution de nom aux métriques et aux libellés, voir [Metric and Label Naming](https://prometheus.io/docs/practices/naming/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Autres considérations
{: #metrics-considerations}

Un chemin en échec est souvent très différent d'un chemin abouti. Ainsi, une réponse d'erreur sur une ressource HTTP peut prendre beaucoup plus de temps  qu'une réponse réussie si l'échec implique des délais d'attente et une collecte de trace de pile. Comptez et traitez les chemins d'erreur séparément des demandes ayant abouti.

Certaines mesures d'un système distribué incluent des variations naturelles. Des erreurs occasionnelles sont normales, car les demandes peuvent être dirigées vers des processus au milieu de l'opération de démarrage ou d'arrêt. Filtrez les données brutes à intercepter quand cette variation naturelle dépasse une plage valide. Par exemple, répartissez les métriques dans des compartiments. Placez la durée des demandes dans des catégories telles 'plus petit/plus rapide', 'moyen/normal' et 'plus long/plus grand', selon les observations dans une fenêtre de durée. Si les durées des demandes sont toujours classées dans le compartiment "plus long/plus grand", vous pouvez identifier un problème. Les métriques d'histogramme ou récapitulatives sont généralement utilisées pour ce type de données. Pour plus d'informations, voir [Histograms and Summaries](https://prometheus.io/docs/practices/histograms/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

Assurez-vous que vos applications et services émettent des métriques avec des noms et des libellés qui respectent les conventions de l'organisation et prennent en charge les activités de surveillance de votre entreprise. Pour plus d'informations, voir [Monitoring distributed systems](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").
