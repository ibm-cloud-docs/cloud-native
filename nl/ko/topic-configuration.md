---

copyright:
  years: 2019
lastupdated: "2019-02-08"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 구성
{: #configuration}

클라우드 네이티브 애플리케이션은 이식 가능해야 합니다. 동일한 고정 아티팩트를 사용하여 코드를 변경하거나 테스트되지 않은 코드 경로를 실행하지 않고도 로컬 하드웨어에서 클라우드 기반 테스트 및 프로덕션 환경에 이르기까지 여러 환경에 배포할 수 있어야 합니다.
{:shortdesc}

[12개 요소 방법론](https://12factor.net/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")의 3개 요소가 다음 실행에 직접 관련됩니다.  

* 첫 번째 요소는 실행 중인 서비스와 버전화된 코드 베이스 사이의 1대1 상관을 권장합니다. 즉, 변경되지 않고 여러 환경에 배치될 수 있는 버전화된 코드 베이스에서 Docker 이미지와 같은 고정 배치 아티팩트를 생성합니다. 
* 세 번째 요소에서는 고정 아티팩트의 일부여야 하는 애플리케이션별 구성과 배치 시 서비스에 제공해야 하는 환경별 구성 간에 분리를 권장합니다. 
* 10 번째 요소는 모든 환경을 가능한 한 유사하게 유지하는 것을 권장합니다. 환경별 코드 경로는 테스트하기가 어렵고 다른 환경에 배치할 때 실패 위험을 증가시킬 수 있습니다. 이는 또한 지원 서비스에도 적용됩니다. 메모리 내 데이터베이스를 개발하고 테스트하는 경우 동작이 다른 데이터베이스를 사용하므로 테스트, 스테이징 또는 프로덕션 환경에서 예기치 않은 오류가 발생할 수 있습니다. 

## 구성 소스
{: #config-inject}

애플리케이션별 구성은 고정 아티팩트의 일부여야 합니다. 예를 들어 WebSphere Liberty에서 실행되는 애플리케이션은 런타임에 활성화된 바이너리와 서비스를 제어하는 설치된 기능 목록을 정의합니다. 이 구성은 애플리케이션에 따라 다르며 Docker 이미지에 포함되어야 합니다. 또한 Docker 이미지는 실행 환경이 컨테이너를 시작할 때 포트 맵핑을 처리하므로 청취 중이거나 노출된 포트를 정의합니다. 

다른 서비스, 데이터베이스 사용자 또는 리소스 활용도 제한조건과 통신하는 데 사용되는 호스트 및 포트와 같은 환경별 구성은 배치 환경이 컨테이너에 제공합니다. 서비스 구성 및 인증 정보의 관리는 크게 달라질 수 있습니다. 

* Kubernetes는 구성 값(문자열화된 JSON 또는 일반 속성)을 ConfigMaps 또는 시크릿에 저장합니다. 환경 변수 또는 가상 파일 시스템이 마운트되면 컨테이너형 애플리케이션으로 전달될 수 있습니다. 서비스에 사용되는 메커니즘은 배치 메타데이터(Kubernetes YAML 또는 Helm 차트)에 지정되어 있습니다. 
* 로컬 개발 환경은 단순한 키 값 환경 변수를 사용하는 단순화된 변형인 경우가 많습니다. 
* Cloud Foundry는 애플리케이션으로 전달되는 문자열화된 JSON 오브젝트에 구성 특성 및 서비스 바인딩 세부사항을 환경 변수로 저장합니다(예: `VCAP_APPLICATION` 및 `VCAP_SERVICES`).
* 환경별 구성 속성을 저장하고 검색하기 위해 etcd, hishicorp Vault, Netflix Archaius 또는 Spring Cloud config와 같은 지원 서비스를 사용하는 것도 모든 환경에서 선택할 수 있습니다. 

대부분의 경우 애플리케이션은 시작 시 환경별 구성을 처리합니다. 예를 들어 프로세스가 시작된 후에는 환경 변수의 값을 변경할 수 없습니다. 그러나, Kubernetes 및 지원 구성 서비스는 애플리케이션이 구성 업데이트에 동적으로 응답할 수 있는 메커니즘을 제공합니다. 이는 선택적인 기능입니다. 상태 비저장(stateless) 임시 프로세스의 경우 서비스를 재시작하는 것으로 충분할 수 있습니다.
{: note}

많은 언어와 프레임워크는 애플리케이션별 구성과 환경별 구성 모두에서 애플리케이션을 지원하는 표준 라이브러리를 제공하여 애플리케이션의 핵심 논리에 집중하고 이러한 기본 기능을 추출할 수 있도록 지원합니다. 

### 서비스 인증 정보에 대한 작업
{: #portable-credentials}

서비스 구성 및 인증 정보(서비스 바인딩) 관리는 플랫폼마다 다릅니다. Cloud Foundry는 애플리케이션으로 전달되는 문자열화된 JSON 오브젝트에 서비스 바인딩 세부사항을 `VCAP_SERVICES` 환경 변수로 저장합니다. Kubernetes는 서비스 바인딩을 문자열화된 JSON 또는 일반 `ConfigMaps` 또는 `Secrets ` 속성으로 저장하며, 이 속성은 환경 변수로 전달하거나 임시 볼륨으로 마운트될 수 있습니다. 자체 구성이 있는 로컬 개발의 경우 로컬 테스트는 클라우드에서 실행 중인 모든 것의 단순화된 버전인 경우가 많습니다. 환경별 코드 경로 없이 이식 가능한 방식으로 이러한 변형에 대해 작업하는 것은 어려울 수 있습니다. 

Cloud Foundry 및 Kubernetes 환경에서 [서비스 브로커](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 사용하여 지원 서비스에 대한 바인딩을 관리하고 연관된 인증 정보를 애플리케이션의 환경에 삽입할 수 있습니다. 이는 여러 환경에서 동일한 방식으로 인증 정보를 애플리케이션에 제공하지 않을 수 있기 때문에 애플리케이션 이식성에 영향을 미칠 수 있습니다. 

{{site.data.keyword.IBM}}에는 `mappings.json` 파일과 함께 작동하여 애플리케이션이 인증 정보를 검색하는 데 사용하는 키를 가능한 소스의 순서 목록에 맵핑하는 여러 오픈 소스 라이브러리가 있습니다. 다음과 같은 세 가지 검색 패턴 유형을 지원합니다.

* `cloudfoundry`: Cloud Foundry의 서비스(VCAP_SERVICES) 환경 변수에서 값을 검색합니다. 
* `env`: 환경 변수에 맵핑되는 값을 검색합니다. 
* `file`: JSON 파일에서 값을 검색합니다. 

다음 예제 `mappings.json` 파일에서 `cloudant-password`는 애플리케이션 코드가 비밀번호 인증 정보를 검색하는 데 사용하는 키입니다. 언어 특정 라이브러리는 일치가 발견될 때까지 특정 순서로 `searchPatterns` 배열을 통해 반복합니다. 

```json
{
   "cloudant_password": {
      "searchPatterns": [
         "cloudfoundry:$['cloudant'][0].credentials.password",
         "env:cloudant_password",
         "file:/server/localdev-config.json:$.cloudant_password"
      ]
   }
}
```
{: codeblock}

라이브러리는 다음 위치에서 cloudant 비밀번호를 검색합니다. 

* Cloud Foundry `VCAP_SERVICES` 환경 변수의 `['cloudant'][0].credentials.password` JSON 경로
* cloudant_password라는 대소문자를 구분하지 않는 환경 변수
* 언어별 리소스 위치에 보관되는 **`localdev-config.json`** 파일의 **cloudant_password** JSON 필드

자세한 정보는 다음을 참조하십시오.

* [{{site.data.keyword.Bluemix}} Environment for Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [{{site.data.keyword.Bluemix_notm}} Environment for Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")
* [{{site.data.keyword.Bluemix_notm}} Service Bindings For Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")

## Kubernetes의 구성 값
{: #config-kubernetes}

Kubernetes는 환경 변수를 정의하고 해당 값을 지정하는 몇 가지 방법을 제공합니다. 

### 리터럴 값
{: #config-literal}

환경 변수를 정의하는 가장 간단한 방법은 서비스의 `deployment.yaml` 파일을 직접 포함하는 것입니다. 다음 [기본 Kubernetes 예제](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에서는, 값이 여러 환경에서 일관될 때 직접 스펙이 제대로 작동합니다. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```
{: codeblock}

### Helm 변수
{: #config-helm}

앞서 언급한 바와 같이 Helm은 템플리트를 사용하여 차트를 작성하므로 나중에 값을 대체할 수 있습니다. `mychart/templates/pod.yaml` 파일 템플리트의 다음 예제를 사용하여 여러 환경에서 보다 유연하게 이전 예제와 동일한 결과를 얻을 수 있습니다. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: {{ .Values.greeting }}
    - name: DEMO_FAREWELL
      value: {{ .Values.farewell }}
```
{: codeblock}

다음은 `mychart/values.yaml` 파일에 있는 예제입니다. 

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

Helm이 템플리트를 렌더링할 때 다음 출력이 생성됩니다.

```bash
$ helm template mychart
---
# Source: mychart/templates/pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: Hello from the environment
    - name: DEMO_FAREWELL
      value: Such a sweet sorrow
```
{:screen}

  이 두 예제 사이에는 약간의 차이가 있습니다. 첫 번째 예제와 샘플 `values.yaml` 파일에서는 사람이 따옴표를 추가했습니다. YAML의 문자열에는 따옴표가 필요하지 않습니다. Helm이 템플리트를 렌더링하면 따옴표가 생략됩니다.
  {: note}

### ConfigMap
{: #kubernetes-configmap}

ConfigMap은 데이터를 키-값 쌍 세트로 정의하는 고유한 Kubernetes 아티팩트입니다. 이전 예제에 표시된 환경 변수의 ConfigMap은 다음 예제와 유사합니다. 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: envar-demo-config
  labels:
    purpose: demonstrate-envars
data:
  DEMO_GREETING: Hello from the environment
  DEMO_FAREWELL: Such a sweet sorrow
```
{: codeblock}

그런 다음 초기 팟(Pod) 정의가 다음과 같이 ConfigMap의 값을 사용하도록 변경됩니다. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    envFrom:
    - configMapRef:
        name: envar-demo-config
```
{: codeblock}

이제 ConfigMap은 팟과 완전히 별개의 아티팩트입니다. 완전히 다른 라이프사이클을 가질 수 있습니다. 이 경우에 팟(Pod)을 다시 배치하지 않고도 ConfigMap에서 값을 업데이트하거나 변경할 수 있습니다. 또한 개발/테스트/디버그 주기에서 유용할 수 있는 명령행에서 직접 ConfigMap을 업데이트 및 조작할 수 있습니다. 

Helm과 함께 사용할 경우 ConfigMap 선언에서 변수를 사용할 수 있습니다. 이러한 변수는 차트가 배치될 때 일반적으로 해결됩니다.

자세한 정보는 [Kubernetes ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오. 

### 인증 정보 및 시크릿
{: #kubernetes-secrets}

구성은 일반적으로 환경 변수 또는  ConfigMap을 통해 Kubernetes에서 실행되는 컨테이너에 제공됩니다. 어느 경우에나 구성 값은 상당히 빨리 발견될 수 있습니다. 이것이 Kubernetes가 시크릿을 이용하여 민감한 정보를 저장하는 이유입니다.

시크릿은 base64 인코딩된 값을 포함하는 독립 오브젝트입니다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```
{: codeblock}

그런 다음 시크릿을 하나 이상의 팟(Pod) 컨테이너에 마운트된 볼륨의 파일로 사용할 수 있습니다. 

```yaml
containers:
- name: myservice
  image: myimage:latest
  volumeMounts:
  - name: foo
    mountPath: "/etc/foo"
    readOnly: true
volumes:
- name: foo
  secret:
    secretName: mysecret
```
{: codeblock}

이러한 방식으로 볼륨으로 마운트되면 시크릿의 데이터 맵에 있는 각 키는 지정된 `mountPath`: `/etc/foo/username` 및 `/etc/foo/password` 아래의 파일 이름이 됩니다. 

시크릿은 환경 변수를 정의하는 데도 사용될 수 있습니다.

```yaml
containers:
- name: mypod
  image: myimage
  env:
  - name: SECRET_USERNAME
      valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
  - name: SECRET_PASSWORD
    valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```
{: codeblock}

Kubernetes는 사용자를 위해서 base64 디코딩을 수행합니다. 팟에서 실행되는 컨테이너는 환경 변수를 검색할 때 base64 디코딩된 값을 인식합니다. 

ConfigMaps와 마찬가지로, 시크릿은 명령행에서 작성되고 조작될 수 있으며 SSL 인증서를 취급할 때 유용하게 사용할 수 있습니다. 

자세한 정보는 [Kubernetes 시크릿](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")을 참조하십시오. 

<!-- SSL EXAMPLE -->
