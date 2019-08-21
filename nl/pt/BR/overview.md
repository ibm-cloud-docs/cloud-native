---

copyright:
  years: 2019
lastupdated: "2019-07-16"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# O que é a abordagem nativa de nuvem?
{: #overview}

Os ambientes de computação em nuvem são dinâmicos, com alocação sob demanda e liberação de recursos por meio de um conjunto virtualizado e compartilhado. Esses ambientes elásticos permitem opções de dimensionamento mais flexíveis quando comparadas à alocação de recurso up-front geralmente usada em data centers locais tradicionais.
{:shortdesc}

De acordo com a [Cloud Native Computing Foundation](https://github.com/cncf/foundation/blob/master/charter.md){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), os sistemas nativos de nuvem têm os atributos a seguir:

- Os aplicativos ou processos são executados em contêineres de software como unidades isoladas.
- Os processos são gerenciados pelos processos de orquestração central para melhorar o uso do recurso e reduzir os custos de manutenção.
- Os aplicativos ou serviços (microsserviços) são fracamente acoplados com dependências descritas explicitamente.

Esses atributos descrevem um sistema altamente dinâmico que é composto de processos independentes que trabalham juntos para fornecer valor de negócios: um sistema distribuído.

A computação distribuída é um conceito que existe há décadas. [As falácias da computação distribuída](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") captura as suposições a seguir que são feitas por arquitetos e designers de sistemas distribuídos que se provaram erradas no final. 

* A rede é confiável.
* A rede é segura.
* A rede é homogênea.
* Não há latência.
* A largura da banda é infinita.
* A topologia não muda.
* Há um administrador.
* Não há custo de transporte.

Tecnologias de nuvem, como o Kubernetes e o Istio, visam abordar essas questões na própria infraestrutura.

## Doze fatores
{: #twelve-factors}

A metodologia de [aplicativo de doze fatores](https://12factor.net){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") foi criada por desenvolvedores no Heroku. As características que são mencionadas nos doze fatores não são específicas para um provedor em nuvem, plataforma ou linguagem. Os fatores representam um conjunto de diretrizes ou boas práticas para aplicativos móveis e resilientes que são bem-sucedidos em ambientes de nuvem (especificamente aplicativos do tipo software como serviço). Os doze fatores são fornecidos na lista a seguir:

1. Há uma associação de um para um entre um código base com versão, por exemplo, um repositório Git, e um serviço implementado. O mesmo código base é usado para muitas implementações.
2. Os serviços declaram explicitamente todas as dependências e não contam com a presença de ferramentas ou bibliotecas de nível de sistema.
3. A configuração que varia entre ambientes de implementação é armazenada no ambiente, especificamente em variáveis de ambiente.
4. Todos os serviços auxiliares são tratados como recursos conectados, sendo gerenciados (conectados e desconectados) pelo ambiente de execução.
5. O delivery pipeline tem estágios estritamente separados: construção, liberação, execução.
6. Os aplicativos são implementados como um ou mais processos stateless. Os processos temporários, especificamente, são stateless e não fazem compartilhamentos. Os dados persistidos são armazenados em um serviço auxiliar apropriado.
7. Os serviços autocontidos tornam-se disponíveis para outros serviços, atendendo em uma porta especificada.
8. A simultaneidade é obtida dimensionando processos individuais (escala horizontal).
9. Os processos são descartáveis: os comportamentos de inicialização rápida e encerramento normal resultam em um sistema mais robusto e resiliente.
10. Todos os ambientes, desde o desenvolvimento local até a produção, são tão semelhantes quanto possível.
11. Os aplicativos produzem logs como fluxos de evento, por exemplo, gravando em `stdout` e `stderr`, e confiam no ambiente de execução para agregar fluxos.
12. Se forem necessárias tarefas de administração únicas, elas serão mantidas no controle de fonte e empacotadas ao lado do aplicativo para assegurar que sejam executadas com o mesmo ambiente que o aplicativo.

Não é necessário seguir estritamente esses fatores para alcançar um ambiente de microsserviço de qualidade. Mas com eles, é possível construir e manter aplicativos ou serviços móveis em ambientes de entrega contínua.

## Aplicativos
{: #apps-intro}

Um app inclui código, dados, serviços e cadeias de ferramentas. Por exemplo, o app móvel do {{site.data.keyword.cloud_notm}} contém o código de dispositivo, com a lógica de back-end,
o armazenamento de dados, a análise de dados e os serviços de segurança, e é configurado para entrega contínua.

![Reutilizar](images/garage_reuse2.png "Com a Experiência do desenvolvedor, é possível reutilizar e evitar a reinvenção")

É possível criar e gerenciar um app usando qualquer Portal do Desenvolvedor do {{site.data.keyword.cloud_notm}} ou o {{site.data.keyword.dev_cli_notm}}.

É possível criar aplicativos em branco simples diretamente ou criar aplicativos mais complexos usando kits do iniciador. Se você optar por criar aplicativos em branco sem a ajuda de um kit do iniciador, será possível fazer isso por meio do [painel do {{site.data.keyword.cloud_notm}} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](https://{DomainName}){: new_window} sem visitar um portal.

É possível usar um padrão de código para criar rapidamente seu app e implementá-lo no {{site.data.keyword.cloud_notm}}. No [website do IBM Developer ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo")](https://developer.ibm.com/patterns/){:new_window}, escolha um padrão de código. É possível visualizar o código no GitHub ou criar e construir um app no {{site.data.keyword.cloud_notm}}, no qual é possível usar uma cadeia de ferramentas do DevOps para implementar automaticamente seu app.

## Microsserviços
{: #microservices}

Um *microsserviço* é um conjunto de componentes arquiteturais pequenos e independentes que se comunicam por meio de uma API leve comum. Cada microsserviço no exemplo simples a seguir é um aplicativo de 12 fatores que usa serviços auxiliares substituíveis para armazenar dados e transmitir mensagens:

![Um aplicativo de microsserviços](images/microservice.png "Um aplicativo de microsserviços")

Os microsserviços são independentes. A agilidade é um dos benefícios das arquiteturas de microsserviço, mas ela existe apenas quando os serviços podem ser regravados sem atrapalhar outros serviços. 

Limites de API limpos fornecem às equipes que trabalham em um serviço a maior flexibilidade para desenvolver a implementação. Essa característica é o que permite a persistência e a programação poliglota.

Os microsserviços são resilientes. A estabilidade do aplicativo depende de que microsserviços individuais sejam robustos com relação às falhas. Essa é uma diferença significativa das arquiteturas tradicionais, em que a infraestrutura de suporte manipula falhas para você. Cada serviço precisa aplicar padrões de isolamento, como disjuntores e anteparas, para conter falhas e definir comportamentos de fallback apropriados para proteger os serviços de envio de dados.

Os microsserviços são stateless e são armazenados em serviços de nuvem auxiliares externos, como o Redis. O comportamento de inicialização rápida e encerramento normal permite ainda mais que os microsserviços funcionem bem em ambientes automatizados que criam e removem instâncias em resposta ao carregamento ou manutenção do funcionamento do sistema.

### O significado de "pequeno"
{: #small-microsvc}

O uso da palavra "pequeno", conforme aplicado a um microsserviço, significa que seu foco é de propósito. Muitas descrições fazem paralelos entre as funções de microsserviços individuais e os comandos encadeados na linha de comandos do Unix:

```
ls | grep 'service' | sort -r
```
{:pre}

Cada um desses comandos do UNIX executa tarefas bem diferentes e é possível encadeá-las independentemente da linguagem de programação ou da quantidade do código.

## Aplicativos poliglotas: escolhendo a ferramenta ideal para a tarefa
{: #polyglot-apps}

Ser poliglota é uma vantagem frequentemente citada das arquiteturas baseadas em microsserviços. Crie um equilíbrio entre a criação de uma lista de tecnologias suportadas para escolher, com uma política definida para estender a lista com novas tecnologias ao longo do tempo. Certifique-se de que requisitos não funcionais ou regulamentares, como a sustentabilidade, a auditabilidade e a segurança de dados, possam ser atendidos, preservando a agilidade e suportando a inovação por meio da experimentação.

... :FIXME: descompactar? Tabela de linguagem aqui?:...

## REST e JSON
{: #rest-json}

Os aplicativos poliglotas somente são possíveis com protocolos abrangentes de linguagem. Os padrões de arquitetura REST definem diretrizes para criar interfaces uniformes que separam a representação de dados na rede da implementação do serviço.

O JSON, em arquiteturas de microsserviços, é o formato de ligação escolhido para dados baseados em texto, deslocando o XML com sua brevidade e simplicidade comparativa. Como uma comparação, o exemplo a seguir é um registro básico que contém dados sobre um funcionário em JSON:

```json
{
  "name": "Marley Cassin",
  "serial": 228264,
  "title": "Senior Software Engineer",
  "address": {
    "office": "501-B101",
    "street": "3858 Kuvalis Pass",
    "city": "East Craig",
    "state": "NC",
    "zip": "64519-8934"
  }
}
```
{: codeblock}

E o exemplo a seguir é o mesmo registro de funcionário em XML:

```xml
<person>
  <name>Marley Cassin</name>
  <serial>228264</serial>
  <title>Senior Software Engineer</title>
  <address>
    <office>501-B101</office>
    <street>3858 Kuvalis Pass</street>
    <city>East Craig</city>
    <state>NC</state>
    <zip>64519-8934</zip>
  </address>
</person>
```
{: codeblock}

O JSON usa pares atributo-valor para representar objetos de dados em uma sintaxe concisa que preserva as informações sobre alguns tipos básicos, como números, sequências, matrizes e objetos. O JSON e o XML claramente representam o objeto de endereço aninhado nos exemplos anteriores, mas o esquema XML associado é necessário para determinar o tipo do elemento `serial`. No JSON, a sintaxe deixa claro que o valor de `serial` é um número e não uma sequência.
