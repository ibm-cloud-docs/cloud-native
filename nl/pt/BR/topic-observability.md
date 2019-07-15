---

copyright:
  years: 2019
lastupdated: "2019-04-09"

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

Há uma mudança de cultura a respeito do monitoramento resultante de uma migração para a abordagem nativa de nuvem. Embora seja esperado que os aplicativos em ambos os ambientes local e na nuvem sejam altamente disponíveis e resilientes a falhas, os métodos usados para alcançar esses objetivos são muito diferentes. Como resultado, o propósito do monitoramento muda: em vez de monitorar para evitar falhas, monitore para gerenciá-las. 
{:shortdesc}

Em ambientes locais, a infraestrutura e o middleware são provisionados com base nos padrões de capacidade planejada e alta disponibilidade, por exemplo, ativo/ativo ou ativo/passivo. Falhas inesperadas podem ser complexas nesse ambiente, exigindo um esforço significativo para a determinação de problemas e a recuperação. O monitoramento externo é executado por agentes que examinam a utilização de recursos para evitar as classes conhecidas de falhas. Como um exemplo, considere o ajuste do tamanho de heap, os tempos limite e as políticas de coleta de lixo para aplicativos Java.

Um aplicativo nativo de nuvem é composto de microsserviços independentes e serviços auxiliares necessários. Mesmo que um aplicativo nativo de nuvem, como um todo, deva permanecer disponível e continuar funcionando, as instâncias de serviço individuais serão iniciadas ou interrompidas, conforme necessário, para a realização de ajustes relativos aos requisitos de capacidade ou para a recuperação de uma falha. 

## Observabilidade
{: #observability}

O monitoramento desse sistema fluido requer que cada participante seja *observável*. Cada entidade deve produzir dados apropriados para suportar a detecção e o alerta de problemas automatizados, a depuração manual, quando necessária, e a análise do funcionamento do sistema (análises e tendências históricas).

Quais tipos de dados um serviço deve produzir para ser observável?

* **Verificações de funcionamento** (geralmente terminais HTTP customizados) ajudam orquestradores, como o Kubernetes ou o Cloud Foundry, na execução de ações automatizadas para manter o funcionamento geral do sistema.
* **Métricas** são uma representação numérica dos dados coletados em intervalos de uma série temporal. Os dados numéricos de série temporal são fáceis de armazenar e consultar, o que ajuda ao procurar tendências históricas. Durante um período mais longo, os dados numéricos podem ser compactados em agregações menos granulares, diárias ou semanais, por exemplo.
* **Entradas de log** representam eventos discretos que ocorreram ao longo do tempo. As entradas de log são essenciais para a depuração, pois incluem, muitas vezes, rastreios de pilha e outras informações contextuais que podem ajudar a identificar a causa raiz das falhas observadas.
* **Rastreamento distribuído, de solicitação ou de ponta a ponta** captura o fluxo de ponta a ponta de uma solicitação por meio do sistema. O rastreamento captura essencialmente os relacionamentos entre os serviços (os serviços atingidos pela solicitação) e a estrutura de trabalho fluindo por meio do sistema (processamento síncrono ou assíncrono, relacionamentos child-of follows-from).

## Telemetria
{: #telemetry}

Os aplicativos nativos de nuvem devem contar com o ambiente para a *telemetria*, que é a coleta e a transmissão de dados automáticas aos locais centralizados para a análise subsequente. Isso é enfatizado por um dos doze fatores pelos quais os estados tratam logs como fluxos de eventos e se estende a todos os dados produzidos por um microsserviço para garantir que ele possa ser observado.

O Kubernetes possui alguns recursos de telemetria integrados, como o Heapster, mas a telemetria é mais provavelmente fornecida por outros sistemas que se integram ao plano de controle do Kubernetes. Como exemplo, dois componentes do Istio, o Mixer e o Envoy, atuam em conjunto para [coletar de forma transparente a telemetria dos aplicativos implementados](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

Falhas não são mais ocorrências raras que causam interrupções. Dividir um aplicativo monolítico em microsserviços envia mais do caminho da linha principal por push para a rede, aumentando o impacto da latência e de outros problemas de rede. As solicitações também atingem processos que não estão prontos para o trabalho por diversas razões. Os serviços são reiniciados automaticamente no caso de uma insuficiência de recursos e as estratégias de tolerância a falhas permitem que o sistema como um todo continue funcionando. A intervenção manual para falhas individuais não é particularmente útil ou factível nesse tipo de ambiente.

## Monitoramento
{: #monitoring}

Os conceitos de observabilidade e telemetria ajudam a destacar algumas diferenças significativas sobre como os aplicativos nativos de nuvem em sistemas distribuídos de larga escala são monitorados. Lembre-se de que os processos em ambientes nativos de nuvem são temporários. Três dos doze fatores (processos, simultaneidade e descartabilidade) enfatizam esse ponto. Processos monolíticos pré-alocados e de longa execução são substituídos, ou auxiliados, por diversos processos de curta duração que são iniciados e interrompidos em resposta à carga para o dimensionamento horizontal ou caso não estejam funcionando corretamente. A telemetria é crítica quando os dados devem ser coletados e persistidos em outro lugar para evitar que sejam perdidos, conforme a criação e destruição dos processos (contêineres). Ela também é frequentemente necessária por motivos de conformidade. 

Por todas as razões descritas anteriormente, o monitoramento muda o foco: em vez de monitorar o comportamento e o funcionamento de recursos (processos ou máquinas individuais), o estado do sistema é monitorado como um todo. Cada serviço individual produz dados que alimentam essa visualização agregada.

