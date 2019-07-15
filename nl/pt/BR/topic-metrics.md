---

copyright:
  years: 2019
lastupdated: "2019-06-11"

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

Métricas são medidas numéricas simples capturadas como pares chave/valor. Algumas métricas incrementam contadores, outras executam agregações, como a soma de todos os valores coletados no último minuto ou o tempo médio decorrido no último minuto. Algumas métricas são apenas calibradores simples, que retornam qualquer que tenha sido o último valor observado. As métricas de captura e processamento podem ajudar a identificar e responder a possíveis problemas antes que eles se intensifiquem e causem problemas mais sérios.
{:shortdesc}

Há três fatores gerais ao falar sobre métricas em um sistema distribuído: produtores, agregadores e processadores. Há algumas combinações bastante comuns desses fatores, como o uso do Prometheus como o agregador e do Grafana para o processamento das métricas coletadas para exibição em painéis gráficos ou o uso do StatsD com o graphite.

![Os três fatores nas métricas do sistema distribuído](images/metrics-systems.png "Os três fatores nas métricas do sistema distribuído")

O produtor é, obviamente, o próprio aplicativo. Em alguns casos, o aplicativo está diretamente envolvido na produção de métricas. Em outros casos, os agentes ou outras infraestruturas observam passivamente ou instruem ativamente o aplicativo a produzir métricas em seu nome. O que acontece a seguir depende do agregador.

Métricas são transferidas do produtor para o agregador por meio de mecanismos "push" ou "pull". Alguns agregadores, como o StatsD, esperam que o aplicativo (ou um agente em seu nome) se conecte a eles para transmitir dados. Isso requer que as informações de conexão para o agregador sejam distribuídas para todos os processos do aplicativo que devem ser medidos. Outros agregadores, como o Prometheus, conectam-se periodicamente a um terminal conhecido para reunir (ou extrair) dados de métricas. Isso requer que o produtor defina e forneça um terminal que aceite extrações e que o agregador possa receber informações sobre o local dos terminais. Quando usado com o Kubernetes, o Prometheus pode descobrir terminais com base em anotações de serviço.

Finalmente, o Processador é quem faz uso de todos os dados agregados. Conforme mencionado anteriormente, serviços como o Grafana processam métricas agregadas para a visualização por meio dos painéis. O Grafana também suporta alertas de regras armazenadas e avaliadas independentemente do painel.

## Descoberta automática de terminais do Prometheus no Kubernetes
{: #prometheus-kubernetes}

O modelo baseado em pull criou seu próprio ecossistema. Outros agregadores, como o Sysdig, também podem extrair dados de métricas dos terminais do Prometheus. Isso pode significar que alguns sistemas usem as métricas do Prometheus sem usar o servidor do Prometheus.

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

Em seguida, qualquer agregador compatível com o Prometheus poderá descobrir esses terminais usando a API do Kubernetes por meio da filtragem da anotação.

## Métricas do aplicativo vs da plataforma
{: #app-platform-metrics}

Coletar métricas requer atenção. Quais pontos de dados devem ser reunidos? Ao considerar um aplicativo nativo de nuvem, é possível dividir amplamente as métricas em duas categorias, de aplicativo e de plataforma:

* As métricas de aplicativo se concentram nas entidades de domínio do aplicativo. Por exemplo, quantos logins foram concluídos nos últimos cinco minutos? Quantos usuários estão conectados atualmente? Quantas trocas foram executadas no último segundo? Reunir métricas específicas de aplicativo requer que o código customizado colete e publique as informações. A maioria das linguagens possui bibliotecas de métricas para simplificar a inclusão de métricas customizadas.
* Por outro lado, as métricas de plataforma hospedam as entidades de domínio. Por exemplo, quanto tempo leva para chamar um serviço? Quanto tempo essa consulta de banco de dados leva? Elas são alinhadas com a forma como o tráfego e o trabalho fluem pelo sistema e geralmente podem ser medidas sem quaisquer mudanças no aplicativo. Algumas estruturas de aplicativo também fornecem suporte integrado para medir essas questões, como tempos de execução de consulta.

Essas categorias não são suficientes por si só para decidir o que você precisa medir. Lembre-se de que em um sistema distribuído, há muitas coisas produzindo métricas. Há alguns poucos métodos conhecidos que tentam extrair do vasto conjunto de métricas as poucas essenciais que devem ser observadas:

* Os [quatro sinais de ouro](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") são as métricas essenciais identificadas pela equipe do Google Site Reliability Engineering (SRE) para a observabilidade em nível de serviço ao monitorar um sistema distribuído. Em suas palavras, "se puder medir apenas quatro métricas de seu sistema voltado ao usuário, concentre-se nelas". As quatro métricas são latência, tráfego, erros e saturação.
* O [método Saturação e Erros de Utilização (USE)](http://www.brendangregg.com/usemethod.html){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") foi projetado como uma lista de verificação emergencial para analisar o desempenho de um sistema. O método USE pode ser resumido em uma frase, "para cada recurso, verifique a utilização, a saturação e os erros". Nesse caso, um recurso físico ou lógico com limites físicos, como CPUs ou discos. Para cada recurso finito, você mede a utilização, a saturação e os erros. 
* O [método RED](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") é um mnemônico derivado dos quatro sinais de ouro, que define três métricas principais que devem ser medidas para cada microsserviço em sua arquitetura. Esse método é muito centrado em solicitações, especialmente quando comparado com o método USE, que se alinha bem com o design e a arquitetura de diversos aplicativos nativos de nuvem. As métricas são taxa, erros e duração. 

Há alguns elementos comuns entre esses métodos e todos rastreiam a taxa de erro, por exemplo. Mas há diferenças notáveis, também. O método USE se concentra nas métricas de infraestrutura (utilização/saturação). Os ambientes nativos de nuvem são projetados para fazer melhor uso do hardware físico (ou virtual). Essas medidas de infraestrutura verificam se o seu sistema está ou não manipulando corretamente o carregamento. O método RED se concentra inteiramente nas métricas de solicitação (latência/duração, tráfego/taxa), o que pode indicar problemas na infraestrutura (problemas de rede) ou em seus aplicativos (erros de configuração, conflitos, falhas de serviço repetidos etc.), especialmente quando combinado às taxas de erro observadas. Os sinais de ouro abrangem ambos, reunindo as métricas de infraestrutura e de solicitação em uma visualização holística.

## Definindo métricas
{: #defining-metrics}

Monitorar as métricas reunidas de um único serviço pode fornecer uma ideia de sua utilização de recursos, mas se houver diversas instâncias do serviço (devido à escala horizontal ou a implementações multiregion), será necessário realizar a distinção entre elas para isolar problemas dentro da massa de dados semelhantes que está sendo recebida. De modo geral, isso é a nomenclatura. Em alguns casos, o sistema de métricas que você está usando pode impor uma estrutura às suas métricas. O Prometheus, por exemplo, recomenda algo como `namespace_subsystem_name`, outros recomendam `namespace.subsystem.targetNoun.actioned`.

Por exemplo, se desejar rastrear o número de "negociações" que um aplicativo de negociação de ações executou, será possível capturá-lo em uma propriedade chamada `stock.trades`. Para distinguir entre as instâncias, é possível prefixar a propriedade com o ID de instância: `<instanceid>.stock.trades`. Isso suporta a coleta de valores de instâncias individuais, bem como os dados agregados, por meio de `*.stock.trades`. Mas, o que acontece quando você implementa em diversos data centers e deseja analisar as métricas dessa maneira? É possível atualizar seu nome para que seja `<datacenter>.<instanceid>.stock.trades`, mas isso invalidaria qualquer relatório usando o curinga anterior de `*.stock.trades`. Em vez disso, `*.*.stock.trades` precisaria ser usado. 

Sem o devido cuidado, o uso de propriedades nomeadas hierarquicamente por si só pode levar a padrões de curinga frágeis ligados à estrutura arbitrária de sua estratégia de nomenclatura, que não ajudam a observar as informações necessárias para garantir que seu aplicativo esteja funcionando corretamente.

Os sistemas de métricas que suportam dados dimensionais permitem associar rótulos de identificação adicionais, ou tags, com os dados de métricas. Em seguida, é possível filtrar, agrupar ou analisar as métricas coletadas usando essas dimensões adicionais sem depender de curingas e convenções de nomenclatura. Os rótulos típicos incluem o nome do terminal ou do serviço, o data center, o código de resposta, o ambiente de hospedagem (produção/preparação/desenvolvimento/teste) ou os identificadores de tempo de execução (versão Java, informações do servidor de aplicativos).

Usando o mesmo exemplo de negociações de ação, a métrica `stock.trades` está associada a diversos rótulos: `{ instanceid=..., datacenter=... }`, que permitem que o valor agregado seja filtrado ou agrupado por `instanceid` ou por `datacenter` sem depender de curingas. Há um equilíbrio entre a métrica nomeada (`stock.trades`) e os rótulos associados (compare com o exemplo hierárquico de `<datacenter>.<instanceid>.stock.trades`): cada métrica deve capturar dados significativos, com rótulos que permitam a desambiguação, quando apropriado.

Além disso, seja cauteloso ao definir rótulos. No Prometheus, por exemplo, cada combinação exclusiva de pares de rótulo chave/valor é tratada como uma série temporal separada. Uma melhor prática para garantir o bom comportamento da consulta e a coleta de dados delimitada é usar rótulos com um número finito de valores permitidos. Por exemplo, se você usar uma métrica que conta o número de erros, será possível usar o código de retorno como um rótulo, no qual os valores estarão dentro de um conjunto razoável (401, 404, 409, 500,... ), mas um rótulo para a URL com falha não seria desejado, pois esse é um conjunto ilimitado (qualquer URL de solicitação que falhou por qualquer motivo, incluindo por ser inválida).

Para obter mais informações sobre as melhores práticas para nomear métricas e rótulos (rótulos), consulte [Nomenclatura de métricas e rótulos](https://prometheus.io/docs/practices/naming/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Considerações Adicionais
{: #metrics-considerations}

Ao coletar suas métricas, lembre-se de que um caminho de falha é, muitas vezes, muito diferente de um caminho de sucesso. Por exemplo, se a falha envolveu tempos limite e coleta de rastreio de pilha, uma resposta de erro em um recurso HTTP pode levar muito mais tempo do que uma resposta bem-sucedida. Conte e trate caminhos de erro separadamente de solicitações bem-sucedidas.

Um sistema distribuído tem variações naturais em determinadas medidas. Erros ocasionais são normais, uma vez que as solicitações podem ser direcionadas para processos em inicialização ou encerramento. Filtre os dados brutos para capturar quando essa variação natural começa a exceder um intervalo válido. Por exemplo, divida métricas em depósitos. Categorize a duração da solicitação em categorias, como 'menor/mais rápida', 'média/normal' e 'maior/mais longa', conforme observado dentro de um espaço de tempo deslizante. Se as durações de solicitação forem continuamente inseridas no depósito "maior/mais longo", será possível identificar um problema. As métricas de histograma ou resumo são geralmente usadas para esse tipo de dados. Para obter mais informações, consulte [Histogramas e resumos](https://prometheus.io/docs/practices/histograms/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

Como um desenvolvedor de aplicativos, certifique-se de que seus aplicativos ou serviços estejam emitindo métricas com nomes e rótulos que sigam convenções corporativas abrangentes para suportar os esforços de monitoramento que estão focados em caminhos de ponta a ponta centrais para seus negócios. Para obter mais informações, consulte [Monitorando sistemas distribuídos](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
