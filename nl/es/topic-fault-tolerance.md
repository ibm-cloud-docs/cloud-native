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

# Tolerancia a errores
{: #fault-tolerance}

La tolerancia a errores es la capacidad que tiene un sistema para continuar funcionando durante una anomalía parcial. La creación de un sistema resistente impone requisitos en todos los servicios que contiene. La naturaleza dinámica de los entornos de nube exige que los servicios se escriben de modo que esperen y respondan sin problemas ante lo inesperado. 

Las soluciones de tolerancia a errores se centran habitualmente en tiempos de espera, retrocesos, barreras de aislamiento e interruptores.

En algunos entornos, los mecanismos de tolerancia a errores los pueden proporcionar los componentes de la infraestructura, como es el caso de Istio. Independientemente de si la infraestructura ayuda o no, un servicio debe suponer que la llamada remota puede fallar y debe estar preparado con acciones de reserva adecuadas.

## Tiempos de espera

La primera línea de defensa frente al fallo parcial es el uso de tiempos de espera. Los tiempos de espera garantizan que la aplicación reciba un error cuando un servicio de respaldo no responda, permitiendo que pueda manejar la condición con un comportamiento de reserva adecuado. Esto no implica que la operación solicitada haya fallado. Los tiempos de espera cambian el tiempo que un cliente que realiza una solicitud espera una respuesta. No afectan al comportamiento de proceso del servicio de destino.

Muchas bibliotecas de lenguaje utilizan un tiempo de espera predeterminado para las solicitudes, e Istio también lo hace. De forma predeterminada, la solicitud del proxy de sidecar falla si no recibe una respuesta en 15 segundos. Este valor se puede cambiar estableciendo una política de tiempo de espera para la ruta en la definición de VirtualService, que tendrá un aspecto similar al siguiente, utilizando un servicio que devuelve cotizaciones bursátiles como ejemplo:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    timeout: 30s
```
{: codeblock}

Las solicitudes realizadas al servicio de cotizaciones bursátiles a través del proxy complementario o la pasarela Ingress esperan durante 30 segundos antes de que falle la solicitud en lugar de esperar 15 segundos, que es el valor predeterminado.

Si se ajustan los tiempos de espera de este modo, todas las solicitudes que utilizan la ruta tienen el tiempo de espera excedido aplicado. Esta es una capa base de seguridad que proporciona Istio. Aunque una biblioteca o una infraestructura de aplicaciones no imponga un tiempo de espera, no esperará de forma ilimitada ya que Istio sí lo impone. Sin embargo, se siguen aplicando los tiempos de espera a nivel de aplicación. El tiempo de espera a nivel de infraestructura para el servicio de cotizaciones bursátiles se ha ampliado a 30 segundos. Si la biblioteca de aplicaciones establece un tiempo de espera durante 5 segundos, la solicitud de la aplicación sigue fallando con el agotamiento del tiempo de espera.

## Retrocesos

Una aplicación define lo que ocurre cuando falla una solicitud a un servicio de respaldo. Existen varias opciones, pero el objetivo es la degradación de forma ordenada cuando estos servicios no respondan de una manera precisa. Cuando falla un servicio remoto, puede volver a intentar la solicitud, intentar una solicitud distinta o devolver datos almacenados en memoria caché en su lugar.

El reintento de una solicitud es el mecanismo de retroceso más sencilla a primera vista. Lo que no resulta tan aparente es que el reintento de solicitudes puede contribuir a encadenar errores del sistema ("tormentas de reintentos", una variación del
[problema de la manada atronadora](https://en.wikipedia.org/wiki/Thundering_herd_problem)). El código a nivel de aplicación no tiene conocimiento suficiente del estado de salud del sistema o de la red, y es difícil llevar a cabo correctamente un algoritmo de retroceso exponencial.

Istio puede realizar reintentos de manera mucho más efectiva. Ya está directamente involucrado en el direccionamiento de solicitudes y proporciona una implementación coherente e independiente del lenguaje para las políticas de reintento. Por ejemplo, puede definir una política como la siguiente para el servicio de cotización en bolsa:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    retries:
      attempts: 3
      perTryTimeout: 5s
```
{: codeblock}

Con esta sencilla configuración, las solicitudes realizadas al servicio de cotizaciones bursátiles a través de un proxy complementario de Istio o de una pasarela Ingress se reintentan hasta tres veces, con un tiempo de espera de cinco segundos para cada intento. Esta política de reintentos se puede restringir aún más con [reglas de coincidencia de rutas adicionales](https://istio.io/docs/reference/config/networking/#HTTPMatchRequest), por ejemplo a las solicitudes `GET`.

Existe un matiz aquí que es fácil pasar por alto: no está especificando ningún intervalo de reintento. El sidecar determina el intervalo entre reintentos e introduce "jitter" deliberadamente entre intentos para evitar el bombardeo de servicios sobrecargados.

<!-- Notes about other approaches here: -->

## Barreras
{: #bulkheads-fault-tolerance}

En navegación, una barrera aislante es una partición que evita que una fuga en un compartimento haga que se hunda todo el barco. El patrón se aplica en entornos de nube para lograr un fin similar, y se lleva a cabo de distintas maneras.

En lenguajes multihebra como Java, se pueden utilizar internamente barreras internas para limitar o controlar la manera en que se utilizan los recursos para la comunicación con recursos remotos, mediante un mecanismo de cola o de semáforo:

- Para utilizar una cola, el servicio asocia un número específico de hebras a una cola concreta. Cualquier solicitud realizada cuando la cola esté completa obtendrá un error rápidamente. En Java, esto podría ser un `ThreadPoolExecutor` asociado a una cola
`BlockingQueue`, por ejemplo.
- El mecanismo de semáforo funciona teniendo un número definido de permisos. Una solicitud de salida requiere un permiso. Una vez que se haya completado correctamente una solicitud, se libera el permiso para que lo pueda utilizar otra solicitud.

También puede definir barreras entre servicios mediante una regla de tipo DestinationRule de Istio para limitar la agrupación de conexiones para un servicio en sentido ascendente, con algo similar a lo del ejemplo siguiente:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
        connectTimeout: 30s
```
{: codeblock}

Esta configuración limita el número máximo de conexiones simultáneas que se realizan en cada instancia del servicio de cotización bursátil a 10. Los servicios que no puedan establecer una conexión en menos de 30 segundos obtienen la respuesta `503 -- Servicio no disponible`. Este tipo de barrera puede utilizarse para evitar que un servicio de cálculo pesado pueda recibir más solicitudes de las que puede gestionar, por ejemplo.

## Interruptores

Los interruptores se utilizan para optimizar el comportamiento de las solicitudes de salida cuando se producen errores. En lugar de realizar solicitudes de forma repetida a un servicio que no responde, un interruptor observa el número de errores que producen durante un periodo de tiempo determinado. Si la tasa de errores supera un umbral, el interruptor abre el circuito y la solicitud falla, como también fallarán todas las solicitudes posteriores hasta que el circuito se cierre de nuevo.

Los interruptores también se definen utilizando una regla de tipo DestinationRule de Istio:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
      interval: 5s
      baseEjectionTime: 5m
      maxEjectionPercent: 100
```
{: codeblock}

Esta configuración impone limitaciones en las solicitudes que realizan otros servicios al servicio de cotización en bolsa. La política de tráfico `outlierDetection` especificada se aplica a cada instancia individual. Resumiendo en una frase la configuración anterior, "Expulsar cualquier instancia del servicio de cotización en bolsa que falle tres veces en cinco segundos durante al menos 5 minutos. Se pueden expulsar todas las instancias". El valor `maxEjectionPercent` del final está relacionado con el equilibrio de carga. Istio mantiene una agrupación de equilibrio de carga y expulsa las instancias que fallen de dicha agrupación. De forma predeterminada, expulsa un máximo del 10 % de las instancias disponibles de la agrupación de equilibrio de carga.

Para aquellos que estén familiarizados con otros mecanismos de interrupción, Istio no tiene un estado medio abierto. Aplica algunas reglas matemáticas sencillas. Una instancia permanece expulsada de la agrupación durante `baseInjectionTime * <number of times it has been ejected>`. Esto permite que las instancias se puedan recuperar de errores transitorios y se mantienen las instancias que fallan fuera de la agrupación.

Con esta configuración, el número máximo de llamadas simultáneas al servicio de cotización bursátil se limita a 10. Las que exceden este límite reciben la respuesta `503 -- Servicio no disponible`.

### Istio -- Límites de velocidad
{: #istio-rate-limits}

También puede definir **Límites de velocidad** con Istio. Se parecen a las barreras, pero se definen dentro de una ventana temporal en lugar de limitar solo las llamadas simultáneas que se procesan en un instante.

Por ejemplo, supongamos que no desea permitir más de un tweet por segundo. Este es un ejemplo del mundo real. Twitter bloqueó automáticamente la cuenta `@IBMStockTrader` hace tiempo cuando se ejecutaron pruebas de estrés sobre Stock Trader.

<!-->
*<Need example. Note that rate limits require us to introduce the concept of Mixer Adapters, like memquota or redisquota; there's more to them than the earlier policies.>*


## Prueba: Inyección de errores con Istio

Puede usar Istio para generar errores deliberadamente para ver cómo respondería su código...
-->
