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

# 建立 RESTful 微服務
{: #rest-api}

不論是否在微服務架構中，雲端原生應用程式都會產生並耗用 API。部分 API 視為內部或專用 API，部分則視為外部 API。
{:shortdesc}

內部 API 只用於防火牆環境內，讓後端服務彼此通訊。外部 API 呈現消費者的統一進入點，通常是由 {{site.data.keyword.apiconnect_long}} 這類工具所**管理**，可強制速率限制或其他使用限制。這類 API 範例是 [GitHub Developer API](https://developer.github.com/v3/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。它提供一個統一的 API，可一致地使用 HTTP 動詞和回覆碼、分頁行為等等，而不需要公開內部實作詳細資料。此 API 可由一個整合型應用程式支援，也可以由一組微服務支援；該細節不會公開給消費者，這使得 GitHub 可視需要自由發展其內部系統。

## RESTful API 的最佳作法
{: #bps-apis}

REST API 應該使用標準 HTTP 動詞來進行「建立」、「擷取」、「更新」及「刪除」(CRUD) 作業，且需要特別注意作業是否為「等冪」（可放心地重試多次）。

* POST 作業可用來建立或更新資源。POST 作業無法反覆呼叫。例如，如果使用 POST 要求來建立資源，且呼叫多次，則在每次呼叫後都會建立新的唯一資源。
* GET 作業必須能夠反覆呼叫，且不得產生負面影響。它們只能用來擷取資訊。不應使用含查詢參數的 GET 要求來變更或更新資訊。請改用 POST、PUT 或 PATCH 作業。
* PUT 作業可用來更新資源。PUT 作業通常包括要更新的完整資源副本，因此可以多次呼叫作業。
* PATCH 作業容許局部更新資源。根據差異的指定方式及其套用至資源的方式，可反覆予以呼叫。例如，如果 PATCH 作業指出應該將值從 A 變更為 B，則可以反覆地呼叫該值。如果已呼叫多次，且值已是 B，則沒有任何效果。
* DELETE 作業可以多次呼叫，因為資源只能刪除一次。不過，回覆碼會改變，因為第一個作業成功（`200` 或 `204`），而後續的呼叫不會尋找資源（`404` 或 `410`）。

### 對機器友善的敘述性結果
{: #rest-results}

鑒於 API 是由軟體而非人類呼叫，因此以最有效且具效率的方式將資訊傳達給呼叫程式時應該盡量小心。

使用相關和有用的 HTTP 狀態碼，如下表所述： 

| HTTP 錯誤碼 | 用法指引 |
|-----------------|----------------|
| `200 (OK)` | 一切正常且有資料要傳回時使用 |
| `204 (NO CONTENT)` | 一切正常但沒有回應資料時使用 |
| `201 (CREATED)` | 用於導致建立資源的 POST 要求，指出是否有回應內文 |
| `409 (CONFLICT)` | 並行變更衝突時使用 |
| `400 (BAD REQUEST)` | 參數的格式錯誤時使用 |

如需相關資訊，請參閱[回應狀態碼](https://tools.ietf.org/html/rfc7231#section-6){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。 

您還應該考量在回應中傳回哪些資料，以提高通訊效率。例如，使用 POST 要求建立資源時，回應應該在 Location 標頭中包括新建立資源的位置。建立的資源通常也會內含在回應內文中，因此不需要使用額外的 GET 要求來提取建立的資源。這同樣適用於 PUT 和 PATCH 要求。

### RESTful 資源 URI
{: #rest-uris}

關於 RESTful 資源 URI 的某些方面，存在不同的意見。一般而言，一致認為資源應該是名詞，而非動詞，且端點應該是複數。這為 CRUD 作業提供一個清楚的結構：

* `POST /accounts`：建立新的帳戶。
* `GET /accounts`：擷取帳戶清單。
* `GET /accounts/16`：擷取特定帳戶。
* `PUT /accounts/16`：更新特定帳戶。
* `PATCH /accounts/16`：更新特定帳戶。
* `DELETE /accounts/16`：刪除特定帳戶。

關係是使用階層式 URI 建模的，例如，`/accounts/16/credentials` 用於管理與帳戶相關聯的認證。

對於與不符合此常用結構的資源相關聯的作業應該發生什麼狀況，一般的共識較少。管理這些作業沒有單一正確的方法：請進行最適合 API 消費者的作業。

### 穩健性和 RESTful API
{: #robust-api}

[穩健性原則](https://tools.ietf.org/html/rfc1122#page-12){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 提供最佳指引：「對於接受的內容請保持開放態度，但在傳送的內容方面請保守」。假設 API 將隨著時間推移而發展，並且能夠容忍您不理解的資料。

#### 產生 API
{: #robust-producer}

向外部用戶端提供 API 時，您必須在接受要求及傳回回應時執行兩個動作。 

* 接受不明屬性作為要求的一部分。
    > 如果服務以不必要的屬性呼叫您的 API，則只會擲出這些值。在此情況下，傳回錯誤可能導致不必要的失敗，對一般使用者會造成負面影響。
* 只傳回消費者所需的屬性
    > 請避免公開內部服務詳細資料。只會將消費者所需的屬性公開為 API 的一部分。

#### 耗用 API
{: #robust-consumer}

只根據您需要的變數或屬性來驗證要求。

    > 不要因為提供了變數就對它們進行驗證。如果您未使用它們作為要求的一部分，則請不要根據它們。

接受不明屬性作為回應的一部分。

    > 如果您收到非預期的變數，請不要發出異常狀況。只要回應包含您需要的資訊，就不需要在意伴隨的其他項目。

這些準則特別適用於 Java 這類高度類型化語言，其中，JSON 序列化和解除序列化通常會透過 Jackson 程式庫或 JSON-P/JSON-B（舉例來說）間接發生。請尋找語言機制，以容許您指定忽略不明屬性這類更寬鬆的行為，或是定義或過濾應該序列化的屬性。

### 版本化 RESTful API
{: #version-api}

微服務的其中一個主要好處是容許服務獨立發展。鑒於微服務會呼叫其他服務，這類獨立性伴隨著一個巨大警告：您不能在 API 中導致重大變更。

如果遵循穩健性原則，則可能要很長的時間才能進行重大變更。最後進行該重大變更時，您可以選擇建置完全不同的服務，並隨著時間推移而淘汰原始服務。

如果您確實需要對現有服務進行重大 API 變更，請決定如何管理這些變更：該服務是否處理 API 的所有版本、您是否維護服務的獨立版本以支援每個 API 版本，或是您的服務是否僅支援 API 的最新版本並根據其他調適性層來與舊 API 進行轉換？

在您判定如何管理變更之後，更容易解決的問題是如何在 API 中反映版本。通常有三種方式可以設定 REST 資源的版本：

* 將版本包括在 URI 路徑中。
* 將版本包括在 HTTP Accept 標頭中，並根據內容協議。
* 使用自訂要求標頭。

#### 將版本包括在 URI 路徑中
{: #version-path}

指定版本的最簡單方式是將它包括在 URI 路徑中。此方式有多項優點：很明顯、在應用程式中建置服務時很容易實現，而且與 Swagger 這類 API 瀏覽工具和 `curl` 這類指令行工具等相容。

如果您要將版本包括在 URI 路徑中，則版本應該整個套用至您的應用程式，例如，`/api/v1/accounts`，而非 `/api/accounts/v1`。「超媒體即應用程式狀態引擎 (HATEOAS)」是將 URI 提供給 API 消費者的一種方式，因此並不負責建構 URI 本身。例如，GitHub 基於此原因會在回應中提供[超媒體 URL](https://developer.github.com/v3/\#hypermedia){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。如果不同的後端服務可以在其 URI 中具有獨立不同的版本，則 HATEOAS 會變得難以實現（如果不是不可能的話）。

#### 修改 Accept 標頭以包括版本
{: #version-accept}

Accept 標頭是定義版本的明顯位置，但卻是其中一個最難測試的位置。Accept 標頭也是常見著手進行特性切換的位置。指定 HTTP 標頭需要更詳細的 API 呼叫。

#### 新增自訂要求標頭
{: #version-custom}

您可以新增自訂要求標頭來指出 API 版本。與 Accept 標頭相同，您也可以使用自訂標頭，將資料流量遞送至特定的後端實例。使用此方法時，您會遇到與使用 Accept 標頭方法時遇到的相同易用性問題，而且消費者需要瞭解此標頭的其他需求。

如需相關資訊，請參閱 [API 版本化錯誤，這是我決定以 3 種不同錯誤方式執行作業的原因](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

## 建立及產生 API
{: #create-api}

[OpenAPI 第 3 版](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是 RESTful 服務的正式規格，由 [OpenAPI Initiative](https://www.openapis.org/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")（Linux 基金會下的同業公會）控管。

您可以使用下列任一方法來建立 API：

  * 從 OpenAPI 定義開始（由上而下）：使用此方式，您可以從使用與語言無關的格式（通常是 YAML）來建立 OpenAPI 定義開始。然後，您可以使用程式碼產生器來建立架構，並從中建置服務實作。此型樣通常由具有中央 API 設計小組的公司所採用，容許開發及測試平行進行。
  * 從程式碼開始（由下而上）：您的程式碼是 API 定義的來源。此方式適用於具有實驗層面的新應用程式，因為您的 API 定義會隨著您更充分瞭解服務需求而發展。此方式在某些語言中的效果也會比其他語言更好，因為它根據從程式碼產生 OpenAPI 定義的工具。例如，Java 極適合支援從註釋型 REST 架構產生 OpenAPI 文件。

在任一情況下，使用 OpenAPI 定義可協助識別 API 與消費者觀點不一致或難以理解的區域。建置工具也可以使用已發佈或版本控制的 OpenAPI 定義，來協助標示可能影響消費者的重大變更。

### 從 OpenAPI 定義建立 API
{: #openapi-first}

您可以使用選擇的任何工具來編寫 OpenAPI YAML 檔案。不過，使用純文字編輯器可能容易發生錯誤。部分編輯器對 YAML 具有基本支援，且部分編輯器可能具有支援 OpenAPI 定義的其他延伸。例如，您可以使用 [Swagger Viewer](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 或 [OpenAPI Preview](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 這類 Visual Studio Code 延伸，來驗證指定規格版本的 OpenAPI 定義，並在預覽窗格中呈現 Web 視圖：

![OpenAPI 預覽](images/create-api-image1.png "OpenAPI 預覽"){: caption="圖 1. OpenAPI 預覽" caption-side="bottom"}  

您也可以在線上或本端使用各種瀏覽器型線上剖析編輯器。以下是部分範例：

* [OpenAPI-GUI 專案](https://github.com/Mermade/openapi-gui){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 同時支援 OpenAPI 規格的第 2 版和第 3 版，並且可以將 OpenAPI 第 2 版定義移轉至第 3 版。
* [SmartBear 的 Swagger Editor](https://editor.swagger.io){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 也同時支援 OpenAPI 的第 2 版和第 3 版。
* [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 提供一組適用於 API 建模及建立的編輯器和工具。

### 產生 API 實作
{: #code-first}

您可以使用開放程式碼 [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，從 OpenAPI 定義建立服務實作的架構專案。您可以從指令行指定架構的語言或架構。例如，若要為範例 PetStore API 建立使用通用 JAX-RS 方法註釋的 Java 專案，您可以指定下列指令：

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

