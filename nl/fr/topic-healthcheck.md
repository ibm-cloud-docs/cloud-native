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

# Diagnostic d'intégrité
{: #healthcheck}

Les diagnostic d'intégrité fournissent un mécanisme simple dans les systèmes automatisés permettant d'examiner l'état de santé d'une instance individuelle. Le système répond aux événements d'inspection de santé en remplaçant une instance en échec, en mettant à jour les tables de routage ou en communiquant à l'utilisateur les états de santé qui en résultent.
{:shortdesc}

Kubernetes définit deux mécanismes intégraux permettant de vérifier la santé d'un conteneur :

* Une sonde de préparation est utilisée pour indiquer si le processus peut traiter les demandes qui sont routables. Kubernetes ne route pas un travail vers un conteneur pour lequel la sonde de préparation a indiqué un échec. Une sonde de préparation échoue si un service est occupé, si son initialisation n'est pas terminée, s'il est surchargé ou s'il ne peut pas traiter les demandes. 
* Une sonde de vivacité est utilisée pour indiquer si le processus doit être redémarré. Kubernetes arrête et redémarre un conteneur avec une sonde de vivacité ayant échoué afin de garantir que les pods inactifs sont terminés et remplacés. Une sonde de vivacité échoue si l'état du service est irrécupérable, en cas d'insuffisance de mémoire, par exemple. Les vérifications de signal de présence simples (initiées par cette sonde), qui renvoient toujours la réponse OK, sont capables d'identifier des conteneurs présentant un état incohérent, ce qui peut survenir quand le processus servant les demandes est arrêté alors que le conteneur est toujours en cours d'exécution.

Les sondes de préparation et de vivacité sont toutes les deux définies en utilisant une structure similaire qui inclut des temps d'attente et des intervalles entre les nouvelles tentatives, des périodes de tolérances aux pannes, des délais d'attente ainsi qu'une définition de l'implémentation des sondes. La sonde peut être implémentée en exécutant une commande, en vérifiant la connectivité d'un noeud final TCP ou en effectuant un appel HTTP. La même implémentation de sonde peut souvent être utilisée à la fois à des fins de préparation et de vivacité mais il est nécessaire d'ajuster le temps d'attente et les intervalles entre les nouvelles tentatives en fonction de l'objectif défini.

## Description et application des sondes
{: #kubernetes-probes}

Le développement d'application Cloud native s'appuie intrinsèquement sur le principe stipulant que les processus de conteneur échouent mais qu'ils sont immédiatement remplacés par un nouveau conteneur. Cette situation se produit en réponse à des événements inattendus, comme des défaillances de conteneur ou de machine. Elle est également provoquée par des événements opérationnels, comme la mise à l'échelle horizontale et de nouveaux déploiements d'images d'application. Les vérifications de préparation garantissent que les nouvelles instances de conteneur sont prêtes à recevoir du travail avant d'initier le routage du trafic. Ces mêmes vérifications empêchent le trafic d'être acheminé vers des instances qui ont été fermées ou supprimées.

Lorsqu'aucune vérification de préparation n'est définie, Kubernetes peut difficilement savoir si une instance de conteneur est prête à traiter le trafic et si elle achemine immédiatement le trafic dès le démarrage du processus de conteneur. Sans vérification de préparation, il est plus probable que des dépassements de délai pour les applications aient lieu et que des réponses de refus de connexion soient générées lorsque des travaux sont acheminés vers une instance qui n'est pas prête à gérer la demande. Les vérifications de préparation réduisent mais ne suppriment pas complètement les erreurs de connectivité client.

Les vérifications de signal de présence dont l'objectif est l'identification sont moins fréquentes et relèvent du domaine de l'exception. Lorsqu'un processus passe à un état dans lequel aucune récupération n'est possible, le processus devient réellement inefficace. Cette situation peut survenir en cas d'insuffisance de mémoire ou d'un blocage provoqué par une erreur de programmation. La meilleure solution de récupération, dans ce cas, est d'arrêter le conteneur, ce qui a pour effet de mettre fin à tout traitement en cours dans ce conteneur. Des boucles d'arrêt ou de redémarrage peuvent aussi se produire dans l'application, qui empêchent la remise en ligne des conteneurs, sans l'arrêt ou le remplacement préalable de ces derniers.

Les sondes de préparation et de vivacité impactent de différentes manières le système. Cette situation peut être envisagée en termes de transition d'état, où l'état positif d'une vérification de préparation est routable contrairement à l'état négatif. De la même façon, l'état positif représente un conteneur qui s'exécute normalement et l'état négatif indique que le conteneur n'est pas fonctionnel. Quand un conteneur démarre, l'état de préparation commence par être négatif et ne passe à l'état positif que lorsque le conteneur fonctionne correctement. L'état d'une vérification de vivacité commence par être positif puis devient négatif dès que le processus devient non fonctionnel.

Une configuration agressive du processus de vérification de préparation, proposant un délai initial faible, par exemple, n'a que peu d'effet, car une exécution trop rapide de la sonde ne permet pas à la vérification de préparation de changer d'état. Par contre, une configuration agressive du processus de vérification de signal de présence, dans laquelle la sonde de vivacité est déclenchée trop tôt, provoque un changement de transition d'état, ce qui conduit le système à arrêter le conteneur plus tôt que prévu.

## Meilleures pratiques pour la configuration des sondes
{: #probe-recommendation}

Si vous implémentez une sonde d'intégrité en utilisant HTTP, prenez en compte les codes de statut HTTP suivants pour la préparation, le signal de présence et l'intégrité.

| Etat    |  Préparation            |  Signal de présence             |
|----------|-----------------------|-----------------------|
|          | Non-OK causes no load | Non-OK causes restart |
| Démarrage | 503 - Unavailable     | 200 - OK              |
| Actif       | 200 - OK              | 200 - OK              |
| Arrêt | 503 - Unavailable     | 200 - OK              |
| Inactif     | 503 - Unavailable     | 503 - Unavailable     |
| Erreur  | 500 - Server Error    | 500 - Server Error    |
{: caption="Tableau 1. Codes d'état HTTP" caption-side="bottom"}

Les noeuds finaux de diagnostic d'intégrité n'exigent aucune autorisation ou authentification. Comme ces protections ne sont pas appliquées sur les noeuds finaux de sonde d'intégrité, restreignez les implémentations de sonde HTTP aux demandes GET qui ne modifient pas les données. Ne renvoyez jamais de données identifiant des caractéristiques spécifiques à votre environnement (système d'exploitation, langage d'implémentation ou versions logicielles, par exemple), car elles peuvent servir à établir un vecteur d'attaque.

Une sonde de vivacité doit être très précise sur les éléments à vérifier, car un échec entraîne un arrêt immédiat du processus. Evitez d'utiliser des métriques ambiguës qui indiquent parfois uniquement un processus défaillant, par exemple un noeud final HTTP simple qui renvoie toujours `{"status": "UP"}` avec un code d'état 200. Cette vérification échoue donc pour la plupart des processus dont l'état est inopérant, ce qui déclenche à raison un redémarrage.

Les diagnostics d'intégrité surviennent à intervalles réguliers, ce qui peut provoquer une surcharge supplémentaire. Les sondes de préparation et de vivacité testent uniquement la viabilité des services de sauvegarde (bases de données ou autres microservices) dans leur résultat, quand il n'existe aucune procédure de secours acceptable. Pour une sonde de vivacité, une vérification de la sauvegarde est incluse si un résultat non concluant  provoque le passage à l'état irrécupérable du conteneur local. Une sonde de préparation ne vérifie un service de sauvegarde que si le conteneur local ne peut pas traiter les demandes en cas de défaillance mais que cette situation n'est pas irréversible.

Lors de la configuration du temps d'attente initial, une sonde de préparation utilise la valeur la plus faible possible alors que la vérification de signal de présence se sert de la valeur temporelle la plus élevée possible. Par exemple, si un serveur d'applications a tendance à démarrer en trente secondes, le délai de préparation standard est de dix secondes. La vérification de signal de présence utilise une valeur de 60 secondes afin de garantir le démarrage complet du serveur avant le contrôle des états susceptibles d'être arrêtés.

L'attribut *periodSeconds* pour les décisions de routage est généralement configuré en tant que valeur à un seul chiffre, si l'implémentation de la sonde est relativement légère. Ainsi, une sonde HTTP qui renvoie un statut 200 OK sans traitement important côté serveur a une charge de processeur minimale et peut être répétée toutes les 1 à 5 secondes.

## Configuration de sondes dans Kubernetes
{: #probe-config}

Déclarez les sondes de préparation et de vivacité lors du déploiement Kubernetes dans le conteneur. Les deux sondes utilisent les mêmes paramètres de configuration :

| Paramètre | Description |
|-----------|-------------|
| *initialDelaySeconds* | Durée pendant laquelle Kubelet attend que le conteneur soit créé avant la première sonde. |
| *periodSeconds* | Fréquence à laquelle Kubelet analyse le service. La valeur par défaut est 1. |
| *timeoutSeconds* | Rapidité à laquelle la sonde arrive à échéance. La valeur par défaut et minimale est 1. |
| *successThreshold* | Nombre de fois où la sonde doit aboutir après un échec. La valeur par défaut et minimale est 1. La valeur doit être 1 pour les sondes de vivacité. |
| *failureThreshold* | Nombre de fois où Kubernetes tente de redémarrer un pod avant d'abandonner quand le pod démarre et que la sonde échoue. La valeur minimale est 1 et la valeur par défaut est 3. |

  Pour une sonde de vivacité, le fait d'abandonner redémarre le pod. Pour une sonde de préparation, le fait d'abandonner marque le pod comme n'étant pas prêt.
  {: note}

Pour éviter des cycles de redémarrage, définissez le paramètre `livenessProbe.initialDelaySeconds` de telle sorte que votre service dispose de suffisamment de temps pour être initialisé. Vous pouvez ensuite utiliser une valeur plus courte pour que l'attribut `readinessProbe.initialDelaySeconds` achemine des demandes au service dès qu'il est prêt.

Vous trouverez ci-dessous une configuration exemple (notez les valeurs path et port) :

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Pour plus d'informations, voir [Configure Liveness and Readiness Probes in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").
