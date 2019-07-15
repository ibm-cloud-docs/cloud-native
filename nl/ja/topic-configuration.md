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

# 構成
{: #configuration}

クラウド・ネイティブ・アプリケーションはポータブルでなければなりません。 ローカル・ハードウェアからクラウド・ベースのテスト環境と実稼働環境に至るまで、コードを変更したり、テストしていないコード・パスを実行したりすることなく、固定の同じ成果物を使用して複数の環境にデプロイできる必要があります。
{:shortdesc}

[12 要素の方法論](https://12factor.net/){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン") の 3 つの要素が、この手法と直接関連しています。

* 第 1 要素では、実行中のサービスとバージョン管理されているコードベースを 1 対 1 で関係付けることが推奨されています。 つまり、Docker イメージなどの固定のデプロイメント成果物を、変更することなく複数の環境にデプロイ可能な、バージョン管理されているコードベースから作成することを意味します。
* 第 3 要素では、固定成果物に含めるべきアプリケーション固有の構成と、デプロイメント時にサービスに渡すべき環境固有の構成を分離することが推奨されています。
* 第 10 要素では、すべての環境をできる限り同じ環境にすることが推奨されています。 環境固有のコード・パスはテストが難しく、別の環境にデプロイすると失敗するリスクが高まります。 これは、バッキング・サービスにも当てはまります。 メモリー内データベースを使用して開発とテストを行うと、テスト環境、ステージング環境、または実稼働環境で使用しているデータベースの動作が異なるために予期しない障害が発生する可能性があります。

## ソースの構成
{: #config-inject}

アプリケーション固有の構成は、固定の成果物に含める必要があります。 例えば、WebSphere Liberty で実行するアプリケーションでは、実行時にアクティブになるバイナリーとサービスを制御するインストール済み機能のリストを定義します。 これはアプリケーションに固有の構成であるため、Docker イメージに含める必要があります。 また、Docker イメージは、コンテナーの起動時に実行環境がポート・マッピングを処理するので、listen するポート、または公開ポートも定義します。

他のサービスとの通信に使用するホストとポート、データベース・ユーザー、リソース使用の制約など、環境固有の構成は、デプロイメント環境によってコンテナーに提供されます。 サービス構成と資格情報の管理は以下の点において大きく異なる可能性があります。

* Kubernetes は構成値 (文字列化された JSON またはフラットな属性) を ConfigMap またはシークレットに格納します。 これらの値は、環境変数または仮想ファイル・システム・マウントとして、コンテナー化アプリケーションに渡すことができます。 サービスで使用するメカニズムは、Kubernetes の YAML または Helm チャートのデプロイメント・メタデータに指定します。
* 多くの場合、ローカル開発環境は、キーと値からなるシンプルな環境変数を使用する、単純化されたバリアントです。
* Cloud Foundry は、構成の属性とサービス・バインディングの詳細を、文字列化された JSON オブジェクトに格納します。これらは、環境変数 (`VCAP_APPLICATION`、`VCAP_SERVICES` など) としてアプリケーションに渡されます。
* どの環境でも、環境固有の構成属性を保管および取得するために、etcd、Hashicorp Vault、Netflix Archaius、Spring Cloud Config などのバッキング・サービスを使用することもできます。

ほとんどの場合、アプリケーションは起動時に環境固有の構成を処理します。 例えば、環境変数の値をプロセスの起動後に変更することはできません。 ただし、Kubernetes とバッキング構成サービスには、アプリケーションが構成更新に動的に反応するためのメカニズムが備わっています。 これはオプションの機能です。 ステートレスで一時的なプロセスであれば、多くの場合、サービスを再起動するだけで十分です。
{: note}

多くの言語およびフレームワークには、アプリケーション固有の構成と環境固有の構成をアプリケーションで行えるようにサポートする標準ライブラリーが用意されているので、アプリケーションの中核ロジックに集中し、こうした基本的な機能を抽象化できます。

### サービス資格情報の操作
{: #portable-credentials}

サービスの構成と資格情報 (サービス・バインディング) の管理はプラットフォームによって異なります。 Cloud Foundry では、文字列化された JSON オブジェクトにサービス・バインディングの詳細を格納し、そのオブジェクトを `VCAP_SERVICES` 環境変数としてアプリケーションに渡します。 Kubernetes では、文字列化された JSON またはフラットな `ConfigMaps` 属性または `Secrets` 属性にサービス・バインディングを格納し、それらを環境変数としてコンテナー化アプリケーションに渡したり、一時ボリュームとしてマウントしたりできます。 ローカル開発環境は独自の構成を持ち、ローカル・テスト環境はクラウドの実行環境をシンプルにしたバージョンであることがほとんどです。 こうしたバリエーションを環境固有のコード・パスがない状態でポータブルに操作するのは簡単ではありません。

Cloud Foundry 環境と Kubernetes 環境では、[サービス・ブローカー](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン") を使用して、バッキング・サービスのバインディング、および、関連する資格情報のアプリケーション環境への注入を管理できます。 これは、アプリケーションのポータビリティーに影響を与える可能性があります。資格情報をアプリケーションに渡す方法は環境によって異なることがあるからです。

{{site.data.keyword.IBM}} には `mappings.json` ファイルを操作するオープン・ソース・ライブラリーがいくつか用意されていて、資格情報を取得するためにアプリケーションが使用するキーを、ソース候補が並べられたリストにマップできます。 以下の 3 種類の検索パターンがサポートされています。

* `cloudfoundry`: Cloud Foundry のサービス (VCAP_SERVICES) 環境変数で値を検索します。
* `env`: 環境変数にマップされている値を検索します。
* `file`: JSON ファイルの値を検索します。

以下の `mappings.json` ファイルの例では、`cloudant-password` が、アプリケーション・コードでパスワード資格情報を検索するために使用するキーです。 言語固有のライブラリーは、一致が見つかるまで特定の順序で `searchPatterns` 配列の処理を繰り返します。

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

ライブラリーは、以下の場所で cloudant パスワードを検索します。

* Cloud Foundry `VCAP_SERVICES` 環境変数の `['cloudant'][0].credentials.password` JSON パス。
* `cloudant_password` という名前の環境変数 (大/小文字の区別なし)。
* 言語固有のリソースの場所に保持されている **`localdev-config.json`** ファイルの **cloudant_password** JSON フィールド。

詳しくは、以下を参照してください。

* [{{site.data.keyword.Bluemix}} Environment for Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン")
* [{{site.data.keyword.Bluemix_notm}} Environment for Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン")
* [{{site.data.keyword.Bluemix_notm}} Service Bindings For Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン")

## Kubernetes の構成値
{: #config-kubernetes}

Kubernetes には、環境変数を定義して値を割り当てるための方法がいくつかあります。

### リテラル値
{: #config-literal}

環境変数を定義する最も簡単な方法は、サービスの `deployment.yaml` ファイルに環境変数を直接指定することです。 以下の [Kubernetes の基本的な例](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン") では直接指定していますが、どの環境でも値が同じであれば、この方法で十分です

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

### Helm 変数
{: #config-helm}

前述のように、Helm はテンプレートを使用してチャートを作成するので、値は後で置換することができます。 `mychart/templates/pod.yaml` ファイル・テンプレートで以下の例を使用すれば、環境ごとに変更できる柔軟性を確保しながら、上記の例と同じ結果を得ることができます。

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

`mychart/values.yaml` ファイルで以下の例を使用します。

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

Helm でテンプレートを表示すると、以下の出力が生成されます。

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

  この 2 つの例には若干の違いがあります。 最初の例とサンプルの `values.yaml` ファイルでは、人が引用符を追加しています。 引用符は YAML の文字列には不要です。 Helm でテンプレートを表示すると、引用符が取り除かれています。
  {: note}

### ConfigMap
{: #kubernetes-configmap}

ConfigMap は、一連のキー値のペアとしてデータを定義する Kubernetes の固有の成果物です。 前述の例で示した環境変数の ConfigMap は、以下の例のようになります。

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

次は、最初のポッド定義を、この ConfigMap の値を使用するように変更します。

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

これで、ConfigMap はポッドとは完全に別の成果物になります。 ライフサイクルも完全に別にすることができます。 この場合、ポッドを再デプロイすることなく、ConfigMap の値を更新したり変更したりできます。 また、ConfigMap はコマンド・ラインから直接更新および操作できるので、開発/テスト/デバッグのサイクルで役に立ちます。

Helm で使用すれば、ConfigMap 宣言で変数を使用できます。 それらの変数は、通常どおり、チャートがデプロイされたときに解決されます。

詳しくは、[Kubernetes ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン") を参照してください。

### 資格情報とシークレット
{: #kubernetes-secrets}

通常、Kubernetes で実行するコンテナーに構成を渡すには、環境変数または ConfigMap のいずれかを使用します。 どちらの場合も、構成値は非常に簡単に露顕します。 そのため、Kubernetes では機密情報の保管にシークレットを使用します。

シークレットは、base64 エンコードの値が含まれる独立したオブジェクトです。

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

その後、シークレットは、ポッドの 1 つ以上のコンテナーにマウントしたボリュームのファイルとして使用できます。

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

この方法でボリュームとしてマウントした場合、シークレットのデータ・マップ内の各キーが、指定した `mountPath` 下のファイル名になります (この場合には、`/etc/foo/username` および `/etc/foo/password`)。

シークレットを使用して環境変数を定義することもできます。

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

Kubernetes が自動で base64 デコードを実行します。 ポッドで実行されるコンテナーは、環境変数を取得するときに base64 でデコードされた値を認識します。

ConfigMap と同様に、シークレットもコマンド・ラインから作成および操作できるので、SSL 証明書を扱うときに便利です。

詳しくは、[Kubernetes シークレット](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![外部リンクのアイコン](../icons/launch-glyph.svg "外部リンクのアイコン") を参照してください。

<!-- SSL EXAMPLE -->
