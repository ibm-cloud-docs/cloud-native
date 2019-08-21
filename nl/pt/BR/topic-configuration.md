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

# Configuração
{: #configuration}

Os aplicativos nativos de nuvem devem ser móveis. É possível usar o mesmo artefato fixo para implementar em múltiplos ambientes sem mudar o código ou exercitar caminhos de código não testados.
{:shortdesc}

Três fatores da [metodologia de doze fatores](https://12factor.net/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") se relacionam diretamente a essa prática:

* O primeiro fator recomenda uma correlação de um para um entre um serviço em execução e um código base com versão. Crie um artefato de implementação fixo, como uma imagem do Docker, por meio de um código base com versão que pode ser implementado, inalterado, para múltiplos ambientes.
* O terceiro fator recomenda uma separação entre a configuração específica do aplicativo, que faz parte do artefato fixo e a configuração específica do ambiente, que deve ser fornecida para o serviço no momento da implementação.
* O décimo fator recomenda manter todos os ambientes o mais semelhantes possível. Os caminhos de código específicos do ambiente são difíceis de testar e aumentam o risco de falhas, conforme a implementação em diferentes ambientes. Isso também se aplica aos serviços auxiliares. Se você desenvolver e testar com um banco de dados na memória, falhas inesperadas poderão ocorrer em ambientes de teste, preparação ou produção, pois eles usam um banco de dados que possui um comportamento diferente.

## Origens de configuração
{: #config-inject}

A configuração específica do aplicativo é parte do artefato fixo. Por exemplo, os aplicativos que são executados no WebSphere Liberty definem uma lista de recursos instalados que controlam os binários e serviços que estão ativos no tempo de execução. Essa configuração é específica para o aplicativo e está incluída na imagem do Docker. As imagens do Docker também definem a porta exposta ou de atendimento, pois o ambiente de execução manipula o mapeamento de porta quando o contêiner é iniciado. 

A configuração específica do ambiente, como o host e a porta que são usados para se comunicar com outros serviços, os usuários do banco de dados ou as restrições de utilização de recursos, é fornecida para o contêiner pelo ambiente de implementação. O gerenciamento da configuração de serviço e as credenciais podem variar significativamente:

* O Kubernetes armazena valores de configuração (atributos JSON ou sem formatação em sequência) no ConfigMaps ou em Segredos. Eles podem ser transmitidos ao aplicativo conteinerizado como variáveis de ambiente ou montagens do sistema de arquivos virtual. O mecanismo usado por um serviço é especificado nos metadados de implementação, seja no YAML do Kubernetes ou no gráfico do Helm).
* Os ambientes de desenvolvimento local são frequentemente variantes simplificadas que usam variáveis de ambiente de chave/valor simples.
* O Cloud Foundry armazena atributos de configuração e detalhes de ligação de serviço em objetos JSON em sequência que são transmitidos ao aplicativo como uma variável de ambiente, por exemplo, `VCAP_APPLICATION` e `VCAP_SERVICES`.
* Usar um serviço auxiliar, como o etcd, o hashicorp Vault, o Netflix Archaius ou o Spring Cloud config, para armazenar e recuperar atributos de configuração específica do ambiente também é uma opção em qualquer ambiente.

Na maioria dos casos, um aplicativo processa uma configuração específica do ambiente no horário de início. O valor das variáveis de ambiente, por exemplo, não pode ser mudado após o início de um processo. No entanto, o Kubernetes e os serviços de configuração auxiliares fornecem mecanismos para que os aplicativos respondam dinamicamente às atualizações de configuração. Este é um recurso opcional. Se processos temporários stateless, a reinicialização do serviço geralmente será suficiente.

Muitas linguagens e estruturas fornecem bibliotecas padrão para auxiliar os aplicativos nas configurações específicas de aplicativo e de ambiente, para que seja possível se concentrar na lógica principal de seu aplicativo e abstrair esses recursos fundamentais.

### Trabalhando com credenciais de serviço
{: #portable-credentials}

O gerenciamento da configuração e das credenciais de serviço (ligações de serviço) varia entre as plataformas. O Cloud Foundry armazena detalhes de ligação de serviço em um objeto JSON em sequência que é passado para o aplicativo como uma variável de ambiente `VCAP_SERVICES`. O Kubernetes armazena as ligações de serviço como atributos `ConfigMaps` ou `Secrets` JSON ou sem formatação em sequência, que podem ser transmitidos ao aplicativo conteinerizado como variáveis de ambiente ou montados como um volume temporário. Se desenvolvimento local, que tem sua própria configuração, o teste local será, muitas vezes, uma versão simplificada do que estiver em execução na nuvem. Trabalhar por essas variações de uma maneira móvel sem ter caminhos de código específicos do ambiente pode ser desafiador.

Em ambientes do Cloud Foundry e do Kubernetes, é possível usar [brokers de serviço](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para gerenciar a ligação a um serviço auxiliar e injetar as credenciais associadas no ambiente do aplicativo. Isso pode impactar a portabilidade do aplicativo, pois as credenciais podem não ser fornecidas ao aplicativo da mesma maneira em ambientes diferentes.

A {{site.data.keyword.IBM}} tem diversas bibliotecas de software livre que trabalham com um arquivo `mappings.json` para mapear a chave usada pelo aplicativo para recuperar as informações de credencial para uma lista ordenada de possíveis origens. Ela suporta três tipos de padrão de procura:

* `cloudfoundry`: procurar um valor na variável de ambiente de serviços do Cloud Foundry (VCAP_SERVICES).
* `env`: procurar um valor mapeado para uma variável de ambiente.
* `file`: procurar um valor em um arquivo JSON.

No arquivo `mappings.json` de exemplo a seguir, `cloudant-password` é a chave usada pelo código do aplicativo para consultar a credencial de senha. Uma biblioteca específica de linguagem itera por meio da matriz `searchPatterns` em uma ordem específica até que uma correspondência seja localizada.

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

A biblioteca procura a senha do cloudant nos locais a seguir:

* O caminho JSON `['cloudant'][0].credentials.password` na variável de ambiente `VCAP_SERVICES` do Cloud Foundry.
* Uma variável de ambiente sem distinção entre maiúsculas e minúsculas, denominada cloudant_password`.
* Um campo JSON **cloudant_password** em um arquivo **`localdev-config.json`** mantido em um local de recurso específico da linguagem.

Para obter mais informações, veja:

* [Ambiente {{site.data.keyword.Bluemix}} para Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Ambiente {{site.data.keyword.Bluemix_notm}} para Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")
* [Ligações de serviço do {{site.data.keyword.Bluemix_notm}} para o Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")

## Valores de configuração no Kubernetes
{: #config-kubernetes}

O Kubernetes fornece algumas maneiras diferentes de definir variáveis de ambiente e designar seus valores.

### Valores literais
{: #config-literal}

A maneira mais direta de definir uma variável de ambiente é incluí-la diretamente no arquivo `deployment.yaml` para o serviço. No seguinte [exemplo básico do Kubernetes](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), a especificação direta funciona corretamente quando o valor é consistente entre os ambientes:

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

### Variáveis do Helm
{: #config-helm}

O Helm utiliza modelos para criar gráficos para que os valores possam ser substituídos posteriormente. É possível obter um resultado igual ao do exemplo anterior, com mais flexibilidade nos ambientes, usando o exemplo a seguir no modelo de arquivo `mychart/templates/pod.yaml`:

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

E o exemplo a seguir em um arquivo `mychart/values.yaml`:

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

A saída a seguir é produzida quando o Helm renderiza o modelo:

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

  Há uma pequena diferença entre esses dois exemplos. No primeiro e no arquivo `values.yaml` de amostra, aspas incluídas manualmente. As aspas não são necessárias para sequências em YAML. Quando o Helm renderiza o modelo, elas são ignoradas.
  {: note}

### ConfigMap
{: #kubernetes-configmap}

Um ConfigMap é um artefato exclusivo do Kubernetes que define dados como um conjunto de pares chave/valor. Um ConfigMap para as variáveis de ambiente que são mostradas nos exemplos anteriores pode ser semelhante ao exemplo a seguir:

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

A definição inicial de Pod é, então, mudada para utilizar valores do ConfigMap da seguinte forma:

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

O ConfigMap agora é um artefato separado do Pod. Ele pode ter um ciclo de vida diferente. Neste caso, é possível atualizar ou mudar os valores no ConfigMap sem ter que reimplementar o Pod. Também é possível atualizar e manipular um ConfigMap diretamente na linha de comandos, o que pode ser útil no ciclo desenvolvimento/teste/depuração.

Quando ele é usado com o Helm, é possível utilizar variáveis em sua declaração ConfigMap. Essas variáveis são resolvidas normalmente quando o gráfico é implementado.

Para obter mais informações, consulte [Kubernetes ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

### Credenciais e Segredos
{: #kubernetes-secrets}

A configuração geralmente é fornecida para contêineres que são executados em Kubernetes por meio de variáveis de ambiente ou ConfigMaps. Em qualquer um dos casos, os valores de configuração podem ser descobertos rapidamente. É por isso que o Kubernetes usa os Segredos para armazenar informações confidenciais.

Os segredos são objetos independentes que contêm valores codificados em base64:

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

Então, os segredos podem ser usados como arquivos em um volume que é montado em um ou mais dos contêineres de um pod:

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

Quando montada como um volume dessa maneira, cada chave no mapa de dados do Segredo se torna um nome de arquivo no `mountPath` especificado: `/etc/foo/username` e `/etc/foo/password`, nesse caso.

Os Segredos também podem ser usados para definir variáveis de ambiente:

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

O Kubernetes realiza a decodificação base64 para você. O contêiner que é executado no pod reconhece o valor decodificado base64 ao recuperar a variável de ambiente.

Como com o ConfigMaps, os Segredos podem ser criados e manipulados na linha de comandos, o que é vantajoso ao lidar com certificados SSL.

Para obter mais informações, consulte [Segredos do Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

<!-- SSL EXAMPLE -->
