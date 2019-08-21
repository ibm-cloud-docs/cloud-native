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

# Comprobación de estado
{: #healthcheck}

Las comprobaciones de estado proporcionan un mecanismo simple dentro de los sistemas automatizados para examinar el estado de salud de una instancia individual. El sistema responde a estos sucesos de inspección de estado sustituyendo una instancia con errores, actualizando tablas de direccionamiento o comunicando los estados resultantes al usuario.
{:shortdesc}

Kubernetes define dos mecanismos integrales para comprobar el estado de un contenedor:

* Se utiliza una prueba de preparación para indicar si el proceso puede manejar solicitudes direccionables. Kubernetes no direcciona el trabajo a un contenedor con una prueba de actividad anómala. Una prueba de preparación falla si un servicio está ocupado, no ha terminado de inicializarse, está sobrecargado o no puede procesar solicitudes. 
* Se utiliza una prueba de actividad para indicar si el proceso se debe reiniciar. Kubernetes detiene y reinicia un contenedor con una prueba de actividad fallida para garantizar que los pods que estén en un estado en desuso se terminen y se sustituyan. Una prueba de actividad falla si el servicio está en un estado irrecuperable, por ejemplo si se produce una condición de falta de memoria. Las comprobaciones de actividad simples que siempre devuelven la respuesta OK pueden identificar contenedores que estén en un estado incoherente, lo que puede ocurrir cuando el proceso que sirve las solicitudes se ha bloqueado pero el contenedor sigue funcionando.

Los analizadores de preparación y de actividad se definen mediante una estructura similar que incluye retardo de tiempo e intervalos de reintento, periodos de tolerancia de errores, tiempos de espera y la definición de la implementación de la prueba. La prueba se puede implementar mediante la ejecución de un mandato, la comprobación de un punto final TCP para ver si hay conectividad o la realización de una invocación HTTP. Con frecuencia, la misma implementación de la prueba puede utilizarse tanto para fines de preparación como de actividad, pero el retardo de tiempo y los intervalos de reintento deben ajustarse para el objetivo concreto.

## Compresión y aplicación de las pruebas
{: #kubernetes-probes}

Fundamentalmente, el desarrollo de aplicaciones nativas de la nube se basa en el principio de que los procesos de contenedores pueden fallar, pero estos procesos se pueden sustituir fácilmente por un nuevo contenedor. Esto ocurre tanto en respuesta a sucesos inesperados, como un error de un contenedor o de una máquina. También se debe a sucesos operativos, como despliegues de escalado horizontal y de nuevas imágenes de aplicación. Las comprobaciones de preparación garantizan que las nuevas instancias de contenedor estén preparadas para recibir trabajo antes de direccionar el tráfico. Las mismas comprobaciones evitan que se pueda direccionar tráfico a las instancias que has salido o que se están eliminando.

Cuando no se definen comprobaciones de preparación, Kubernetes tiene poca información sobre si una instancia de contenedor está preparada para gestionar tráfico, y direcciona el tráfico inmediatamente después de que se inicie el proceso del contenedor. Sin las comprobaciones de preparación, es probable que las aplicaciones experimenten que se agote el tiempo de espera de conectividad y respuestas de rechazo de conexión cuando se direccione trabajo a una instancia que no esté preparada para atender la solicitud. Las comprobaciones de preparación reducen, pero no eliminan por completo, los errores de conectividad de cliente.

Las comprobaciones de actividad que están pensadas para identificar son menos frecuentes y representan la excepción. Cuando un proceso entra en un estado desde el cual no es posible la recuperación, el proceso queda inoperativo de forma efectiva. Esto puede ocurrir debido a condiciones de falta de memoria o un punto muerto provocado por un error de programación. La mejor manera de recuperarse es terminar el contenedor, lo que termina cualquier proceso que se esté produciendo actualmente en el contenedor como resultado. Es posible que se produzcan bucles de terminación o de reinicio en la aplicación donde los contenedores no pueden entrar en línea antes de terminen y se sustituyan.

Las pruebas de preparación y actividad afectan al sistema de distintas maneras. Puede pensarse en esto en términos de transición de estados, donde el estado positivo de una comprobación de preparación es direccionable y el estado negativo no lo es. Del mismo modo, el estado positivo de una prueba de actividad representa un contenedor que se ejecuta con normalidad y el estado negativo es inoperativo. Cuando se inicia un contenedor, el estado de preparación es negativo inicialmente, y entra en un estado positivo únicamente después de que el contenedor funcione correctamente. Una comprobación de actividad se inicia en un estado positivo, y entra en un estado negativo únicamente cuando el proceso pasa a estar inoperativo.

La configuración de una prueba de preparación de manera agresiva, por ejemplo con un retardo inicial bajo, tiene poco efecto, ya que la ejecución de la prueba demasiado pronto no hace que la comprobación de preparación cambie de estado. Sin embargo, una comprobación de actividad agresiva, donde la prueba se desencadena demasiado pronto, provoca un cambio de transición de estado, haciendo que el sistema termine el contenedor antes de lo esperado.

## Métodos recomendados para configurar pruebas
{: #probe-recommendation}

Si implementa una prueba de estado mediante HTTP, tenga en cuenta los códigos de estado HTTP siguientes correspondientes a preparación, actividad y estado.

| Estado    |  Preparación            |  Actividad             |
|----------|-----------------------|-----------------------|
|          | No es correcta y no se carga | No es correcta y provoca un reinicio |
| Iniciando | 503 - No disponible     | 200 - Correcto              |
| Activo       | 200 - Correcto              | 200 - Correcto              |
| Deteniéndose | 503 - No disponible     | 200 - Correcto              |
| Inactivo     | 503 - No disponible     | 503 - No disponible     |
| Con errores  | 500 - Error del servidor    | 500 - Error del servidor    |
{: caption="Tabla 1. Códigos de estado HTTP" caption-side="bottom"}

Los puntos finales de comprobación de estado no requieren autorización ni autenticación. Dado que estas protecciones no se ponen en marcha en puntos finales de pruebas de estado, limite las implementaciones de pruebas HTTP a las solicitudes GET que no modifiquen ningún dato. No devuelva nunca datos que identifiquen detalles específicos del entorno, como el sistema operativo, el lenguaje de implementación o las versiones de software. Se podrían utilizar para establecer un vector de ataque.

Una prueba de actividad debe ser prudente en cuanto a lo que comprueba, ya que un error provocará la terminación inmediata del proceso. Evite métricas ambiguas que solo indiquen el fallo de un proceso a veces, por ejemplo, un punto final HTTP simple que siempre devuelve
`{"status": "UP"}` con un código de estado 200. Por lo tanto, la mayoría de los procesos que estén en estado inoperativo no pasarán esta comprobación, desencadenando correctamente un reinicio.

Las comprobaciones de estado se producen a intervalos frecuentes, lo que puede provocar más sobrecarga. Las pruebas de preparación y actividad prueban únicamente la viabilidad de servicios de respaldo como bases de datos u otros microservicios en el resultado cuando no haya una reserva aceptable. Para una prueba de actividad, se incluye una comprobación de respaldo si un resultado no satisfactorio provocaría que el contenedor local entrase en un estado irrecuperable. Una prueba de preparación verifica un servicio de respaldo solo cuando el contenedor local no pueda manejar solicitudes si este falla, pero la condición sea recuperable.

Cuando se configura el retardo de tiempo inicial, una prueba de preparación utiliza el valor menos probable y una comprobación de actividad utiliza el valor de tiempo más probable. Por ejemplo, si un servidor de aplicaciones tiende a iniciarse en 30 segundos, un retardo de preparación típico es de 10 segundos. La comprobación de actividad utiliza un valor de 60 segundos para garantizar que el inicio del servidor se complete antes de comprobar si hay estados terminables.

El atributo *periodSeconds* para las decisiones de direccionamiento se configura habitualmente en un valor de un solo dígito, si la implementación de la prueba es relativamente ligera. Por ejemplo, una prueba HTTP que devuelve el estado 200 OK sin proceso sustancial del lado del servidor tiene una carga de procesador mínima y se puede repetir fácilmente cada 1 - 5 segundos.

## Configuración de pruebas en Kubernetes
{: #probe-config}

Declare pruebas de actividad y preparación junto a su despliegue de Kubernetes en el elemento contenedor. Ambas pruebas utilizan los mismos parámetros de configuración:

| Parámetro | Descripción |
|-----------|-------------|
| *initialDelaySeconds* | La cantidad de tiempo que el kubelet espera después de que se cree el contenedor antes de la primera prueba. |
| *periodSeconds* | Frecuencia en que el kubelet prueba el servicio. El valor predeterminado es 1. |
| *timeoutSeconds* | Rapidez con la que se agota el tiempo de espera de la prueba. El valor predeterminado y mínimo es 1. |
| *successThreshold* | El número de veces que la prueba debe ser correcta tras un fallo. El valor predeterminado y mínimo es 1. El valor debe ser 1 en las pruebas de actividad. |
| *failureThreshold* | El número de veces que Kubernetes intenta reiniciar un pod antes de desistir cuando el pod se inicia y la prueba falla. El valor mínimo es 1 y el valor predeterminado es 3. |

  Para una prueba de actividad, desistir implica reiniciar el pod. Para una prueba de preparación, desistir significa marcar el pod como no preparado.
  {: note}

Para evitar ciclos de reinicio, establezca el parámetro `livenessProbe.initialDelaySeconds` de manera que, con seguridad, sea más largo que el tiempo que tarda su servicio en inicializarse. A continuación, utilice un valor más corto para el atributo
`readinessProbe.initialDelaySeconds` para direccionar solicitudes al servicio tan pronto como esté preparado.

Una configuración de ejemplo puede tener un aspecto similar al ejemplo siguiente (observe los valores de vía de acceso y puerto):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Para obtener más información, consulte
[Configurar pruebas de actividad y preparación en Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/){: new_window} ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo").
