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

# 容錯
{: #fault-tolerance}

容錯是系統在局部失敗期間能夠繼續運作的能力。其中所有服務都需要建立具復原力系統。雲端環境的動態本質要求服務撰寫為能預期並平穩地回應非預期情況。 

容錯解決方案通常專注在逾時、撤回、隔板及斷路器。

在某些環境中，Istio 這類基礎架構元件可能會提供容錯機制。不論基礎架構是否有幫助，服務都必須假設遠端呼叫可能會失敗，且必須準備好採取適當的撤回動作。

## 逾時

局部失敗的防禦第一線是使用逾時。逾時確保應用程式在支援服務無回應時收到錯誤，讓它能夠採取適當的撤回動作來處理該狀況。這不表示所要求的作業失敗。逾時會變更提出要求的用戶端等待回應的時間。它們不會影響目標服務的處理行為。

許多語言程式庫都會使用要求的預設逾時，而 Istio 也是。依預設，如果要求未在 15 秒內收到回應，則 Sidecar Proxy 會讓要求失敗。使用傳回股票報價的服務作為範例，您可以藉由在 VirtualService 定義中設定路徑的逾時原則來變更此值，其與下列內容類似：

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

透過 Sidecar Proxy 或 Ingress 閘道對股票報價服務提出的要求會先等待 30 秒，再讓要求失敗，而不是預設值 15 秒。

以這種方式調整逾時，所有使用路徑的要求都會套用逾時。這是 Istio 所提供的基本安全層。即使應用程式架構或程式庫未強制逾時，也不會永久等待，因為 Istio 會強制逾時。不過，仍然會套用應用程式層次逾時。股票報價服務的基礎架構層次逾時已延長至 30 秒。如果應用程式庫設定 5 秒的逾時，則應用程式的要求仍會因逾時而失敗。

## 撤回

應用程式會定義對支援服務的要求失敗時會發生什麼情況。有幾個選擇，但目標是在這些服務未及時回應時溫和地降級。遠端服務失敗時，您可以重試要求、嘗試不同的要求，或改為傳回快取的資料。

乍看之下，重試要求是最簡單的撤回機制。但不明顯的是，重試要求可能會導致階層式系統失敗（「重試風暴」，這是 [Thundering Herd Problem](https://en.wikipedia.org/wiki/Thundering_herd_problem) 的變異）。應用程式層次程式碼無法足夠瞭解系統或網路性能，而且指數後退演算法很難正確處理。

Istio 可以更有效地執行重試。它已直接涉入要求遞送，並為重試原則提供一致的不限語言實作。例如，您可以為股票報價服務定義下列這類原則：

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

使用此簡單配置，透過 Istio Sidecar Proxy 或 Ingress 閘道對股票報價服務提出的要求會重試最多 3 次，而且每次嘗試都有 5 秒的逾時。例如，[其他路徑比對規則](https://istio.io/docs/reference/config/networking/#HTTPMatchRequest)可以進一步將此重試原則限制為 `GET` 要求。

這裡有一個容易遺漏的細微差異：您未指定重試間隔。Sidecar 會判定重試之間的間隔，並故意在嘗試之間加入「雜訊」，以避免不斷攻擊超載服務。

<!-- Notes about other approaches here: -->

## 隔板
{: #bulkheads-fault-tolerance}

出貨時，隔板是防止某區段的裂縫而導致整艘船下沈的分割區。此型樣會套用至雲端環境，以實現類似的目的，而且是以幾種不同的方式來完成。

對於 Java 這類多執行緒語言，可以在內部使用內部隔板，來限制或控制如何利用佇列或號誌機制以使用資源進行與遠端資源的通訊：

- 若要使用佇列，服務會建立特定數目的執行緒與特定佇列的關聯。任何在佇列已滿時所提出的要求都會快速失敗。例如，在 Java 中，這可能是與 `BlockingQueue` 相關聯的 `ThreadPoolExecutor`。
- 號誌機制的運作方式是要具有一定數量的允許。出埠要求需要有允許。在順利完成要求之後，即會釋放該允許，以供其他要求使用。

您也可以使用 Istio DestinationRule 來定義服務間隔板，以限制上游服務的連線儲存區，如下列範例所示：

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

此配置會將對每個股票報價服務實例所建立的並行連線數目上限限制為 10。無法在 30 秒內建立連線的服務將會取得 `503 -- Service Unavailable` 回應。例如，這類型的隔板可以用來防止密集運算服務收到超過其管理能力的要求數目。

## 斷路器

發生故障時，會使用斷路器來最佳化出埠要求的行為。斷路器會觀察給定時段內發生的故障次數，而非反覆地對無回應服務提出要求。如果錯誤率超過臨界值，則斷路器會開啟電路並讓要求失敗，而且除非再次關閉電路，否則不會處理所有後續要求。

斷路器也是使用 Istio DestinationRule 所定義：

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

此配置會限制要求，而這些要求就是其他服務將對股票報價服務提出的要求。指定的 `outlierDetection` 資料流量原則會套用至每個個別實例。以一個句子表達之前的配置：「在至少 5 分鐘的期間，退出任何在 5 秒內失敗 3 次的股票報價服務實例。可以退出所有實例。」後面的 `maxEjectionPercent` 設定與負載平衡相關。Istio 會維護負載平衡儲存區，並從該儲存區中退出失敗的實例。依預設，它會從負載平衡儲存區中退出所有可用實例的 10% 上限。

對於熟悉其他斷路器機制的項目，Istio 沒有半開啟狀態。它會套用一些簡單的數學方式。仍會從儲存區退出實例 `baseInjectionTime * <number of times it has been ejected>`。這容許從暫時性失敗中回復實例，並保留儲存區中的失敗實例。

備妥之後，對股票報價服務的並行呼叫數目上限限制為至 10 個。只要超過此數量，就會收到 `503 -- Service Unavailable` 回應。

### Istio - 速率限制
{: #istio-rate-limits}

您也可以使用 Istio 來定義**速率限制**。這些與隔板類似，但透過時間範圍進行定義，而不是僅限制在同一時刻處理的並行呼叫數。

例如，您不要容許每秒有多條推文。這是真實範例。以前，對 Stock Trader 執行壓力測試時，Twitter 會自動鎖定 `@IBMStockTrader` 帳戶。

<!-->
*<需要範例。請注意，速率限制要求介紹混合器配接器（例如，memquota 或 redisquota）的概念；除了先前的原則外，還有更多相關內容。>*


## 測試：使用 Istio 進行錯誤注入

您可以使用 Istio 來有意導致錯誤，以查看程式碼如何回應這些錯誤...
-->
