---

copyright:
  years: 2019
lastupdated: "2019-02-18"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Conceitos da plataforma de nuvem
{: #platform}

Esta seção fornece uma breve visão geral das tecnologias e conceitos principais com os quais os desenvolvedores interagem ao construir aplicativos nativos de nuvem, iniciando com os Contêineres, o Kubernetes, o Helm e o Istio.
{:shortdesc}

## Contêineres
{: #containers}

Os contêineres são um mecanismo padrão para empacotar um aplicativo e todas as suas dependências em uma unidade única e autocontida. Eles resolvem o problema da portabilidade: o artefato contêiner (imagem) garante que tudo que um aplicativo precise executar esteja no local correto, possibilitando que os mecanismos de contêiner foquem na execução de contêineres como processos isolados, de forma eficiente e segura.

As imagens de contêiner são geralmente construídas por meio de uma lista de instruções definidas em um `Dockerfile`. Elas são praticamente sempre construídas por meio de outras imagens de contêiner (como, essencialmente, uma continuação das instruções do estado anterior conhecido). É possível usar o fragmento a seguir para criar sua própria imagem do Open Liberty, por exemplo:

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

Depois que uma imagem é construída, pode ser executada. Os mecanismos de execução de contêiner, como o Docker ou o [containerd](https://containerd.io/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), utilizam essa definição de imagem e executam o ponto de entrada definido como um processo isolado de recurso diretamente no sistema operacional do host, eliminando a sobrecarga de máquinas virtuais.

As imagens de contêiner são armazenadas em *registros*. O mais conhecido é o registro público do Docker Hub, mas é mais comum enviar imagens por push e extraí-las de registros de contêiner controlados por acesso, como o {{site.data.keyword.registryshort_notm}}, que estão mais intimamente associados a seus pipelines de infraestrutura e CI/CD.

## Kubernetes
{: #kubernetes}

As plataformas em nuvem da IBM utilizam o Kubernetes para a orquestração de contêiner. Portanto, além de conhecer os conceitos básicos do contêiner, é importante que os desenvolvedores estejam familiarizados com os fundamentos do Kubernetes, incluindo os comandos básicos e os artefatos de implementação. A tabela a seguir inclui alguns conceitos importantes do Kubernetes:

| Conceito | Descrição |
|---------|-------------|
| Pod | Um grupo localizado de contêineres implementados em conjunto como uma única unidade. Os pods são relativamente imutáveis, exigindo que o Pod original seja substituído para fazer modificações em diversos de seus atributos. Um aplicativo típico possui um contêiner com lógica de negócios principal e, opcionalmente, Pods adicionais que fornecem recursos de plataforma no nível granular do Pod. |
| Implementação | Um modelo repetido para um Pod stateless, incluindo uma dimensão de escala no conceito de Pod. Além disso, a definição modelada pode ser atualizada e as instâncias de Pod subjacentes podem ser substituídas. Uma configuração de Implementação do Kubernetes é monitorada por um Controlador de Implementação do Kubernetes para garantir que o número declarado de Pods para uma determinada Implementação seja mantido. Uma Implementação é exibida como `kind: Deployment` em arquivos `.yaml`. |
| Serviço | Um nome bem conhecido que representa um conjunto de endereços IP de Pod relativamente instáveis. Um Serviço somente pode existir na rede privada do cluster ou pode ser exposto externamente, geralmente por meio de um balanceador de carga específico do provedor em nuvem. Um Serviço é exibido como `kind: Service` em arquivos `.yaml`. |
| Entrada | Fornece a capacidade de compartilhar um único endereço de rede com diversos serviços por meio de hospedagem virtual ou de roteamento baseado em contexto. Um Ingresso também pode executar atividades de gerenciamento de conexão de rede, como a finalização de TLS. Um Ingresso é exibido como `kind: Ingress` em arquivos `.yaml`. |
| Segredo | Um objeto que armazena informações confidenciais para o uso do tempo de execução do Pod e separa as informações específicas da implementação da orquestração ou da imagem de contêiner. Um segredo pode ser exposto a um Pod no tempo de execução por meio de variáveis de ambiente ou de montagens do sistema de arquivos virtual. Sem segredos, os dados sensíveis são armazenados na imagem do contêiner ou na orquestração, criando mais oportunidades para a exposição acidental ou o acesso indesejado. |
| ConfigMap | Desempenha uma função semelhante àquela dos Segredos na separação das informações específicas da implementação com relação à orquestração de contêiner. No entanto, um ConfigMap é uma estrutura de configuração de propósito geral. Ele é usado para ligar informações, como argumentos de linha de comandos, variáveis de ambiente e outros artefatos de configuração, aos contêineres e componentes do sistema do seu Pod no tempo de execução. | 
{: caption="Tabela 1. Conceitos do Kubernetes" caption-side="bottom"}

Todos os recursos são definidos dentro do modelo de recurso do Kubernetes, que pode ser configurado pela API RESTful ou pelos arquivos de configuração enviados por meio da linha de comandos `kubectl`.

Para obter mais informações, consulte [Conceitos básicos do Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), [Modelo de objeto do Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") e [Linha de comandos `kubectl`](https://kubernetes.io/docs/reference/kubectl/overview/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"). 

## Helm
{: #helm}

O Helm é um gerenciador de pacote que fornece uma maneira fácil de localizar, compartilhar e usar o software construído para o Kubernetes. Ele também aborda uma necessidade comum do usuário: a implementação do mesmo aplicativo em diversos ambientes. Além disso, ele usa *gráficos*, que são coleções de modelos que produzem objetos válidos do Kubernetes (YAML) no momento da instalação. Esses gráficos são construídos por meio de uma linguagem de modelo que inclui o suporte para variáveis, operações de intervalo e outros itens que demandam mão de obra excessiva na manutenção dos metadados de implementação do Kubernetes.

Para obter mais informações, consulte [Helm](https://helm.sh/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Malha de serviços do Istio
{: #istio}

O Istio é uma plataforma de software livre para o gerenciamento e a proteção de microsserviços. Ele funciona com orquestradores como o Kubernetes, fornecendo uma maneira de gerenciar e controlar a comunicação entre os serviços.

O Istio opera usando um modelo de sidecar. Um sidecar (um proxy Envoy) é um processo separado que é executado junto ao seu aplicativo. Ele gerencia toda a comunicação recebida e enviada de seu serviço e aplica um nível comum de capacidade a todos os serviços, independentemente da linguagem de programação ou da estrutura com a qual o serviço foi construído. Efetivamente, o Istio fornece um mecanismo para configurar centralmente as políticas de roteamento e segurança, aplicando essas políticas por meio de sidecars de maneira descentralizada.

De modo geral, recomendamos usar os recursos fornecidos pelo Istio em vez dos recursos semelhantes fornecidos pelas linguagens de programação ou estruturas individuais. Por exemplo, as políticas de balanceamento de carga e outras políticas de roteamento são definidas, gerenciadas e aplicadas pela infraestrutura de maneira mais consistente.

Em alguns casos, como com o rastreio distribuído, o Istio e as bibliotecas de nível de aplicativo são complementares. É possível melhorar as operações usando ambos. No caso do rastreio distribuído, o Istio somente pode garantir que os cabeçalhos de rastreio estejam presentes, pois são as bibliotecas de aplicativos que fornecem o contexto importante sobre os relacionamentos entre as solicitações. Seu entendimento do sistema como um todo melhora quando o Istio e as bibliotecas auxiliares ou de estrutura são usados juntos.

No nível mais alto, o Istio estende a plataforma do Kubernetes, fornecendo conceitos de gerenciamento, visibilidade e segurança adicionais. Os recursos do Istio podem ser divididos nas quatro categorias a seguir:

* Gerenciamento de tráfego: controle o tráfego entre seus microsserviços para executar a divisão de tráfego, a recuperação de falhas e as liberações canary.
* Segurança: forneça autenticação, autorização e criptografia fortes e baseadas em identidade entre os microsserviços.
* Observabilidade: colete métricas e logs para melhorar a visibilidade nos aplicativos que estão em execução em seu cluster.
* Políticas: aplique controles de acesso, limites de taxa e cotas para proteger seus aplicativos.

Consulte [O que é o Istio?](https://istio.io/docs/concepts/what-is-istio/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para obter mais informações.



