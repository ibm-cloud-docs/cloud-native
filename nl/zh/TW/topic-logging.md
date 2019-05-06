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

# 記載
{: #logging}

日誌訊息這類字串包含建立日誌項目時，與微服務的狀態和活動相關的環境定義資訊。需要有日誌，才能診斷服務失敗的程度和原因。它們扮演監視應用程式性能之度量值的支援角色。
{:shortdesc}

請確保將日誌項目直接寫入至標準輸出和錯誤串流。這會使用指令行工具來檢視日誌項目，並容許在基礎架構層次配置日誌轉遞服務（例如 Logstash、Filebeat 或 Fluentd）來管理日誌收集和資料管理。

處理日誌檔時，需要深入思考是否無法將容器化應用程式配置成將日誌寫入至標準輸出或標準錯誤。

* 其中一個選項是使用磁區來記載資料、是否將簡單連結裝載用於本端開發及測試，或作為 Kubernetes 部署一部分的適當「持續性磁區」。您可以從共用磁區讀取[專用 Sidecar 或記載代理程式](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")，以將日誌轉遞至集中化聚集器。必須明確地配置日誌替換，才能控制磁區上所儲存的日誌資料量。
* 另一個選項是使用應用程式庫或代理程式，直接將日誌轉遞至聚集器。此選項會在部署環境之間攜帶一些配置複雜性。

## JSON 記載
{: #json-logging}

因為您的應用程式會隨著時間有所進展，因此您所記載事物的本質也會變更。使用 JSON 日誌格式，您可以獲得下列好處：

* 可以建立日誌的索引，以更輕易地搜尋聚集的日誌內文。
* 日誌在變更後可以復原，因為剖析並非仰賴元素在字串中的位置。

雖然 JSON 格式化日誌較容易供機器剖析，但人類卻難以閱讀。請考慮使用環境變數，將日誌格式在純文字（以便用於本端開發及除錯）與 JSON 格式日誌（以便用於較長期的儲存和聚集）之間進行切換。

可以使用 JSON Query 工具 (jq) 這類指令行 JSON 剖析器來建立人類可讀的 JSON 格式日誌視圖。在下列範例中，會在 jq 剖析該行之前，先透過 grep 對日誌進行管道處理，以確保訊息欄位存在：

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## 使用 `kubectl` 檢視日誌
{: #view-logs-kube}

透過主控台或具有 `kubectl logs <podname>` 格式的 `kubectl` 指令方式，即可在 Kubernetes 中檢視傳送至標準輸出和錯誤串流的日誌。

如果您使用 stock-trader 這類自訂名稱空間，請在指令中包括該名稱空間，例如，`kubectl logs -n stock-trader <podname>`。

如果每個 Pod 都有多個容器，則像使用 Istio Sidecar 時一樣，您也需要指定容器。在下列範例中，使用 stock-trader 名稱空間來檢視 `portfolio-54b4d579f7-4zvzk` Pod 的 `portfolio` 容器中的日誌：

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

對於 JSON 格式的日誌，您可以使用 `jq` 來擷取訊息欄位，例如：

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

透過 `grep` 對日誌項目進行管道處理，以確保 `jq` 僅剖析包含訊息欄位的數行。
{: note}
