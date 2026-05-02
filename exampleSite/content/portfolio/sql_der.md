+++
categories = ["sql"]
coders = []
date = 2020-06-19T23:00:00Z
description = "Diagrama entidade relacionamento e construção de banco de dados usando SQL Server inspirado no workflow das ordens de produção da empresa The Fini Company."
github = ["https://github.com/Brunorwell/Production_Order"]
image = "https://res.cloudinary.com/samrobbins/image/upload/q_auto/v1591793276/logos/logos_hugo_h2xbne.svg"
title = "Diagrama Entidade-Relacionamento (DER)"
type = "post"
[[tech]]
logo = "/logos/sql_server.svg"
[[tech]]
logo = "/logos/docker.svg"
name = "Docker"


+++
Esse projeto reproduz o que acontece por de baixo dos panos do ERP Protheus na empresa em que eu trabalhava. Eu percebi que as ordens de produção que eu tinha de lidar eram basicamente um banco de dados relacional - Você consegue até identificar as **chaves primarias e estrangeiras** apenas observando atentamente os campos! Eu decidi reproduzir em um escala pequena um banco de dados inspirado por esse processo respeitando as boas praticas de **modelagem de banco de dados relacionais**.

O objetivo desse projeto, é, portanto, demonstrar habilidades em **modelagem de dados**: Um processo que envolve principalmente a **normalização** - identificação de quais **tabelas** e **relacionamentos** são necessárias para o modelo.

No mundo real, por conta de trade-offs, a normalização extrita dos dados nem sempre é utilizada, mas para esse projeto foi considerado normalização até a **terceira Forma Normal (3NF)** para fins de aplicabilidade de conhecimentos adquiridos na faculdade.

## Breve contexto

A The Fini Company é um empresa espanhola que fabrica doces. Eu trabalhava na fabrica do Brasil.

Uma **Ordem de Produção** é um documento que orienta o **operador de maquina** a **gerenciar a produção solicitada** pelo engenheiro de produção. A ordem de produção contém informações do tipo:
  - Quantidade a ser produzida
  - Lote
  - Nome do produto
  - A unidade de medidas das embalagens
  - Tipo de embalagem a ser retirada na logistica

Como o documento não é organizado como um banco de dados relacional, foi necessário identificar todas as entidades (tabelas) envolvidas em todo o processo e garantir que as tabelas tivesse relacionamentos apropriados para garantir as **regras de negocio** da companhia.

**A imagem abaixo representa o Diagrama Entidade Relacionamento do modelo:**

![](/images/erd.jpeg)

**Abaixo segue o MER (Modelo entidade relacionamento):**


```
  PRODUCTION_ORDER(Po_id[PK], Issue_Date, Beginning_date, Delivery_date, Total_2_To_One, OBS, Lot_id[FK], Weight_id[FK])

  PRODUCT_UNITY(Product_id[PK], product_name) 

  PRODUCT_WEIGHT_UNITY(Product_id[PK and FK], Weight_id[PK and FK]) 

  PRODUCT_WEIGHT(Weight_id[PK], Box_id[FK], Weight_desc)

  TRANSLATION_WEIGHT(Weight_id[PK and FK], Idiom_id[PK and FK], Weight_desc_trans)

  IDIOM(Idiom_id[PK], Idiom_name)

  BOX_REGISTER(Box_id[PK], Box_desc) 	

  LOT(Lot_id[PK], Lot_cod_id[FK], Lot_number) 

  LOT_CODE(Lot_cod_id[PK], Lot_cod)

  BOX_MOVEMENT(Box_id[PK AND FK], Po_id[PK AND FK], lot_id[FK], Quantity)

  PRODUCT_MOVEMENT(Product_id[PK], Lot_id[FK], Quantity)
```
## Relacionamentos criticos

### 1. [PRODUCT_UNITY] N:N [PRODUCT_WEIGHT] (Muitos para muitos):

#### Definição de tabelas
A entidade **PRODUCT_UNITY** é responsável por armazenar todos os doces produzidos:

| product_id | product_name |
|---|---|
| A008 | AMORAS 
| D017 | DENTADURAS


A entidade **PRODUCT_WEIGHT** é responsável por armazenar os tipos de pesos que um produto pode ser associado:

| weight_id | weight_desc | box_id |
|---|---|---|
| 111070F07611 | 12X70G | ME0BR257681 |
| 113018A00104 | 24X12X15G | ME0BR257703 |

#### A solução para o relacionamento muitos para muitos (N:M): Entidade associativa

Como **muitos produtos podem ser associados a N pesos** e **muitos pesos podem ser associados a N produtos** é necessário a criação de uma **tabela associativa** para garantir que todas as combinações serão possíveis. Em tabelas desse tipo é criada um **chave primaria** formada pelas duas chaves primarias das tabelas que precisam ser relacionadas:
``` 
  PRODUCT_WEIGHT_UNITY(Product_id[PK and FK], Weight_id[PK and FK]) 
```

### 2. [PRODUCT_WEIGHT] 1:N [BOX_REGISTER] (Um para N):

#### Definição de tabelas

Como vimos no tópico anterior, a entidade **PRODUCT_WEIGHT** armazena os valores de peso que um produto pode assumir. Introduso agora a entidade **BOX_REGISTER** que armazena especificações de todas as embagens disponíveis pela logistica:

| box_id | box_desc |
|---|---|
| ME0BR257681	| CX PAP COLOR 12X100G V0 |
| ME0BR257703	| CX COLOR DP AUT 24X12X15 V0 | 

#### O relacionamento ideal:

Como um tipo de peso pode assumiir N tipos de caixas, o relacionamento escolhido é um para N.Em modelagens star squema - muito utilizada no Power BI -, a **cardinalidade ideal** é **um para muitos** por se tratar do modelo que mais se aproxima da lógica de bancos de dados relacionais.

### 3. [LOT] 1:N [LOT_CODE] (Um para um)

#### Definição de tabelas

Produtos são organizados em categorias e a **tabela LOT reflete essa regra**:
| lot_id | lot_number | lot_cod_id| 
|---|---|---|
|1|446|2|
|2|450|1|
|3|514|3|
|4|994|3|

A entidade **LOT_CODE** é responsável por **armazenar essas categorias**:

|lot_cod_id| lot_cod|
|---|---|
|1|GAMO|
|2|GFRS|
|3|CX|  

#### Regra de négocio garantida: Um lote pode ter apenas uma categória.
Como um lote pode pertecer a apenas **um tipo de lote**, a chave primária de **LOT_CODE** entra como chave estrangeira na tabela **LOT**.

### 4. [PRODUCT_UNITY] 1:1 [PRODUCT_MOVEMENT] 1:1 [LOT] 1:1 [PRODUCTION_ORDER] 

#### Definição de tabelas

A entidade **PRODUCT_MOVEMENT** é responsável por armazenar toda a movimentação de um produto. Essa tabela é o que chamamons de **entidade fraca**, porque ela tem como **sua chave primária, uma chave estrangeira (a chave primaria da tabela PRODUCT_UNITY)**

A entidade **PRODUCTION_ORDER**, como o nome diz a ordem de produção.

#### A normalização até a 3NF força um número maior de joins:

Uma ordem de produção e um produto/produtos especifico compartilham um mesmo LOTE. Portanto, a conexão entre **PRODUCTION_ORDER** e **PRODUCTION_MOVEMENT** é pela tabela **LOT**

Por conta da normalização até a terceira forma normal, cada tabela é especialmente designida para armazenar um tipo de informação, 

Para a formulação completa de uma ordem de produção, são nessários diversos joins, 

