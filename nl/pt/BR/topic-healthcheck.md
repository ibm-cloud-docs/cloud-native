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

# Verificação de funcionamento
{: #healthcheck}

As verificações de funcionamento fornecem um mecanismo simples dentro de sistemas automatizados para examinar o status de funcionamento de uma instância individual. O sistema responde aos eventos de inspeção de funcionamento, substituindo uma instância com falha, atualizando tabelas de roteamento ou comunicando os estados de funcionamento resultantes para o usuário.
{:shortdesc}

O Kubernetes define dois mecanismos integrais para verificar o funcionamento de um contêiner:

* Uma análise de prontidão é usada para indicar se o processo pode manipular solicitações que são roteáveis. O Kubernetes não roteia trabalho para um contêiner com uma análise de prontidão com falha. Uma análise de prontidão falhará se um serviço estiver ocupado, não terminar a inicialização, estiver sobrecarregado ou não for capaz de processar as solicitações. 
* Uma análise de atividade é usada para indicar se o processo deve ser reinicializado. O Kubernetes para e reinicia um contêiner com uma análise de atividade com falha para garantir que os Pods em estado defeituoso sejam finalizados e substituídos. Uma análise de vivacidade falhará se o serviço estiver em um estado irrecuperável, por exemplo, se ocorrer uma condição de falta de memória. As verificações de vivacidade simples que sempre retornam uma resposta OK podem identificar contêineres em um estado inconsistente, o que pode acontecer quando o processo que atende as solicitações trava, mas o contêiner ainda está em execução.

As análises de prontidão e vivacidade são ambas definidas usando uma estrutura semelhante que inclui atraso de tempo e intervalos de novas tentativas, períodos de tolerância a falhas, tempos limite e a definição da implementação da análise. A análise pode ser implementada executando um comando, verificando a conectividade de um terminal TCP ou executando uma chamada HTTP. A mesma implementação de análise pode frequentemente ser usada para propósitos de prontidão ou atividade, mas o atraso e os intervalos de novas tentativas precisam ser ajustados para o propósito específico.

## Entendendo e aplicando análises
{: #kubernetes-probes}

Fundamentalmente, o desenvolvimento de aplicativos nativos de nuvem é baseado no princípio de que os processos de contêiner realmente falham, mas são prontamente substituídos por um novo contêiner. Isso ocorre em resposta a eventos inesperados, como falha de contêiner ou de máquina. Também é devido a eventos operacionais, como escala horizontal e novos lançamentos de imagem de aplicativo. As verificações de prontidão asseguram que as novas instâncias de contêiner estejam prontas para receber trabalho antes do tráfego de roteamento. As mesmas verificações impedem que o tráfego seja roteado para instâncias saíram ou estão sendo destruídas.

Quando as verificações de prontidão não são definidas, o Kubernetes tem poucos insights sobre se uma instância de contêiner está pronta para manipular o tráfego e roteia-o imediatamente após o processo de contêiner ser iniciado. Sem verificações de prontidão, os aplicativos têm mais probabilidade de experimentar tempos limite de conectividade e respostas de rejeição de conexão quando o trabalho é roteado para uma instância que não está pronta para entregar a solicitação. As verificações de prontidão reduzem, mas não eliminam completamente, os erros de conectividade do cliente.

As verificações de vivacidade que são destinadas à identificação são menos frequentes e representam a exceção. Quando um processo entra em um estado que não tem uma recuperação possível, ele torna-se efetivamente inoperante. Isso pode ocorrer devido a condições de falta de memória ou a um conflito que é causado por um erro de programação. A melhor maneira de recuperar é finalizar o contêiner, que finaliza qualquer processamento que esteja acontecendo atualmente no contêiner como um resultado. O término ou a reinicialização dos loops pode ocorrer no aplicativo em que os contêineres não são capazes de ficar on-line antes de serem finalizados e substituídos.

As análises de prontidão e atividade afetam o sistema de diferentes maneiras. Isso pode ser pensado em termos de transição de estado, em que o estado positivo de uma verificação de prontidão é roteável e o estado negativo não é roteável. Da mesma forma, o estado positivo de uma verificação de atividade representa um contêiner que está em execução normalmente e o estado negativo representa um contêiner que está inoperante. Quando um contêiner é iniciado, o estado de prontidão é inicialmente negativo e ele entra em um estado positivo apenas depois que o contêiner está funcionando corretamente. Uma verificação de atividade é iniciada em um estado positivo e entra em um estado negativo somente quando o processo se torna inoperante.

Configurar uma verificação de prontidão agressivamente, como com um atraso inicial baixo, tem pouco efeito, pois executar a análise muito antecipadamente não faz com que a verificação de prontidão mude de estado. No entanto, uma verificação de vivacidade agressiva, onde a análise dispara muito antecipadamente, causa uma mudança de transição de estado, fazendo com que o sistema finalize o contêiner mais cedo do que o desejado.

## Melhores práticas para configurar análises
{: #probe-recommendation}

Se você implementar uma análise de funcionamento usando HTTP, considere os códigos de status HTTP a seguir para prontidão, vivacidade e funcionamento.

| Status    |  Prontidão            |  Atividade             |
|----------|-----------------------|-----------------------|
|          | Não OK causa nenhum carregamento | Não OK causa reinicialização |
| Iniciando | 503 - Indisponível     | 200 - OK              |
| Para cima       | 200 - OK              | 200 - OK              |
| Parando | 503 - Indisponível     | 200 - OK              |
| Inativo     | 503 - Indisponível     | 503 - Indisponível     |
| Com erro  | 500 - Erro do servidor    | 500 - Erro do servidor    |
{: caption="Tabela 1. Códigos de status HTTP" caption-side="bottom"}

Os terminais de verificação de funcionamento não devem requerer autorização ou autenticação. Como essas proteções não entram em vigor em terminais de análise de funcionamento, restrinja quaisquer implementações de análise de HTTP a solicitações GET que não modifiquem nenhum dado. Nunca retorne dados que identifiquem as especificidades sobre o ambiente, como o sistema operacional, a linguagem de implementação ou as versões de software. Eles podem ser usados para estabelecer um vetor de ataque.

Uma análise de vivacidade é deliberada sobre o que ela verifica porque uma falha resulta em término imediato do processo. Evite métricas ambíguas que indicam apenas ocasionalmente um processo com falha, por exemplo, um terminal HTTP simples que sempre retorna `{"status": "UP"}` com um código de status 200. Portanto, a maioria dos processos que estão no estado inoperante falha nessa verificação, acionando corretamente uma reinicialização.

As verificações de funcionamento ocorrem em intervalos frequentes, o que pode causar mais sobrecarga. As análises de prontidão e vivacidade testam apenas a viabilidade dos serviços auxiliares, como bancos de dados ou outros microsserviços, em seu resultado quando não há um fallback aceitável. Para uma análise de vivacidade, uma verificação auxiliar será incluída se um resultado malsucedido fizer com que o contêiner local entre em um estado irrecuperável. Uma análise de prontidão verificará um serviço auxiliar apenas quando o contêiner local for incapaz de manipular solicitações se ele falhar, mas a condição for recuperável.

Ao configurar o atraso de tempo inicial, uma análise de prontidão usa o valor mais baixo provável e uma verificação de vivacidade usa o valor de tempo mais alto provável. Por exemplo, se um servidor de aplicativos tende a ser iniciado em 30 segundos, um atraso de prontidão típico é de 10 segundos. A verificação de vivacidade usa um valor de 60 segundos para assegurar a conclusão da inicialização do servidor antes que os estados suscetíveis a término sejam verificados.

O atributo *periodSeconds* para as decisões de roteamento será geralmente configurado para um valor de dígito único se a implementação da análise for relativamente leve. Por exemplo, uma análise HTTP que retorna um status 200 OK sem um processamento substancial do lado do servidor tem uma carga mínima do processador e pode ser prontamente repetida a cada 1 a 5 segundos.

## Configurando análises no Kubernetes
{: #probe-config}

Declare as análises de atividade e prontidão com a implementação do seu Kubernetes no elemento do contêiner. As duas análises usam os mesmos parâmetros de configuração:

| Parâmetro | Descrição |
|-----------|-------------|
| *initialDelaySeconds* | A quantidade de tempo que o Kubernetes espera após a criação do contêiner antes da primeira análise. |
| *periodSeconds* | A frequência com que o kubelet analisa o serviço. O padrão é 1. |
| *timeoutSeconds* | Com que rapidez a análise atinge o tempo limite. O valor padrão e o mínimo é 1. |
| *successThreshold* | O número necessário de execuções bem-sucedidas da análise após uma falha. O valor padrão e mínimo é 1. O valor deve ser 1 para as análises de vivacidade. |
| *failureThreshold* | O número de vezes que o Kubernetes tenta reiniciar um pod antes de desistir quando o pod inicia e a análise falha. O valor mínimo é 1 e o valor padrão é 3. |

  Para uma análise de atividade, desistir significa reiniciar o Pod. Para uma análise de prontidão, desistir significa marcar o Pod como não preparado.
  {: note}

Para evitar os ciclos de reinicialização, configure o parâmetro `livenessProbe.initialDelaySeconds` para um valor seguramente maior do que o tempo que o seu serviço leva para ser inicializado. Em seguida, será possível usar um valor menor para que o atributo `readinessProbe.initialDelaySeconds` roteie solicitações para o serviço assim que ele estiver pronto.

O exemplo a seguir pode ser semelhante a uma configuração de exemplo (observe os valores de caminho e porta):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

Para obter mais informações, consulte [Configurar análises de prontidão e atividade no Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").
