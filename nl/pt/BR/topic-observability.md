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

# Observabilidade, telemetria e monitoramento
{: #observability-cn}

Há uma mudança de cultura a respeito do monitoramento resultante de uma migração para a abordagem nativa de nuvem. Embora os aplicativos em ambientes locais e nativos de nuvem sejam esperados como altamente disponíveis e resilientes à falha, os métodos que são usados para atingir esses objetivos são diferentes. Como resultado, o propósito do monitoramento muda: em vez de monitorar para evitar falhas, monitore para gerenciá-las. 
{:shortdesc}

Em ambientes locais, a infraestrutura e o middleware são provisionados com base nos padrões de capacidade planejada e de alta disponibilidade, por exemplo, ativo/ativo ou ativo/passivo. Falhas inesperadas podem ser complexas nesse ambiente, exigindo um esforço significativo para a determinação de problemas e a recuperação. O monitoramento externo é executado por agentes que examinam a utilização de recursos para evitar as classes conhecidas de falhas. Como um exemplo, considere o ajuste do tamanho de heap, os tempos limite e as políticas de coleta de lixo para aplicativos Java.

Um aplicativo nativo de nuvem é composto de microsserviços independentes e serviços auxiliares necessários. Mesmo que um aplicativo nativo de nuvem como um todo deva permanecer disponível e continuar a funcionar, as instâncias de serviço individuais iniciarão ou pararão para se ajustarem aos requisitos de capacidade ou para se recuperarem da falha. 

## Observabilidade
{: #observability}

O monitoramento desse sistema fluido requer que cada participante seja *observável*. Cada entidade deve produzir dados apropriados para suportar a detecção e o alerta de problemas automatizados, a depuração manual, quando necessária, e a análise do funcionamento do sistema (análises e tendências históricas).

Quais tipos de dados um serviço deve produzir para ser observável?

* **Verificações de funcionamento** (geralmente terminais HTTP customizados) ajudam orquestradores, como o Kubernetes ou o Cloud Foundry, na execução de ações automatizadas para manter o funcionamento geral do sistema.
* **Métricas** são uma representação numérica de dados que são coletados em intervalos em uma série temporal. Os dados numéricos de série temporal são fáceis de armazenar e consultar, o que ajuda ao procurar tendências históricas. Durante um período mais longo, os dados numéricos podem ser compactados em agregados menos granulares, diários ou semanais, por exemplo.
* **Entradas de log** representam eventos discretos. As entradas de log são essenciais para a depuração, pois incluem, muitas vezes, rastreios de pilha e outras informações contextuais que podem ajudar a identificar a causa raiz das falhas observadas.
* **Rastreamento distribuído, de solicitação ou de ponta a ponta** captura o fluxo de ponta a ponta de uma solicitação por meio do sistema. O rastreio captura essencialmente os relacionamentos entre os serviços (os serviços nos quais a solicitação executou touch) e a estrutura de trabalho por meio do sistema (processamento síncrono ou assíncrono, relacionamentos "filho do" ou "seguir do").

## Telemetria
{: #telemetry}

Os aplicativos nativos de nuvem dependem do ambiente para *telemetria*, que é a coleta automática e a transmissão de dados para localizações centralizadas para análise subsequente. Isso é enfatizado por um dos doze fatores pelos quais os estados tratam logs como fluxos de eventos e se estende a todos os dados produzidos por um microsserviço para garantir que ele possa ser observado.

O Kubernetes possui alguns recursos de telemetria integrados, como o Heapster, mas a telemetria é mais provavelmente fornecida por outros sistemas que se integram ao plano de controle do Kubernetes. Como exemplo, dois componentes do Istio, o Mixer e o Envoy, atuam em conjunto para [coletar de forma transparente a telemetria dos aplicativos implementados](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

Falhas não são mais ocorrências raras que causam interrupções. Dividir um aplicativo monolítico em microsserviços envia mais do caminho da linha principal por push para a rede, aumentando o impacto da latência e de outros problemas de rede. As solicitações também atingem processos que não estão prontos para o trabalho por diversas razões. Os serviços são reiniciados automaticamente no caso de uma insuficiência de recursos e as estratégias de tolerância a falhas permitem que o sistema como um todo continue funcionando. A intervenção manual para falhas individuais não é útil ou viável nesse tipo de ambiente.

## Monitoramento
{: #monitoring}

Os conceitos de observabilidade e telemetria ajudam a destacar algumas diferenças significativas sobre como os aplicativos nativos de nuvem em sistemas distribuídos de larga escala são monitorados. Lembre-se de que os processos em ambientes nativos de nuvem são temporários. Três dos doze fatores (processos, simultaneidade e descartabilidade) enfatizam esse ponto. Processos monolíticos pré-alocados e de longa execução são substituídos, ou auxiliados, por diversos processos de curta duração que são iniciados e interrompidos em resposta à carga para o dimensionamento horizontal ou caso não estejam funcionando corretamente. A telemetria é crítica quando os dados devem ser coletados e persistidos em outro lugar para evitar que sejam perdidos, conforme a criação e destruição dos processos (contêineres). Ela também é frequentemente necessária por motivos de conformidade. 

Por todas as razões descritas anteriormente, o monitoramento muda o foco: em vez de monitorar o comportamento e o funcionamento de recursos (processos ou máquinas individuais), o estado do sistema é monitorado como um todo. Cada serviço individual produz dados que alimentam essa visualização agregada.

## Rastreamento versus criação de log versus métricas
{: #trace-log-metrics}

:FIXME -- comparação de refrase como resumo do tópico:

A criação de log é usada quando o desenvolvedor deseja gerar explicitamente alguma mensagem para que alguém veja. Ela é codificada diretamente na classe Java, incluindo a transmissão de valores de variáveis relevantes. Quando ocorrerem problemas, os logs são úteis para propósitos de depuração, mostrando onde ocorreu uma falha, como um rastreio de pilha para uma exceção que foi lançada. Você usa o *Kibana* para ver uma visualização federada de tais logs em pods/microsserviços.

O rastreio ocorre automaticamente, portanto, o desenvolvedor não executa a ação. Por exemplo, é possível configurar o Liberty para enviar registros de rastreio para um servidor de rastreio compatível com Open Tracing sempre que qualquer método anotado JAX-RS for chamado. Desta forma, você tem um registro de auditoria do que foi chamado quando, por quem e quanto tempo levou. Também é possível aumentar esse rastreio, como para incluir informações sobre quais métodos privados foram chamados em seu código incluindo as anotações do Open Tracing para tais métodos que são rastreados. 

É possível usar uma ferramenta como *Zipkin* ou *Jaeger* para mostrar uma visualização federada de rastreios em pods/microsserviços. Um *mesh de serviço* também pode fornecer rastreio automático de chamadas que são transmitidas por meio de um sidecar para o seu contêiner.  

As métricas são usadas para rastrear valores agregados. Em vez de perder tempo com um rastreio ou um log para ver com que frequência alguém está criando portfólios, é possível tornar isso uma métrica de contador customizada. É possível rotular a sua implementação de forma que o *Prometheus* a "toque" levemente, como ao obter periodicamente o URI de /metrics em seus pods. Em seguida, é possível usar uma ferramenta como o *Grafana* para obter uma visualização federada das métricas nos pods/microsserviços.

É possível olhar para o rastreamento para descobrir que "esse método foi chamado". Mas você olharia para a criação de log para descobrir que "isso é o que aconteceu dentro desse método quando ele foi chamado". E você olharia para as métricas para ver "quantas vezes ele foi chamado". Geralmente, o rastreio está implícito, ocorrendo automaticamente sem qualquer esforço da parte do desenvolvedor. Considerando que a criação de log é explícita, exigindo que o desenvolvedor codifique o envio de informações que possam ser relevantes para a análise de um problema já ocorrido. As métricas também são explícitas, na medida em que você precisa incluir a anotação no método apropriado de seu código (embora muitas vezes as métricas padrão de baixo nível estejam disponíveis sem esforço da parte do programador, como para uso de memória, uso de CPU, contagens de encadeamentos).

Geralmente, as métricas são úteis para a análise de dados, enquanto a criação de log é mais útil para propósitos de determinação de problemas e o rastreio é mais útil para entender o fluxo de controle de microsserviço para microsserviço.
