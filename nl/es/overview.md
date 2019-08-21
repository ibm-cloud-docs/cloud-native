---

copyright:
  years: 2019
lastupdated: "2019-07-16"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# ¿Qué significa nativo de la nube?
{: #overview}

Los entornos de computación en la nube son dinámicos, con asignación y liberación a demanda de recursos de una agrupación compartida y virtualizada. Estos entornos elásticos permiten más opciones de escalado flexible si se comparan con la asignación de recursos frontal que se utiliza habitualmente en los centros de datos locales tradicionales.
{:shortdesc}

De acuerdo con [Cloud Native Computing Foundation](https://github.com/cncf/foundation/blob/master/charter.md){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), los sistemas nativos de la nube tienen los atributos siguientes:

- Las aplicaciones o procesos se ejecutan en contenedores de software como unidades aisladas.
- Los procesos se gestionan mediante procesos de coordinación centrales para mejorar el uso de recursos y reducir los costes de mantenimiento.
- Las aplicaciones o servicios (microservicios) están débilmente acopladas con las dependencias descritas explícitamente.

Estos atributos describen un sistema altamente dinámico compuesto por procesos independientes que trabajan de forma conjunta para proporcionar un valor empresarial: un sistema distribuido.

El cálculo distribuido es un concepto con raíces que se remontan a décadas atrás. [Falacias del cómputo distribuido](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") recoge las suposiciones siguientes hechas por arquitectos y diseñadores de sistemas distribuidos que finalmente han resultado ser erróneas. 

* La red es fiable.
* La red es segura.
* La red es homogénea.
* La latencia es cero.
* El ancho de banda es infinito.
* La topología no cambia.
* Hay un administrador.
* El coste de transporte es cero.

Las tecnologías de la nube como Kubernetes e Istio tienen como objetivo abordar estas preocupaciones en la propia infraestructura.

## Doce factores
{: #twelve-factors}

Los desarrolladores de Heroku han creado un borrador de la metodología de [aplicación en doce factores](https://12factor.net){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"). Las características mencionadas en los doce factores no son específicas de un proveedor de nube, plataforma o lenguaje. Los factores representan un conjunto de directrices o métodos recomendados para aplicaciones portables y resistentes que se desarrollan en entornos de nube (específicamente la aplicación de software como servicio). Los doce factores se proporcionan en la lista siguiente:

1. Existe una asociación uno a uno entre un código base con versión, por ejemplo, un repositorio Git, y un servicio desplegado. Se utiliza el mismo código base para muchos despliegues.
2. Los servicios declaran explícitamente todas las dependencias y no se basan en la presencia de bibliotecas ni herramientas a nivel del sistema.
3. La configuración que varía entre entornos de despliegue se almacena en el entorno, específicamente en variables de entorno.
4. Todos los servicios de respaldo se tratan como recursos conectados, que se gestionan (se conectan y se desconectan) por medio del entorno de ejecución.
5. El conducto de entrega tiene etapas estrictamente separadas: compilar, publicar, ejecutar.
6. Las aplicaciones se despliegan como uno o más procesos sin estado. Específicamente, los procesos transitorios no tienen estado y no comparten nada. Los datos persistentes se almacenan en un servicio de respaldo adecuado.
7. Los servicios autocontenidos se establecen a sí mismos como disponibles para otros servicios mediante la escucha en un puerto especificado.
8. La simultaneidad se obtiene escalando procesos individuales (escalado horizontal).
9. Los procesos son desechables: los comportamientos de inicio rápido y cierre ordenado permiten que un sistema sea más sólido y resistente.
10. Todos los entornos, desde desarrollo local a producción, son tan similares como sea posible.
11. Las aplicaciones producen registros como secuencias de sucesos, por ejemplo, la escritura en `stdout` y `stderr`, y confían en el entorno de ejecución para agregar las secuencias.
12. Si se necesitan tareas de administración únicas, se mantienen en el control de origen y se empaquetan junto con la aplicación para garantizar que se ejecutan con el mismo entorno que la aplicación.

No tiene que seguir estos factores de manera estricta para conseguir un entorno de microservicios de calidad. No obstante, tenerlos en cuenta le permitirá crear y mantener aplicaciones portables o servicios en entornos de entrega continua.

## Aplicaciones
{: #apps-intro}

Una app consta de código, datos, servicios y cadenas de herramientas. Por ejemplo, la app móvil de {{site.data.keyword.cloud_notm}} contiene código de fondo junto con lógica de fondo, almacenamiento de datos, analíticas y servicios de seguridad y, además, está configurado para una entrega continua.

![Reutilizar](images/garage_reuse2.png "Con Experiencia del desarrollador, puede reutilizar para no tener que reinventar")

Puede crear y gestionar una app utilizando cualquier portal de desarrollador de {{site.data.keyword.cloud_notm}} o {{site.data.keyword.dev_cli_notm}}.

Puede crear apps sencillas vacías directamente o crear apps más complejas mediante kits de inicio. Si decide crear apps vacías sin la ayuda de un kit de inicio, puede hacerlo desde el [panel de control de {{site.data.keyword.cloud_notm}} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://{DomainName}){: new_window} sin visitar ningún portal.

Puede utilizar un patrón de código para crear rápidamente una app y desplegarla en {{site.data.keyword.cloud_notm}}. En el [sitio web de IBM Developer ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](https://developer.ibm.com/patterns/){:new_window}, elija un patrón de código. Puede ver el código en GitHub o crear y compilar una app en {{site.data.keyword.cloud_notm}}, donde puede utilizar una cadena de herramientas de DevOps para desplegar automáticamente la app.

## Microservicios
{: #microservices}

Un *microservicio* es un conjunto de componentes de arquitectura pequeños e independientes que se comunican a través de una API ligera común. Cada microservicio del ejemplo simple siguiente es una aplicación de doce factores que utiliza servicios de respaldo reemplazables para almacenar datos y pasar mensajes:

![Una aplicación de microservicios](images/microservice.png "Una aplicación de microservicios")
{: caption="Figura 1. Una aplicación de microservicios" caption-side="bottom"}

Los microservicios son independientes. La agilidad es una de las ventajas de las arquitecturas de microservicios, pero solo existe cuando los servicios se pueden sobrescribir sin que ello afecte a otros servicios. 

Borrar límites de API ofrece a los equipos que trabajan en un servicio la mayor flexibilidad para mejorar la implementación. Esta característica es lo que permite la persistencia y la programación políglota.

Los microservicios son resistentes. La estabilidad de la aplicación depende de la solidez de los servicios individuales frente a los errores. Esta es una diferencia significativa con respecto a las arquitecturas tradicionales, en las que la infraestructura que les da soporte gestiona los errores automáticamente. Cada servicio necesita aplicar patrones de aislamiento, como interruptores y barreras, para contener errores y definir comportamientos de retroceso adecuados para proteger los servicios en sentido ascendente.

Los microservicios no tienen estado y se almacenan en servicios de nube con respaldo externo, como Redis. El comportamiento de inicio rápido y cierre ordenado posibilita en mayor medida que los servicios funcionen bien en entornos automatizados que crean y eliminan instancias en respuesta a la carga o para mantener el buen estado del sistema.

### El significado de "pequeño"
{: #small-microsvc}

El uso de la palabra "pequeño", aplicada a un microservicio, significa que se centra en un propósito. Muchas descripciones establecen paralelismos entre los roles de microservicios individuales y los mandatos encadenados en la línea de mandatos de Unix:

```
ls | grep 'service' | sort -r
```
{:pre}

Estos mandatos de UNIX realizan cada uno una tarea claramente distinta, y puede encadenarlos independientemente del lenguaje de programación o de la cantidad de código.

## Aplicaciones políglotas: elección de la herramienta adecuada para el trabajo
{: #polyglot-apps}

El poliglotismo es una ventaja frecuentemente mencionada de las arquitecturas basadas en microservicios. Cree un equilibrio entre las mismas mediante la creación de una lista de tecnologías admitidas entre las que elegir, con una política definida para ampliar la lista con nuevas tecnologías a lo largo del tiempo. Asegúrese de que se puedan cumplir requisitos no funcionales o de normativa como la capacidad de mantenimiento y de auditoría y la seguridad de los datos, mientras se conserva la agilidad y se da soporte a la innovación a través de la experimentación.

... :FIXME: unpack? Language table here?:...

## REST y JSON
{: #rest-json}

Las aplicaciones políglotas solo son posibles con protocolos independientes del lenguaje. Los patrones de arquitectura REST definen directrices para la creación de interfaces uniformes que separan la representación de datos en la red de la implementación del servicio.

JSON, en las arquitecturas de microservicios, es el formato de cableado adecuado para datos basados en texto, desplazando a XML por su simpleza y claridad. Como comparación, el ejemplo siguiente es un registro básico que contiene datos acerca de un empleado en JSON:

```json
{
  "name": "Marley Cassin",
  "serial": 228264,
  "title": "Senior Software Engineer",
  "address": {
    "office": "501-B101",
    "street": "3858 Kuvalis Pass",
    "city": "East Craig",
    "state": "NC",
    "zip": "64519-8934"
  }
}
```
{: codeblock}

Y el ejemplo siguiente es el mismo registro de empleado en XML:

```xml
<person>
  <name>Marley Cassin</name>
  <serial>228264</serial>
  <title>Senior Software Engineer</title>
  <address>
    <office>501-B101</office>
    <street>3858 Kuvalis Pass</street>
    <city>East Craig</city>
    <state>NC</state>
    <zip>64519-8934</zip>
  </address>
</person>
```
{: codeblock}

JSON utiliza pares de atributo-valor para representar objetos de datos en una sintaxis concisa que conserva la información sobre unos pocos tipos básicos, como números, series, matrices y objetos. Tanto JSON como XML representan de forma clara el objeto de dirección anidado en los ejemplos anteriores, pero necesita el esquema XML asociado para determinar el tipo del elemento `serial`. En JSON, la sintaxis deja claro que el valor de `serial` es un número y no una serie.
