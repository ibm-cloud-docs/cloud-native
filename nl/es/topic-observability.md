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

# Observabilidad, telemetría y supervisión
{: #observability-cn}

Hay un cambio cultural en torno a la supervisión que se produce con un cambio a la nube nativa. Aunque se espera que las aplicaciones tanto locales como nativas de la nube tengan una alta disponibilidad y resistencia a errores, los métodos utilizados para lograr esos objetivos son diferentes. Como resultado, la finalidad de la supervisión cambia: en lugar de realizar la supervisión para evitar la anomalía, se realiza para gestionar la anomalía. 
{:shortdesc}

En entornos locales, se suministran la infraestructura y el middleware según la capacidad planificada y los patrones de alta disponibilidad, por ejemplo, activo-activo o activo-pasivo. Las anomalías inesperadas pueden ser complejas en este entorno, lo que requiere un esfuerzo significativo para la determinación y recuperación de problemas. La supervisión externa la realizan los agentes que examinan la utilización de recursos para evitar clases conocidas de anomalías. Como ejemplo, tenga en cuenta el ajuste del tamaño de almacenamiento dinámico, los tiempos de espera y las políticas de recogida de basura para las aplicaciones Java.

Una aplicación nativa de la nube está formada por microservicios independientes y los servicios de respaldo necesarios. Aunque una aplicación nativa en la nube como un todo debe permanecer disponible y seguir funcionando, se iniciarán o detendrán instancias de servicio individuales para que se ajusten a los requisitos de capacidad o para recuperarse de una anomalía. 

## Observabilidad
{: #observability}

La supervisión de este sistema fluido requiere que cada participante sea *observable*. Cada entidad debe producir los datos adecuados para dar soporte a la creación de alertas y la detección de problemas automatizados, a la depuración manual cuando sea necesario y al análisis del estado del sistema (tendencias históricas y análisis).

¿Qué tipos de datos debe producir un servicio para que sea observable?

* Las **comprobaciones de estado** (a menudo puntos finales HTTP personalizados) ayudan a los coordinadores, como Kubernetes o Cloud Foundry, a realizar acciones automatizadas para mantener la salud global del sistema.
* Las **métricas** son una representación numérica de los datos recopilados a intervalos dentro de una serie temporal. Los datos de series de tiempo numéricos son fáciles de almacenar y consultar, lo que resulta de ayuda a la hora de buscar tendencias históricas. En un periodo más largo, los datos numéricos se pueden comprimir en agregados de menor granularidad, diaria o semanalmente, por ejemplo.
* Las **entradas de registro** representan sucesos individuales. Las entradas de registro son esenciales para la depuración, ya que a menudo incluyen rastreos de pila y otra información contextual que puede ayudar a identificar la causa raíz de las anomalías observadas.
* El **rastreo distribuido, de solicitudes o de extremo a extremo** captura el flujo de extremo a extremo de una solicitud a través del sistema. El rastreo captura esencialmente las relaciones entre los servicios (los servicios a los que ha afectado la solicitud) y la estructura del trabajo que pasa a través del sistema (proceso síncrono o asíncrono, relaciones padre-hijo o de derivación).

## Telemetría
{: #telemetry}

Las aplicaciones nativas de la nube se basan en el entorno para la *telemetría*, que es la recopilación y transmisión automática de los datos a ubicaciones centralizadas para su análisis posterior. Esto se pone de manifiesto mediante uno de los doce factores que indica tratar los registros como secuencias de sucesos, y se amplía a todos los datos que produce un microservicio para garantizar que se pueda observar.

Kubernetes tiene algunas funciones de telemetría incorporadas, como Heapster, pero es más probable que la telemetría la proporcionen otros sistemas que se integran con el plano de control de Kubernetes. Como ejemplo, dos de los componentes de Istio, Mixer y Envoy, actúan de manera conjunta para [recopilar la telemetría de las aplicaciones desplegadas](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") de forma transparente.

Las anomalías ya no son ocurrencias disruptivas y raras. El desglose de una aplicación monolítica en microservicios empuja una parte mayor de la vía de acceso principal a la red, aumentando el impacto de la latencia y de otros problemas de red. También llegan solicitudes a procesos que no están preparados para trabajar por diversos motivos. Los servicios se reinician automáticamente si agotan los recursos, y las estrategias de tolerancia a errores permiten que el sistema en su conjunto pueda seguir en funcionamiento. La intervención manual ante errores individuales no resulta útil ni factible en este tipo de entorno.

## Supervisión
{: #monitoring}

Los conceptos de observabilidad y telemetría ayudan a resaltar algunas diferencias significativas en el modo en que se supervisan las aplicaciones nativas de la nube en sistemas distribuidos a gran escala. Recuerde que los procesos en los entornos nativos en la nube son transitorios. Tres de los doce factores (procesos, simultaneidad y desechabilidad) hacen hincapié en este punto. Los procesos monolíticos preasignados de larga duración se sustituyen, o son rodeados, por muchos procesos de corta duración que se inician y detienen en respuesta a la carga para el escalado horizontal o si no funcionan correctamente. La telemetría es crítica cuando es necesario que los datos se recopilen y que persistan en algún otro lugar para evitar que se pierdan a medida que se crean y se destruyen los procesos (contenedores). Con frecuencia, la telemetría también es necesaria por motivos de conformidad. 

Por todos los motivos descritos anteriormente, la supervisión cambia su enfoque: en lugar de supervisar el comportamiento y el estado de los recursos (procesos individuales o máquinas individuales), se supervisa el estado del sistema como un todo. Cada servicio individual produce datos que alimentan esta vista agregada.

## Comparación entre rastreo, registro y métricas
{: #trace-log-metrics}

:FIXME -- rephrase comparison as topic summary:

El registro se utiliza cuando el desarrollador desea enviar de forma explícita algún mensaje para que alguien lo vea. Se codifica directamente en la clase Java, y se pasan valores de las variables relevantes. Cuando se producen problemas, los registros resultan útiles para realizar una depuración, ya que muestran dónde se ha producido una anomalía, como por ejemplo un rastreo de pila para una excepción que se ha generado. Puede utilizar *Kibana* para obtener una vista federada de dichos registros en los pods y microservicios.

El rastreo se produce automáticamente, por lo que el desarrollador no toma ninguna acción. Por ejemplo, puede configurar Liberty de modo que envíe registros de rastreo a un servidor de rastreo conforme con Open Tracing siempre que se invoca un método anotado JAX-RS. De esta manera consigue un registro de auditoría que indica el contenido de la llamada, cuándo se ha realizado, quién la ha hecho y cuánto ha tardado. También puede aumentar este rastreo, por ejemplo para incluir información sobre los métodos privados que se han invocado en el código, añadiendo anotaciones de Open Tracing a estos métodos que se rastrean. 

Puede utilizar una herramienta como *Zipkin* o *Jaeger* para obtener una vista federada de los rastreos en los pods y microservicios. Una *red de servicios* también puede proporcionar rastreo automático de las llamadas que se pasan a través de un complemento del contenedor.  

Las métricas se utilizan para realizar un seguimiento de los valores agregados. En lugar de utilizar un rastreo o un registro para ver la frecuencia con que alguien crea portafolios, puede utilizar una métrica de recuento personalizada. Puede etiquetar el despliegue de forma que *Prometheus* lo recopile, como por ejemplo el acceso periódico al URI /metrics en los pods. Luego puede utilizar una herramienta como *Grafana* para obtener una vista federada de las métricas de los pods y microservicios.

Puede examinar el rastreo para averiguar que "se ha llamado a este método". Pero consultaría el registro para saber "qué ha sucedido en este método cuando se ha llamado". Y examinaría las métricas para ver "cuántas veces que se ha llamado". Por lo general, el rastreo es implícito, es decir, se produce automáticamente sin esfuerzo por parte del desarrollador. Sin embargo el registro es explícito: requiere que el desarrollador codifique el envío de la información que podría ser relevante para el análisis posterior de un problema. Las métricas también son explícitas, ya que tiene que añadir la anotación al método adecuado en el código (aunque generalmente hay métricas predeterminadas disponibles de bajo nivel que no requieren esfuerzo por parte del programador, como uso de memoria, uso de CPU y recuento de hebras).

Por lo general, las métricas resultan útiles para la analítica, mientras que el registro es más útil para la determinación de problemas y el rastreo es más útil para comprender el flujo de control entre microservicio y microservicio.
