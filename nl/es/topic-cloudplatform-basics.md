---

copyright:
  years: 2019
lastupdated: "2019-02-18"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Conceptos de la plataforma de nube
{: #platform}

En esta sección se proporciona una breve visión general de las tecnologías y conceptos principales con los que interactúan los desarrolladores al crear aplicaciones nativas de la nube, empezando por los contenedores, Kubernetes, Helm e Istio.
{:shortdesc}

## Contenedores
{: #containers}

Los contenedores son un mecanismo estándar para empaquetar una aplicación y todas sus dependencias en una única unidad autocontenida. Los contenedores resuelven el problema de la portabilidad: el artefacto de contenedor (imagen) garantiza que todo lo que necesita ejecutar una aplicación esté en el lugar adecuado; así, los motores de contenedor pueden centrarse en la ejecución de contenedores como procesos aislados de una manera eficiente y segura.

Las imágenes de contenedor se crean habitualmente a partir de una lista de instrucciones definida en un
`Dockerfile`. Las imágenes de contenedor se crean casi siempre a partir de otras imágenes de contenedor (esencialmente como una continuación de las instrucciones de un estado anterior conocido). Puede utilizar el fragmento de código siguiente para crear su propia imagen de Open Liberty, por ejemplo:

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

Una vez que se haya creado una imagen, se puede ejecutar. Lo motores de ejecución de contenedores, como Docker o
[containerd](https://containerd.io/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), utilizan dicha definición de imagen y ejecutan el punto de entrada definido como un proceso aislado de los recursos directamente en el sistema operativo de host, evitando la sobrecarga de las máquinas virtuales.

Las imágenes de contenedor se almacenan en *registros*. El más conocido es el registro Docker Hub, pero, de manera más común, introducirá y extraerá imágenes de registros de contenedores controlados mediante acceso, como {{site.data.keyword.registryshort_notm}}, que están asociados en mayor medida con su infraestructura y las interconexiones CI/CD.

## Kubernetes
{: #kubernetes}

Las plataformas de nube de IBM aprovechan Kubernetes para la orquestación de contenedores. Por lo tanto, además de conocer los aspectos básicos de los contenedores, es importante que los desarrolladores se familiaricen con los aspectos principales de Kubernetes, incluyendo los mandatos básicos y los artefactos de despliegue. La tabla siguiente incluye algunos conceptos importantes de Kubernetes:

| Concepto | Descripción |
|---------|-------------|
| Pod | Un grupo localizado de contenedores que se despliegan conjuntamente como una unidad individual. Los pods son relativamente inmutables, siendo necesario que se sustituya el pod original para realizar modificaciones en diversos atributos del pod. Una aplicación típica tiene un contenedor con lógica de negocio principal y, opcionalmente, pods adicionales que proporcionan funciones de la plataforma a nivel granular del pod. |
| Despliegue | Una plantilla repetible para un pod sin estado, añadiendo una dimensión de escala al concepto de pod. Además, la definición en plantilla se puede actualizar y se pueden sustituir las instancias subyacentes del pod. Una configuración de despliegue de Kubernetes se supervisa mediante un controlador de despliegue de Kubernetes para garantizar que se mantiene el número declarado de pods en un despliegue determinado. Un despliegue se muestra como `kind: Deployment` en los archivos
`.yaml`. |
| Servicio | Un nombre ampliamente conocido que representa un conjunto de direcciones IP de pod relativamente inestable. Un servicio puede existir únicamente en la red privada del clúster, o se puede exponer de manera externa, normalmente utilizando un equilibrador de carga específico del proveedor de nube. Un servicio se muestra como `kind: Service` en los archivos
`.yaml`. |
| Ingress | Proporciona la capacidad de compartir una dirección de red individual con varios servicios por medio del alojamiento virtual o de direccionamiento basado en el contexto. Ingress también puede realizar actividades de gestión de conexiones de red como la terminación TLS. Ingress se visualiza como `kind: Ingress` en los archivos `.yaml`. |
| Secreto | Un objeto que almacena información confidencial para su uso por parte del tiempo de ejecución de pod y que separa la información específica del despliegue de la coordinación o la imagen del contenedor. Se puede exponer un secreto a un pod en tiempo de ejecución a través de variables de entorno o de montajes de sistema de archivos virtual. Sin los secretos, los datos confidenciales se almacenan en la imagen de contenedor o la coordinación, creando los dos más oportunidades de exposición accidental o de acceso no deseado. |
| ConfigMap | Juega un papel similar a los secretos en cuanto a que separa la información específica del despliegue de la coordinación del contenedor. No obstante, un ConfigMap es una estructura de configuración de propósito general. Se utiliza para enlazar información, como argumentos de línea de mandatos, variables de entorno y otros artefactos de configuración, con los componentes del sistema y contenedores del pod en tiempo de ejecución. | 
{: caption="Tabla 1. Conceptos de Kubernetes" caption-side="bottom"}

Todos los recursos se definen dentro del modelo de recursos de Kubernetes, que se puede configurar a través de la API RESTful o a través de los archivos de configuración enviados mediante la línea de mandatos de `kubectl`.

Para obtener más información, consulte [Aspectos básicos de
Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), [Modelo de objetos de
Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") y
[Línea de mandatos de `kubectl`](https://kubernetes.io/docs/reference/kubectl/overview/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"). 

## Helm
{: #helm}

Helm es un gestor de paquetes que proporciona una manera sencilla para encontrar, compartir y utilizar el software creado para Kubernetes. Helm también aborda una necesidad común del usuario: el despliegue de la misma aplicación en varios entornos. Helm utiliza *diagramas*, que son colecciones de plantillas que producen objetos de Kubernetes (YAML) válidos en el momento de la instalación. Estos diagramas se crean a partir de un lenguaje de plantilla que incluye soporte para variables, operaciones de rango y otros aspectos que quitan una gran carga de trabajo del mantenimiento de metadatos de despliegue de Kubernetes.

Para obtener más información, consulte [Helm](https://helm.sh/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Istio Service Mesh
{: #istio}

Istio es una plataforma de código abierto para la gestión y protección de microservicios. Funciona con coordinadores como Kubernetes, proporcionando una manera de gestionar y controlar la comunicación entre servicios.

Istio trabaja utilizando un modelo de sidecar. Un sidecar (un proxy Envoy) es un proceso independiente que se encuentra junto a la aplicación. El sidecar gestiona toda la comunicación entrante y saliente del servicio, y aplica un nivel común de capacidad a todos los servicios independientemente del lenguaje de programación o la infraestructura con los que se haya creado el servicio. En efecto, Istio proporciona un mecanismo para configurar de forma centralizada las políticas de direccionamiento y seguridad, mientras esas políticas se aplican mediante sidecars de una manera descentralizada.

En la mayoría de los casos, se recomienda utilizar las funciones proporcionadas por Istio en lugar de funciones similares proporcionadas por los entornos o lenguajes de programación individuales. El equilibrio de carga y otras políticas de direccionamiento se definen, gestionan e imponen de una manera más coherente por medio de la infraestructura, por ejemplo.

En algunos casos, tal como ocurre con el rastreo distribuido, Istio y las bibliotecas a nivel de aplicación son complementarios. Puede mejorar las operaciones utilizando ambos conjuntamente. En el caso del rastreo distribuido, Istio solo puede garantizar que las cabeceras de rastreo estén presentes; las bibliotecas de aplicación proporcionan el contexto importante acerca de las relaciones entre solicitudes. Su comprensión del sistema en su totalidad mejora cuando se utilizan de forma conjunta tanto Istio como las bibliotecas del entorno o las bibliotecas de soporte.

A más alto nivel, Istio amplía la plataforma Kubernetes, proporcionando conceptos de gestión, visibilidad y seguridad adicionales. Las características de Istio se pueden desglosar en las cuatro categorías siguientes:

* Gestión de tráfico: controlar el tráfico entre los microservicios para llevar a cabo la división del tráfico, la recuperación tras error y releases mínimos.
* Seguridad: proporcionar una autenticación, autorización y cifrado fuertes y basados en la identidad entre los microservicios.
* Observabilidad: recopilar métricas y registros para una mejor visibilidad en las aplicaciones que se ejecutan en el clúster.
* Políticas: imponer controles de acceso, límites de velocidad y cuotas para proteger las aplicaciones.

Consulte [¿Qué es Istio?](https://istio.io/docs/concepts/what-is-istio/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para obtener más información.



