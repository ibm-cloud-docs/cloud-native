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

# Métricas
{: #metrics}

Métricas são medidas numéricas simples capturadas como pares chave-valor. Algumas métricas estão incrementando contadores e outras executam agregações. Por exemplo, a soma de todos os valores coletados dentro do último minuto ou o tempo médio decorrido no último minuto. Algumas métricas são medidores simples que retornam qualquer que seja o último valor observado. As métricas de captura e processamento podem ajudar a identificar e responder a possíveis problemas antes que eles se intensifiquem e causem problemas mais sérios.
{:shortdesc}

Há três fatores gerais em métricas do sistema distribuído: produtores, agregadores e processadores. Há combinações comuns de fatores, como usar o Prometheus como o agregador com o Grafana processando as métricas coletadas para exibição em painéis gráficos. Outra é StatsD com graphite.

![Os três fatores nas métricas do sistema distribuído](images/metrics-systems.png "Os três fatores nas métricas do sistema distribuído")

O produtor é o aplicativo. Em alguns casos, o aplicativo está diretamente envolvido na produção de métricas. Em outros casos, os agentes ou outras infraestruturas observam passivamente ou instruem ativamente o aplicativo a produzir métricas em seu nome. O que acontece a seguir depende do agregador.

Métricas são transferidas do produtor para o agregador por meio de mecanismos "push" ou "pull". Alguns agregadores, como o StatsD, esperam que o aplicativo se conecte ao agregador para transmitir os dados. As informações de conexão para o agregador devem ser distribuídas para todos os processos do aplicativos que são medidos. Outros agregadores, como o Prometheus, se conectam periodicamente a um terminal conhecido para reunir os dados de métricas. Isso requer que o produtor defina e forneça um terminal que aceite extrações e que o agregador possa receber informações sobre o local dos terminais. Quando usado com o Kubernetes, o Prometheus pode descobrir terminais com base em anotações de serviço.

Finalmente, o processador usa todos os dados agregados. Serviços como as métricas agregadas de processo do Grafana para visualização usando painéis. O Grafana também suporta alertas de regras armazenadas e avaliadas independentemente do painel.

## Descoberta automática de terminais do Prometheus no Kubernetes
{: #prometheus-kubernetes}

O modelo baseado em pull cria seu próprio ecossistema. Outros agregadores, como o Sysdig, também podem extrair dados de métricas dos terminais do Prometheus. Isso pode significar que alguns sistemas usem as métricas do Prometheus sem usar o servidor do Prometheus.

Em ambientes do Kubernetes, as anotações são usadas para a descoberta de terminal do Prometheus. Por exemplo, um serviço que fornece um terminal `/metrics` por HTTP na porta 8080 inclui as anotações a seguir para a definição de serviço:

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

Qualquer agregador compatível com o Prometheus pode, então, descobrir esses terminais usando a API do Kubernetes filtrando por anotação.

## Métricas do aplicativo vs da plataforma
{: #app-platform-metrics}

Quais pontos de dados precisam ser reunidos? Com um aplicativo nativo de nuvem, é possível dividir amplamente as métricas em duas categorias, aplicativo e plataforma:

* As métricas de aplicativo se concentram nas entidades de domínio do aplicativo. Por exemplo, quantos logins foram concluídos nos últimos cinco minutos? Quantos usuários estão conectados atualmente? Quantas trocas foram executadas no último segundo? Reunir métricas específicas de aplicativo requer que o código customizado colete e publique as informações. A maioria das linguagens possui bibliotecas de métricas para simplificar a inclusão de métricas customizadas.
* Por outro lado, as métricas de plataforma hospedam as entidades de domínio. Por exemplo, quanto tempo leva para chamar um serviço? Quanto tempo essa consulta de banco de dados leva? Eles são alinhados com a forma como o tráfego e o trabalho estão fluindo pelo sistema e podem, muitas vezes, ser medidos sem nenhuma mudança no aplicativo. Algumas estruturas de aplicativo também fornecem suporte integrado para medir essas questões, como tempos de execução de consulta.

Essas categorias não são suficientes por si só para decidir o que você precisa medir. Em um sistema distribuído, há muitas coisas produzindo métricas. Há alguns métodos conhecidos que tentam reduzir o vasto conjunto de métricas para as poucas essenciais que devem ser observadas:

* Os [quatro sinais dourados](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") são as métricas essenciais para monitorar um sistema. Eles são identificados pela equipe do Google Site Reliability Engineering (SRE) para a observabilidade de nível de serviço durante o monitoramento. Em suas palavras, "se puder medir apenas quatro métricas de seu sistema voltado ao usuário, concentre-se nelas". As quatro métricas são latência, tráfego, erros e saturação.
* O [método Saturação e Erros de Utilização (USE)](http://www.brendangregg.com/usemethod.html){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") foi projetado como uma lista de verificação emergencial para analisar o desempenho de um sistema. O método USE pode ser resumido em uma frase, "para cada recurso, verifique a utilização, a saturação e os erros". Nesse caso, um recurso físico ou lógico com limites físicos, como CPUs ou discos. Para cada recurso finito, você mede a utilização, a saturação e os erros. 
* O [método RED](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") é um mnemônico derivado dos quatro sinais dourados que define três métricas principais que você mede para cada microsserviço em sua arquitetura. Este método é centrado na solicitação, especialmente quando comparado ao método USE, que se alinha bem com o design e a arquitetura de muitos aplicativos nativos de nuvem. As métricas são taxa, erros e duração. 

O método USE é centrado nas métricas de infraestrutura. Os ambientes nativos de nuvem são projetados para fazer melhor uso do hardware físico ou virtual. Essas medidas de infraestrutura analisam se o seu sistema está manipulando corretamente o carregamento. O método RED foca inteiramente nas métricas da solicitação que indicam problemas com a infraestrutura e os aplicativos. Os sinais de ouro abrangem ambos, reunindo as métricas de infraestrutura e de solicitação em uma visualização holística.

## Definindo métricas
{: #defining-metrics}

Métricas de monitoramento de um único serviço podem fornecer uma ideia de sua utilização de recurso. Mas, se houver múltiplas instâncias do serviço, será necessário distingui-las para isolar problemas com dados de entrada semelhantes. De modo geral, isso é a nomenclatura. Em alguns casos, o sistema de métricas que você está usando pode impor uma estrutura às suas métricas. O Prometheus recomenda algo como `namespace_subsystem_name`, outros recomendam `namespace.subsystem.targetNoun.actioned`.

Por exemplo, se desejar rastrear o número de "negociações" que um aplicativo de negociação de ações executou, será possível capturá-lo em uma propriedade chamada `stock.trades`. Para distinguir entre as instâncias, é possível prefixar a propriedade com o ID de instância: `<instanceid>.stock.trades`. Isso suporta a coleta de valores de instância individuais e agrega dados usando `*.stock.trades`. Mas, o que acontece quando você implementa em diversos data centers e deseja analisar as métricas dessa maneira? É possível atualizar seu nome para que seja `<datacenter>.<instanceid>.stock.trades`, mas isso invalidaria qualquer relatório usando o curinga anterior de `*.stock.trades`. Em vez disso, `*.*.stock.trades` precisaria ser usado. 

O mal uso das propriedades apenas hierarquicamente nomeadas pode levar a padrões de curinga frágeis ligados à estrutura arbitrária de sua estratégia de nomenclatura. Os frágeis padrões de curinga não ajudam a observar as informações que você precisa para assegurar que seu aplicativo esteja funcionando bem.

Com os sistemas de métricas que suportam dados dimensionais, é possível associar rótulos de identificação ou tags. Os rótulos típicos incluem o nome do terminal ou do serviço, o data center, o código de resposta, o ambiente de hospedagem (produção/preparação/desenvolvimento/teste) ou os identificadores de tempo de execução (versão Java, informações do servidor de aplicativos).

Usando o mesmo exemplo de negociação de ações, a métrica `stock.trades` está associada a vários rótulos, como `{ instanceid=..., datacenter=... }`. Isso permite que o valor agregado seja filtrado ou agrupado por `instanceid` ou `datacenter` sem depender de curingas. Há um equilíbrio entre a métrica nomeada, `stock.trades` e os rótulos associados. Cada métrica captura dados significativos, com rótulos para a desambiguação.

Defina rótulos com cuidado. No Prometheus, cada combinação única de pares chave:valor é tratada como uma série temporal separada. Uma boa prática para assegurar o bom comportamento da consulta e a coleta de dados delimitada é usar rótulos com um número finito de valores permitidos. 

Se você usar uma métrica que conta o número de erros, será possível usar o código de retorno como um rótulo, em que os valores estão dentro de um conjunto razoável. Não rotule para a URL com falha. É um conjunto desvinculado.

Para obter mais informações sobre as melhores práticas para nomear métricas e rótulos, consulte [Nomenclatura de métrica e rótulo](https://prometheus.io/docs/practices/naming/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Mais considerações
{: #metrics-considerations}

Frequentemente, um caminho de falha é muito diferente de um caminho de sucesso. Por exemplo, uma resposta de erro em um recurso HTTP poderá levar muito mais tempo do que uma resposta bem-sucedida se a falha envolver tempos limites e coleta de rastreio de pilha. Conte e trate caminhos de erro separadamente de solicitações bem-sucedidas.

Um sistema distribuído tem variações naturais em determinadas medidas. Erros ocasionais são normais, uma vez que as solicitações podem ser direcionadas para processos em inicialização ou encerramento. Filtre os dados brutos para capturar quando essa variação natural excede um intervalo válido. Por exemplo, divida métricas em depósitos. Categorize a duração da solicitação em categorias, como 'menor/mais rápida', 'média/normal' e 'maior/mais longa', conforme observado dentro de um espaço de tempo deslizante. Se as durações de solicitação forem continuamente inseridas no depósito "maior/mais longo", será possível identificar um problema. As métricas de histograma ou resumo são geralmente usadas para esse tipo de dados. Para obter mais informações, consulte [Histogramas e resumos](https://prometheus.io/docs/practices/histograms/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

Assegure-se de que seus aplicativos e serviços emitam métricas com nomes e rótulos que sigam as convenções de toda a organização que suportam os esforços de monitoramento de seus negócios. Para obter mais informações, consulte [Monitorando sistemas distribuídos](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
