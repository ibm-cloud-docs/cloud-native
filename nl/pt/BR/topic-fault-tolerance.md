---

copyright:
  years: 2019
lastupdated: "2019-02-11"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Tolerância a Fal
{: #fault-tolerance}

A tolerância a falhas é a capacidade do sistema de continuar funcionando em caso de falha parcial. A criação de um sistema resiliente aplica os requisitos a todos os serviços que estão nele. A natureza dinâmica dos ambientes de nuvem exige que os serviços sejam escritos para esperar e responder normalmente ao inesperado, como o recebimento de dados inválidos, a falta de capacidade de acessar um serviço auxiliar necessário ou a manipulação de conflitos devido a atualizações simultâneas em um sistema distribuído. 

As soluções de tolerância a falhas geralmente se concentram em tempos limite, fallbacks, anteparas e disjuntores.

Em alguns ambientes, os mecanismos de tolerância a falhas podem ser fornecidos por componentes de infraestrutura, como o Istio. Independentemente da ajuda ou não da infraestrutura, um serviço deve presumir que a chamada remota pode falhar e deve estar preparado com ações de fallback apropriadas.

## Tempos limite

A primeira linha de defesa contra falhas parciais é o uso de tempos limite. Os tempos limite garantem que seu aplicativo receba um erro quando um serviço auxiliar não estiver responsivo, permitindo que ele manipule a condição com um comportamento de fallback apropriado. Isso não significa necessariamente que a operação solicitada falhou. Os tempos limite mudam o tempo de espera do cliente da solicitação por uma resposta e não têm impacto no comportamento de processamento do serviço de destino.

Muitas bibliotecas de linguagem usam um tempo limite padrão para solicitações e o Istio também age dessa forma. Por padrão, o proxy do sidecar falha na solicitação se não receber uma resposta em 15 segundos. Esse valor pode ser mudado configurando uma política de tempo limite para a rota na definição VirtualService, que se parece com o seguinte, usando um serviço que retorna cotações de ação como um exemplo:

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

Com essa configuração, as solicitações feitas ao serviço de cotação de ações por meio do proxy do sidecar ou do gateway de ingresso aguardam 30 segundos antes de uma falha ocorrer na solicitação, em vez do valor padrão de 15 segundos.

O aspecto interessante dos tempos limite de ajuste é que todas as solicitações usando a rota têm o tempo limite aplicado. Essa é uma camada base de segurança fornecida pelo Istio: mesmo que uma biblioteca ou estrutura de aplicativo não imponha um tempo limite, ela não esperará para sempre, porque o Istio impõe. No entanto, os tempos limite de nível de aplicativo ainda se aplicam. Dado o exemplo acima, o tempo limite de nível de infraestrutura para o serviço de cotação de ações foi estendido para 30 segundos. Se a biblioteca do aplicativo configurar um tempo limite para 5 segundos, a solicitação do aplicativo ainda falhará com um tempo limite.

## Fallbacks

Um aplicativo deve definir o que acontece quando uma solicitação para um serviço auxiliar falha. Existem algumas poucas opções, mas o objetivo é que o comprometimento aconteça normalmente quando esses serviços não responderem de maneira oportuna. Quando um serviço remoto falha, é possível tentar executar novamente a solicitação, executar uma solicitação diferente ou retornar dados em cache em vez disso.

Tentar executar a solicitação novamente é o mecanismo de fallback mais fácil, à primeira vista. O que não é tão aparente é que as solicitações de nova tentativa podem contribuir para falhas do sistema em cascata ("tempestades de novas tentativas", uma variação do [problema de thundering herd](https://en.wikipedia.org/wiki/Thundering_herd_problem)). O código no nível do aplicativo não está ciente o suficiente do funcionamento da rede ou do sistema e algoritmos de backoff exponencial são difíceis de corrigir.

O Istio pode realizar novas tentativas de maneira muito mais efetiva. Ele já está diretamente envolvido no roteamento de solicitações e fornece uma implementação consistente e com abrangência de linguagem para políticas de nova tentativa. Por exemplo, poderíamos definir uma política como a seguinte para o nosso serviço de cotação de ações:

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

Com essa configuração simples, as solicitações feitas ao serviço de cotação de ações por meio de um proxy de sidecar ou gateway de ingresso do Istio terão novas tentativas realizadas até 3 vezes, com um tempo limite de 5 segundos para cada uma. [Regras de correspondência de rota adicional](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPMatchRequest) poderiam restringir ainda mais essa política de nova tentativa para solicitações `GET`, por exemplo.

Nesse caso, há uma nuance que deve ser notada: você não está especificando o intervalo de nova tentativa. O sidecar determina o intervalo entre as novas tentativas e, deliberadamente, introduz "jitter" entre elas para evitar grandes volumes em serviços sobrecarregados.

## Anteparas

Em navegações, uma antepara é uma partição que evita que um vazamento em um compartimento afunde todo o barco. O padrão é aplicado em ambientes de nuvem para alcançar um propósito semelhante e é feito de algumas maneiras diferentes.

Para linguagens multiencadeadas, como Java, as anteparas internas podem ser usadas internamente para restringir ou controlar como os recursos são usados para a comunicação com recursos remotos, usando os mecanismos de fila ou de semáforo:

- Para usar uma fila, o serviço associa um número específico de encadeamentos a uma fila específica. Quaisquer solicitações feitas depois que a fila estiver cheia obterão uma falha rápida. Em Java, isso pode ser um `ThreadPoolExecutor` associado a um `BlockingQueue`, como um exemplo.
- O mecanismo de semáforo funciona com um número definido de permissões. Uma solicitação de saída requer uma permissão. Depois que uma solicitação foi concluída com sucesso, a permissão é liberada para ser usada por outra solicitação.

Também é possível definir anteparas entre serviços utilizando um Istio DestinationRule para restringir o conjunto de conexões para um serviço de envio de dados, utilizando algo como o exemplo a seguir:

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

Essa configuração limita o número máximo de conexões simultâneas sendo feitas para cada instância do serviço de cotação de ações para 10. Os serviços que não puderem fazer uma conexão dentro de 30 segundos obterão uma resposta `503 -- Service Unavailable`. Esse tipo de antepara pode ser usada para evitar que um serviço de cálculo intensivo receba mais solicitações do que pode gerenciar, por exemplo.

## Disjuntores

Os disjuntores são usados para otimizar o comportamento de solicitações de saída quando falhas ocorrem. Em vez de fazer solicitações repetidamente para um serviço não responsivo, um disjuntor observa o número de falhas que ocorreram dentro de um determinado período. Se a taxa de erro exceder um limite, o disjuntor abrirá o circuito e causará uma falha na solicitação e em todas as subsequentes, até que o circuito esteja fechado novamente.

Os disjuntores também são definidos usando um Istio DestinationRule:

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

Essa configuração coloca restrições nas solicitações que outros serviços estão fazendo para o serviço de cotação de ações. A política de tráfego `outlierDetection` especificada é aplicada a cada instância individual. Para colocar a configuração anterior em uma frase, "ejete qualquer instância de serviço de cotação de ações que falhar 3 vezes em 5 segundos por pelo menos 5 minutos. Além disso, todas as instâncias podem ser ejetadas". A última configuração `maxEjectionPercent` está relacionada ao balanceamento de carga. O Istio mantém um conjunto de balanceamento de carga e ejeta instâncias com falha desse conjunto. Por padrão, ele ejeta um máximo de 10% de todas as instâncias disponíveis do conjunto de balanceamento de carga.

Para aqueles que estão familiarizados com outros mecanismos de interrupção do circuito, o Istio não tem um estado meio aberto. Em vez disso, aplica uma matemática simples: uma instância permanece ejetada do conjunto por `baseInjectionTime * <number of times it has been ejected>`. Isso permite que instâncias se recuperem de falhas temporárias, além de manter as instâncias que falham consistentemente fora do conjunto.

