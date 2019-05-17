---

copyright:
  years: 2019
lastupdated: "2019-04-30"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Métricas
{: #metrics}

Las métricas son mediciones numéricas simples que se capturan como pares de clave/valor. Algunas métricas son contadores que se incrementan; otras realizan agregaciones, como la suma de todos los valores recopilados en el último minuto o el tiempo promedio transcurrido en el último minuto. Algunas métricas son solo indicadores simples que devuelven el último valor que se haya observado. La captura y procesamiento de las métricas puede ayudarle a identificar y responder ante problemas potenciales antes de que escalen y provoquen problemas más serios.
{:shortdesc}

Existen tres factores generales cuando se habla de métricas en un sistema distribuido: productores, agregadores y procesadores. Existen algunas combinaciones comunes de estos factores, como el uso de Prometheus como agregador con Grafana como procesador de las métricas recopiladas para mostrarlas en paneles de control gráficos, o el uso de StatsD con graphite.

![Los tres factores en las métricas de sistemas distribuidos](images/metrics-systems.png "Los tres factores en las métricas de sistemas distribuidos")
{: caption="Figura 1. Los tres factores en las métricas de sistemas distribuidos" caption-side="bottom"}

El productor es, por supuesto, la propia aplicación. En algunos casos, la aplicación está implicada directamente en producir métricas. En otros casos, los agentes u otra infraestructura observan de manera pasiva o equipan de forma activa la aplicación para producir métricas en su nombre. Lo que ocurre después dependerá del agregador.

Las métricas se transfieren del productor al agregador mediante mecanismos "push" (envío) o "pull" (extracción). Algunos agregadores, como StatsD, esperan que la aplicación (o un agente en nombre de la aplicación) se conecte al agregador para transmitir datos. Esto requiere que la información de conexión para el agregador se distribuya a todos los procesos de la aplicación que se deben medir. Otros agregadores, como Prometheus, se conectan periódicamente a un punto final conocido para recopilar (o reunir) datos de métricas. Esto requiere que el productor defina y proporcione un punto final que se pueda recopilar, y que se indique al agregador dónde se encuentran los puntos finales. Cuando se utiliza con Kubernetes, Prometheus puede descubrir puntos finales basándose en anotaciones de servicio.

Por último, el procesador es algo que hace uso de todos los datos agregados. Como se ha mencionado anteriormente, los servicios como Grafana agregaban métricas para su visualización utilizando paneles de control. Grafana admite también alertas mediante reglas que se almacenan y evalúan independientemente del panel de control.

## Descubrimiento automático de puntos finales de Prometheus en Kubernetes
{: #prometheus-kubernetes}

El modelo basado en la extracción ha creado su propio ecosistema. Otros agregadores, como Sysdig, también pueden recopilar datos de métricas de puntos finales de Prometheus. Esto puede implicar que algunos sistemas utilizan métricas de Prometheus sin utilizar el servidor de Prometheus.

En entornos de Kubernetes, se utilizan anotaciones para el descubrimiento de puntos finales de Prometheus. Por ejemplo, un servicio que proporciona un punto final `/metrics` a través de HTTP en el puerto 8080 añade las anotaciones siguientes en la definición del servicio:

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

Cualquier agregador compatible con Prometheus puede entonces descubrir estos puntos finales utilizando la API de Kubernetes mediante el filtrado de la anotación.

## Métricas de aplicación frente a métricas de plataforma
{: #app-platform-metrics}

La recopilación de métricas requiere estudio. ¿Qué puntos de datos se deben recopilar? Al tener en cuenta una aplicación nativa en la nube, puede dividir en términos generales las métricas en dos categorías, aplicación y plataforma:

* Las métricas de aplicación se centran en las entidades del dominio de la aplicación. Por ejemplo, ¿cuántos inicios de sesión se han completado en los últimos cinco minutos? ¿Cuántos usuarios están conectados actualmente? ¿Cuántas transacciones se realizaron en el último segundo? La recopilación de métricas específicas de la aplicación requiere código personalizado para recopilar y publicar la información. La mayoría de los lenguajes tienen bibliotecas de métricas para simplificar la adición de métricas personalizadas.
* Las métricas de plataforma, por el contrario, son entidades del dominio del alojamiento. Por ejemplo, ¿cuánto se tarda en invocar un servicio? ¿Cuánto tarda esta consulta de base de datos? Están alineadas con el modo en que el tráfico y el trabajo fluye a través del sistema y con frecuencia se pueden medir sin realizar cambios en la aplicación. Algunos marcos de aplicaciones proporcionan también soporte incorporado para medir estos aspectos, como tiempos de ejecución de consulta.

Estas categorías no son suficientes por sí solas para decidir realmente qué se necesita medir. Recuerde que, en un sistema distribuido, hay muchas cosas que producen métricas. Existen varios métodos bien conocidos que intentan filtrar el amplio abanico de métricas para reducirlo a unas pocas esenciales que se deben observar:

* Las [cuatro señales doradas](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") son las métricas esenciales identificadas por el equipo de Google Site Reliability Engineering (SRE) para la observabilidad a nivel de servicio al supervisar un sistema distribuido. En sus palabras, "Si solo puede medir cuatro métricas de su sistema de cara al usuario, céntrese en esas cuatro". Las cuatro métricas son latencia, tráfico, errores y saturación.
* El [método de Saturación de utilización y errores (USE)](http://www.brendangregg.com/usemethod.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") está diseñado como una lista de comprobación de emergencia para analizar el rendimiento de un sistema. El método USE se puede resumir en una frase, "Para cada recurso, compruebe la utilización, saturación y errores". Un recurso, en este caso, son los recursos físicos o lógicos con límites estrictos, como CPU o discos. Para cada recurso finito, mida la utilización, la saturación y los errores. 
* El [método RED](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") es una regla nemotécnica derivada de las cuatro señales doradas que define tres métricas clave que debe medir para cada microservicio de su arquitectura. Este método está muy centrado en las solicitudes, especialmente cuando se compara con el método USE, que se ajusta bien al diseño y la arquitectura de muchas aplicaciones nativas de la nube. Las métricas son velocidad, errores y duración. 

Hay algunos elementos comunes entre estos métodos, por ejemplo, que todos ellos realizan el seguimiento de la tasa de errores. Sin embargo, también existen diferencias notables. El método USE se centra en métricas de infraestructura (utilización/saturación). Los entornos nativos de la nube están diseñados para hacer un mejor uso del hardware físico (o virtual). Estas medidas de infraestructura miran si el sistema gestiona la carga adecuadamente o no. El método RED se centra enteramente en métricas de solicitud (latencia/duración, tráfico/velocidad), que pueden indicar problemas con la infraestructura (problemas de red) o con las aplicaciones (configuraciones erróneas, puntos muertos, errores de servicio repetidos, etc.), especialmente cuando se combinan con tasas de error observadas. Las señales doradas abarcan las dos, combinando las métricas tanto de infraestructura como de solicitud en una vista holística.

## Definición de métricas
{: #defining-metrics}

La supervisión de las métricas recopiladas de un servicio individual puede ofrecerle una idea de la utilización de recursos, pero si hay varias instancias del servicio (debido a despliegue de escalado horizontal o de varias regiones), necesita distinguir entre ellas para aislar problemas dentro la masa de datos similares que entran. En última instancia, esto se reduce a la denominación. En algunos casos, el sistema de métricas que utiliza puede imponer una estructura en sus métricas. Prometheus, por ejemplo, recomienda algo como `namespace_subsystem_name`, mientras que otros recomiendan `namespace.subsystem.targetNoun.actioned`.

Por ejemplo, si desea realizar el seguimiento del número de "transacciones" que ha realizado una aplicación de comercio de acciones, puede capturarla en una propiedad denominada `stock.trades`. Para distinguir entre instancias, puede poner como prefijo el ID de instancia en la propiedad: `<instanceid>.stock.trades`. Esto da soporte a la recopilación de valores de instancia individuales y también de datos agregados utilizando `*.stock.trades`. Pero, ¿qué ocurre cuando realiza el despliegue en varios centros de datos y desea analizar las métricas de esa manera? Puede actualizar el nombre a `<datacenter>.<instanceid>.stock.trades`, pero eso podría romper cualquier informe que utilice el comodín anterior de `*.stock.trades`. Necesitaría utilizar `*.*.stock.trades` en su lugar. 

Si no se tiene cuidado, el uso de propiedades con nombres jerárquicos únicamente puede resultar en patrones de comodines frágiles vinculados a la estructura arbitraria de su estrategia de denominación, lo que no le ayuda a observar la información que necesita para asegurarse de que su aplicación está funcionando bien.

Los sistemas de métricas que dan soporte a datos dimensionales le permiten asociar etiquetas de identificación adicionales, o códigos, a los datos de métricas. A continuación, puede filtrar, agrupar o analizar las métricas recopiladas utilizando estas dimensiones adicionales sin depender de comodines ni convenios de denominación. Las etiquetas típicas incluyen el nombre de punto final o de servicio, centro de datos, código de respuesta, entorno de alojamiento (prod/staging/dev/test), o identificadores de tiempo de ejecución (versión de Java, información del servidor de apps).

Utilizando el mismo ejemplo de comercio de acciones, la métrica `stock.trades` está asociada con varias etiquetas: `{ instanceid=..., datacenter=... }`, lo que permite que se filtre o agrupe el valor agregado por `instanceid` o `datacenter` sin depender de comodines. Hay un equilibrio entre la métrica denominada (`stock.trades`) y las etiquetas asociadas (compare esto con el ejemplo jerárquico de `<datacenter>.<instanceid>.stock.trades`): cada métrica debe capturar datos significativos, mientras que las etiquetas eliminan ambigüedades donde sea necesario.

Además, tenga cuidado al definir etiquetas. En Prometheus, por ejemplo, cada combinación exclusiva de pares de etiquetas clave/valor se trata como una serie temporal independiente. Un método recomendado para garantizar un buen comportamiento de consulta y la recopilación de datos limitada es utilizar etiquetas con un número finito de valores permitidos. Por ejemplo, si utiliza una métrica que cuenta el número de errores, puede utilizar el código de retorno como etiqueta, donde los valores están dentro de un conjunto razonable (401, 404, 409, 500... ), pero es posible que no quiera una etiqueta para el URL con errores, ya que este es un conjunto ilimitado (cualquier URL de solicitud que haya fallado por cualquier motivo, incluyendo que no sea válido).

Para obtener más información sobre los métodos recomendados para la denominación de métricas y etiquetas (etiquetas), consulte [Denominación de métricas y etiquetas](https://prometheus.io/docs/practices/naming/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Consideraciones adicionales
{: #metrics-considerations}

Al recopilar las métricas, recuerde que una vía de error es con frecuencia sumamente distinta a una vía de éxito. Por ejemplo, una respuesta de error en un recurso HTTP puede tardar mucho más tiempo que una respuesta correcta si el error implicaba la recopilación del rastreo de pila y tiempos de espera. Haga el recuento y trate las vías de error de forma independiente a las solicitudes correctas.

Un sistema distribuido tiene variaciones naturales en determinadas medidas. Los errores ocasionales son normales, ya que las solicitudes pueden estar dirigidas a procesos en medio de su inicio o de su conclusión. Filtre los datos en bruto para capturar cuándo esta variación natural empieza a sobrepasar un rango válido. Por ejemplo, divida las métricas en grupos. Categorice la duración de la solicitud en categorías como 'más pequeña/más rápida', 'media/normal' y 'más larga/más extensa', tal como se observa en una ventana de tiempo deslizante. Si las duraciones de solicitud caen de forma constante en el grupo "más larga/más extensa", puede identificar un problema. El histograma o las métricas de resumen se utilizan habitualmente para este tipo de datos. Para obtener más información, consulte [Histogramas y resúmenes](https://prometheus.io/docs/practices/histograms/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

Como desarrollador de aplicaciones, asegúrese de que sus aplicaciones o servicios emiten métricas con nombres y etiquetas que siguen los convenios generales de la organización para dar soporte a los esfuerzos de supervisión que se centran en las vías de extremo a extremo más importantes para su negocio. Para obtener más información, consulte [Supervisión de sistemas distribuidos](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").
