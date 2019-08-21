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

# Configuración
{: #configuration}

Las aplicaciones nativas de la nube deben ser portátiles. Puede utilizar el mismo artefacto fijo para el despliegue en varios entornos sin cambiar el código ni utilizar vías de acceso de código sin probar.
{:shortdesc}

Tres de los factores de la [metodología de doce factores](https://12factor.net/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") están directamente relacionados con esta práctica:

* El primer factor recomienda una correlación de 1 a 1 entre un servicio en ejecución y un código base con versión. Cree un artefacto de despliegue fijo, como una imagen de Docker, a partir de un código base con versión que se puede desplegar, sin cambios, en varios entornos.
* El tercer factor recomienda una separación entre la configuración específica de la aplicación, que forma parte del artefacto fijo, y la configuración específica del entorno, que se proporciona al servicio en el momento del despliegue.
* El décimo factor recomienda mantener todos los entornos de la forma más parecida posible. Las vías de acceso de código específicas del entorno son difíciles de probar y aumentan el riesgo de que se produzcan errores a medida que se realiza el despliegue en distintos entornos. Esto se aplica también a los servicios de respaldo. Si desarrolla y prueba con una base de datos en memoria, se posible que se produzcan errores no esperados en los entornos de pruebas, transferencia o producción, ya que utilizan una base de datos que tiene un comportamiento distinto.

## Orígenes de configuración
{: #config-inject}

La configuración específica de la aplicación forma parte del artefacto fijo. Por ejemplo, las aplicaciones que se ejecutan en WebSphere Liberty definen una lista de características instaladas que controlan los binarios y los servicios que están activos en el tiempo de ejecución. Esta configuración es específica de la aplicación y se incluye en la imagen de Docker. Las imágenes de Docker definen también el puerto de escucha o expuesto, ya que el entorno de ejecución gestiona la correlación de puertos cuando se inicia el contenedor. 

La configuración específica del entorno, como el host y el puerto utilizados para la comunicación con otros servicios, los usuarios de base de datos o las limitaciones de utilización de recursos, la proporciona el entorno de despliegue al contenedor. La gestión de la configuración y las credenciales del servicio puede variar de forma significativa:

* Kubernetes almacena valores de configuración (atributos planos o JSON en serie) en mapas de configuración (ConfigMaps) o secretos. Estos se pueden pasar a la aplicación contenerizada como variables de entorno o como montajes de sistema de archivos virtual. El mecanismo utilizado por un servicio se especifica en los metadatos de despliegue, ya sea en el YAML de Kubernetes o el diagrama de Helm.
* Los entornos de desarrollo locales suelen ser variantes simplificadas que utilizan variables de entorno clave/valor simples.
* Cloud Foundry almacena atributos de configuración y detalles de enlace de servicio en objetos JSON en serie que se pasan a la aplicación como una variable de entorno, por ejemplo, `VCAP_APPLICATION` y `VCAP_SERVICES`.
* El uso de un servicio de respaldo, como etcd, hashicorp Vault, Netflix Archaius o Spring Cloud config, para almacenar y recuperar atributos de configuración específicos del entorno es también una opción en cualquier entorno.

En la mayoría de los casos, una aplicación procesa una configuración específica del entorno en el inicio. El valor de las variables de entorno, por ejemplo, no se puede modificar una vez que inicia un proceso. No obstante, Kubernetes y los servicios de configuración de respaldo proporcionan mecanismos para que las aplicaciones puedan responder de forma dinámica a actualizaciones en la configuración. Esta es una función opcional. En el caso de procesos transitorios sin estado, reiniciar el servicio suele ser suficiente.

Muchos lenguajes e infraestructuras proporcionan bibliotecas estándar para ayudar a las aplicaciones en las configuraciones específicas de la aplicación y específicas del entorno, de manera que puede centrarse en la lógica principal de la aplicación y abstraerse de estas funciones fundamentales.

### Cómo trabajar con credenciales de servicio
{: #portable-credentials}

La gestión de la configuración y las credenciales del servicio (enlaces de servicio) varía entre las distintas plataformas. Cloud Foundry almacena detalles de enlace de servicio en un objeto JSON en serie que se pasa a la aplicación como una variable de entorno
`VCAP_SERVICES`. Kubernetes almacena enlaces de servicio en un JSON en serie o en atributos sin formato
`ConfigMaps` o `Secrets`, que se pueden pasar a la aplicación contenerizada como variables de entorno o montarse como un volumen temporal. En el caso del desarrollo local, que tiene su propia configuración, las pruebas locales suelen ser una versión simplificada de lo que se ejecuta en la nube. Trabajar con estas variaciones de una forma portátil sin tener vías de acceso de código específicas del entorno puede ser un reto.

En entornos de Cloud Foundry y Kubernetes, puede utilizar [intermediarios de servicio](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para gestionar el enlace con un servicio de respaldo e inyectar las credenciales asociadas en el entorno de la aplicación. Esto puede afectar a la portabilidad de las aplicaciones, ya que es posible que no se proporcionen las credenciales a la aplicación de la misma manera en distintos entornos.

{{site.data.keyword.IBM}} tiene varias bibliotecas de código abierto que trabajan con un archivo
`mappings.json` para correlacionar la clave que utiliza la aplicación para recuperar información de credenciales con una lista ordenada de posibles orígenes. Tiene soporte para tres tipos de patrones de búsqueda:

* `cloudfoundry`: buscar un valor en la variable de entorno de los servicios de Cloud Foundry (VCAP_SERVICES).
* `env`: buscar un valor correlacionado con una variable de entorno.
* `file`: buscar un valor en un archivo JSON.

En el archivo `mappings.json` de ejemplo siguiente, `cloudant-password` es la clave que utiliza el código de la aplicación para buscar la credencial de contraseña. Una biblioteca específica del lenguaje itera a través de la matriz
`searchPatterns` en un orden específico hasta que se encuentra una coincidencia.

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

La biblioteca busca la contraseña de cloudant en los lugares siguientes:

* La vía de acceso JSON `['cloudant'][0].credentials.password` en la variable de entorno
`VCAP_SERVICES` de Cloud Foundry.
* Una variable de entorno que no distingue entre mayúsculas y minúsculas denominada cloudant_password`.
* Un campo JSON **cloudant_password** en un archivo **`localdev-config.json`** que se guarda en una ubicación de recursos específica del lenguaje.

Para obtener más información, consulte:

* [Entorno {{site.data.keyword.Bluemix}} para Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Entorno {{site.data.keyword.Bluemix_notm}} para Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")
* [Enlaces de servicio de {{site.data.keyword.Bluemix_notm}} para Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")

## Valores de configuración en Kubernetes
{: #config-kubernetes}

Kubernetes proporciona varias maneras distintas para definir variables de entorno y asignar sus valores.

### Valores literales
{: #config-literal}

La manera más directa de definir una variable de entorno es incluirla directamente en el archivo
`deployment.yaml` del servicio. En el [ejemplo básico de Kubernetes](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") siguiente, la especificación directa funciona bien cuando el valor es coherente entre entornos:

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

### Variables de Helm
{: #config-helm}

Helm utiliza plantillas para crear diagramas, de manera que los valores se puedan sustituir posteriormente. Puede conseguir el mismo resultado que en el ejemplo anterior, con mayor flexibilidad entre entornos, utilizando el ejemplo siguiente en la plantilla del archivo `mychart/templates/pod.yaml`:

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

Y el ejemplo siguiente en un archivo `mychart/values.yaml`:

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

La salida siguiente se produce cuando Helm representa la plantilla:

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

  Existe una diferencia menor entre estos dos ejemplos. En el primer ejemplo y el archivo `values.yaml` de ejemplo, una persona ha añadido comillas. Las comillas no son necesarias para las series de YAML. Cuando Helm represente la plantilla, dejará fuera las comillas.
  {: note}

### ConfigMap
{: #kubernetes-configmap}

Un ConfigMap es un artefacto de Kubernetes exclusivo que define los datos como un conjunto de pares de clave/valor. Un ConfigMap para las variables de entorno que se muestran en los ejemplos anteriores puede tener un aspecto similar al del ejemplo siguiente:

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

La definición de pod inicial se cambia luego para que se utilicen los valores del ConfigMap tal como se indica a continuación:

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

El ConfigMap es ahora un artefacto independiente del pod. Puede tener un ciclo de vida distinto. Puede actualizar o cambiar los valores del ConfigMap sin tener que volver a desplegar el pod en este caso. También puede actualizar y manipular un ConfigMap directamente desde la línea de mandatos, lo que puede resultar útil en el ciclo de desarrollo/pruebas/depuración.

Cuando se utiliza con Helm, puede utilizar variables en la declaración del ConfigMap. Dichas variables se resuelven de la manera habitual cuando se despliega el diagrama.

Para obtener más información, consulte
[ConfigMaps de Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

### Credenciales y secretos
{: #kubernetes-secrets}

Generalmente, se proporciona una configuración a los contenedores que se ejecutan en Kubernetes mediante variables de entorno o ConfigMaps. En cualquier caso, los valores de configuración se pueden descubrir bastante rápido. Este es el motivo por el que Kubernetes utiliza secretos para almacenar información confidencial.

Los secretos son objetos independientes que contienen valores codificados en base64:

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

Así pues, se pueden utilizar los secretos como archivos en un volumen montado en uno o más contenedores de un pod:

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

Cuando se montan como un volumen de esta manera, cada clave de la correlación de datos del secreto pasa a ser un nombre de archivo bajo la vía de montaje (`mountPath`) especificada: `/etc/foo/username` y `/etc/foo/password` en este caso.

Los secretos se pueden utilizar también para definir variables de entorno:

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

Kubernetes realiza la decodificación de base64 automáticamente. El contenedor que se ejecuta en el pod reconoce el valor decodificado de base64 al recuperar la variable de entorno.

Al igual que los ConfigMaps, se pueden crear y manipular los secretos desde la línea de mandatos, lo que resulta útil cuando se trata con certificados SSL.

Para obtener más información, consulte [Secretos de Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

<!-- SSL EXAMPLE -->
