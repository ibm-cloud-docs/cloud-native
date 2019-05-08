---

copyright:
  years: 2019
lastupdated: "2019-04-09"

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

Hay un cambio cultural en torno a la supervisión que se produce con un cambio a la nube nativa. Aunque se espera que las aplicaciones tanto locales como nativas de la nube tenga una alta disponibilidad y resistencia a errores, los métodos utilizados para lograr esos objetivos son muy diferentes. Como resultado, la finalidad de la supervisión cambia: en lugar de realizar la supervisión para evitar la anomalía, se realiza para gestionar la anomalía. 
{:shortdesc}

En entornos locales, se suministran la infraestructura y el middleware según la capacidad planificada y los patrones de alta disponibilidad, por ejemplo, activo-activo o activo-pasivo. Las anomalías inesperadas pueden ser complejas en este entorno, lo que requiere un esfuerzo significativo para la determinación y recuperación de problemas. La supervisión externa la realizan los agentes que examinan la utilización de recursos para evitar clases conocidas de anomalías. Como ejemplo, tenga en cuenta el ajuste del tamaño de almacenamiento dinámico, los tiempos de espera y las políticas de recogida de basura para las aplicaciones Java.

Una aplicación nativa de la nube está formada por microservicios independientes y los servicios de respaldo necesarios. Aunque una aplicación nativa en la nube como un todo debe permanecer disponible y seguir funcionando, se iniciarán o detendrán instancias de servicio individuales según sea necesario para ajustarse a los requisitos de capacidad o para recuperarse de una anomalía. 

## Observabilidad
{: #observability}

La supervisión de este sistema fluido requiere que cada participante sea *observable*. Cada entidad debe producir los datos adecuados para dar soporte a la creación de alertas y la detección de problemas automatizados, a la depuración manual cuando sea necesario y al análisis del estado del sistema (tendencias históricas y análisis).

¿Qué tipos de datos debe producir un servicio para que sea observable?

* Las **comprobaciones de estado** (a menudo puntos finales HTTP personalizados) ayudan a los coordinadores, como Kubernetes o Cloud Foundry, a realizar acciones automatizadas para mantener la salud global del sistema.
* Las **métricas** son una representación numérica de los datos recopilados a intervalos dentro de una serie temporal. Los datos de series de tiempo numéricos son fáciles de almacenar y consultar, lo que resulta de ayuda a la hora de buscar tendencias históricas. En un periodo de tiempo más largo, los datos numéricos se pueden comprimir en agregados de menor granularidad, diaria o semanalmente, por ejemplo.
* Las **entradas de registro** representan sucesos discretos que se han producido a lo largo del tiempo. Las entradas de registro son esenciales para la depuración, ya que a menudo incluyen rastreos de pila y otra información contextual que puede ayudar a identificar la causa raíz de las anomalías observadas.
* El **rastreo distribuido, de solicitudes o de extremo a extremo** captura el flujo de extremo a extremo de una solicitud a través del sistema. El rastreo captura esencialmente las relaciones entre los servicios (los servicios a los que ha afectado la solicitud) y la estructura del trabajo que fluye a través del sistema (proceso síncrono o asíncrono, relaciones padre-hijo o de derivación).

## Telemetría
{: #telemetry}

Las aplicaciones nativas de la nube deben basarse en el entorno para la *telemetría*, que es la recopilación y transmisión automática de los datos a ubicaciones centralizadas para su análisis posterior. Esto se pone de manifiesto mediante uno de los doce factores que indica tratar los registros como secuencias de sucesos, y se amplía a todos los datos que produce un microservicio para garantizar que se pueda observar.

Kubernetes tiene algunas funciones de telemetría incorporadas, como Heapster, pero es más probable que la telemetría la proporcionen otros sistemas que se integran con el plano de control de Kubernetes. Como ejemplo, dos de los componentes de Istio, Mixer y Envoy, actúan de manera conjunta para [recopilar la telemetría de las aplicaciones desplegadas](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") de forma transparente.

Las anomalías ya no son ocurrencias disruptivas y raras. El desglose de una aplicación monolítica en microservicios empuja una parte mayor de la vía de acceso principal a la red, aumentando el impacto de la latencia y de otros problemas de red. También llegan solicitudes a procesos que no están preparados para trabajar por diversos motivos. Los servicios se reinician automáticamente si agotan los recursos, y las estrategias de tolerancia a errores permiten que el sistema en su conjunto pueda seguir en funcionamiento. La intervención manual ante errores individuales no resulta especialmente útil ni factible en este tipo de entorno.

## Supervisión
{: #monitoring}

Los conceptos de observabilidad y telemetría ayudan a resaltar algunas diferencias significativas en el modo en que se supervisan las aplicaciones nativas de la nube en sistemas distribuidos a gran escala. Recuerde que los procesos en los entornos nativos en la nube son transitorios. Tres de los doce factores (procesos, simultaneidad y desechabilidad) hacen hincapié en este punto. Los procesos monolíticos preasignados de larga duración se sustituyen, o son rodeados, por muchos procesos de corta duración que se inician y detienen en respuesta a la carga para el escalado horizontal o si no funcionan correctamente. La telemetría es crítica cuando es necesario que los datos se recopilen y que persistan en algún otro lugar para evitar que se pierdan a medida que se crean y se destruyen los procesos (contenedores). Con frecuencia, la telemetría también es necesaria por motivos de conformidad. 

Por todos los motivos descritos anteriormente, la supervisión cambia su enfoque: en lugar de supervisar el comportamiento y el estado de los recursos (procesos individuales o máquinas individuales), se supervisa el estado del sistema como un todo. Cada servicio individual produce datos que alimentan esta vista agregada.

