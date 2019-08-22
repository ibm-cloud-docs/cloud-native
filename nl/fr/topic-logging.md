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

# Journalisation
{: #logging}

Les messages de journal sont des chaînes contenant des informations contextuelles relatives à l'état et à l'activité d'un microservice lors de la création de l'entrée de journal. Les journaux permettent de diagnostiquer comment et pourquoi les services échouent. Ils jouent un rôle de prise en charge des métriques pour la surveillance de l'état des applications.
{:shortdesc}

Assurez-vous d'écrire les entrées de journal directement dans les flux d'erreurs et de sortie standard. Ainsi, il est possible de consulter les entrées de journal en utilisant des outils de ligne de commande. De plus, les services de transmission de journal configurés au niveau de l'infrastructure peuvent prendre en charge la collecte des journaux et la gestion des données.

Vous devez être vigilant lors de la gestion des fichiers journaux si une application conteneurisée ne peut pas être configurée pour écrire des journaux dans le flux de sortie standard ou d'erreurs standard.

* Une option consiste à utiliser un volume pour les données de journal, soit un montage d'association simple pour le test et le développement locaux, soit un volume persistant approprié faisant partie d'un déploiement Kubernetes. Un [agent de consignation ou un composant sidecar dédié](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") peut lire des données dans un volume partagé afin de transmettre les journaux à un agrégateur centralisé. La rotation des journaux doit être configurée explicitement pour contrôler la quantité de données de journal stockées sur les volumes.
* Une autre option consiste à utiliser les agents ou les bibliothèques d'applications pour transmettre les journaux directement aux agrégateurs. L'utilisation de cette option peut être à l'origine d'une configuration complexe dans les environnements de déploiement.

## Journalisation JSON
{: #json-logging}

Lorsque votre application évolue au fil du temps, la nature des éléments journalisés peut changer. En utilisant un format de journal JSON, vous bénéficiez des avantages suivants :

* Les journaux sont indexables, ce qui facilite la recherche d'un corps agrégé de journaux.
* Les journaux sont résilients face aux modifications car l'analyse ne dépend pas de l'emplacement des éléments dans une chaîne.

Il est plus facile pour les machines d'analyser les journaux formatés mais les humains ont du mal à lire ces journaux. Envisagez d'utiliser des variables d'environnement pour passer du format de journal en texte en clair pour le débogage et le développement local au format de journal JSON pour le stockage et l'agrégation à plus long terme.

Les analyseurs syntaxiques JSON de ligne de commande, comme l'outil JSON Query (jq), peuvent être utilisés pour créer des vues lisibles des journaux formatés JSON. Dans l'exemple suivant, les journaux sont dirigés via grep afin de garantir que la zone de message apparaît avant l'analyse de la ligne par jq :

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## Affichage des journaux en utilisant `kubectl`
{: #view-logs-kube}

Les journaux envoyés aux flux d'erreur et de sortie standard peuvent être affichés dans Kubernetes via la console ou les commandes `kubectl` au format `kubectl logs <podname>`.

Si vous utilisez un espace de nom personnalisé, comme stock-trader, incluez ce dernier dans la commande (`kubectl logs -n stock-trader <podname>`).

S'il existe plusieurs conteneurs pour chaque pod, comme c'est le cas avec les composants sidecar istio, vous devez également spécifier le conteneur. Dans l'exemple suivant, l'espace de nom stock-trader permet d'afficher les journaux du conteneur `portfolio` dans le pod `portfolio-54b4d579f7-4zvzk` :

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

Pour les journaux au format JSON, vous pouvez utiliser `jq` pour extraire une zone de message, par exemple :

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Les entrées de journal sont dirigées via `grep` afin de garantir que `jq` analyse les lignes contenant une zone de message.
{: note}
