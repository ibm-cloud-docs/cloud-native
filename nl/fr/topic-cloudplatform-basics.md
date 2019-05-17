---

copyright:
  years: 2019
lastupdated: "2019-02-18"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Concepts de plateforme Cloud
{: #platform}

Cette section présente brièvement les principaux concepts et technologies utilisés par les développeurs lors de la génération d'applications Cloud native, à commencer par les conteneurs, Kubernetes, Helm et Istio.
{:shortdesc}

## Conteneurs
{: #containers}

Les conteneurs constituent un mécanisme de base pour le conditionnement d'une application et de toutes ses dépendances dans une seule unité autonome. Les conteneurs permettent de résoudre le problème de portabilité : l'artefact de conteneur (image) permet de s'assurer que tous les éléments devant être exécutés par une application se trouvent au bon emplacement. Les moteurs de conteneur peuvent ensuite se concentrer sur l'exécution de conteneurs en tant que processus isolés de manière sécurisée et efficace. 

Les images de conteneur sont généralement générées à partir d'une liste d'instructions définie dans un élément `Dockerfile`. Elles sont presque toujours générées à partir d'autres images de conteneur (principalement en tant que suite d'instructions d'un état précédent connu). Vous pouvez utiliser le fragment suivant pour créer votre propre image Open Liberty, par exemple :

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

Une fois qu'une image est générée, elle peut être exécutée. Les moteurs d'exécution de conteneur, comme Docker ou [containerd](https://containerd.io/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), utilisent cette définition d'image et exécutent le point d'entrée défini en tant que processus isolé par les ressources directement sur le système d'exploitation hôte, supprimant la surcharge des machines virtuelles.

Les images de conteneur sont stockées dans des *registres*. Le plus connu est le registre Docker Hub public. Généralement, vous transmettez et recevez des messages provenant de/à destination de registres de conteneurs contrôlés par l'accès, comme {{site.data.keyword.registryshort_notm}}, qui sont associés à votre infrastructure et aux pipelines CI/CD.

## Kubernetes
{: #kubernetes}

Les plateformes de cloud d'IBM optimisent Kubernetes pour l'orchestration de conteneur. C'est pourquoi, non seulement les développeurs doivent connaître les principes de base des conteneurs mais ils doivent également maîtriser les fondamentaux de Kubernetes, y compris les commandes et les artefacts de déploiement de base. Le tableau suivant présente certains des principaux concepts Kubernetes :

| Concept | Description |
|---------|-------------|
| Pod | Groupe localisé de conteneurs déployés ensemble en tant qu'unité unique. Les pods ne sont pas modifiables, ce qui fait qu'il est nécessaire de remplacer le pod d'origine pour apporter des modifications à différents attributs du pod. Une application standard inclut un conteneur avec une logique métier centrale et éventuellement des pods supplémentaires offrant des fonctionnalités de plateforme au niveau granulaire du pod. |
| Déploiement | Modèle reproductible pour un pod sans état, ajoutant une dimension d'échelle au concept de pod. De plus, la définition modélisée peut être mise à jour et les instances de pod sous-jacentes remplacées. Une configuration de déploiement Kubernetes est surveillée par un contrôleur de déploiement Kubernetes afin de garantir que le nombre déclaré de pods d'un déploiement indiqué est conservé. Un déploiement s'affiche sous la forme de `kind: Deployment` dans les fichiers `.yaml`. |
| Service | Nom connu qui représente un ensemble d'adresses IP de pod relativement instables. Un service peut exister sur le réseau privé de cluster uniquement ou être exposé en externe, généralement en utilisant un équilibreur de charge spécifique au fournisseur de cloud. Un service s'affiche sous la forme de `kind: Service` dans les fichiers `.yaml`. |
| Ingress | Permet de partager une adresse réseau unique avec plusieurs services via l'hébergement virtuel ou le routage contextuel. Un élément Ingress peut également effectuer des activités de gestion des connexions réseau, comme l'arrêt TLS. Un élément Ingress s'affiche comme `kind: Ingress` dans les fichiers `.yaml`. |
| Secret | Objet qui stocke les informations sensibles pour l'utilisation d'environnement d'exécution de Pod et sépare les informations spécifiques au déploiement de l'orchestration ou de l'image de conteneur. Un secret peut être exposé à un pod lors de l'exécution via des variables d'environnement ou des montages de système de fichiers virtuels. Sans secret, les données sensibles sont stockées dans l'orchestration ou l'image de conteneur. Ainsi, les risques d'exposition accidentelle ou d'accès indésirable sont plus élevés. |
| ConfigMap | Joue un rôle similaire aux secrets par le fait que cet élément sépare les informations spécifiques au déploiement de l'orchestration de conteneur. Toutefois, un élément ConfigMap constitue une structure de configuration générale. Il permet d'associer des informations, comme des arguments de ligne de commande, des variables d'environnement et d'autres artefacts de configuration, à vos conteneurs de pod et à vos composants système lors de l'exécution. | 
{: caption="Tableau 1. Concepts Kubernetes" caption-side="bottom"}

Toutes les ressources sont définies dans le modèle de ressource Kubernetes, qui peut être configuré via l'API RESTful ou via les fichiers de configuration soumis à partir de la ligne de commande `kubectl`.

Pour plus d'informations, voir les pages [Kubernetes basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), [Kubernetes Object Model](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") ainsi que les informations sur la ligne de commande [`kubectl`](https://kubernetes.io/docs/reference/kubectl/overview/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"). 

## Helm
{: #helm}

Helm est un gestionnaire de package permettant de facilement trouver, partager et utiliser des logiciels générés pour Kubernetes. Helm prend également en charge un besoin courant des utilisateurs : le déploiement de la même application dans plusieurs environnements. Helm utilise des *graphiques*, qui sont des collections de modèles générant des objets Kubernetes (YAML) lors de l'installation. Ces graphiques sont générés à partir d'un langage de modèle qui inclut le support des variables, des opérations de plage et d'autres opérations qui facilitent grandement la tâche de gestion des métadonnées de déploiement Kubernetes.

Pour plus d'informations, voir [Helm](https://helm.sh/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

## Service Mesh - Istio
{: #istio}

Istio est une plateforme open source permettant de gérer et de sécuriser les microservices. Cette plateforme fonctionne avec des orchestrateurs, ce qui permet de gérer et de contrôler la communication entre les services.

Istio utilise un modèle de composant sidecar. Un tel composant (proxy Envoy) est un processus distinct utilisé conjointement à votre application. Il gère toutes les communications de votre service, qu'elles soient entrantes ou sortantes, et applique un niveau commun de capacité à tous les services indépendants de la structure ou du langage de programmation utilisés pour la génération du service. En effet, Istio offre un mécanisme permettant de configurer centralement les règles de routage et de sécurité. De plus, ces règles sont appliquées par des composants sidecar de manière décentralisée. 

Nous vous recommandons d'utiliser les fonctions fournies par Istio et non les fonctions similaires fournies par les infrastructures ou les langages de programmation individuels. L'équilibrage de charge et d'autres règles d'acheminement sont définis, gérés et appliqués par l'infrastructure de manière cohérente et ce en tant qu'exemple.

Dans certains cas, comme avec le traçage distribué, Istio et les bibliothèques de niveau d'application sont complémentaires. Vous pouvez améliorer les performances en utilisant ensemble ces deux éléments. Pour le traçage distribué, Istio peut uniquement garantir que les en-têtes de trace sont présents. Les bibliothèques d'applications fournissent le contexte important sur les relations entre les demandes. Votre compréhension du système en tant qu'ensemble s'améliore lorsque Istio et les bibliothèques de prise en charge ou les bibliothèques d'infrastructure sont utilisés ensemble.

Au niveau le plus élevé, Istio étend la plateforme Kubernetes, offrant sécurité, visibilité et concepts de gestion supplémentaires. Les fonctions d'Istio peuvent être réparties dans les quatre catégories suivantes :

* Gestion du trafic : Contrôle le trafic entre vos microservices afin d'effectuer le fractionnement du trafic, la reprise après incident et de créer des versions canary.
* Sécurité : Permet le chiffrement, l'autorisation et l'authentification forte utilisant les identités entre les microservices.
* Observabilité : Collecte les métriques et les journaux pour une meilleure visibilité des applications s'exécutant dans votre cluster.
* Règles : Applique des contrôles d'accès, des limites de débit et des quotas pour protéger vos applications.

Voir la page [What is Istio?](https://istio.io/docs/concepts/what-is-istio/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour plus d'informations. 



