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

# Criando microsserviços RESTful
{: #rest-api}

Os aplicativos nativos de nuvem produzem e usam APIs, seja em uma arquitetura de microsserviços ou não. Algumas APIs são consideradas internas, ou privadas, e algumas são consideradas externas. 
{:shortdesc}

As APIs internas são usadas apenas em um ambiente com firewall para os serviços de back-end se comunicarem uns com os outros. As APIs externas apresentam um ponto de entrada unificado para os consumidores e são frequentemente **gerenciadas** por ferramentas como a {{site.data.keyword.apiconnect_long}}, que podem impor limites de taxa ou outras restrições de uso. Um exemplo desse tipo de API é a [API do GitHub Developer](https://developer.github.com/v3/){: new_window}![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"). Ela fornece uma API unificada com o uso consistente de verbos HTTP, códigos de retorno e comportamento de paginação sem mostrar os detalhes da implementação interna. Essa API pode ser apoiada por um aplicativo grande ou por uma coleção de microsserviços. Esse detalhe não é mostrado ao consumidor, portanto, o GitHub pode desenvolver seus sistemas internos conforme necessário.

## Melhores práticas para APIs RESTful
{: #bps-apis}

As APIs de REST usam verbos HTTP padrão para as operações Criar, Recuperar, Atualizar e Excluir (CRUD), com atenção especial a se a operação é idempotente (segura para tentar novamente diversas vezes).

* As operações POST podem ser usadas para criar ou atualizar recursos. No entanto, não podem ser chamadas repetidamente. Por exemplo, se uma solicitação POST for usada para criar recursos e for chamada diversas vezes, um recurso novo e exclusivo será criado como resultado de cada chamada.
* As operações GET devem poder ser chamadas repetidamente e não devem causar efeitos colaterais. Elas devem ser usadas para recuperar informações. Solicitações GET com parâmetros de consulta não devem ser usadas para mudar ou atualizar informações. Em vez disso, use as operações POST, PUT ou PATCH.
* As operações PUT podem ser usadas para atualizar recursos. Geralmente, elas incluem uma cópia completa do recurso a ser atualizado, possibilitando a chamada da operação diversas vezes.
* As operações PATCH permitem uma atualização parcial dos recursos. Elas podem ser chamadas repetidamente, dependendo de como o delta foi especificado e, em seguida, aplicado ao recurso. Por exemplo, se uma operação PATCH indicar que um valor deva ser mudado de A para B, ela poderá ser chamada repetidamente. Não haverá efeito se ela for chamada diversas vezes e o valor já for B.
* As operações DELETE podem ser chamadas diversas vezes, pois um recurso pode ser excluído apenas uma vez. No entanto, o código de retorno varia, já que a primeira operação foi bem-sucedida (`200` ou `204`) e as chamadas subsequentes não localizarão o recurso (`404` ou `410`).

### Resultados descritivos que podem ser compreendidos por máquinas
{: #rest-results}

Como as APIs são chamadas pelo software e não por humanos, tome cuidado ao comunicar as informações ao responsável pela chamada da forma mais eficiente e eficiente possível.

Use códigos de status HTTP relevantes e úteis, conforme descrito na tabela a seguir: 

| Código de erro HTTP | Orientação de uso |
|-----------------|----------------|
| `200 (OK)` | Use quando tudo estiver correto e houver dados a serem retornados |
| `204 (NO CONTENT)` | Use quando tudo estiver correto, mas não houver dados de resposta |
| `201 (CREATED)` | Use para solicitações POST que resultam na criação de um recurso, haja ou não um corpo de resposta |
| `409 (CONFLICT)` | Use quando mudanças simultâneas entrarem em conflito |
| `400 (BAD REQUEST)` | Use quando os parâmetros estiverem malformados |

Para obter mais informações, consulte [Códigos de status de resposta](https://tools.ietf.org/html/rfc7231#section-6){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"). 

Considere quais dados retornarão em suas respostas para tornar a comunicação eficiente. Por exemplo, quando um recurso é criado com uma solicitação POST, a resposta deve incluir a localização do recurso recém-criado em um cabeçalho Localização. O recurso criado é frequentemente incluído no corpo de resposta, além de eliminar a solicitação GET extra para buscar o recurso criado. O mesmo se aplica a solicitações PUT e PATCH.

### URIs do recurso RESTful
{: #rest-uris}

Há opiniões diferentes sobre alguns aspectos dos URIs do recurso RESTful. Em geral, há concordância de que os recursos devem ser substantivos, não verbos, e os terminais devem ser plurais. Isso resulta em uma estrutura clara para operações CRUD:

* `POST /accounts`: criar uma nova conta.
* `GET /accounts`: recuperar uma lista de contas.
* `GET /accounts/16`: recuperar uma conta específica.
* `PUT /accounts/16`: atualizar uma conta específica.
* `PATCH /accounts/16`: atualizar uma conta específica.
* `DELETE /accounts/16`: excluir uma conta específica.

Os relacionamentos são modelados usando URIs hierárquicas, por exemplo, ` /accounts/16/credentials` para gerenciar credenciais associadas a uma conta.

Não há uma única maneira de gerenciar operações com um recurso que não se ajusta a uma estrutura típica. Essas operações fazem o que funciona melhor para o consumidor da API.

<!-- Operation example -->

### Robustez e APIs RESTful
{: #robust-api}

O [princípio de robustez](https://tools.ietf.org/html/rfc1122#page-12){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") fornece a melhor orientação: "seja liberal quanto ao que aceita e conservador quanto ao que envia". Parta do princípio de que as APIs evoluem ao longo do tempo e seja tolerante quanto aos dados que você não compreende.

#### Produzindo APIs
{: #robust-producer}

Ao fornecer uma API para clientes externos, há duas coisas que devem ser feitas ao aceitar solicitações e retornar respostas: 

* Aceite os atributos desconhecidos como parte da solicitação.
    > Se um serviço chamar sua API com atributos desnecessários, jogue esses valores fora. A reversão de um erro nesse cenário pode causar falhas desnecessárias, impactando negativamente o usuário.
* Retorne os atributos requeridos por seus consumidores
    > Evitar a exposição de detalhes do serviço interno. Expor atributos que os consumidores precisam como parte da API.

#### Consumindo APIs
{: #robust-consumer}

Ao consumir APIs:

* Valide a solicitação com relação às variáveis ou atributos que você precisa.
    > Não valide com relação às variáveis apenas porque elas são fornecidas. Se não as estiver usando como parte de sua solicitação, não se preocupe se elas não estiverem presentes.

* Aceite os atributos desconhecidos como parte da resposta.
    > Não emita uma exceção se você receber uma variável inesperada. Se a resposta contiver as informações que você precisa, não importará o que mais esteja envolvido.

Essas diretrizes são especialmente relevantes para linguagens fortemente tipificadas como a Java, em que a serialização e a desserialização JSON geralmente ocorrem indiretamente. Por exemplo, as bibliotecas Jackson ou JSON-P/JSON-B. Procure mecanismos de linguagem que permitam especificar um comportamento mais generoso, como ignorar atributos desconhecidos ou definir ou filtrar quais atributos devem ser serializados.

### Versionando APIs RESTful
{: #version-api}

Um dos principais benefícios dos microsserviços é a capacidade de permitir que os serviços evoluam independentemente. Como os microsserviços chamam outros serviços, essa independência ocorre com uma ressalva gigantesca: não é possível causar mudanças interruptivas em sua API.

Se o princípio de robustez for seguido, pode demorar um tempo até que uma mudança interruptiva seja necessária. Quando essa alteração que afeta o processamento da mensagem acontece, é possível optar por construir um serviço completamente diferente e obsoletar o original ao longo do tempo.

Se você precisar fazer alterações de API que afetam o processamento da mensagem para um serviço existente, decida como gerenciar essas mudanças: o serviço manipulará todas as versões da API, você manterá versões independentes do serviço para suportar cada versão da API ou seu serviço suportará apenas a versão mais recente da API e dependerá de outras camadas adaptáveis para converter para/de a API mais antiga?

Depois de determinar como gerenciar as mudanças, o problema mais fácil de resolver será como refletir a versão em sua API. Geralmente, há três maneiras de realizar a versão de um recurso REST:

* Inclua a versão no caminho do URI.
* Inclua a versão no cabeçalho Aceitar HTTP e baseie-se na negociação de conteúdo.
* Use um cabeçalho de solicitação customizado.

#### Incluindo a versão no caminho do URI
{: #version-path}

A maneira mais fácil de especificar uma versão é incluí-la no caminho do URI. Há vantagens nessa abordagem: ela é claramente fácil de alcançar ao construir os serviços em seu aplicativo e é compatível com ferramentas de navegação de API, como o Swagger, e ferramentas de linha de comandos, como o `curl`, e assim por diante.

Se for incluir a versão no caminho do URI, ela deverá se aplicar a seu aplicativo como um todo, por exemplo, `/api/v1/accounts` em vez de `/api/accounts/v1`. Hipermídia como o Engine of Application State (HATEOAS) é uma maneira de fornecer URIs aos consumidores de API para que eles não sejam responsáveis por construir URIs por conta própria. O GitHub, por exemplo, fornece [URLs de hipermídia](https://developer.github.com/v3/#hypermedia){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") nas respostas por esse motivo. Isso faz com que HATEOAS seja difícil, se não impossível, de obter, caso diferentes serviços de back-end puderem ter versões de variação independente em seus URIs.

#### Modificando o cabeçalho Aceitar para incluir a versão
{: #version-accept}

O cabeçalho Aceitar é um local óbvio para definir uma versão, mas é um dos mais difíceis de testar. Ele também é um local inicial frequente para a alternância de recursos. Especificar cabeçalhos HTTP requer chamadas de API mais detalhadas.

#### Incluindo um cabeçalho de solicitação customizado
{: #version-custom}

É possível incluir um cabeçalho de solicitação customizado para indicar a versão da API. Assim como com o cabeçalho Aceitar, também é possível usar cabeçalhos customizados para rotear o tráfego para instâncias de back-end específicas. Com esse método, você encontra os mesmos problemas de fácil utilização que encontraria com o método de cabeçalho Aceitar, mas com o requisito adicional de que os consumidores precisam aprender sobre esse cabeçalho.

Para obter mais informações, consulte [Seu versionamento de API está errado e, por isso, decidi fazê-lo de três maneiras erradas diferentes](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo").

## Criando e gerando APIs
{: #create-api}

O [OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") é a especificação oficial para serviços RESTful, governada pela [OpenAPI Initiative](https://www.openapis.org/){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo"), uma associação de empresas sob a Linux Foundation.

É possível usar uma das maneiras a seguir para criar uma API:

  * Comece com uma Definição OpenAPI (de cima para baixo): nessa abordagem, você começa criando uma definição OpenAPI em um formato independente de idioma (geralmente YAML). Em seguida, use um gerador de código para criar uma estrutura básica e construa sua implementação de serviço com base nela. Esse padrão é geralmente adotado por empresas que têm uma equipe central de design de API e permite que o desenvolvimento e o teste sejam realizados em paralelo.
  * Comece com o código (de baixo para cima): seu código é a origem de sua definição de API. Essa abordagem funciona bem para novos aplicativos com um aspecto experimental, pois sua definição de API evolui conforme você obtém um melhor entendimento do que seu serviço precisa fazer. Essa abordagem também funciona melhor em algumas linguagens do que em outras, já que depende de ferramentas que geram uma definição OpenAPI por meio de seu código. Java, por exemplo, tem um excelente suporte para gerar documentos OpenAPI por meio de estruturas REST baseadas em anotação.

Em qualquer caso, trabalhar com uma definição OpenAPI pode ajudar a identificar áreas nas quais a API é inconsistente ou difícil de entender do ponto de vista de um consumidor. As definições OpenAPI publicadas ou controladas por versão também podem ser usadas por ferramentas de construção para ajudar a sinalizar mudanças interruptivas que afetam os consumidores.

### Criando uma API por meio de uma Definição OpenAPI
{: #openapi-first}

É possível criar seu arquivo YAML de OpenAPI em qualquer ferramenta que você escolher. Usar um editor de texto sem formatação, no entanto, pode aumentar a probabilidade de erros. Alguns editores têm suporte básico para YAML e alguns podem ter extensões adicionais para suportar definições OpenAPI. Por exemplo, é possível usar extensões do Visual Studio Code, como o [Swagger Viewer](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") ou o [OpenAPI Preview](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") para validar sua definição OpenAPI com relação a uma versão de especificações especificada e renderizar uma visualização da web na área de janela de visualização:

![OpenAPI Preview](images/create-api-image1.png "OpenAPI Preview") 

Há também uma variedade de editores de análise em tempo real baseados em navegador, que podem ser usados on-line ou localmente. Alguns exemplos incluem:

* O [Projeto OpenAPI-GUI](https://github.com/Mermade/openapi-gui){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") suporta a v2 e a v3 da especificação OpenAPI e pode migrar uma definição OpenAPI v2 para a v3.
* O [Swagger Editor do SmartBear](https://editor.swagger.io){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") também suporta a v2 e a v3 do OpenAPI.
* O [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") fornece um conjunto de editores e ferramentas para a criação e modelagem de API.

### Gerando a implementação da API
{: #code-first}

É possível usar o [Gerador de OpenAPI](https://github.com/OpenAPITools/openapi-generator){: new_window} ![Ícone de link externo](../icons/launch-glyph.svg "Ícone de link externo") de software livre para criar um projeto de estrutura básica para a implementação de seu serviço por meio de uma definição OpenAPI. É possível especificar a estrutura ou a linguagem para a estrutura básica na linha de comandos. Por exemplo, para criar um projeto Java para a API do PetStore de amostra que usa as anotações de método JAX-RS genéricas, especifique o comando a seguir:

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

