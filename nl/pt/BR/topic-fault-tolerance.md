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

# Tolerância a Fal
{: #fault-tolerance}

A tolerância a falhas é uma capacidade do sistema de continuar a funcionar durante uma falha parcial. A criação de um sistema resiliente aplica os requisitos a todos os serviços que estão nele. As naturezas dinâmicas dos ambientes de nuvem exigem que os serviços sejam gravados para esperar e responder normalmente ao inesperado. 

As soluções de tolerância a falhas geralmente se concentram em tempos limite, fallbacks, anteparas e disjuntores.

Em alguns ambientes, os mecanismos de tolerância a falhas podem ser fornecidos por componentes de infraestrutura, como o Istio. Independentemente de a infraestrutura ajudar, um serviço deve assumir que a chamada remota pode falhar e deve estar preparado com ações de fallback apropriadas.

## Tempos limite

A primeira linha de defesa contra falhas parciais é o uso de tempos limite. Os tempos limites asseguram que seu aplicativo receba um erro quando um serviço auxiliar não estiver respondendo, permitindo que ele manipule a condição com um comportamento de fallback apropriado. Isso não significa que a operação solicitada falhou. Os tempos limites mudam de acordo com o tempo que um cliente faz uma solicitação esperar por uma resposta. Eles não têm nenhum impacto sobre o comportamento de processamento do serviço de destino.

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

As solicitações feitas no serviço de cotação de ações por meio do proxy sidecar ou gateway do Ingress esperam 30 segundos antes de a solicitação falhar, em vez dos 15 segundos padrão.

Ao ajustar os tempos limites dessa maneira, todas as solicitações que estão usando a rota têm o tempo limite aplicado. Essa é uma camada base de segurança fornecida pelo Istio. Mesmo que uma estrutura de aplicativo ou uma biblioteca não imponha um tempo limite, ele não esperará para sempre porque o Istio o fará. No entanto, os tempos limites do nível do aplicativo ainda se aplicam. O tempo limite de nível de infraestrutura para o serviço de cotação de ações foi estendido para 30 segundos. Se a biblioteca do aplicativo configurar um tempo limite para 5 segundos, a solicitação do aplicativo ainda falhará com um tempo limite.

## Fallbacks

Um aplicativo define o que acontece quando uma solicitação para um serviço auxiliar falha. Existem algumas poucas opções, mas o objetivo é que o comprometimento aconteça normalmente quando esses serviços não responderem de maneira oportuna. Quando um serviço remoto falha, é possível tentar executar novamente a solicitação, executar uma solicitação diferente ou retornar dados em cache em vez disso.

Tentar executar a solicitação novamente é o mecanismo de fallback mais fácil, à primeira vista. O que não é tão aparente é que as solicitações de nova tentativa podem contribuir para falhas do sistema em cascata ("tempestades de novas tentativas", uma variação do [problema de thundering herd](https://en.wikipedia.org/wiki/Thundering_herd_problem)). O código no nível do aplicativo não está ciente o suficiente do funcionamento da rede ou do sistema e algoritmos de backoff exponencial são difíceis de corrigir.

O Istio pode realizar novas tentativas de maneira muito mais efetiva. Ele já está diretamente envolvido no roteamento de solicitações e fornece uma implementação consistente e com abrangência de linguagem para políticas de nova tentativa. Por exemplo, é possível definir uma política como a seguinte para o serviço de cotação de ações:

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

Com essa configuração simples, as solicitações que são feitas para o serviço de cotação de ações por meio de um proxy sidecar do Istio ou gateway do Ingress tentam novamente até três vezes, com um tempo limite de cinco segundos para cada tentativa. [Regras de correspondência de rota adicional](https://istio.io/docs/reference/config/networking/#HTTPMatchRequest) podem restringir ainda mais essa política de nova tentativa para solicitações `GET`, por exemplo.

Nesse caso, há uma nuance que deve ser notada: você não está especificando o intervalo de nova tentativa. O sidecar determina o intervalo entre as novas tentativas e, deliberadamente, introduz "jitter" entre elas para evitar grandes volumes em serviços sobrecarregados.

<!-- Notes about other approaches here: -->

## Anteparas
{: #bulkheads-fault-tolerance}

Em navegações, uma antepara é uma partição que evita que um vazamento em um compartimento afunde todo o barco. O padrão é aplicado em ambientes de nuvem para alcançar um propósito semelhante e é feito de algumas maneiras diferentes.

Para linguagens multiencadeadas, como a Java, as anteparas internas podem ser usadas internamente para restringir ou controlar como os recursos são usados para comunicação com recursos remotos, usando um mecanismo de fila ou semáforo:

- Para usar uma fila, o serviço associa um número específico de encadeamentos a uma fila específica. Qualquer solicitação feita quando a fila estiver cheia obterá uma falha rápida. Em Java, isso pode ser um `ThreadPoolExecutor` associado a um `BlockingQueue`, como um exemplo.
- O mecanismo de semáforo funciona com um número definido de permissões. Uma solicitação de saída requer uma permissão. Depois que uma solicitação foi concluída com sucesso, a permissão é liberada para ser usada por outra solicitação.

Também é possível definir as anteparas entre serviços usando um Istio DestinationRule para restringir o conjunto de conexões para um serviço de envio de dados, usando algo como o exemplo a seguir:

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

Essa configuração limita o número máximo de conexões simultâneas feitas para cada instância do serviço de cotação de ações a 10. Os serviços que não podem fazer uma conexão dentro de 30 segundos obtêm uma resposta `503 -- Service Unavailable`. Esse tipo de antepara pode ser usada para evitar que um serviço de cálculo intensivo receba mais solicitações do que pode gerenciar, por exemplo.

## Disjuntores

Os disjuntores são usados para otimizar o comportamento de solicitações de saída quando falhas ocorrem. Em vez de fazer solicitações repetidamente para um serviço não responsivo, um disjuntor observa o número de falhas que ocorrem dentro de um determinado período de tempo. Se a taxa de erro exceder um limite, o disjuntor abrirá o circuito e causará uma falha na solicitação e em todas as subsequentes, até que o circuito esteja fechado novamente.

Os disjuntores também são definidos usando um DestinationRule do Istio:

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

Essa configuração coloca restrições nas solicitações que outros serviços estão fazendo para o serviço de cotação de ações. A política de tráfego `outlierDetection` especificada é aplicada a cada instância individual. Para redigir a configuração anterior como uma sentença, "Ejete qualquer instância de serviço de cotação de ações que falhar três vezes em cinco segundos durante pelo menos 5 minutos. Todas as instâncias podem ser ejetadas". A última configuração `maxEjectionPercent` está relacionada ao balanceamento de carga. O Istio mantém um conjunto de balanceamento de carga e ejeta instâncias com falha desse conjunto. Por padrão, ele ejeta um máximo de 10% de todas as instâncias disponíveis do conjunto de balanceamento de carga.

Para aqueles que estão familiarizados com outros mecanismos de interrupção do circuito, o Istio não tem um estado meio aberto. Ele aplica um pouco de matemática simples. Uma instância permanece ejetada do conjunto por `baseInjectionTime * <number of times it has been ejected>`. Isso permite que as instâncias se recuperem de falhas temporárias e mantenham as instâncias com falha fora do conjunto.

Com isso em vigor, o número máximo de chamadas simultâneas para o serviço de cotação de ações é limitado a 10. Qualquer coisa além disso obterá uma resposta `503 -- Service Unavailable`.

### Istio -- Limites de taxa
{: #istio-rate-limits}

Também é possível definir **Limites de taxa** com o Istio. Elas são semelhantes às anteparas, mas são definidas ao longo de um espaço de tempo, em vez de limitar apenas as chamadas simultâneas que são processadas no mesmo instante.

Por exemplo, você não deseja permitir mais de um tweet por segundo. Este é um exemplo do mundo real. O Twitter bloqueou automaticamente a conta `@IBMStockTrader` no passado quando testes de tensão foram realizados em relação ao negociador de ações.

<!-->
*< É necessário um exemplo. Observe que os limites de taxa requerem a introdução do conceito de Mixer Adapters, como memquota ou redisquota. Há mais para eles do que nas políticas anteriores. >*


## Teste: injeção de falha com o Istio

É possível usar o Istio para causar falhas deliberadamente para ver como seu código responderia a elas....
-->
