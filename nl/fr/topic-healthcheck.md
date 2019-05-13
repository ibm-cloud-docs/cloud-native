---

copyright:
  years: 2019
lastupdated: "2019-02-10"

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

Les diagnostic d'intégrité fournissent un mécanisme simple dans les systèmes automatisés permettant d'examiner l'état de santé d'une instance individuelle. Le système répond ensuite à ces événements d'inspection de santé en effectuant une action, comme le remplacement d'une instance défaillante, la mise à jour des tables de routage ou la communication à l'utilisateur des états de santé en résultant.
{:shortdesc}

Kubernetes définit deux mécanismes intégraux permettant de vérifier la santé d'un conteneur :

* La sonde de préparation est utilisée pour indiquer si le processus peut traiter les demandes (est routable). Kubernetes n'achemine pas les travaux vers un conteneur si la sonde de préparation a échoué. Une sonde de préparation échoue si l'initialisation d'un service n'est pas terminée ou si ce dernier est occupé, surchargé ou ne peut pas traiter les demandes.
* Une sonde de vivacité est utilisée pour indiquer si le processus doit être redémarré. Kubernetes arrête et redémarre un conteneur avec une sonde de vivacité ayant échoué afin de garantir que les pods inactifs sont terminés et remplacés. Une sonde de vivacité échoue si le service est à un état irrécupérable, par exemple lorsque la mémoire est insuffisante. Les vérifications de vivacité simples qui renvoient toujours la réponse OK peuvent identifier les conteneurs à un état incohérent. Cette situation peut survenir lorsque le processus traitant les demandes s'est arrêté mais que le conteneur est toujours en cours d'exécution.

Les sondes de préparation et de vivacité sont toutes les deux définies en utilisant une structure similaire qui inclut des temps d'attente et des intervalles entre les nouvelles tentatives, des périodes de tolérances aux pannes, des délais d'attente ainsi que la définition de l'implémentation de sonde. La sonde peut être implémentée en exécutant une commande, en vérifiant la connectivité d'un noeud final TCP ou en effectuant un appel HTTP. La même implémentation de sonde peut souvent être utilisée à la fois à des fins de préparation et de vivacité mais il est nécessaire d'ajuster le temps d'attente et les intervalles entre les nouvelles tentatives en fonction de l'objectif défini.

## Description et application des sondes
{: #kubernetes-probes}

Le développement d'application Cloud native s'appuie intrinsèquement sur le principe stipulant que les processus de conteneur échouent mais qu'ils sont immédiatement remplacés par un nouveau conteneur. Cette situation se produit en réponse à des événements inattendus, comme des défaillances de conteneur ou de machine mais également en raison d'événements opérationnels, comme la mise à l'échelle horizontale et les nouveaux déploiements d'image d'application. Les vérifications de préparation sont importantes car elles permettent de garantir que les nouvelles instances de conteneur sont prêtes à recevoir des travaux avant d'acheminer le trafic. De plus, les mêmes vérifications empêchent le trafic d'être acheminé vers des instances fermées ou supprimées.

Lorsque aucune vérification de préparation n'est définie, Kubernetes peut difficilement savoir si une instance de conteneur est prête à traiter le trafic et si elle achemine immédiatement le trafic dès le démarrage du processus de conteneur. Sans vérification de préparation, il est plus probable que des dépassements de délai pour les applications aient lieu et que des réponses de refus de connexion soient générées lorsque des travaux sont acheminés vers une instance qui n'est pas prête à gérer la demande. Les vérifications de préparation réduisent mais ne suppriment pas complètement les erreurs de connectivité client.

Même s'il est normal de modifier des cibles d'acheminement d'instance dans le cycle de vie d'une application activée par conteneur, le processus indique que les vérifications de vivacité conçues pour effectuer des tâches d'identification sont moins fréquentes et qu'elles représentent une exception et non la norme. Lorsqu'un processus passe à un état dans lequel aucune récupération n'est possible, le processus devient réellement inefficace. Cette situation peut survenir lorsque les conditions de mémoire sont insuffisantes ou en raison d'un interblocage provoqué par une erreur de programmation. La meilleure méthode de résolution de ce problème consiste à arrêter le conteneur, ce qui arrête également tout traitement en cours dans le conteneur. Cette opération permet également d'arrêter ou de redémarrer des boucles dans l'application, où les conteneurs ne peuvent pas revenir complètement en ligne avant d'être arrêtés et remplacés.

Les sondes de préparation et de vivacité impactent de différentes manières le système. Cette situation peut être envisagée en termes de transition d'état, où l'état positif d'une vérification de préparation est routable contrairement à l'état négatif. De la même façon, l'état positif représente un conteneur qui s'exécute normalement et l'état négatif indique que le conteneur n'est pas fonctionnel. Lorsqu'un conteneur démarre, l'état de préparation commence par être négatif puis passe à l'état positif dès que le conteneur est sain. L'état d'une vérification de vivacité commence par être positif puis devient négatif dès que le processus devient non fonctionnel.

La configuration agressive d'une vérification de préparation (avec un faible délai initial, par exemple) a peu de conséquences car l'exécution anticipée de la sonde ne permet pas à la vérification de préparation de changer d'état. Inversement, une vérification de vivacité agressive, dans laquelle la sonde est déclenchée trop tôt, provoque un changement de transition d'état, qui provoque à son tour un arrêt du conteneur plus tôt que prévu.

## Meilleures pratiques pour la configuration des sondes
{: #probe-recommendation}

Lors de l'implémentation d'une sonde d'intégrité en utilisant HTTP, prenez en compte les codes d'état HTTP suivants pour la préparation, la vivacité et l'intégrité :

| Etat    |  Préparation            |  Vivacité             |
|----------|-----------------------|-----------------------|
|          | Non-OK causes no load | Non-OK causes restart |
| Démarrage | 503 - Unavailable     | 200 - OK              |
| Actif       | 200 - OK              | 200 - OK              |
| Arrêt | 503 - Unavailable     | 200 - OK              |
| Inactif     | 503 - Unavailable     | 503 - Unavailable     |
| Erreur  | 500 - Server Error    | 500 - Server Error    |

Les noeuds finaux de diagnostic d'intégrité n'exigent aucune autorisation ou authentification. Etant donné que ces protections ne sont pas appliquées sur les noeuds finaux de sonde d'intégrité, restreignez les implémentations de sonde HTTP aux demandes GET qui ne modifient pas les données. Ne renvoyez jamais les données qui identifient des caractéristiques de l'environnement, comme le système d'exploitation, le langage d'implémentation ou les versions logicielles, car elles peuvent être utilisées pour établir un vecteur d'attaque.

Une sonde de vivacité doit être très précise sur les éléments à vérifier, car un échec entraîne un arrêt immédiat du processus. Evitez d'utiliser des métriques ambiguës qui indiquent parfois uniquement un processus défaillant, par exemple un noeud final HTTP simple qui renvoie toujours `{"status": "UP"}` avec un code d'état 200. Cette vérification n'aboutit pas pour la plupart des processus à l'état inopérant, ce qui déclenche à raison un redémarrage.

Les diagnostics d'intégrité surviennent à intervalles réguliers, ce qui peut provoquer une surcharge supplémentaire. Les sondes de préparation et de vivacité testent uniquement la viabilité des services de sauvegarde, comme les bases de données ou d'autres microservices, se trouvant dans leurs résultats lorsqu'il n'existe aucune rétromigration acceptable. Pour une sonde de vivacité, une vérification de sauvegarde doit être incluse uniquement si un mauvais résultat provoque le passage à l'état irrécupérable du conteneur local. Une sonde de préparation doit vérifier un service de sauvegarde uniquement lorsque le conteneur local ne peut pas traiter les demandes en cas de défaillance mais que cette situation n'est pas irréversible.

Lors de la configuration du temps d'attente initial, une sonde de préparation doit utiliser la valeur la plus faible possible et une vérification d'activité doit utiliser la valeur temporelle la plus élevée possible. Par exemple, si un serveur d'applications a tendance à démarrer en trente secondes, le délai de préparation standard est de dix secondes. La vérification de vivacité utilise la valeur de 60 secondes afin de garantir que le démarrage du serveur aboutit avant la vérification des états pouvant être arrêtés.

L'attribut *periodSeconds* pour les décisions de routage est généralement configuré en tant que valeur numérique unique, à condition que l'implémentation de sonde soit relativement légère. Par exemple, une sonde HTTP qui renvoie un statut 200 OK sans traitement côté serveur substantiel a une charge de processeur minimale et peut immédiatement être répétée toutes les 1 à 5 secondes.

## Configuration de sondes dans Kubernetes
{: #probe-config}

Déclarez les sondes de préparation et de vivacité lors du déploiement Kubernetes dans le conteneur. Les deux sondes utilisent les mêmes paramètres de configuration :

| Paramètre | Description |
|-----------|-------------|
| *initialDelaySeconds* | Durée pendant laquelle Kubelet attend que le conteneur soit créé avant la première sonde. |
| *periodSeconds* | Fréquence à laquelle Kubelet analyse le service. La valeur par défaut est 1. |
| *timeoutSeconds* | Rapidité à laquelle la sonde arrive à échéance. La valeur par défaut et minimale est 1. |
| *successThreshold* | Nombre de fois où la sonde doit aboutir après un échec. La valeur par défaut et minimale est 1. La valeur doit être 1 pour les sondes de vivacité. |
| *failureThreshold* | Nombre de fois où Kubernetes tente de redémarrer un pod avant d'abandonner lorsque le pod démarre et que la sonde échoue (voir remarque). La valeur minimale est 1 et la valeur par défaut est 3. |

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
