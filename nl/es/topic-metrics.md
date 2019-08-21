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

# Métricas
{: #metrics}

Las métricas son mediciones numéricas simples que se capturan como pares de clave/valor. Algunas métricas consisten en recuentos que se incrementan y otras realizan agregaciones. Por ejemplo, la suma de todos los valores recopilados en el último minuto o el tiempo promedio transcurrido en el último minuto. Algunas métricas son medidores que devuelven el último valor observado. La captura y el proceso de métricas puede ayudarle a identificar y responder ante problemas potenciales antes de que escalen y provoquen problemas más serios.
{:shortdesc}

Existen tres factores generales cuando se habla de métricas en un sistema distribuido: productores, agregadores y procesadores. Existen algunas combinaciones comunes de estos factores, como el uso de Prometheus como agregador con Grafana como procesador de las métricas recopiladas para mostrarlas en paneles de control gráficos. Otra combinación es StatsD con Graphite.

![Los tres factores en las métricas de sistemas distribuidos](images/metrics-systems.png "Los tres factores en las métricas de sistemas distribuidos")

El productor es la aplicación. En algunos casos, la aplicación está implicada directamente en producir métricas. En otros casos, los agentes u otra infraestructura observan de manera pasiva o equipan de forma activa la aplicación para producir métricas en su nombre. Lo que ocurre después dependerá del agregador.

Las métricas se transfieren del productor al agregador mediante mecanismos "push" (envío) o "pull" (extracción). Algunos agregadores, como StatsD, esperan que la aplicación se conecte al agregador para transmitir datos. La información de conexión para el agregador se debe distribuir a todos los procesos de la aplicación que se van a medir. Otros agregadores, como Prometheus, se conectan periódicamente a un punto final conocido para recopilar datos de métricas. Esto requiere que el productor defina y proporcione un punto final que se pueda recopilar, y que se indique al agregador dónde se encuentran los puntos finales. Cuando se utiliza con Kubernetes, Prometheus puede descubrir puntos finales basándose en anotaciones de servicio.

Por último, el procesador utiliza todos los datos agregados. Los servicios como Grafana procesan las métricas agregadas para mostrarlas mediante paneles de control. Grafana admite también alertas mediante reglas que se almacenan y evalúan independientemente del panel de control.

## Descubrimiento automático de puntos finales de Prometheus en Kubernetes
{: #prometheus-kubernetes}

El modelo basado en la extracción crea su propio ecosistema. Otros agregadores, como Sysdig, también pueden recopilar datos de métricas de puntos finales de Prometheus. Esto puede implicar que algunos sistemas utilizan métricas de Prometheus sin utilizar el servidor de Prometheus.

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

¿Qué puntos de datos se deben recopilar? Con una aplicación nativa en la nube, puede dividir en términos generales las métricas en dos categorías, aplicación y plataforma:

* Las métricas de aplicación se centran en las entidades del dominio de la aplicación. Por ejemplo, ¿cuántos inicios de sesión se han completado en los últimos cinco minutos? ¿Cuántos usuarios están conectados actualmente? ¿Cuántas transacciones se realizaron en el último segundo? La recopilación de métricas específicas de la aplicación requiere código personalizado para recopilar y publicar la información. La mayoría de los lenguajes tienen bibliotecas de métricas para simplificar la adición de métricas personalizadas.
* Las métricas de plataforma, por el contrario, son entidades del dominio del alojamiento. Por ejemplo, ¿cuánto se tarda en invocar un servicio? ¿Cuánto tarda esta consulta de base de datos? Están alineadas con el modo en que el tráfico y el trabajo fluyen a través del sistema y con frecuencia se pueden medir sin realizar cambios en la aplicación. Algunos marcos de aplicaciones proporcionan también soporte incorporado para medir estos aspectos, como tiempos de ejecución de consulta.

Estas categorías no son suficientes por sí solas para decidir realmente qué se necesita medir. En un sistema distribuido, hay muchas cosas que producen métricas. Existen varios métodos conocidos que intentan filtrar el amplio abanico de métricas para reducirlo a unas pocas esenciales que se deben observar:

* Las [cuatro señales doradas](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") son las métricas esenciales para supervisar un sistema. Han sido identificadas por el equipo de Google Site Reliability Engineering (SRE) para la observabilidad a nivel de servicio cuando se realiza la supervisión. En sus palabras, "Si solo puede medir cuatro métricas de su sistema de cara al usuario, céntrese en esas cuatro". Las cuatro métricas son latencia, tráfico, errores y saturación.
* El [método de Saturación de utilización y errores (USE)](http://www.brendangregg.com/usemethod.html){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") está diseñado como una lista de comprobación de emergencia para analizar el rendimiento de un sistema. El método USE se puede resumir en una frase, "Para cada recurso, compruebe la utilización, saturación y errores". Un recurso, en este caso, son los recursos físicos o lógicos con límites estrictos, como CPU o discos. Para cada recurso finito, mida la utilización, la saturación y los errores. 
* El [método RED](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") es una regla nemotécnica derivada de las cuatro señales doradas que define tres métricas clave que se van a medir para cada microservicio de su arquitectura. Este método está centrado en las solicitudes, especialmente cuando se compara con el método USE, que se ajusta bien al diseño y la arquitectura de muchas aplicaciones nativas de la nube. Las métricas son velocidad, errores y duración. 

El método USE se centra en métricas de infraestructura. Los entornos nativos de la nube están diseñados para hacer un mejor uso del hardware físico o virtual. Estas medidas de infraestructura miran si el sistema gestiona la carga adecuadamente. El método RED se centra por completo en las métricas de solicitud que indican problemas con la infraestructura y las aplicaciones. Las señales doradas abarcan las dos, combinando las métricas tanto de infraestructura como de solicitud en una vista holística.

## Definición de métricas
{: #defining-metrics}

La supervisión de métricas desde un solo servicio le puede dar una idea de la utilización de recursos. Pero, si hay varias instancias del servicio, es necesario distinguirlas para aislar problemas con datos de entrada similares. En última instancia, esto se reduce a la denominación. En algunos casos, el sistema de métricas que utiliza puede imponer una estructura en sus métricas. Prometheus recomienda algo como `namespace_subsystem_name`, mientras que otros recomiendan `namespace.subsystem.targetNoun.actioned`.

Por ejemplo, si desea realizar el seguimiento del número de "transacciones" que ha realizado una aplicación de comercio de acciones, puede capturarla en una propiedad denominada `stock.trades`. Para distinguir entre instancias, puede poner como prefijo el ID de instancia en la propiedad: `<instanceid>.stock.trades`. Esto da soporte a la recopilación de valores de instancia individuales y agrega datos mediante `*.stock.trades`. Pero, ¿qué ocurre cuando realiza el despliegue en varios centros de datos y desea analizar las métricas de esa manera? Puede actualizar el nombre a `<datacenter>.<instanceid>.stock.trades`, pero eso podría romper cualquier informe que utilice el comodín anterior de `*.stock.trades`. Necesitaría utilizar `*.*.stock.trades` en su lugar. 

Si no se tiene cuidado, el uso de propiedades con nombres jerárquicos únicamente puede dar lugar a patrones de comodines frágiles vinculados a la estructura arbitraria de su estrategia de denominación. Los patrones de comodines frágiles no le ayudan a observar la información que necesita para asegurarse de que la aplicación está funcionando correctamente.

Con sistemas de métricas que den soporte a datos dimensionales puede asociar etiquetas de identificación. Las etiquetas típicas incluyen el nombre de punto final o de servicio, centro de datos, código de respuesta, entorno de alojamiento (prod/staging/dev/test), o identificadores de tiempo de ejecución (versión de Java, información del servidor de apps).

Utilizando el mismo ejemplo de comercio de acciones, la métrica `stock.trades` está asociada con varias etiquetas, como `{ instanceid=..., datacenter=... }`. Esto permite que se filtre o agrupe el valor agregado por `instanceid` o `datacenter` sin depender de comodines. Hay un equilibrio entre la métrica mencionada, `stock.trades`, y las etiquetas asociadas. Cada métrica captura datos significativos, con etiquetas para la desambiguación.

Defina las etiquetas con precaución. En Prometheus, cada combinación exclusiva de pares de clave y valor se trata como una serie temporal independiente. Una práctica recomendada para garantizar un buen comportamiento de consulta y la recopilación de datos limitada consiste en utilizar etiquetas con un número finito de valores permitidos. 

Si utiliza una métrica que cuenta el número de errores, puede utilizar el código de retorno como etiqueta, donde los valores están dentro de un conjunto razonable. No etiquete el URL que ha fallado. Se trata de un conjunto no enlazado.

Para obtener más información sobre los métodos recomendados para la denominación de métricas y etiquetas, consulte [Denominación de métricas y etiquetas](https://prometheus.io/docs/practices/naming/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Más consideraciones
{: #metrics-considerations}

Una vía de error es con frecuencia sumamente distinta a una vía de éxito. Por ejemplo, una respuesta de error en un recurso HTTP puede tardar mucho más tiempo que una respuesta correcta si el error implicaba la recopilación del rastreo de pila y tiempos de espera. Haga el recuento y trate las vías de error de forma independiente a las solicitudes correctas.

Un sistema distribuido tiene variaciones naturales en determinadas medidas. Los errores ocasionales son normales, ya que las solicitudes pueden estar dirigidas a procesos en medio de su inicio o de su conclusión. Filtre los datos en bruto para capturar cuándo esta variación natural supera un rango válido. Por ejemplo, divida las métricas en grupos. Categorice la duración de la solicitud en categorías como 'más pequeña/más rápida', 'media/normal' y 'más larga/más extensa', tal como se observa en una ventana de tiempo deslizante. Si las duraciones de solicitud caen de forma constante en el grupo "más larga/más extensa", puede identificar un problema. El histograma o las métricas de resumen se utilizan habitualmente para este tipo de datos. Para obtener más información, consulte [Histogramas y resúmenes](https://prometheus.io/docs/practices/histograms/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

Asegúrese de que sus aplicaciones o servicios emiten métricas con nombres y etiquetas que siguen los convenios generales de la organización para dar soporte a los esfuerzos de supervisión de la empresa. Para obtener más información, consulte [Supervisión de sistemas distribuidos](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").
