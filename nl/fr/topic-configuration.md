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

# Configuration
{: #configuration}

Les applications Cloud native doivent être portables. Vous pouvez utiliser le même artefact fixe pour effectuer le déploiement dans plusieurs environnements, sans changer de code ou en utilisant des chemins de code autrement non testés.
{:shortdesc}

Trois facteurs de la [méthodologie à douze facteurs](https://12factor.net/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") sont directement liés à cette pratique :

* Le premier facteur recommande une corrélation individuelle entre un service en cours d'exécution et un codebase versionné. Créez un artefact de déploiement fixe, comme une image Docker, à partir d'un codebase versionné qui peut être déployé, non modifié, dans plusieurs environnements différents.
* Le troisième facteur recommande une séparation entre la configuration spécifique à l'application, qui fait partie de l'artefact fixe, et la configuration spécifique à l'environnement, qui est fournie au service lors du déploiement.
* Le dixième facteur recommande de conserver tous les environnements le plus similaire possible. Il est difficile de tester les chemins de code spécifiques à l'environnement. De plus, ces éléments augmentent le risque de défaillances lorsque vous effectuez le déploiement dans différents environnements. Ces remarques s'appliquent également aux services de sauvegarde. Si vous effectuez le développement et le test avec une base de données en mémoire, des erreurs inattendues peuvent se produire dans les environnements de test, de préproduction ou de production car ces derniers utilisent une base de données ayant un autre comportement.

## Sources de configuration
{: #config-inject}

La configuration spécifique à l'application fait partie de l'artefact fixe. Par exemple, les applications qui s'exécutent sur WebSphere Liberty définissent une liste de fonctions installées qui contrôlent les fichiers binaires et les services actifs dans l'environnement d'exécution. Cette configuration, qui est spécifique à l'application, est incluse dans l'image Docker. Les images Docker définissent également le port d'écoute ou exposé, puisque l'environnement d'exécution gère le mappage de port au démarrage du conteneur.  

La configuration spécifique à l'environnement (hôte et port utilisés pour communiquer avec les autres services, utilisateurs de base de données ou contraintes liées à l'utilisation des ressources, par exemple) est fournie au conteneur par l'environnement de déploiement. La gestion de la configuration des services et des données d'identification peut varier de manière significative :

* Kubernetes stocke les valeurs de configuration (élément JSON sous forme de chaîne ou attributs à plat) dans des éléments ConfigMap ou dans des secrets. Ces valeurs peuvent être transmises à l'application conteneurisée en tant que variables de configuration ou de montages de système de fichiers. Le mécanisme utilisé par un service est spécifié dans les métadonnées de déploiement (dans un élément YAML Kubernetes ou dans la charte Helm).
* Les environnements de développement locaux sont souvent des variantes simplifiées qui utilisent des variables d'environnement clé/valeur simples.
* Cloud Foundry stocke les attributs de configuration et les détails de liaison de service dans des objets JSON sous forme de chaînes qui sont transmis à l'application en tant que variable d'environnement, par exemple `VCAP_APPLICATION` et `VCAP_SERVICES`.
* Dans tout environnement, il est également possible d'utiliser un service de sauvegarde, tel etcd, hashicorp Vault, Netflix Archaius ou Spring Cloud config pour stocker et extraire les attributs de configuration spécifiques à l'environnement.

Dans la plupart des cas, une application traite une configuration spécifique à l'environnement lors du démarrage. La valeur des variables d'environnement, par exemple, ne peut pas être changée après le démarrage d'un processus. Toutefois, Kubernetes et les services de configuration de sauvegarde fournissent des mécanismes permettant aux applications de répondre dynamiquement aux mises à jour de configuration. Il s'agit d'une fonction facultative. Dans le cas de processus transitoires sans état, le redémarrage du service est souvent suffisant.

Un grand nombre de langages et d'infrastructures fournissent des bibliothèques standard assistant les applications pour les configurations spécifiques à l'environnement et à l'application afin que vous puissiez vous concentrer sur la logique centrale de votre application et abstraire ces fonctions fondamentales.

### Utilisation de données d'identification de service
{: #portable-credentials}

La gestion de la configuration de service et des données d'identification (liaisons de service) varie entre les plateformes. Cloud Foundry stocke les détails de liaison de service dans un objet JSON sous forme de chaîne qui est transmis à l'application en tant que variable d'environnement `VCAP_SERVICES`. Kubernetes stocke les liaisons de service en tant qu'élément JSON sous forme de chaîne ou d'attributs `ConfigMaps` ou `Secrets`. Ces éléments peuvent être transmis à l'application conteneurisée en tant que variables d'environnement ou montés en tant que volume temporaire. Dans le cas d'un développement local, qui dispose de sa propre configuration, le test local est souvent une version simplifiée de l'exécution en cours dans le cloud. Prendre en charge ces variations de manière portable sans avoir de chemins d'accès au code spécifiques à l'environnement peut être un véritable défi.

Dans les environnements Cloud Foundry et Kubernetes, vous pouvez utiliser des [courtiers de services](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe") pour gérer la liaison à un service de sauvegarde et l'injection des données d'identification associées dans l'environnement de l'application. Cela peut avoir des conséquences sur la portabilité de l'application car il est possible que les données d'identification ne soient pas fournies à l'application de la même façon dans les différents environnements.

{{site.data.keyword.IBM}} inclut plusieurs bibliothèques open source qui fonctionnent avec un fichier `mappings.json` afin de mapper la clé que l'application utilise pour extraire des informations de données d'identification dans une liste ordonnée de sources possibles. Il prend en charge trois types de modèle de recherche :

* `cloudfoundry` : Recherche d'une valeur dans les variables d'environnement de services de Cloud Foundry (VCAP_SERVICES).
* `env` : Recherche d'une valeur mappée à une variable d'environnement.
* `file` : Recherche d'une valeur dans un fichier JSON.

Dans le fichier `mappings.json` exemple suivant, `cloudant-password` est la clé utilisée par le code d'application pour rechercher le mot de passe. Une bibliothèque spécifique au langage est utilisée plusieurs fois dans la grappe `searchPatterns` dans un ordre spécifique jusqu'à ce qu'une correspondance soit trouvée.

```json
{
   "cloudant_password": {
      "searchPatterns": [
         "cloudfoundry:$['cloudant'][0].credentials.password",
         "env:cloudant_password",
         "file:/server/localdev-config.json:$.cloudant_password"
      ]
   }
}
```
{: codeblock}

La bibliothèque recherche les emplacements suivants pour le mot de passe cloudant :

* Chemin JSON `['cloudant'][0].credentials.password` dans la variable d'environnement `VCAP_SERVICES` Cloud Foundry.
* Variable d'environnement insensible à la casse nommée cloudant_password.
* Zone JSON **cloudant_password** dans un fichier **`localdev-config.json`** conservé à un emplacement de ressource spécifique au langage.

Pour plus d'informations, voir :

* [{{site.data.keyword.Bluemix}} Environment for Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [{{site.data.keyword.Bluemix_notm}} Environment for Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")
* [{{site.data.keyword.Bluemix_notm}} Service Bindings For Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe")

## Valeurs de configuration dans Kubernetes
{: #config-kubernetes}

Kubernetes propose différents méthodes permettant de définir des variables d'environnement et d'affecter leurs valeurs.

### Valeurs littérales
{: #config-literal}

La manière la plus simple de définir une variable d'environnement consiste à l'inclure directement dans le fichier `deployment.yaml` pour le service. Dans l'[exemple Kubernetes de base](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} suivant ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe"), la spécification directe fonctionne correctement lorsque la valeur est cohérente dans les différents environnements :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```
{: codeblock}

### Variables Helm
{: #config-helm}

Helm utilise des modèles pour créer des chartes afin que ces valeurs puissent être remplacées ultérieurement. Vous pouvez obtenir le même résultat que dans l'exemple précédent, avec plus de flexibilité dans les différents environnements, en utilisant l'exemple suivant dans le modèle de fichier `mychart/templates/pod.yaml` :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: {{ .Values.greeting }}
    - name: DEMO_FAREWELL
      value: {{ .Values.farewell }}
```
{: codeblock}

Et l'exemple suivant dans un fichier `mychart/values.yaml` :

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

La sortie suivante est générée lorsque Helm affiche le modèle :

```bash
$ helm template mychart
---
# Source: mychart/templates/pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: Hello from the environment
    - name: DEMO_FAREWELL
      value: Such a sweet sorrow
```
{:screen}

  Il existe une différence mineure entre ces deux exemples. Dans le premier exemple et le fichier `values.yaml` exemple, une personne a ajouté des guillemets. Ces caractères ne sont pas requis pour les chaînes en YAML. Lorsque Helm affiche le modèle, les guillemets n'apparaissent pas.
  {: note}

### ConfigMap
{: #kubernetes-configmap}

Un élément ConfigMap est un artefact Kubernetes unique qui définit les données comme un ensemble de paires clé/valeur. Un élément ConfigMap pour les variables d'environnement qui apparaissent dans les exemples précédents peut être similaire à l'exemple suivant :

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: envar-demo-config
  labels:
    purpose: demonstrate-envars
data:
  DEMO_GREETING: Hello from the environment
  DEMO_FAREWELL: Such a sweet sorrow
```
{: codeblock}

La définition de pod initiale est ensuite changée afin d'utiliser des valeurs de l'élément ConfigMap de la manière suivante :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    envFrom:
    - configMapRef:
        name: envar-demo-config
```
{: codeblock}

ConfigMap est désormais un artefact séparé du pod. Il peut avoir un cycle de vie différent. Vous pouvez mettre à jour ou changer les valeurs dans l'élément ConfigMap sans qu'il soit nécessaire de redéployer le pod. Vous pouvez également mettre à jour et manipuler un élément ConfigMap directement à partir de la ligne de commande, ce qui peut s'avérer utile dans le cycle de développement/test/débogage.

En cas d'utilisation avec Helm, vous pouvez employer des variables dans votre déclaration ConfigMap. Ces variables sont résolues lors du déploiement de la charte.

Pour plus d'informations, voir [Kubernetes ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

### Données d'identification et secrets
{: #kubernetes-secrets}

La configuration est généralement fournie aux conteneurs qui s'exécutent dans Kubernetes via les variables d'environnement ou les éléments ConfigMap. Dans les deux cas, les valeurs de configuration sont détectées assez rapidement. C'est pourquoi, Kubernetes utilise des secrets pour stocker des informations sensibles.

Les secrets sont des objets indépendants qui contiennent des valeurs codées en base64 :

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```
{: codeblock}

Les secrets peuvent ensuite être utilisés en tant que fichiers dans un volume qui est monté sur un ou plusieurs conteneurs d'un pod :

```yaml
containers:
- name: myservice
  image: myimage:latest
  volumeMounts:
  - name: foo
    mountPath: "/etc/foo"
    readOnly: true
volumes:
- name: foo
  secret:
    secretName: mysecret
```
{: codeblock}

En cas de montage en tant que volume, chaque clé de la mappe de données du secret devient un nom de fichier sous l'élément `mountPath` spécifié : `/etc/foo/username` et `/etc/foo/password` dans ce cas.

Les secrets peuvent également être utilisés pour définir des variables d'environnement :

```yaml
containers:
- name: mypod
  image: myimage
  env:
  - name: SECRET_USERNAME
      valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
  - name: SECRET_PASSWORD
    valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```
{: codeblock}

Kubernetes effectue le décodage base64 pour vous. Le conteneur qui s'exécute dans le pod reconnaît la valeur décodée en base64 lors de l'extraction de la variable d'environnement.

Tout comme avec ConfigMaps, les secrets peuvent être créés et manipulés à partir de la ligne de commande, ce qui est fort utile lors du traitement des certificats SSL.

Pour plus d'informations, voir [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![Icône de lien externe](../icons/launch-glyph.svg "Icône de lien externe").

<!-- SSL EXAMPLE -->
