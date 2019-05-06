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

# 度量值
{: #metrics}

度量值是擷取為鍵值組的簡單數值測量。有些度量值是增量計數器；其他度量值會執行聚集，例如在過去一分鐘內收集到的所有值總和，或在過去一分鐘內的平均經歷時間。有些度量值只是簡單量規，會傳回最後觀察到的任何值。擷取及處理度量值可協助您在呈報潛在問題並導致更嚴重問題之前，先識別及回應潛在問題。
{:shortdesc}

談論分散式系統中的度量值時，有三個一般因素：生產者、聚集器及處理器。這些因素有一些常見的一般組合，例如搭配使用 Prometheus（作為聚集器）與 Grafana（其處理為了要顯示在圖形儀表板中而收集的度量值），或搭配使用 StatsD 與 Graphite。

![分散式系統度量值中的三個因素](images/metrics-systems.png "分散式系統度量值中的三個因素"){: caption="圖 1. 分散式系統度量值中的三個因素" caption-side="bottom"}

生產者當然就是應用程式本身。在某些情況下，應用程式直接涉及產生度量值。在其他情況下，代理程式或其他基礎架構會被動觀察或主動檢測應用程式，以代表應用程式產生度量值。接下來會發生的狀況視聚集器而定。

藉由「推送」或「取回」機制的方式，將度量值從生產者傳送至聚集器。部分聚集器（如 StatsD）預期應用程式（或代表應用程式的代理程式）會連接至聚集器來傳輸資料。這需要將聚集器的連線資訊配送至應該測量的所有應用程式處理程序。其他聚集器（如 Prometheus）會定期連接至已知的端點來收集（或提取）度量值資料。這需要生產者定義及提供可提取的端點，以作為聚集器並且將端點位置告知聚集器。與 Kubernetes 搭配使用時，Prometheus 可以根據服務註釋來探索端點。

最後，「處理器」會利用所有聚集資料。如前所述，Grafana 這類服務會使用儀表板來處理視覺化的聚集度量值。Grafana 也支援與儀表板分開儲存及評估的規則所發出的警示。

## 在 Kubernetes 中自動探索 Prometheus 端點
{: #prometheus-kubernetes}

取回型模型已建立自己的生態系統。Sysdig 這類其他聚集器也可以從 Prometheus 端點提取度量值資料。這可能表示部分系統使用 Prometheus 度量，但不使用 Prometheus 伺服器。

在 Kubernetes 環境中，註釋用於 Prometheus 端點探索。例如，在埠 8080 透過 HTTP 提供 `/metrics` 端點的服務，會將下列註釋新增至服務定義：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /metrics
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

然後，任何 Prometheus 相容聚集器可以都使用 Kubernetes API，藉由過濾註釋來探索這些端點。

## 應用程式與平台度量值
{: #app-platform-metrics}

收集度量值需要思考。您應該收集哪些資料點？考量雲端原生應用程式時，您可以將度量值廣泛分成兩個種類，即應用程式和平台：

* 應用程式度量值著重在應用程式網域實體。例如，在過去五分鐘內完成多少次登入？目前連接了多少位使用者？在最後一秒內執行多少次交易？收集應用程式特定度量值，需要有自訂程式碼來收集及發佈資訊。大部分的語言具有度量值庫，可簡化自訂度量值的新增。
* 相對地，平台度量值是管理網域實體。例如，呼叫服務需要多長時間？此資料庫查詢需要多長時間？它們與資料流量和工作流經系統的方式保持一致，而且通常不需要對應用程式進行任何變更即可進行測量。部分應用程式架構還會提供內建的支援來測量這些問題，例如查詢執行時間。

這些種類本身不足以實際決定您需要測量的項目。請記住，在分散式系統中，有許多項目都會產生度量值。有一些已知的方法會嘗試將大量的度量值儲存區減少到應該監看的一些基本度量值：

* [四個黃金信號](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是 Google Site Reliability Engineering (SRE) 團隊所識別的基本度量值，用於監視分散式系統的服務層級觀察。換言之，「如果您只能測量使用者面向系統的四個度量值，則請專注在這四個度量值。」四個度量值為：延遲、資料流量、錯誤及飽和度。
* [使用率飽和度和錯誤 (USE) 方法](http://www.brendangregg.com/usemethod.html){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 設計作為緊急核對清單，用來分析系統的效能。USE 方法可以彙總為一句話：「對於每個資源，檢查使用率、飽和度及錯誤」。在此情況下，為具有強迫限制的實體或邏輯資源（例如 CPU 或磁碟）。對於每個有限資源，您可以測量使用率、飽和度及錯誤。 
* [RED 方法](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示") 是衍生自四個黃金信號的助記鍵，用於定義架構中每個微服務都應該測量的三個關鍵測量值。此方法非常以要求為中心，尤其是與 USE 方法相較時，而這與許多雲端原生應用程式的設計與架構一致。度量值為：速率、錯誤及持續時間。 

這些方法有一些通用元素，所有這些元素都會追蹤錯誤率（舉例來說）。但也有值得注意的差異。USE 方法著重在基礎架構度量值（使用率/飽和度）。雲端原生環境的設計是要充分使用實體（或虛擬）硬體。這些基礎架構測量會查看您的系統是否適當地處理負載。RED 方法完全著重在要求度量值（延遲/持續時間、資料流量/速率），其指出基礎架構的問題（網路問題）或應用程式的問題（錯誤配置、死鎖、重複的服務失敗等），尤其是與觀察的錯誤率結合時。黃金信號跨越兩者，可將基礎架構度量值和要求度量值合併為一個整體視圖。

## 定義度量值
{: #defining-metrics}

收集自單一服務的監視度量值可讓您瞭解其資源使用率，但是，如果服務因水平調整或多地區部署而有多個實例，則需要識別它們來隔離大量類似進入資料的問題。這最終會將範圍縮小到命名。在某些情況下，您所使用的度量值系統可能會對您的度量值強制執行結構。例如，Prometheus 建議使用 `namespace_subsystem_name` 這類項目，其他項目則建議使用 `namespace.subsystem.targetNoun.actioned`。

例如，如果您要追蹤股票交易應用程式所執行的「交易」數目，則可以將它擷取到稱為 `stock.trades` 的內容中。若要識別實例，您可以在內容的前面加上實例 ID：`<instanceid>.stock.trades`。這支援收集個別實例值，以及使用 `*.stock.trades` 來聚集資料。但是，當您部署至多個資料中心，並且要透過該方式來分析度量值時，會發生什麼情況？您可以將您的名稱更新為 `<datacenter>.<instanceid>.stock.trades`，但這樣會中斷任何使用先前萬用字元 `*.stock.trades` 的報告。需要改用 `*.*.stock.trades`。 

單獨使用階層式命名的內容可能會不小心地導致關聯命名策略之任意結構的易損壞萬用字元型樣，這些型樣無法協助您觀察應用程式運作良好所需的資訊。

支援維度資料的度量值系統可讓您建立其他識別標籤或標記與度量值資料的關聯。您接著可以使用這些額外的維度來過濾、分組或分析所收集的度量值，而不需要依賴萬用字元和命名慣例。一般標籤包括端點或服務名稱、資料中心、回應碼、管理環境 (prod/staging/dev/test) 或運行環境 ID（Java 版本、應用程式伺服器資訊）。

使用相同的股票交易範例，`stock.trades` 度量值會與數個標籤相關聯：`{ instanceid=..., datacenter=... }`，以根據 `instanceid` 或 `datacenter` 來過濾或分組所聚集的值，而不需要依賴萬用字元。具名度量值 (`stock.trades`) 與關聯的標籤之間有其平衡（與階層式範例 `<datacenter>.<instanceid>.stock.trades` 相較之下）：每個度量值都應該擷取有意義的資料，而且標籤容許在適當時進行澄清。

此外，定義標籤時也請小心。例如，在 Prometheus 中，鍵值標籤組的每個唯一組合都被視為個別的時間序列。確保良好查詢行為和有限資料收集的最佳作法是使用具有有限容許值數目的標籤。例如，如果您使用的度量值會計算錯誤數目，則可以使用回覆碼作為標籤，其中值位於合理的集內（401、404、409、500...），但您不想要失敗 URL 的標籤，因為那是無界限集（因任何原因而失敗的任何要求 URL，包括無效的要求 URL）。

如需度量值和標籤（標籤）命名最佳作法的相關資訊，請參閱[度量值和標籤命名](https://prometheus.io/docs/practices/naming/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

## 其他考量
{: #metrics-considerations}

收集度量值時，請記住，失敗路徑通常與成功路徑極為不同。例如，如果失敗涉及逾時和堆疊追蹤集合，則 HTTP 資源的錯誤回應可能會花費比成功回覆還要長的時間。根據成功的要求，個別地計算並處理錯誤路徑。

分散式系統在某些測量中具有自然變化。偶發性錯誤是正常的，因為可能會在啟動或關閉過程中將要求導向至處理程序。過濾要在此自然變化開始超出有效範圍時捕捉的原始資料。例如，將度量值分割成儲存區。將要求持續時間分類為 'smallest/quickest'、'medium/normal' 及 'longest/largest' 這類種類，如滑動時間範圍內觀察到的值。如果要求持續時間一致地落入 "longest/largest" 儲存區，您便可以識別出問題。直方圖或摘要度量值通常用於這類型的資料。如需相關資訊，請參閱[直方圖和摘要](https://prometheus.io/docs/practices/histograms/){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。

身為應用程式開發人員，請確定您應用程式或服務所發出的度量值具有遵循全組織使用慣例的名稱和標籤，以支援著重在商業核心端對端路徑的監視工作。如需相關資訊，請參閱[監視分散式系統](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")。
