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

# Verificação de funcionamento
{: #healthcheck}

As verificações de funcionamento fornecem um mecanismo simples dentro de sistemas automatizados para examinar o status de funcionamento de uma instância individual. Em seguida, o sistema responde a esses eventos de inspeção de funcionamento executando uma ação, como a substituição de uma instância com falha, a atualização de tabelas de roteamento ou a comunicação dos estados de funcionamento resultantes para o usuário.
{:shortdesc}

O Kubernetes define dois mecanismos integrais para verificar o funcionamento de um contêiner:

* Uma análise de prontidão é usada para indicar se o processo pode manipular solicitações (é roteável). O Kubernetes não roteia trabalho para um contêiner com uma análise de prontidão falha. Uma análise de prontidão falhará se um serviço não tiver concluído a inicialização ou estiver ocupado, sobrecarregado ou sem capacidade de processar solicitações.
* Uma análise de atividade é usada para indicar se o processo deve ser reinicializado. O Kubernetes para e reinicia um contêiner com uma análise de atividade com falha para garantir que os Pods em estado defeituoso sejam finalizados e substituídos. Uma análise de atividade falhará se o serviço estiver em um estado irrecuperável, por exemplo, se uma condição de falta de memória ocorrer. As verificações de atividade simples que sempre retornam uma resposta OK podem identificar contêineres em um estado inconsistente, o que pode acontecer quando as solicitações de entrega de processo travam, mas o contêiner ainda está em execução.

As análises de prontidão e atividade são definidas usando uma estrutura semelhante, que inclui atraso e intervalos de novas tentativas, períodos de tolerância a falhas, tempos limite, bem como a definição da implementação da análise. A análise pode ser implementada executando um comando, verificando a conectividade de um terminal TCP ou executando uma chamada HTTP. A mesma implementação de análise pode frequentemente ser usada para propósitos de prontidão ou atividade, mas o atraso e os intervalos de novas tentativas precisam ser ajustados para o propósito específico.

## Entendendo e aplicando análises
{: #kubernetes-probes}

Fundamentalmente, o desenvolvimento de aplicativos nativos de nuvem é baseado no princípio de que os processos de contêiner realmente falham, mas são prontamente substituídos por um novo contêiner. Isso acontece em resposta a eventos inesperados, como uma falha de contêiner ou máquina, mas também devido a eventos operacionais, como o ajuste de escala horizontal e novos lançamentos de imagem do aplicativo. As verificações de prontidão são importantes porque garantem que novas instâncias de contêiner estejam prontas para receber trabalho antes de rotear o tráfego e evitam que ele seja roteado para instâncias que tenham sido removidas ou que estejam sendo destruídas.

Quando as verificações de prontidão não são definidas, o Kubernetes tem poucos insights sobre se uma instância de contêiner está pronta para manipular o tráfego e roteia-o imediatamente após o processo de contêiner ser iniciado. Sem verificações de prontidão, os aplicativos têm mais probabilidade de experimentar tempos limite de conectividade e respostas de rejeição de conexão quando o trabalho é roteado para uma instância que não está pronta para entregar a solicitação. As verificações de prontidão reduzem, mas não eliminam completamente, os erros de conectividade do cliente.

Embora as mudanças nos destinos de roteamento de instância sejam normais dentro do ciclo de vida de um aplicativo ativado para contêiner, os estados de processo que as verificações de atividade são destinadas a identificar são menos frequentes e representam uma exceção em vez da norma. Quando um processo entra em um estado que não tem uma recuperação possível, ele torna-se efetivamente inoperante. Alguns exemplos de possíveis motivos incluem condições de falta de memória ou um conflito causado por um erro de programação. A melhor maneira de recuperar-se de situações como essa é finalizar o contêiner, o que também finalizará qualquer processamento que esteja ocorrendo atualmente nele. Isso também cria a possibilidade de finalizar ou reiniciar loops no aplicativo, no qual os contêineres não conseguem ficar completamente on-line antes de serem finalizados e substituídos.

As análises de prontidão e atividade afetam o sistema de diferentes maneiras. Isso pode ser entendido em termos de transição de estado, na qual o estado positivo de uma verificação de prontidão é roteável e o estado negativo não é. Da mesma forma, o estado positivo de uma verificação de atividade representa um contêiner que está em execução normalmente e o estado negativo representa um contêiner que está inoperante. Quando um contêiner é iniciado, o estado de prontidão é inicialmente negativo e ele entrará em um estado positivo somente depois que estiver funcional. Uma verificação de atividade é iniciada em um estado positivo e entra em um estado negativo somente quando o processo se torna inoperante.

A configuração excessivamente agressiva de uma verificação de prontidão, como no caso de um atraso inicial baixo, tem pouco efeito porque executar a análise muito antecipadamente não faz com que a verificação de prontidão mude o estado. Por outro lado, uma verificação de atividade agressiva, na qual a análise é acionada muito antecipadamente, causa uma mudança de transição de estado, fazendo com que o sistema finalize o contêiner mais cedo do que o desejado.

## Melhores práticas para configurar análises
{: #probe-recommendation}

Ao implementar uma análise de funcionamento usando HTTP, considere os códigos de status HTTP a seguir para prontidão, atividade e funcionamento.

| Status    |  Prontidão            |  Atividade             |
|----------|-----------------------|-----------------------|
|          | Não OK causa nenhum carregamento | Não OK causa reinicialização |
| Iniciando | 503 - Indisponível     | 200 - OK              |
| Para cima       | 200 - OK              | 200 - OK              |
| Parando | 503 - Indisponível     | 200 - OK              |
| Inativo     | 503 - Indisponível     | 503 - Indisponível     |
| Com erro  | 500 - Erro do servidor    | 500 - Erro do servidor    |
{: caption="Tabela 1. Códigos de status HTTP" caption-side="bottom"}

Os terminais de verificação de funcionamento não devem requerer autorização ou autenticação. Como essas proteções não entram em vigor em terminais de análise de funcionamento, restrinja quaisquer implementações de análise de HTTP a solicitações GET que não modifiquem nenhum dado. Nunca retorne dados que identifiquem as especificidades sobre o ambiente, como o sistema operacional, a linguagem de implementação ou as versões de software, pois elas podem ser usadas para estabelecer um vetor de ataque.

Uma análise de atividade deve indicar claramente o que verifica, pois uma falha resulta em término imediato do processo. Evite métricas ambíguas que indicam apenas ocasionalmente um processo com falha, por exemplo, um terminal HTTP simples que sempre retorna `{"status": "UP"}` com um código de status 200. A maioria dos processos que estão em estado inoperante falha nessa verificação, acionando corretamente uma reinicialização.

As verificações de funcionamento ocorrem em intervalos frequentes, o que pode causar sobrecarga adicional. As análises de prontidão e atividade somente devem testar a viabilidade de serviços auxiliares, como bancos de dados ou outros microsserviços, em seu resultado quando não houver um fallback aceitável. Para uma análise de atividade, uma verificação auxiliar deve ser incluída somente caso um resultado malsucedido possa fazer com que o contêiner local entre em um estado irrecuperável. Uma análise de prontidão somente deve verificar um serviço auxiliar quando o contêiner local não é capaz de manipular solicitações devido a uma falha, mas a condição é recuperável.

Ao configurar o atraso inicial, uma análise de prontidão deve usar o valor mais baixo provável e uma verificação de atividade deve usar o valor de tempo mais alto provável. Por exemplo, se um servidor de aplicativos tende a ser iniciado em 30 segundos, um atraso de prontidão típico é de 10 segundos. A verificação de atividade usa um valor de 60 segundos para garantir que a inicialização do servidor seja sempre concluída antes da verificação dos estados termináveis.

O atributo *periodSeconds* para decisões de roteamento é geralmente configurado para um valor de dígito único, desde que a implementação da análise seja relativamente leve. Por exemplo, uma análise HTTP que retorna um status 200 OK sem um processamento do lado do servidor substancial tem um carregamento de processador mínimo e pode ser prontamente repetida a cada 1 a 5 segundos.

## Configurando análises no Kubernetes
{: #probe-config}

Declare as análises de atividade e prontidão com a implementação do seu Kubernetes no elemento do contêiner. As duas análises usam os mesmos parâmetros de configuração:

| Parâmetro | Descrição |
|-----------|-------------|
| *initialDelaySeconds* | A quantidade de tempo que o kubelet aguarda após o contêiner ser criado antes da primeira análise. |
| *periodSeconds* | A frequência com que o kubelet analisa o serviço. O padrão é 1. |
| *timeoutSeconds* | Com que rapidez a análise atinge o tempo limite. O valor padrão e o mínimo é 1. |
| *successThreshold* | O número necessário de execuções bem-sucedidas da análise após uma falha. O valor padrão e mínimo é 1. O valor deve ser 1 para as análises de vivacidade. |
| *failureThreshold* | O número de tentativas de reinicialização de um Pod pelo Kubernetes antes de ele desistir quando o Pod é iniciado e a análise falha (consulte a nota). O valor mínimo é 1 e o valor padrão é 3. |

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
