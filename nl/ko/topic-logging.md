---

copyright:
  years: 2019
lastupdated: "2019-02-10"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 로깅
{: #logging}

로그 메시지는 로그 항목이 작성될 때 마이크로서비스의 상태 및 활동과 관련된 컨텍스트 정보를 포함하는 문자열입니다. 서비스가 실패하는 방법과 이유를 진단하려면 로그가 필요합니다. 로그는 애플리케이션 상태를 모니터할 때 메트릭에 대한 지원 역할을 합니다.
{:shortdesc}

로그 항목을 표준 출력 및 오류 스트림에 직접 기록해야 합니다. 이렇게 하면 명령행 도구를 사용하여 로그 항목을 볼 수 있고 Logstash, Filebeat 또는 Fluentd와 같은 인프라 레벨에서 구성된 로그 전달 서비스를 사용하여 로그 콜렉션과 데이터를 관리할 수 있습니다. 

표준 출력 또는 표준 오류에 로그를 쓰도록 컨테이너형 애플리케이션을 구성할 수 없는 경우 로그를 처리하려면 더 많은 것을 고려해야 합니다. 

* 한 가지 옵션은 로그 데이터의 볼륨을 사용하는 것입니다. 즉, 로컬 개발 및 테스트를 위한 단순한 바인딩 마운트 또는 Kubernetes 배치의 일부로 적절한 지속적 볼륨을 사용하는 것입니다. [전용 사이드카 또는 로깅 에이전트](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 공유 볼륨에서 로그를 읽고 중앙 집중식 집계기에 로그를 전달할 수 있습니다. 볼륨에 저장된 로그 데이터의 양을 제어하도록 로그 회전을 명시적으로 구성해야 합니다. 
* 다른 옵션은 집계기에 직접 로그를 전달하기 위해 애플리케이션 라이브러리 또는 에이전트를 사용하는 것입니다. 이 옵션을 사용하면 배치 환경 전반에서 몇 가지 구성이 복잡해질 수 있습니다. 

## JSON 로깅
{: #json-logging}

시간이 지나면서 애플리케이션이 진화함에 따라 로그의 속성이 변경될 수 있습니다. JSON 로그 형식을 사용하여 다음과 같은 이점을 얻을 수 있습니다.

* 로그를 색인화화할 수 있으므로 로그의 집계된 본문을 훨씬 쉽게 검색할 수 있습니다. 
* 구문 분석이 문자열의 요소 위치에 의존하지 않기 때문에 로그가 변경에 대해 탄력적입니다. 

JSON 형식 로그는 기계가 구문 분석하기가 더 쉽지만, 사람이 읽기는 더 어렵습니다. 로컬 개발 및 디버깅을 위한 일반 텍스트와 장기 저장 및 집계를 위한 JSON 형식의 로그 간에 로그 형식을 전환하기 위해 환경 변수를 사용하는 것이 좋습니다.

JSON 조회 도구(jq)와 같은 명령행 JSON 구문 분석기를 사용하여 JSON 형식 로그를 사용자가 읽을 수 있는 보기로 생성할 수 있습니다. 다음 예제에서는 jq가 라인을 구문 분석하기 전에 메시지 필드가 있는지 확인하기 위해 grep을 통해 로그가 연결됩니다. 

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## `kubectl`을 사용하여 로그 보기
{: #view-logs-kube}

표준 출력 및 오류 스트림들에 전송되는 로그는 콘솔을 통해 또는 `kubectl logs <podname>` 형식의 `kubectl` 명령을 통해 Kubernetes에서 볼 수 있습니다. 

stock-trader와 같은 사용자 정의 네임스페이스를 사용하는 경우 `kubectl logs -n stock-trader<podname>`과 같이 명령에 이를 포함하십시오. 

istio 사이드카를 사용할 때와 같이 각 팟에 대해 여러 개의 컨테이너가 있는 경우 해당 컨테이너도 지정해야 합니다. 다음 예제에서는 stock-trader 네임스페이스를 사용하여 `portfolio-54b4d579f7-4zvzk` 팟(Pod)의 `portfolio` 컨테이너에서 로그를 봅니다. 

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

JSON 형식의 로그인 경우, `jq`를 사용하여 메시지 필드를 추출할 수 있습니다. 예를 들어, 다음과 같습니다.

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

로그 항목은 `jq`에서 메시지 필드가 포함된 행만 구문 분석하도록 `grep`을 통해 연결됩니다.
{: note}
