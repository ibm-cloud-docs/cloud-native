---

copyright:
  years: 2019
lastupdated: "2019-06-06"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:external: target="_blank" .external}

# 결함 허용
{: #fault-tolerance}

결함 허용(fault tolerance)은 부분적 고장 시에도 계속 작동할 수 있는 시스템 기능입니다. 탄력적 시스템을 생성하면 해당 시스템의 모든 서비스에 요구사항이 적용됩니다. 클라우드 환경의 동적 특성은 서비스가 잘못된 데이터 수신, 필요한 지원 서비스에 연결 불가능 또는 분산 시스템의 동시 업데이트로 인한 충돌 처리 등을 예측하고 예상치 못한 상황에 올바르게 대응할 수 있게 작성될 것을 요구합니다. 

결함 허용 솔루션은 일반적으로 제한시간, 대체 조치, 벌크헤드 및 회로 차단기에 중점을 두고 있습니다.

일부 환경에서, 결함 허용 메커니즘은 Istio와 같은 인프라 컴포넌트가 제공할 수 있습니다. 인프라가 도움이 되는지에 관계없이, 서비스는 원격 호출이 실패할 수 있다고 가정하고 적절한 대체 조치로 준비해야 합니다.

## 제한시간

부분적인 실패에 대한 첫 번째 방어선은 제한시간을 사용하는 것입니다. 제한시간은 백업 서비스가 응답하지 않는 경우 애플리케이션이 오류를 수신하여 적절한 대체 동작으로 조건을 처리할 수 있도록 합니다. 이는 요청된 조작이 실패했음을 의미하지는 않습니다. 제한시간은 클라이언트가 응답을 기다리는 시간을 변경하며, 대상 서비스의 처리 동작에 영향을 주지 않습니다.

대부분의 언어 라이브러리는 요청에 대해 기본 제한시간을 사용하고, Istio도 마찬가지입니다. 기본적으로, 사이드카 프록시는 15초 이내에 응답을 수신하지 않으면 요청에 실패합니다. 이 값은 다음과 같은 가상 서비스 정의의 라우트에 대한 제한시간 정책을 설정하여 변경할 수 있습니다. 예를 들어, 주식 시세를 리턴하는 서비스를 사용하여 다음과 같이 표시됩니다.

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

이 구성을 사용하면 사이드카 프록시 또는 Ingress 게이트웨이를 통해 주식 시세 서비스에 작성된 요청은 기본값인 15초가 아니라 30초 동안 대기하고 그 후에 요청에 실패합니다.

이러한 방식으로 제한시간을 조정할 때 흥미로운 점은 경로를 사용하는 모든 요청에 제한시간이 적용된다는 것입니다. 이것은 Istio가 제공하는 기본적인 안전 계층입니다. 애플리케이션 프레임워크나 라이브러리가 제한시간을 적용하지 않더라도, Istio가 제한시간을 적용하기 때문에 계속 대기하지 않습니다. 애플리케이션 레벨 제한시간은 여전히 적용됩니다. 위 예제에서는 주식 시세 서비스에 대한 인프라 레벨 제한시간이 30초로 확장되었습니다. 애플리케이션 라이브러리가 제한시간을 5초로 설정하는 경우 애플리케이션의 요청은 여전히 제한시간 초과로 실패합니다.

## 대체

애플리케이션은 지원 서비스에 대한 요청이 실패할 때 발생하는 사항을 정의해야 합니다. 몇 가지 옵션이 있지만, 이러한 서비스가 적절한 시기에 응답하지 않을 경우, 단계적으로 성능을 저하시키는 것이 목표입니다. 원격 서비스가 실패하면 요청을 다시 시도하거나 다른 요청을 시도하거나 캐시된 데이터를 대신 리턴할 수 있습니다.

요청을 재시도하는 것이 처음에는 가장 쉬운 대체 메커니즘입니다. 하지만 재시도 요청은 계단식 시스템 장애("재시도 스톰", [썬더링 허드(thundering herd) 문제](https://en.wikipedia.org/wiki/Thundering_herd_problem){: external}의 변형)의 원인이 될 수 있습니다. 애플리케이션 레벨 코드가 시스템 또는 네트워크 상태를 충분히 인식하지 못하며 기하급수적인 백오프 알고리즘은 정확하기 어렵습니다.

Istio는 재시도를 훨씬 더 효과적으로 수행할 수 있습니다. 이미 요청 라우팅에 직접 관여하고 있으며 재시도 정책에 대해 일관되고 언어에 구애받지 않는 구현을 제공합니다. 예를 들어, 주식 시세 서비스에 대해 다음과 같은 정책을 정의할 수 있습니다.

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

이 간단한 구성을 사용하면, Istio 사이드카 프록시 또는 Ingress 게이트웨이를 통해 주식 시세 서비스에 대해 작성된 요청은 각 시도에 대해 5초의 제한시간으로 최대 3회 재시도됩니다. 예를 들어 [추가 라우트 일치 규칙](https://istio.io/docs/reference/config/networking/#HTTPMatchRequest){: external}은 이 재시도 정책을 `GET` 요청으로 더 제한할 수 있습니다.

여기에는 놓치기 쉬운 뉘앙스가 있습니다. 재시도 간격을 지정하지 않았습니다. 사이드카는 재시도 간격을 결정하고 과부하된 서비스의 충돌을 피하기 위한 시도 사이에 의도적으로 "지터(jitter)"를 도입합니다.

## 벌크헤드

선적에서 벌크헤드는 한 칸에 있는 누수로 인해 배 전체가 침몰되는 것을 막는 칸막이입니다. 패턴은 유사한 목적을 달성하기 위해 클라우드 환경에 적용되며 몇 가지 다른 방식으로 수행됩니다.

Java와 같은 다중 스레드 언어의 경우 내부 벌크헤드를 내부적으로 사용하여 큐 또는 세마포어 메커니즘을 통해 원격 리소스에 대한 통신에 리소스가 사용되는 방식을 제한하거나 제어할 수 있습니다.

- 큐를 사용하기 위해 서비스는 특정 스레드 수를 특정 큐와 연관시킵니다. 큐가 가득 차면 모든 요청이 빠르게 실패합니다. Java의 경우, 예를 들어, `BlockingQueue`와 연관된 `ThreadPoolExecutor`가 될 수 있습니다.
- 세마포어 메커니즘은 권한 수를 설정하여 작동합니다. 아웃바운드 요청에는 권한이 필요합니다. 요청이 완료되면 다른 요청을 사용하도록 권한이 해제됩니다.

또한 다음 예제와 유사하게 Istio DestinationRule을 사용하여 서비스 간 벌크헤드를 정의하고 업스트림 서비스의 연결 풀을 제한할 수 있습니다.

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

이 구성은 주식 시세 서비스의 각 인스턴스에 대해 작성되는 동시 연결의 최대 수를 10개로 제한합니다. 30초 내에 연결할 수 없는 서비스는 `503 -- Service Unavailable` 응답을 받습니다. 예를 들어, 이러한 종류의 벌크헤드를 사용하여 컴퓨팅 집약적 서비스가 관리할 수 있는 것보다 더 많은 요청을 수신하지 못하게 할 수 있습니다.

## 회로 차단기

회로 차단기는 장애 발생 시 아웃바운드 요청의 작동을 최적화하는 데 사용됩니다. 반복적으로 응답하지 않는 서비스에 요청을 작성하는 대신, 회로 차단기는 지정된 기간 내에 발생한 장애 수를 관찰합니다. 오류 비율이 임계값을 초과하면 회로 차단기가 회로를 열고 요청에 실패하며, 회로가 다시 닫힐 때까지 모든 후속 요청이 실패합니다.

회로 차단기는 또한 Istio DestinationRule을 사용하여 정의됩니다.

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

이 구성은 다른 서비스가 주식 시세 서비스에 적용하는 요청에 제한을 가합니다. 지정된 `outlierDetection` 트래픽 정책이 각 개별 인스턴스에 적용됩니다. 앞의 구성을 문장으로 표현하면 다음과 같습니다. "5초 동안 3번 실패한 모든 주식 시세 서비스 인스턴스를 5분 이상 방출합니다. 또한 모든 인스턴스가 방출될 수 있습니다." 후자의 `maxEjectionPercent` 설정은 로드 밸런싱과 관련되어 있습니다. Istio는 로드 밸런싱 풀을 유지하고 실패한 인스턴스를 해당 풀에서 방출합니다. 기본적으로 이는 로드 밸런싱 풀에서 사용 가능한 모든 인스턴스의 최대 10%를 방출합니다.

회로 차단과 관련된 다른 메커니즘에 익숙한 사람들에게 Istio는 반쯤 개방된 상태를 가지고 있습니다. 대신 몇 가지 간단한 수학을 적용합니다. 예를 들어 `baseInjectionTime * <number of times it has been ejected>` 동안 인스턴스가 풀에서 방출된 상태로 유지됩니다. 따라서 인스턴스가 일시적인 장애로부터 복구되는 동시에 지속적으로 장애가 발생한 인스턴스는 풀 밖으로 이동할 수 있습니다.

