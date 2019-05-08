---

copyright:
  years: 2019
lastupdated: "2019-03-26"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# RESTful 마이크로서비스 작성
{: #rest-api}

클라우드 네이티브 애플리케이션은 마이크로서비스 아키텍처에 있든 없든 API를 생성하고 이용합니다. 일부 API는 내부 또는 사설 API로 간주되고 일부는 외부 API로 간주됩니다.
{:shortdesc}

내부 API는 백엔드 서비스가 서로 통신할 수 있도록 방화벽을 갖춘 환경 내에서만 사용됩니다. 외부 API는 이용자를 위한 통합 진입점을 제공하며, 종종 비율 제한이나 기타 사용 제한을 가할 수 있는 {{site.data.keyword.apiconnect_long}}와 같은 도구로 **관리**됩니다. 이와 같은 API의 예에는 [GitHub Developer API](https://developer.github.com/v3/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")가 있습니다. 내부 구현 세부 정보를 노출하지 않고 HTTP verb와 리턴 코드, 페이지 매김 동작 등을 일관되게 사용하는 단일 통합 API를 제공합니다.  이 API는 한 개의 단일 애플리케이션으로 지원되거나 마이크로서비스 모음으로 지원될 수 있습니다. 이 세부사항은 이용자에게 노출되지 않으므로 GitHub는 필요에 따라 내부 시스템을 자유롭게 발전시킬 수 있습니다. 

## RESTful API 우수 사례
{: #bps-apis}

REST API는 CRUD(Create, Retrieve, Update, Delete) 조작에 표준 HTTP verb를 사용해야 하며, 작업이 유휴 상태인지 여부에 특히 주의해야 합니다(여러 번 재시도할 수 있음).  

* POST 오퍼레이션을 사용하여 리소스를 작성하거나 업데이트할 수 있습니다. POST 오퍼레이션은 반복적으로 호출할 수 없습니다. 예를 들어 POST 요청이 리소스를 생성하는 데 사용되고 여러 번 호출되는 경우 각 호출의 결과로 고유한 새 리소스가 생성됩니다. 
* GET 오퍼레이션은 반복적으로 호출할 수 있어야 하며 부작용을 유발하지 않아야 합니다. 정보를 검색하는 데만 사용해야 합니다. 조회 매개변수를 사용한 GET 요청을 사용하여 정보를 변경하거나 갱신할 수 없습니다. 대신 POST, PUT 또는 PATCH 오퍼레이션을 사용합니다. 
* PUT 오퍼레이션을 사용하여 리소스를 업데이트할 수 있습니다. 일반적으로 PUT 오퍼레이션에는 업데이트할 리소스의 전체 사본이 포함되어 있으므로 오퍼레이션을 여러 번 호출할 수 있습니다. 
* PATCH 오퍼레이션을 통해 리소스를 부분적으로 업데이트할 수 있습니다. 델타 지정 방법에 따라 반복적으로 호출한 다음 리소스에 적용할 수 있습니다. 예를 들어 PATCH 오퍼레이션이 값을 A에서 B로 변경해야 함을 나타내는 경우 반복해서 호출할 수 있습니다. 여러 번 호출되고 값이 이미 B인 경우에는 아무런 영향이 없습니다. 
* 리소스는 한 번만 삭제할 수 있으므로 DELETE 오퍼레이션은 여러 번 호출할 수 있습니다. 하지만 첫 번째 오퍼레이션이 성공하므로 리턴 코드가 달라지고(`200` 또는 `204`), 후속 호출에서 리소스를 찾지 못합니다(`404` 또는 `410`).

### 기계 친화적이고 설명적인 결과
{: #rest-results}

인간이 아닌 소프트웨어가 API를 호출하는 것을 고려하면, 가능한 한 가장 효과적이고 효율적인 방법으로 정보를 호출자에게 전달하도록 주의해야 합니다. 

다음 표에 설명된 대로 관련되고 유용한 HTTP 상태 코드를 사용하십시오.  

| HTTP 오류 코드 |사용법 안내 |
|-----------------|----------------|
| `200 (OK)` | 모든 것이 정상이고 리턴할 데이터가 있을 때 사용합니다. |
| `204 (NO CONTENT)` | 모든 것이 정상이지만 응답 데이터가 없는 경우 사용합니다. |
| `201 (CREATED)` | 응답 본문이 있는지 여부에 관계없이 리소스를 생성하게 되는 POST 요청에 사용합니다. |
| `409 (CONFLICT)` | 동시 변경이 충돌할 때 사용합니다. |
| `400 (BAD REQUEST)` | 매개변수가 잘못 구성되었을 때 사용합니다.|

자세한 정보는 [응답 상태 코드](https://tools.ietf.org/html/rfc7231#section-6){: new_window} ![외부 링크아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 참조하십시오.  

또한 응답에서 어떤 데이터를 리턴해야 통신의 효율을 높일 수 있는지 고려해야 합니다. 예를 들어 POST 요청을 사용하여 리소스를 생성하는 경우 응답의 위치 헤더에 새로 생성된 리소스의 위치가 포함되어야 합니다. 생성된 리소스를 가져오는 추가 GET 요청을 제거하기 위해 생성된 리소스가 응답 본문에도 포함되는 경우가 많습니다. PUT 및 PATCH 요청도 마찬가지입니다.

### RESTful 리소스 URI
{: #rest-uris}

RESTful 리소스 URI의 일부 측면에 대해서 다양한 의견이 있습니다. 일반적으로 리소스는 동사가 아니라 명사여야 하고, 끝점은 복수여야 한다는 합의가 있습니다. 따라서 CRUD 오퍼레이션을 위한 구조가 명확해집니다. 

* `POST /accounts`: 새 계정을 작성합니다. 
* `GET /accounts`:계정 목록을 검색합니다. 
* `GET /accounts/16`: 특정 계정을 검색합니다. 
* `PUT /accounts/16`: 특정 계정을 업데이트합니다. 
* `PATCH /accounts/16`: 특정 계정을 업데이트합니다. 
* `DELETE /accounts/16`: 특정 계정을 삭제합니다. 

계정과 관련된 인증 정보를 관리하기 위한 계층적 URI(예: ` /accounts/16/credentials`)를 사용하여 관계를 모델링합니다. 

이 일반적 구조 내에 맞지 않는 리소스와 연관된 오퍼레이션에서 수행할 작업에 대한 합의가 줄어듭니다. 이러한 오퍼레이션을 관리할 수 있는 유일한 올바른 방법은 없습니다. API의 이용자에게 가장 적합한 작업을 수행하면 됩니다. 

### 견고성 및 RESTful API
{: #robust-api}

[견고성 원칙(Robustness Principle)](https://tools.ietf.org/html/rfc1122#page-12){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")은 "수락하는 것에는 자유롭고, 보내는 것에는 보수적이어야 한다"는 최고의 지침을 제공합니다. API가 시간이 지남에 따라 발전하고 사용자가 이해하지 못하는 데이터를 허용하게 될 것이라고 가정하십시오. 

#### API 생성
{: #robust-producer}

외부 클라이언트에 API를 제공하는 경우 요청을 수락하고 응답을 리턴할 때 반드시 수행해야 하는 두 가지가 있습니다.  

* 알 수 없는 특성을 요청의 일부로 수락합니다.
    > 서비스에서 불필요한 속성을 사용하여 API를 호출하는 경우 이러한 값을 버립니다. 이 시나리오에서 오류를 리턴하면 불필요한 장애가 발생하여 일반 사용자에게 부정적인 영향을 미칠 수 있습니다. 
* 이용자가 요구하는 속성만 리턴합니다.
> 내부 서비스 세부 정보가 노출되지 않도록 합니다. API의 일부로 이용자에게 필요한 속성만 노출합니다. 

#### API 이용
{: #robust-consumer}

필요한 변수 또는 속성에 대한 요청만 유효성 검증하십시오.

    > 단지 변수가 제공되었다고 해서 변수에 대해 유효성을 검증하지 않습니다. 이를 사용자 요청의 일부로 사용하지 않는 경우에는 해당 요청에 의존하지 마십시오.

알 수 없는 특성을 응답의 일부로 수락합니다. 

    > 예기치 않은 변수를 수신하는 경우에는 예외를 발행하지 마십시오. 응답에 필요한 정보가 포함되어 있는 한, 실행에 필요한 다른 정보는 중요하지 않습니다. 

이러한 지침은 JSON 직렬화 및 직렬화 해제가 JSON 라이브러리 또는 JSON-P/JSON-B와 같이 자주 간접적으로 발생하는 Java와 같은 강력한 유형의 언어와 특히 관련이 있습니다. 알 수 없는 속성 무시와 같은 보다 관대한 동작을 지정하거나, 직렬화할 속성을 정의하거나 필터링할 수 있는 언어 메커니즘을 찾으십시오. 

### RESTful API 버전화
{: #version-api}

마이크로서비스의 주요 이점 중 하나는 서비스를 독립적으로 발전시킬 수 있는 능력입니다. 마이크로서비스에서 다른 서비스를 호출하는 경우 독립성은 중대한 경고와 함께 나타납니다: API에 중단적 변경을 일으킬 수 없습니다. 

견고성 원칙을 준수할 경우 중단적 변경이 필요할 때까지 오랜 시간이 걸릴 수 있습니다. 마침내 이러한 중단적 변경이 도래하면 완전히 다른 서비스를 구축하여 시간이 지남에 따라 원래의 서비스를 폐기할 수 있습니다. 

기존 서비스에 대해 API를 변경해야 하는 경우, 이러한 변경을 관리하는 방법을 결정하십시오. 서비스가 모든 버전의 API를 처리할지, 각 버전의 API를 지원하기 위해 서비스의 독립 버전을 유지할지, 아니면 서비스가 최신 버전의 API만 지원하고, 이전 API에서 변환하거나 이전 API로 변환할 수 있는 다른 적응형 계층에 의존할지를 결정합니다. 

변경을 관리하는 방법을 결정한 후에는 훨씬 쉽게 해결할 수 있는 문제가 API에 버전을 반영하는 방법입니다. 일반적으로 REST 리소스를 버전화하는 방법은 세 가지가 있습니다. 

* URI 경로에 버전을 포함합니다. 
* HTTP Accept 헤더에 버전을 포함시키고 컨텐츠 조정에 의존합니다. 
* 사용자 정의 요청 헤더를 사용합니다. 

#### URI 경로에 버전 포함
{: #version-path}

버전을 지정하는 가장 쉬운 방법은 URI 경로에 버전을 포함하는 것입니다. 이러한 접근 방식에는 이점이 있습니다. 애플리케이션에서 서비스를 빌드할 때 쉽게 달성할 수 있으며, Swagger와 같은 API 탐색 도구 및 `curl` 등과 같은 명령행 도구와 호환됩니다. 

URI 경로에 버전을 포함하려는 경우 버전은 애플리케이션 전체에 적용해야 합니다. 예를 들어 `/api/accounts/v1`가 아니라 `/api/v1/accounts`를 지정해야 합니다. HATEOAS(Hypermedia as the Engine of Application State)는 API 이용자에게 URI를 제공하는 한 가지 방법으로, URI를 스스로 생성할 책임이 없습니다. 예를 들어 GitHub는 이런 이유로 응답에서 [하이퍼미디어 URL](https://developer.github.com/v3/\#hypermedia){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")을 제공합니다. 서로 다른 백엔드 서비스가 해당 URI에서 독립적으로 다른 버전을 가질 수 있는 경우, HATEOAS는 불가능하지는 않더라도 달성하기가 어려워집니다.

#### 버전을 포함하도록 Accept 헤더 수정
{: #version-accept}

Accept 헤더는 버전을 정의할 수 있는 명백한 위치이지만 테스트하기가 가장 어렵습니다. Accept 헤더는 기능 토글에 대한 시작 위치가 되는 경우가 많습니다. HTTP 헤더를 지정하려면 보다 자세한 API 호출이 필요합니다. 

#### 사용자 정의 요청 헤더 추가
{: #version-custom}

사용자 정의 요청 헤더를 추가하여 API 버전을 표시할 수 있습니다. Accept 헤더와 마찬가지로 사용자 정의 헤더를 사용하여 트래픽을 특정 백엔드 인스턴스로 라우트할 수도 있습니다. 이 방법을 사용하면 이용자가 이 헤더에 대해 알아야 하는 추가 요구사항과 함께 Accept 헤더 방법에 대해 발생하는 것과 동일한 사용의 용이성 문제가 발생합니다.

자세한 정보는 [사용자의 API 버전화가 잘못되었기 때문에 3가지 다른 잘못된 방식으로 작업을 수행하기로 결정](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")의 내용을 참조하십시오.

## API 작성 및 생성
{: #create-api}

[OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 Linux Foundation 산하 기업의 협회인 [OpenAPI Initiative](https://www.openapis.org/){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에서 관리하는 RESTful 서비스의 공식 스펙입니다. 

다음 방법 중 하나를 사용하여 API를 작성할 수 있습니다.

  * OpenAPI 정의로 시작(하향식): 이 접근 방식에서는, 언어 독립적 형식(일반적으로 YAML)에서 OpenAPI 정의를 작성하는 것으로 시작합니다. 그런 다음 코드 생성기를 사용하여 스켈레톤을 작성하고 여기에서 서비스 구현을 빌드합니다. 이 패턴은 일반적으로 중앙 API 설계 팀이 있는 회사에서 채택되며 개발 및 테스트가 병렬로 진행될 수 있게 합니다. 
  * 코드로 시작(상향식): 코드는 API 정의의 소스입니다. 이 접근 방식은 서비스가 수행해야 하는 작업에 대해 더 잘 이해할 수 있으므로 API 정의가 발전함에 따라 실험적인 측면이 있는 새 애플리케이션에 적합합니다. 또한 이러한 접근 방식은 코드에서 OpenAPI를 생성하는 도구에 의존하기 때문에 어떤 언어에서는 다른 언어보다 더 효과적입니다.  예를 들어, Java는 어노테이션 기반 REST 프레임워크에서 OpenAPI 문서를 생성하는 데 매우 적합합니다. 

어느 경우에나 OpenAPI 정의에 대한 작업은 API가 일관되지 않거나 이용자 관점에서 이해하기 어려운 영역을 식별하는 데 도움이 될 수 있습니다. 또한, 공개되거나 버전 제어되는 OpenAPI 정의를 빌드 도구에서 사용하여 이용자에게 영향을 주는 중단적 변경을 플래그 지정하도록 지원합니다. 

### OpenAPI 정의에서 API 작성
{: #openapi-first}

선택한 도구가 무엇이든 이를 사용하여 OpenAPI YAML 파일을 작성할 수 있습니다. 단, 일반 텍스트 편집기를 사용하면 오류가 발생할 수 있습니다. 일부 편집기에는 YAML에 대한 기본적인 지원이 있으며 일부 편집기에는 OpenAPI 정의를 지원하기 위한 추가 확장기능이 있을 수 있습니다. 예를 들어 Visual Studio Code 확장기능(예: [Swagger 뷰어](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 또는 [OpenAPI 미리보기](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 사용하여 특정 스펙 버전에 대해 OpenAPI 정의를 유효성 검증하고 미리보기 분할창에서 웹 보기를 렌더링할 수 있습니다. 

![OpenAPI 미리보기](images/create-api-image1.png "OpenAPI 미리보기"){: caption="그림 1. OpenAPI 미리보기" caption-side="bottom"} 

또한 온라인 또는 로컬에서 사용할 수 있는 다양한 브라우저 기반의 활성 구문 분석 편집기도 있습니다. 다음은 일부 예입니다.

* [OpenAPI-GUI 프로젝트](https://github.com/Mermade/openapi-gui){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")는 v2 및 v3의 OpenAPI 스펙을 모두 지원하며 OpenAPI v2 정의를 v3으로 마이그레이션할 수 있게 해줍니다. 
* [SmartBear의 Swagger 편집기](https://editor.swagger.io){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘") 역시 v2 및 v3의 OpenAPI를 지원합니다. 
* [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")에서는 API 모델링을 위한 일련의 편집기와 도구를 제공합니다.  

### API 구현 생성
{: #code-first}

오픈 소스 [OpenAPI 생성기](https://github.com/OpenAPITools/openapi-generator){: new_window} ![외부 링크 아이콘](../icons/launch-glyph.svg "외부 링크 아이콘")를 사용하여 OpenAPI 정의에서 서비스 구현을 위해 스켈레톤 프로젝트를 작성할 수 있습니다. 명령행에서 스켈레톤에 대한 언어 또는 프레임워크를 지정할 수 있습니다. 예를 들어 일반 JAX-RS 메소드 어노테이션을 사용하는 샘플 PetStore API에 대한 Java 프로젝트를 작성하기 위해 다음을 지정합니다. 

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

