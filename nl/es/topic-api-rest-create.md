---

copyright:
  years: 2019
lastupdated: "2019-06-05"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Creación de microservicios RESTful
{: #rest-api}

Las aplicaciones nativas en la nube producen y consumen API, ya sea en una arquitectura de microservicios o no. Algunas API se consideran internas, o privadas, y algunas se consideran externas. 
{:shortdesc}

Las API internas solo se utilizan dentro de un entorno con cortafuegos para que los servicios de fondo se puedan comunicar entre sí. Las API externas presentan un punto de entrada unificado para los consumidores y, con frecuencia, son **gestionadas** por herramientas como {{site.data.keyword.apiconnect_long}}, que pueden imponer una limitación de velocidad u otras restricciones de uso. Un ejemplo de una API como esta es la [API de desarrollador de GitHub](https://developer.github.com/v3/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"). Proporciona una API unificada con el uso coherente de códigos de retorno y verbos HTTP, comportamiento de paginación, etc., sin exponer los detalles de la implementación interna. Esta API puede estar respaldada por una aplicación monolítica, o por una colección de microservicios; dicho detalle no se expone al consumidor, lo que deja libertad a GitHub para evolucionar sus propios sistemas internos según sea necesario.

## Métodos recomendados para las API RESTful
{: #bps-apis}

Las API REST deben utilizar verbos HTTP estándar para las operaciones de crear, recuperar, actualizar y suprimir (CRUD), prestando especial atención a si la operación es o no idempotente (segura para su reintento en múltiples ocasiones).

* Las operaciones POST se pueden utilizar para crear o actualizar recursos. Las operaciones POST no se pueden invocar repetidamente. Por ejemplo, si se utiliza una solicitud POST para crear recursos y se invoca varias veces, se crea un nuevo recurso exclusivo como resultado de cada invocación.
* Las operaciones GET se deben poder invocar repetidamente y no deben provocar efectos secundarios. Solo se deben utilizar para recuperar información. No se deben utilizar solicitudes GET con parámetros de consulta para cambiar o actualizar la información. Utilice las operaciones POST, PUT o PATCH en su lugar.
* Las operaciones PUT se pueden utilizar para actualizar recursos. Las operaciones PUT incluyen habitualmente una copia completa del recurso a actualizar, haciendo que sea posible invocar la operación varias veces.
* Las operaciones PATCH permiten la actualización parcial de los recursos. Se pueden invocar repetidamente dependiendo de cómo se especifique el delta y se aplique luego al recurso. Por ejemplo, si una operación PATCH indica que un valor se debe cambiar de A a B, se puede invocar repetidamente. No hay ningún efecto si se invoca varias veces y el valor ya es B.
* Las operaciones DELETE se pueden invocar varias veces, ya que un recurso solo se puede suprimir una vez. No obstante, el código de retorno varía, ya que la primer operación tiene éxito (`200` o `204`), mientras que las invocaciones posteriores no encuentran el recurso (`404` o `410`).

### Resultados descriptivos y legibles para la máquina
{: #rest-results}

Dado que las API las invoca el software en lugar del usuario, tenga cuidado de que se comunique la información al proceso llamante de la forma más efectiva y eficiente posible.

Utilice códigos de estado HTTP relevantes y útiles, tal como se describe en la tabla siguiente: 

| Código de error HTTP | Instrucciones de uso |
|-----------------|----------------|
| `200 (OK)` | Utilizar cuando todo sea correcto y existan datos que devolver. |
| `204 (NO CONTENT)` | Utilizar cuando todo sea correcto pero no existan datos de respuesta. |
| `201 (CREATED)` | Utilizar para solicitudes POST que provoquen que se cree un recurso, exista o no un cuerpo de respuesta. |
| `409 (CONFLICT)` | Utilizar cuando haya un conflicto de cambios simultáneos. |
| `400 (BAD REQUEST)` | Utilizar cuando los parámetros tengan un formato incorrecto. |
{: caption="Tabla 1. Códigos de estado HTTP." caption-side="bottom"}

Para obtener más información, consulte [Códigos de estado de respuesta](https://tools.ietf.org/html/rfc7231#section-6){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"). 

También debe tener en cuenta qué datos devolver en las respuestas para hacer que la comunicación sea eficiente. Por ejemplo, cuando se crea un recurso con una solicitud POST, la respuesta debe incluir la ubicación del recurso recién creado en una cabecera Location. El recurso creado se suele incluir también en el cuerpo de la respuesta, para evitar que una solicitud GET adicional tenga que buscar el recurso creado. Lo mismo se aplica a las solicitudes PUT y PATCH.

### URI de recursos RESTful
{: #rest-uris}

Existen diferentes opiniones acerca de algunos aspectos de los URI de recursos RESTful. En general, hay consenso en que los recursos deben ser nombres, no verbos, y que los puntos finales deben ser plurales. Esto resulta en una estructura clara para las operaciones CRUD:

* `POST /accounts`: crear una nueva cuenta.
* `GET /accounts`: recuperar una lista de cuentas.
* `GET /accounts/16`: recuperar una cuenta específica.
* `PUT /accounts/16`: actualizar una cuenta específica.
* `PATCH /accounts/16`: actualizar una cuenta específica.
* `DELETE /accounts/16`: suprimir una cuenta específica.

Las relaciones se modelan utilizando URI jerárquicos, por ejemplo, ` /accounts/16/credentials`, para gestionar las credenciales asociadas a una cuenta.

Hay menos consenso en relación con qué debe ocurrir con las operaciones asociadas al recurso que no se ajusten a esta estructura habitual. No hay una única manera correcta de gestionar estas operaciones: haga lo que funcione mejor para el consumidor de la API.

### Solidez y API RESTful
{: #robust-api}

El [Principio de solidez](https://tools.ietf.org/html/rfc1122#page-12){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") proporciona la mejor guía: "Sea liberal en lo que acepta y conservador en lo que envía". Suponga que las API evolucionarán con el tiempo y sea tolerante con los datos que no entienda.

#### Producción de API
{: #robust-producer}

Al proporcionar una API a clientes externos, hay dos acciones que se deben realizar al aceptar solicitudes y devolver respuestas: 

* Aceptar atributos desconocidos como parte de la solicitud.
    - Si un servicio llama a la API con atributos innecesarios, simplemente descarte dichos valores. La devolución de un error en esta situación puede provocar fallos innecesarios, afectando negativamente al usuario final.
* Devolver solo los atributos necesarios para los consumidores
    - Evite exponer detalles internos del servicio. Exponga solo los atributos que necesiten los consumidores como parte de la API.

#### Consumo de API
{: #robust-consumer}

Al consumir las API:

* Valide la solicitud únicamente con las variables o atributos que necesite.
    - No valide con las variables solo por el hecho de que se proporcionen. Si no las utiliza como parte de su solicitud, no se base simplemente en que estén ahí.
* Acepte atributos desconocidos como parte de la respuesta.
    - No emita una excepción si recibe una variable no esperada. Siempre que la respuesta contenga la información que necesita, no importa que venga algo más en el camino.

Estas directrices son especialmente relevantes para lenguajes fuertemente tipados como Java, donde la serialización y deserialización de JSON a menudo se produce indirectamente, por ejemplo, mediante bibliotecas Jackson o JSON-P/JSON-B. Busque mecanismos del lenguaje que le permitan especificar un comportamiento más generoso, como ignorar atributos desconocidos, o definir o filtrar qué atributos se deben serializar.

### Creación de versiones de API RESTful
{: #version-api}

Una de las ventajas principales de los microservicios es la posibilidad de permitir que evolucionen de forma independiente. Dado que los microservicios llaman a otros servicios, esta independencia conlleva una gran advertencia: no puede provocar cambios que rompan la API.

Si se sigue el principio de solidez, puede pasar mucho tiempo hasta que se requiera un cambio de ruptura. Cuando finalmente se produzca dicho cambio de ruptura, puede optar por crear un servicio distinto por completo y retirar el original con el tiempo.

En el caso de que necesite realizar cambios que rompan la API para un servicio existente, decida cómo gestionar dichos cambios: si el servicio gestionará todas las versiones de la API, si ha decidido mantener versiones independientes del servicio para dar soporte a cada versión de la API, o si el servicio solo admitirá la versión más reciente de la API y se basará en otras capas adaptativas para realizar la conversión entre esta versión de la API y la anterior.

Después de determinar cómo gestionar los cambios, otro problema a resolver, aunque esta vez mucho más fácil, es cómo reflejar la versión en la API. Generalmente hay tres maneras de indicar la versión de un recurso REST:

* Incluir la versión en la vía de acceso de URI.
* Incluir la versión en la cabecera HTTP Accept y basarla en la negociación de contenido.
* Utilizar una cabecera de solicitud personalizada.

#### Incluir la versión en la vía de acceso de URI
{: #version-path}

La forma más fácil de especificar una versión es incluirla en la vía de acceso del URI. Existen ventajas en este enfoque: obviamente, es fácil de conseguir al crear los servicios en la aplicación, y es compatible con herramientas de navegación de API como Swagger y herramientas de línea de mandatos como `curl`, etc.

Si va a incluir la versión en la vía de acceso de URI, la versión debe aplicarse a la aplicación en su totalidad, por ejemplo, `/api/v1/accounts` en lugar de `/api/accounts/v1`. Hypermedia como motor de estado de aplicación (HATEOAS) es una manera de proporcionar URI a consumidores de API para que no tengan la responsabilidad de construir los URI ellos mismos. GitHub, por ejemplo, proporciona [URL de Hypermedia](https://developer.github.com/v3/#hypermedia){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") en las respuestas por este motivo. HATEOAS empieza a ser difícil, si no imposible, de conseguir si distintos servicios de respaldo pueden tener versiones que varían de forma independiente en sus URI.

#### Modificar la cabecera Accept para incluir la versión
{: #version-accept}

La cabecera Accept es un lugar obvio en el que definir una versión, pero es uno de los más difíciles de probar. La cabecera Accept es también un lugar de destino frecuente para conmutadores de características. La especificación de cabeceras HTTP requiere invocaciones de API más detalladas.

#### Añadir una cabecera de solicitud personalizada
{: #version-custom}

Puede añadir una cabecera de solicitud personalizada para indicar la versión de API. Al igual que con la cabecera Accept, también puede usar cabeceras personalizadas para direccionar el tráfico a instancias de programa de fondo específicas. Con este método, encontrará los mismos problemas de facilidad de uso que puede encontrar con el método de la cabecera Accept, con el requisito adicional de que los consumidores necesitan información sobre esta cabecera.

Para obtener más información, consulte [El mantenimiento de versiones de la API es incorrecto, por lo que he decidido hacerlo de 3 maneras incorrectas](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").

## Creación y generación de API
{: #create-api}

[OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") es la especificación oficial para los servicios RESTful, controlada por la [Iniciativa OpenAPI](https://www.openapis.org/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo"), una asociación de empresas bajo la Fundación Linux.

Puede utilizar cualquiera de los métodos siguientes para crear una API:

  * Empezar por una definición de OpenAPI (de arriba a abajo): en este enfoque, comienza creando una definición de OpenAPI en un formato independiente del lenguaje (normalmente YAML). A continuación, utilizará un generador de código para crear un esqueleto y creará la implementación del servicio a partir de ahí. Este patrón lo adoptan habitualmente las empresas que tienen un equipo de diseño de API central, y permite que el desarrollo y las pruebas progresen en paralelo.
  * Empezar con código (de abajo a arriba): el código es el origen de su definición de API. Este enfoque funciona bien para aplicaciones nuevas con un aspecto experimental, ya que la definición de la API evoluciona a medida que se entiende mejor lo que debe hacer el servicio. Este enfoque también funciona mejor en algunos lenguajes que en otros, ya que se basa en herramientas que generan una definición de OpenAPI a partir del código. Java, por ejemplo, tiene un soporte excelente para generar documentos de OpenAPI a partir de infraestructuras REST basados en anotaciones.

En cualquier caso, trabajar con una definición de OpenAPI puede servir de ayuda para identificar áreas donde la API es incoherente o difícil de entender desde el punto de vista de un consumidor. Las definiciones de OpenAPI publicadas o controladas por versiones también las pueden utilizar las herramientas de compilación como ayuda para marcar cambios de ruptura que puedan afectar a los consumidores.

### Creación de una API a partir de una definición de OpenAPI
{: #openapi-first}

Puede crear su archivo YAML de OpenAPI en cualquier herramienta de su elección. No obstante, el uso de un editor de texto sin formato puede provocar errores. Algunos editores tienen soporte básico para YAML, y algunos pueden tener extensiones adicionales para dar soporte a definiciones de OpenAPI. Por ejemplo, puede utilizar extensiones de Visual Studio Code como [Visor de Swagger](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") o [Vista previa de OpenAPI](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") para validar la definición de OpenAPI frente a una versión de especificación determinada y representar una vista web en el panel de vista previa:

![Vista previa de OpenAPI](images/create-api-image1.png "Vista previa de OpenAPI")
{: caption="Figura 1. Vista previa de OpenAPI" caption-side="bottom"} 

También hay una serie de editores basados en navegador y de análisis en vivo que puede utilizar en línea o de manera local. Algunos ejemplos incluyen:

* El [Proyecto de GUI de OpenAPI](https://github.com/Mermade/openapi-gui){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") admite tanto la versión 2 como la versión 3 de la especificación de OpenAPI y puede migrar una definición de OpenAPI v2 a v3 automáticamente.
* El [editor Swagger de SmartBear](https://editor.swagger.io){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") también admite la versión 2 y la versión 3 de OpenAPI.
* [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") proporciona un conjunto de editores y herramientas para el modelado y la creación de API.

### Generación de la implementación de API
{: #code-first}

Puede utilizar el [generador OpenAPI](https://github.com/OpenAPITools/openapi-generator){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo") de código abierto para crear un proyecto esqueleto para la implementación de su servicio a partir de una definición de OpenAPI. Puede especificar el lenguaje o la infraestructura para el esqueleto desde la línea de mandatos. Por ejemplo, para crear un proyecto Java para la API PetStore de ejemplo que utiliza anotaciones del método JAX-RS genérico, especifique el siguiente mandato:

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

