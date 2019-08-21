---

copyright:
  years: 2019
lastupdated: "2019-07-18"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 配置
{: #configuration}

雲端原生應用程式必須為可攜式。您可以使用相同的固定構件來部署至多個環境，而不需要變更程式碼或執行未測試的程式碼路徑。
{:shortdesc}

[十二因素方法](https://12factor.net/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 中有三個因素直接與此作法相關：

* 第一個因素建議執行中服務與已版本化程式碼庫之間的 1 對 1 相關性。從可在未進行變更的情況下部署至多個環境的已版本化程式碼庫，建立 Docker 映像檔這類固定部署構件。
* 第三個因素建議分隔屬於固定構件一部分的應用程式特定配置與部署時提供給服務的環境特定配置。
* 第十個因素建議盡可能將所有環境都保留為類似。環境特定程式碼路徑很難測試，並在您部署至不同環境時增加失敗的風險。這也適用於支援服務。如果您使用記憶體內資料庫進行開發及測試，則在測試、編譯打包或正式作業環境中可能會發生非預期的失敗，因為它們使用的資料庫具有不同的行為。

## 配置來源
{: #config-inject}

應用程式特定配置是固定構件的一部分。例如，在 WebSphere Liberty 上執行的應用程式會定義一份已安裝特性清單，這些特性可控制運行環境內的作用中二進位檔和服務。此配置是應用程式所特有，並且包括在 Docker 映像檔中。Docker 映像檔也會定義接聽埠或已公開埠，因為執行環境會在啟動容器時處理埠對映。 

部署環境會將環境特定配置提供給容器，例如用來與其他服務、資料庫使用者或資源使用率限制通訊的主機和埠。服務配置和認證的管理可能會有明顯的差異：

* Kubernetes 將配置值（字串化 JSON 或平面屬性）儲存在 ConfigMaps 或 Secrets 中。這些可以當作環境變數或虛擬檔案系統裝載傳遞至容器化應用程式。服務所使用的機制指定於部署 meta 資料中，格式為 Kubernetes YAML 或 Helm Chart。
* 本端開發環境通常是使用簡單鍵值環境變數的簡化變式。
* Cloud Foundry 會將配置屬性和服務連結詳細資料儲存在當作環境變數傳遞至應用程式的字串化 JSON 物件中，例如，`VCAP_APPLICATION` 和 `VCAP_SERVICES`。
* 使用 etcd、hashicorp 儲存庫、Netflix Archaius 或 Spring Cloud 配置這類支援服務來儲存及擷取環境特定配置屬性，也是任何環境中的一個選項。

在大部分情況下，應用程式會在開始時處理環境特定配置。例如，在處理程序啟動之後，就無法變更環境變數的值。不過，Kubernetes 和支援配置服務提供機制，讓應用程式動態回應配置更新。這是選用功能。如果是無狀態的暫時性處理程序，則通常重新啟動服務就已足夠。

許多語言和架構都會提供標準程式庫來輔助應用程式特定配置和環境特定配置中的應用程式，因此您可以專注於應用程式的核心邏輯，以及摘要這些基本功能。

### 使用服務認證
{: #portable-credentials}

服務配置和認證（服務連結）的管理在平台之間有所不同。Cloud Foundry 會將服務連結詳細資料儲存在當作 `VCAP_SERVICES` 環境變數傳遞至應用程式的字串化 JSON 物件中。Kubernetes 會將服務連結儲存為字串化 JSON 或平面 `ConfigMaps` 或 `Secrets` 屬性，這些屬性可以當作環境變數傳遞至容器化應用程式，或裝載為暫時磁區。如果是具有其專屬配置的本端開發，則本端測試通常是雲端中執行項目的簡化版本。在沒有環境特定程式碼路徑的情況下，以可攜式方式在這些變動因素下作業，相當具有挑戰性。

在 Cloud Foundry 和 Kubernetes 環境中，您可以使用[服務分配管理系統](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 來管理連結至支援服務，並將關聯的認證注入應用程式的環境。這可能會影響應用程式可攜性，因為可能無法在不同環境中使用相同的方式將認證提供給應用程式。

{{site.data.keyword.IBM}} 有數個使用 `mappings.json` 檔案的開放程式碼庫，可將應用程式用來擷取認證資訊的索引鍵對映至已排序的可能來源清單。它支援三種搜尋型樣類型：

* `cloudfoundry`：搜尋 Cloud Foundry 服務 (VCAP_SERVICES) 環境變數中的值。
* `env`：搜尋對映至環境變數的值。
* `file`：搜尋 JSON 檔案中的值。

在下列範例 `mappings.json` 檔案中，`cloudant-password` 是應用程式碼用來查閱密碼認證的金鑰。語言特定程式庫會依特定順序反覆運算 `searchPatterns` 陣列，直到找到相符項為止。

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

程式庫會在下列位置中搜尋 cloudant 密碼：

* Cloud Foundry `VCAP_SERVICES` 環境變數中的 `['cloudant'][0].credentials.password` JSON 路徑。
* 名為 cloudant_password 且不區分大小寫的環境變數。
* 保留在語言特定資源位置內的 **`localdev-config.json`** 檔案中的 **cloudant_password** JSON 欄位。

如需相關資訊，請參閱：

* [{{site.data.keyword.Bluemix}} Environment for Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [{{site.data.keyword.Bluemix_notm}} Environment for Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")
* [適用於 Spring Boot 的 {{site.data.keyword.Bluemix_notm}} 服務連結](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")

## Kubernetes 中的配置值
{: #config-kubernetes}

Kubernetes 提供數種不同的方式來定義環境變數，並指派其值。

### 文字值
{: #config-literal}

定義環境變數的最直接明確方式是直接在服務的 `deployment.yaml` 檔案中併入它。在下列[基本 Kubernetes 範例](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 中，當值在環境中都一致時，直接規格會運作良好：

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

### Helm 變數
{: #config-helm}

Helm 使用範本來建立圖表，因此您稍後可以替換這些值。您可以實現與前述範例相同的結果，而且環境之間具有更多彈性，方法是在 `mychart/templates/pod.yaml` 檔案範本中使用下列範例：

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

以及，在 `mychart/values.yaml` 檔案中使用下列範例：

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

Helm 呈現範本時，會產生下列輸出：

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

  這兩個範例之間存在次要差異。在第一個範例及範例 `values.yaml` 檔案中，人員已新增引號。YAML 中的字串不需要引號。Helm 呈現範本時，會忽略引號。
  {: note}

### ConfigMap
{: #kubernetes-configmap}

ConfigMap 是唯一的 Kubernetes 構件，可將資料定義為一組鍵值組。前述範例中所顯示環境變數的 ConfigMap 與下列範例類似：

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

然後，會將起始 Pod 定義變更為使用 ConfigMap 中的值，如下所示：

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

ConfigMap 現在是與 Pod 不同的構件。它可以具有不同的生命週期。您可以更新或變更 ConfigMap 中的值，而不需要在此情況下重新部署 Pod。您也可以直接從指令行更新及操作 ConfigMap，這在開發/測試/偵錯循環中可能會很有幫助。

與 Helm 一起使用時，您可以在 ConfigMap 宣告中使用變數。部署圖表時，會如常解析這些變數。

如需相關資訊，請參閱 [Kubernetes ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

### 認證和密碼
{: #kubernetes-secrets}

通常是透過環境變數或 ConfigMap 方式，將配置提供給 Kubernetes 中執行的容器。在任一情況下，都可以相當快速地探索配置值。這是 Kubernetes 使用 Secrets 來儲存機密性資訊的原因。

Secret 是包含 base64 編碼值的獨立物件：

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

然後，可以將 Secret 用作一個以上 Pod 容器上所裝載磁區中的檔案：

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

透過這種方式裝載為磁區時，Secret 資料對映中的每個索引鍵都會變成所指定 `mountPath` 下的檔案名稱：在此情況下為 `/etc/foo/username` 和 `/etc/foo/password`。

Secret 也可以用來定義環境變數：

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

Kubernetes 會為您執行 base64 解碼。擷取環境變數時，Pod 中執行的容器會辨識 base64 解碼值。

如同 ConfigMap，可以從指令行建立及操作 Secret，這在處理 SSL 憑證時會很方便。

如需相關資訊，請參閱 [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

<!-- SSL EXAMPLE -->
