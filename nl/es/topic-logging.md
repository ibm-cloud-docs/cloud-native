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

# Registro
{: #logging}

Los mensajes de registro son series con información contextual que hace referencia al estado y la actividad de un microservicio cuando se crea la entrada de registro. Los registros son necesarios para diagnosticar cómo y por qué fallan los servicios. Juegan un rol de soporte para las métricas en la supervisión del estado de las aplicaciones.
{:shortdesc}

Asegúrese de escribir las entradas de registro directamente en las secuencias de salida y de error estándares. Esto hace que las entradas de registro se puedan visualizar mediante herramientas de línea de mandatos y permite que los servicios de reenvío de registros configurados a nivel de infraestructura puedan gestionar la recopilación de registros y la gestión de datos.

El manejo de los archivos de registro requiere más reflexión si no se puede configurar una aplicación contenerizada para escribir registros en la corriente de salida estándar o de error estándar.

* Una opción es utilizar un volumen para datos de registro, ya sea un montaje de enlace simple para desarrollo y pruebas o un volumen persistente adecuado como parte de un despliegue de Kubernetes. Un
[sidecar o agente de registro dedicado](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") puede leer de un volumen compartido para reenviar registros a un agregador centralizado. La rotación de registros se debe configurar de forma explícita de modo que controle la cantidad de datos que se almacenan en los volúmenes.
* Otra opción es utilizar bibliotecas de aplicación o agentes para reenviar registros directamente a los agregadores. Esta acción puede acarrear cierta complejidad de configuración en los entornos de despliegue

## Registro JSON
{: #json-logging}

A medida que la aplicación evoluciona a lo largo del tiempo, la naturaleza de lo que puede registrar puede cambiar. Al utilizar un formato de registro JSON, obtendrá las ventajas siguientes:

* Los registros son indexables, lo que hace que la búsqueda de un cuerpo agregado de registros sea mucho más fácil.
* Los registros son resistentes al cambio, ya que el análisis no depende de la posición de los elementos en una serie.

Aunque los registros con formato JSON son más fáciles de analizar para las máquinas, son menos legibles para el usuario. Considere la posibilidad de utilizar variables de entorno para alternar el formato de registro entre texto sin formato para desarrollo y depuración en local, y registros con formato JSON para el almacenamiento a más largo plazo y la agregación.

Los analizadores JSON de línea de mandatos, como la herramienta JSON Query (jq), se pueden utilizar para crear vistas legibles para el usuario de registros con formato JSON. En el ejemplo siguiente, se utiliza un conducto para los registros a través de grep para garantizar que el campo de mensaje esté ahí antes de que jq analice la línea:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## Visualización de registros mediante `kubectl`
{: #view-logs-kube}

Los registros enviados a las corrientes de salida estándar y de error se pueden visualizar en Kubernetes a través de la consola o mediante mandatos `kubectl` que tengan el formato: `kubectl logs <podname>`.

Si utiliza un espacio de nombres personalizado, como stock-trader, inclúyalo en el mandato, `kubectl logs -n stock-trader <podname>`.

Si hay varios contenedores para cada pod, como ocurre cuando se utilizan sidecars de istio, también deberá especificar el contenedor. En el ejemplo siguiente, se utiliza el espacio de nombres stock-trader para ver registros del contenedor `portfolio` en el pod
`portfolio-54b4d579f7-4zvzk`:

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

Para los registros en formato JSON, puede utilizar `jq` para extraer un campo de mensaje, por ejemplo:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Se utiliza un conducto para las entradas de registro mediante `grep` para garantizar que
`jq` analiza las líneas con un campo de mensaje.
{: note}
