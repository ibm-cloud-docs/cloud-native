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

# 雲端平台概念
{: #platform}

這裡簡要概述開發人員在建置雲端原生應用程式時與之互動的核心技術和概念（從 Containers、Kubernetes、Helm 及 Istio 開始）。
{:shortdesc}

## 容器
{: #containers}

容器是一種標準機制，可將應用程式及其所有相依關係都包裝成單一自行包含的單元。容器解決可攜性問題。容器構件（映像檔）確保應用程式執行所需的所有項目都已就緒。容器引擎著重在使用有效率、放心且安全的方式，以隔離的處理程序來執行容器。

容器映像檔通常是從 `Dockerfile` 中定義的指令清單來建置。容器映像檔幾乎一律是從其他容器映像檔所建置（實際上是接續已知先前狀態的指令）。例如，您可以使用下列 Snippet 來建立自己的 Open Liberty 映像檔：

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

映像檔在建置之後即可執行。Docker 或 [containers](https://containerd.io/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 這類容器執行引擎會採用該映像檔定義，並對主機作業系統直接執行定義的進入點作為資源隔離的處理程序。這會刪除虛擬機器的額外負擔。

容器映像檔會儲存至*登錄* 中。最熟知的是公用 Docker Hub 登錄。您可以將映像檔推送至存取控制的容器登錄（如 {{site.data.keyword.registryshort_notm}}），並從中取回映像檔，而這些登錄與基礎架構和 CI/CD 管線更為緊密相關。

## Kubernetes
{: #kubernetes}

IBM 的雲端平台使用 Kubernetes 來進行容器編排。因此，除了知道容器基本觀念之外，重要的是讓開發人員熟悉 Kubernetes 基本概念，其中包括基本指令和部署構件。下表包括一些重要的 Kubernetes 概念：

|概念| 說明 |
|---------|-------------|
| Pod | 一起部署為單一單元的本地化容器群組。Pod 相對地不可變，並且需要取代原始 Pod 以修改 Pod 的各種屬性。一般應用程式會有一個具有核心商業邏輯的容器，以及以精細層次提供平台功能的額外 Pod。|
| 部署 | 無狀態 Pod 的可重複範本，將規模維度新增至 Pod 概念。此外，還可以更新範本定義，並取代基礎 Pod 實例。「Kubernetes 部署控制器」會監視「Kubernetes 部署」配置，以確保維護「部署」的已宣告數目的 Pod。在 `.yaml` 檔案中，將「部署」顯示為 `kind: Deployment`。|
| 服務 | 代表一組相對不穩定 Pod IP 位址的已知名稱。「服務」只能存在於叢集專用網路，或向外部公開（使用雲端提供者特定的負載平衡器）。在 `.yaml` 檔案中，將「服務」顯示為 `kind: Service`。|
| Ingress | 透過虛擬主機作業或環境定義型遞送方式，與多個服務共用單一網址的能力。Ingress 也可以執行 TLS 終止這類網路連線管理活動。在 `.yaml` 檔案中，將 Ingress 顯示為 `kind: Ingress`。|
| 密碼 | 該物件儲存用於 Pod 運行環境的機密性資訊，並且區隔來自容器映像檔或編排的部署特定資訊。在運行環境，可以透過環境變數或虛擬檔案系統裝載，向 Pod 公開密碼。如果沒有密碼，則會將機密資料儲存在容器映像檔或編排中，這兩者較可能會不小心曝露或發生非預期的存取。|
| ConfigMap | 扮演與「密碼」類似的角色，因為它會將部署特定資訊與容器編排分開。不過，ConfigMap 是一般用途配置結構。在運行環境，它用來將指令行引數、環境變數及其他配置構件這類資訊連結至 Pod 的容器和系統元件。| 
{: caption="表 1. Kubernetes 概念" caption-side="bottom"}

所有資源都定義在 Kubernetes 資源模型內，其配置方式是透過 RESTful API 或透過 `kubectl` 指令行提交的配置檔。

如需相關資訊，請參閱 [Kubernetes 基本觀念](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")、[Kubernetes 物件模型](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 及 [`kubectl` 指令行](https://kubernetes.io/docs/reference/kubectl/overview/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。 

## Helm
{: #helm}

Helm 是一個套件管理程式，提供簡單的方式來尋找、共用及使用針對 Kubernetes 所建置的軟體。Helm 也會解決一般使用者需求：將相同的應用程式部署至多個環境。Helm 使用*圖表*，圖表是在安裝時間產生有效 Kubernetes 物件 (YAML) 的範本集合。這些圖表是從範本語言所建置，範本語言支援變數、範圍作業，以及其他需要大量人力來維護 Kubernetes 部署 meta 資料的事項。

如需相關資訊，請參閱 [Helm](https://helm.sh/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

## Istio 服務網格
{: #istio}

Istio 是用來管理及保護微服務的開放程式碼平台。它與 Kubernetes 這類編排器一起使用，提供一種方式來管理及控制服務之間的通訊。

Istio 是使用 Sidecar 模型運作的。Sidecar (Envoy Proxy) 是與您的應用程式並存的不同處理程序。Sidecar 可管理與服務之間的所有通訊，並將一般功能層次套用至所有與用來建置服務之程式設計語言或架構無關的服務。實際上，Istio 提供一種機制來集中配置遞送和安全原則，同時透過 Sidecar 以非集中方式來套用這些原則。

建議您使用 Istio 所提供的功能，而非個別程式設計語言或架構所提供的類似功能。舉例來說，基礎架構會更一致地定義、管理及強制執行負載平衡和其他遞送原則。

在某些情況下，與分散式追蹤相同，Istio 與應用程式層次程式庫是互補的。您可以同時使用兩者來改善作業。如果是分散式追蹤，Istio 只能確定追蹤標頭存在；應用程式庫提供要求之間關係的重要環境定義。Istio 和支援程式庫或架構程式庫一起使用時，會改善您對系統的整體瞭解。

在最高層次，Istio 會延伸 Kubernetes 平台，提供其他管理概念、可見性及安全。Istio 的特性可以分成下列四種種類：

* 資料流量管理：控制微服務之間的資料流量，以執行資料流量分割、失敗回復及 Canary 版本。
* 安全：在微服務之間提供強大的身分型鑑別、授權及加密。
* 觀察：收集度量值和日誌，以更適當的查看叢集中執行的應用程式。
* 原則：強制執行存取控制、速率限制及配額，以保護應用程式。

如需相關資訊，請參閱[何謂 Istio？](https://istio.io/docs/concepts/what-is-istio/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。



