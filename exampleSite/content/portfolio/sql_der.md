+++
categories = ["sql"]
coders = []
date = 2026-05-13T23:00:00Z
description = "Diagrama entidade relacionamento e construção de banco de dados usando SQL Server inspirado no workflow das ordens de produção da empresa The Fini Company."
github = ["https://github.com/Brunorwell/Production_Order"]
image = "/logos/sql1.png"
title = "Diagrama Entidade - Relacionamento (DER)"
type = "post"
[[tech]]
logo = "/logos/sql_server.svg"
[[tech]]
logo = "/logos/docker.svg"
name = "Docker"


+++
Esse projeto reproduz o que acontece por baixo dos panos do ERP Protheus na empresa em que eu trabalhava. Eu percebi que as ordens de produção que eu tinha de lidar eram basicamente um banco de dados relacional — você consegue até identificar as **chaves primárias e estrangeiras** apenas observando atentamente os campos! Eu decidi reproduzir em uma escala pequena um banco de dados inspirado por esse processo, respeitando as boas práticas de **modelagem de banco de dados relacionais**.

O objetivo desse projeto é, portanto, demonstrar habilidades em **modelagem de dados**: um processo que envolve principalmente a **normalização** — identificação de quais **tabelas** e **relacionamentos** são necessários para o modelo.

No mundo real, por conta de trade-offs, a normalização estrita dos dados nem sempre é utilizada, mas para esse projeto foi considerada normalização até a **Terceira Forma Normal (3NF)** para fins de aplicação dos conhecimentos adquiridos na faculdade.

## Breve contexto

A The Fini Company é uma empresa espanhola que fabrica doces. Eu trabalhava na fábrica do Brasil.

Uma **Ordem de Produção** é um documento que orienta o **operador de máquina** a **gerenciar a produção solicitada** pelo engenheiro de produção. A ordem de produção contém informações do tipo:
  - Quantidade a ser produzida
  - Lote
  - Nome do produto
  - A unidade de medidas das embalagens
  - Tipo de embalagem a ser retirada na logística

Como o documento não é organizado como um banco de dados relacional, foi necessário identificar todas as entidades (tabelas) envolvidas em todo o processo e garantir que as tabelas tivessem relacionamentos apropriados para garantir as **regras de negócio** da companhia.

**A imagem abaixo representa o Diagrama Entidade Relacionamento do modelo:**

<style>
  .content img.sql-image-fullsize {
    max-width: 100% !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/erd.jpeg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: 100% !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>

**Abaixo segue o MER (Modelo Entidade Relacionamento):**

<style>
  .content img.sql-image-fullsize {
    max-width: 100% !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/mer10.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: 100% !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>

## Relacionamentos críticos

### 1. [PRODUCT_UNITY] N:N [PRODUCT_WEIGHT] (Muitos para muitos): 

#### Definição de tabelas
A entidade **PRODUCT_UNITY** é responsável por armazenar todos os doces produzidos:

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 300px; font-family: Arial, sans-serif; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Product_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Product_name</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">A008</td>
        <td style="padding: 8px; border: 1px solid #ccc;">AMORAS</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">D017</td>
        <td style="padding: 8px; border: 1px solid #ccc;">DENTADURAS</td>
    </tr>
</table>
<br>

A entidade **PRODUCT_WEIGHT** é responsável por armazenar os tipos de pesos que um produto pode ser associado:

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 500px; font-family: Arial, sans-serif; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Weight_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Weight_desc</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Box_id</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">111070F07611</td>
        <td style="padding: 8px; border: 1px solid #ccc;">12X70G</td>
        <td style="padding: 8px; border: 1px solid #ccc;">ME0BR257681</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">113018A00104</td>
        <td style="padding: 8px; border: 1px solid #ccc;">24X12X15G</td>
        <td style="padding: 8px; border: 1px solid #ccc;">ME0BR257703</td>
    </tr>
</table>

#### A solução para o relacionamento muitos para muitos (N:N): A entidade associativa

Como **muitos produtos podem ser associados a N pesos** e **muitos pesos podem ser associados a N produtos**, é necessária a criação de uma **tabela associativa** para garantir que todas as combinações sejam possíveis


Em tabelas desse tipo é criada uma **chave primária composta** formada pelas duas chaves primárias das tabelas que precisam ser relacionadas. Desse modo, é possível registrar o valor produto-peso no banco de dados!


<style>
  .content img.sql-image-fullsize {
    max-width: 100% !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/mer_product_unity3.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: 100% !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>

Podemos notar o registros de produtos tendo associação com diversos id de pesos.

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 400px; font-family: Arial, sans-serif; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Product_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Weight_id</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F001</td>
        <td style="padding: 8px; border: 1px solid #ccc;">111070F07611</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F001</td>
        <td style="padding: 8px; border: 1px solid #ccc;">113018A00104</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F002</td>
        <td style="padding: 8px; border: 1px solid #ccc;">111070F07611</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F002</td>
        <td style="padding: 8px; border: 1px solid #ccc;">113018A00104</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F003</td>
        <td style="padding: 8px; border: 1px solid #ccc;">113018A00104</td>
    </tr>
</table>

#### Demonstração do comportamento da tabela associativa:

##### Query:

<style>
  .content img.sql-image-fullsize {
    max-width: 100% !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/sql_querry_1v6.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: 100% !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>

##### Output:

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 900px; font-family: Arial, sans-serif; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Product_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Product Name</th>
        <th style="padding: 8px; border: 1px solid #ccc;">weight_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Weight description</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F001</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM MORANGO CREMOSO</td>
        <td style="padding: 8px; border: 1px solid #ccc;">111070F07611</td>
        <td style="padding: 8px; border: 1px solid #ccc;">12X70G</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F001</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM MORANGO CREMOSO</td>
        <td style="padding: 8px; border: 1px solid #ccc;">113018A00104</td>
        <td style="padding: 8px; border: 1px solid #ccc;">24X12X15G</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F002</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM PAVE DE ABACAXI</td>
        <td style="padding: 8px; border: 1px solid #ccc;">111070F07611</td>
        <td style="padding: 8px; border: 1px solid #ccc;">12X70G</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F002</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM PAVE DE ABACAXI</td>
        <td style="padding: 8px; border: 1px solid #ccc;">113018A00104</td>
        <td style="padding: 8px; border: 1px solid #ccc;">24X12X15G</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">F003</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM TORTA DE LIMAO</td>
        <td style="padding: 8px; border: 1px solid #ccc;">113018A00104</td>
        <td style="padding: 8px; border: 1px solid #ccc;">24X12X15G</td>
    </tr>
</table>
<br>

Podemos observar o comportamento de muitos para muitos nessa tabela retornada pela query, notando que um **weight_id** pode estar associado a mais de um doce e um **product_id** pode estar associado a mais de um peso.

### 2. [PRODUCT_WEIGHT] 1:N [BOX_REGISTER] (Um para N):

#### Definição de tabelas

Como vimos no tópico anterior, a entidade **PRODUCT_WEIGHT** armazena os valores de peso que um produto pode assumir. Introduzo agora a entidade **BOX_REGISTER**, que armazena especificações de todas as embalagens disponíveis pela logística:

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 600px; font-family: Arial, sans-serif; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Box_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Box_desc</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">ME0BR257681</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CX PAP COLOR 12X100G V0</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">ME0BR257703</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CX COLOR DP AUT 24X12X15 V0</td>
    </tr>
</table> 

#### O relacionamento ideal:

Como um tipo de peso pode assumir N tipos de caixas, o relacionamento escolhido é um para N. Em modelagens **star schema — muito utilizadas no Power BI —**, a cardinalidade ideal é um para muitos, por se tratar do modelo que mais se aproxima da lógica de bancos de dados relacionais, onde, em um modelo tabular, o que se espera é que um elemento se comunique com N valores ou apenas um.

### 3. [LOT] N:1 [LOT_CATEGORY] (Muitos para um)

#### Definição de tabelas

Produtos são organizados em categorias e a **tabela LOT reflete essa regra**:

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 500px; font-family: Arial, sans-serif; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Lot_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Lot_number</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Lot_category_id</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">1</td>
        <td style="padding: 8px; border: 1px solid #ccc;">446</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">2</td>
        <td style="padding: 8px; border: 1px solid #ccc;">450</td>
        <td style="padding: 8px; border: 1px solid #ccc;">1</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">3</td>
        <td style="padding: 8px; border: 1px solid #ccc;">514</td>
        <td style="padding: 8px; border: 1px solid #ccc;">3</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">4</td>
        <td style="padding: 8px; border: 1px solid #ccc;">994</td>
        <td style="padding: 8px; border: 1px solid #ccc;">3</td>
    </tr>
</table>
<br>

A entidade **LOT_CATEGORY** é responsável por **armazenar essas categorias**:

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 400px; font-family: Arial, sans-serif; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Lot_category_id</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Lot_cod</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">1</td>
        <td style="padding: 8px; border: 1px solid #ccc;">GOMA</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">2</td>
        <td style="padding: 8px; border: 1px solid #ccc;">GEL</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">3</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CHICLETE</td>
    </tr>
</table>

#### Regra de negócio garantida: um lote pode ter apenas uma categoria.
Como um lote pode pertencer a apenas **um tipo de lote**, a chave primária de **LOT_CATEGORY** entra como chave estrangeira na tabela **LOT**.

<style>
  .content img.sql-image-fullsize {
    max-width: 100% !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/mer_lot2.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: 100% !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>

### 4. [PRODUCT_MOVEMENT] 1:1 [LOT] 1:1 [PRODUCTION_ORDER] 

#### Definição de tabelas:

A entidade **PRODUCTION_ORDER**, como o nome diz, é a ordem de produção.

A entidade **PRODUCT_MOVEMENT** é responsável por armazenar toda a movimentação de um produto. Essa tabela é o que chamamos de **entidade fraca**, porque ela tem como **sua chave primária uma chave composta estrangeira formada pela chave de PRODUCT_WEIGHT_UNITY e a chave de LOT**.

#### A normalização até a 3NF força um número maior de joins: um trade-off entre performance e organização:

Uma ordem de produção compartilha um mesmo lote registrado na tabela **PRODUCT_MOVEMENT**. Portanto, a conexão entre **PRODUCTION_ORDER** e **PRODUCT_MOVEMENT** é feita pela tabela **LOT**.

Por conta da normalização até a Terceira Forma Normal, cada tabela é especialmente designada para armazenar um tipo de informação. Desse modo, para a formulação de uma query que detalhe todos os elementos associados a uma ordem de produção, são necessários diversos joins:

##### Query:

<style>
  .content img.sql-image-fullsize {
    max-width: 100% !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/sql_query_2V6.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: 100% !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>

##### Output:

<table style="border-collapse: collapse; margin-left: auto; margin-right: auto; width: 100%; font-family: Arial, sans-serif; font-size: 14px; border-radius: 14px; overflow: hidden;">
    <tr style="background-color: #007BFF; color: white;">
        <th style="padding: 8px; border: 1px solid #ccc;">Product Order</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Issue Date</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Start Date</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Delivery Date</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Product Name</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Quantity of Products</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Weight Description</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Box Description</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Box Quantity</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Lot Number</th>
        <th style="padding: 8px; border: 1px solid #ccc;">Lot Code</th>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">61549</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2024-12-12</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2024-12-09</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2024-12-15</td>
        <td style="padding: 8px; border: 1px solid #ccc;">AMORAS</td>
        <td style="padding: 8px; border: 1px solid #ccc;">180000</td>
        <td style="padding: 8px; border: 1px solid #ccc;">12X70G</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CX PAP COLOR 12X100G V0</td>
        <td style="padding: 8px; border: 1px solid #ccc;">200</td>
        <td style="padding: 8px; border: 1px solid #ccc;">450</td>
        <td style="padding: 8px; border: 1px solid #ccc;">GOMA</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">61489</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-10</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-11</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-15</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM PAVE DE ABACAXI</td>
        <td style="padding: 8px; border: 1px solid #ccc;">1000</td>
        <td style="padding: 8px; border: 1px solid #ccc;">12X70G</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CX PAP COLOR 12X100G V0</td>
        <td style="padding: 8px; border: 1px solid #ccc;">600</td>
        <td style="padding: 8px; border: 1px solid #ccc;">994</td>
        <td style="padding: 8px; border: 1px solid #ccc;">GOMA</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">61487</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2024-11-08</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2024-11-11</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2024-11-17</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM TORTA DE LIMAO</td>
        <td style="padding: 8px; border: 1px solid #ccc;">43200</td>
        <td style="padding: 8px; border: 1px solid #ccc;">24X12X15G</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CX COLOR DP AUT 24X12X15 V0</td>
        <td style="padding: 8px; border: 1px solid #ccc;">150</td>
        <td style="padding: 8px; border: 1px solid #ccc;">446</td>
        <td style="padding: 8px; border: 1px solid #ccc;">GOMA</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">61488</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-03</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-06</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-10-06</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM MORANGO CREMOSO</td>
        <td style="padding: 8px; border: 1px solid #ccc;">1008</td>
        <td style="padding: 8px; border: 1px solid #ccc;">24X12X15G</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CX COLOR DP AUT 24X12X15 V0</td>
        <td style="padding: 8px; border: 1px solid #ccc;">500</td>
        <td style="padding: 8px; border: 1px solid #ccc;">514</td>
        <td style="padding: 8px; border: 1px solid #ccc;">GOMA</td>
    </tr>
    <tr>
        <td style="padding: 8px; border: 1px solid #ccc;">61490</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-10</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-11</td>
        <td style="padding: 8px; border: 1px solid #ccc;">2026-05-17</td>
        <td style="padding: 8px; border: 1px solid #ccc;">FRUTIE SOBREM PAVE DE ABACAXI</td>
        <td style="padding: 8px; border: 1px solid #ccc;">1000</td>
        <td style="padding: 8px; border: 1px solid #ccc;">24X12X15G</td>
        <td style="padding: 8px; border: 1px solid #ccc;">CX COLOR DP AUT 24X12X15 V0</td>
        <td style="padding: 8px; border: 1px solid #ccc;">180</td>
        <td style="padding: 8px; border: 1px solid #ccc;">995</td>
        <td style="padding: 8px; border: 1px solid #ccc;">GOMA</td>
    </tr>
</table>

## Criação do banco de dados:

<style>
  .content img.sql-image-fullsize {
    max-width: none !important;
    max-height: none !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/sql_creation6.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: none !important; max-height: none !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>


## Inserção dos dados 

<style>
  .content img.sql-image-fullsize {
    max-width: none !important;
    max-height: none !important;
    width: auto !important;
    height: auto !important;
    display: block;
    margin: 0 auto;
  }
</style>

<img
  src="/images/sql_input2.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: none !important; max-height: none !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>
