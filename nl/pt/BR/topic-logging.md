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

# Criação de log
{: #logging}

As mensagens de log são sequências com informações contextuais que pertencem ao estado e atividade de um microsserviço quando a entrada de log é criada. Os logs são necessários para diagnosticar como e por que os serviços falham. Eles desempenham uma função de suporte para as métricas no monitoramento do funcionamento do aplicativo.
{:shortdesc}

Certifique-se de gravar as entradas de log diretamente na saída padrão e nos fluxos de erro. Isso torna as entradas de log visualizáveis usando ferramentas de linha de comandos e permite que os serviços de encaminhamento de log sejam configurados no nível de infraestrutura para gerenciar a coleção de logs e o gerenciamento de dados.

A manipulação de arquivos de log requer mais atenção, caso não seja possível configurar um aplicativo conteinerizado para gravar logs em fluxos de saída e de erro padrão.

* Uma opção é usar um volume para os dados de log, seja uma montagem de ligação simples para o desenvolvimento e o teste locais ou um Volume Persistente adequado como parte de uma implementação do Kubernetes. Um [agente de criação de log ou sidecar dedicado](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") pode ler um volume compartilhado para encaminhar logs para um agregador centralizado. A rotação de log deve ser configurada explicitamente para controlar a quantidade de dados de log armazenados nos volumes.
* Outra opção é usar agentes ou bibliotecas de aplicativos para encaminhar logs diretamente aos agregadores. Essa opção pode oferecer alguma complexidade de configuração em ambientes de implementação

## Criação de log JSON
{: #json-logging}

À medida que seu aplicativo se desenvolve ao longo do tempo, a natureza do que você registra pode mudar. Ao usar um formato de log JSON, você obtém os benefícios a seguir:

* Os logs são indexáveis, o que facilita muito mais a procura de um corpo agregado de logs.
* Os logs são resilientes à mudança, pois a análise não é dependente da posição dos elementos em uma sequência.

Embora os logs formatados em JSON sejam mais fáceis para a análise das máquinas, são mais difíceis para a leitura humana. Considere usar variáveis de ambiente para alternar o formato de log entre texto simples para desenvolvimento local e depuração, e logs formatados por JSON para armazenamento e agregação de longo prazo.

Os analisadores JSON da linha de comandos, como a ferramenta de Consulta JSON (jq), podem ser usados para criar visualizações de logs formatados em JSON legíveis para os humanos. No exemplo a seguir, os logs são canalizados por meio do grep para assegurar que o campo de mensagem esteja lá antes que o jq analise a linha:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## Visualizando logs usando o `kubectl`
{: #view-logs-kube}

Os logs enviados para os fluxos de saída e de erro padrão podem ser visualizados no Kubernetes por meio do console ou de comandos `kubectl` do seguinte formato: `kubectl logs<podname>`.

Se você usar um namespace customizado, como o stock-trader, inclua-o no comando, `kubectl logs -n stock-trader <podname>`.

Se houver múltiplos contêineres para cada pod, como há com os sidecars do Istio, também será necessário especificar o contêiner. No exemplo a seguir, o namespace stock-trader é usado para visualizar logs do contêiner `portfolio` no Pod `portfolio-54b4d579f7-4zvzk`:

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

Para logs no formato JSON, é possível usar a `jq` para extrair um campo de mensagem, por exemplo:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

As entradas de log são canalizadas por meio de `grep` para assegurar que o `jq` analise as linhas com um campo da mensagem.
{: note}
