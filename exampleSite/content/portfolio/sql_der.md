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

![](/images/erd.jpeg)

**Abaixo segue o MER (Modelo Entidade Relacionamento):**


```SQL
  PRODUCTION_ORDER(Po_id[PK], Issue_Date, Beginning_date, Delivery_date, OBS, Lot_id[FK])

  PRODUCT_UNITY(Product_id[PK], product_name) 

  PRODUCT_WEIGHT_UNITY(Product_id[PK and FK], Weight_id[PK and FK]) 

  PRODUCT_WEIGHT(Weight_id[PK], Box_id[FK], Weight_desc)

  TRANSLATION_PRODUCT(PRODUCT_id[PK and FK], Idiom_id[PK and FK], product_name_trans)

  IDIOM(Idiom_id[PK], Idiom_name)

  BOX_REGISTER(Box_id[PK], Box_desc) 	

  LOT(Lot_id[PK], Lot_category_id[FK], Lot_number) 

  LOT_CATEGORY(Lot_category_id[PK], Lot_cod)

  BOX_MOVEMENT(Box_id[PK AND FK], lot_id[PK AND FK], Quantity)

  PRODUCT_MOVEMENT(Product_id[PK and FK], Weight_id[PK and FK], Lot_id[PK AND FK], Quantity)
```
## Relacionamentos críticos

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

#### A solução para o relacionamento muitos para muitos (N:N): A entidade associativa

Como **muitos produtos podem ser associados a N pesos** e **muitos pesos podem ser associados a N produtos**, é necessária a criação de uma **tabela associativa** para garantir que todas as combinações sejam possíveis. Em tabelas desse tipo é criada uma **chave primária composta** formada pelas duas chaves primárias das tabelas que precisam ser relacionadas. Desse modo, é possível registrar o valor produto-peso no banco de dados!

```SQL
  PRODUCT_WEIGHT_UNITY(Product_id[PK and FK], Weight_id[PK and FK]) 
```

| product_id | weight_id
|---|---|
| F001 | 111070F07611|
| F001 | 113018A00104|
| F002 | 111070F07611|
| F002 | 113018A00104|
| F003 | 113018A00104|
#### Demonstração do comportamento da tabela associativa:

##### Query:
```SQL
--Functionality of the associative table PRODUCT_WEIGHT_UNITY:
SELECT pu.product_id, pu.product_name 'Product Name', pw.weight_id, pw.weight_desc 'Weight description'
FROM product_weight_unity AS pwu -- Associative table playing its rule by connecting the tables
    INNER JOIN product_unity AS pu
        ON pu.product_id = pwu.product_id
    INNER JOIN product_weight AS pw
        ON pw.weight_id = pwu.weight_id
WHERE pu.product_id like 'F%'
```
##### Output:
| product_id | Product Name |	weight_id |	Weight description
|---|---|---|---|
|F001	| FRUTIE SOBREM MORANGO CREMOSO	| 111070F07611	| 12X70G
|F001	| FRUTIE SOBREM MORANGO CREMOSO	| 113018A00104	| 24X12X15G
|F002	| FRUTIE SOBREM PAVE DE ABACAXI	| 111070F07611	| 12X70G
|F002	| FRUTIE SOBREM PAVE DE ABACAXI	| 113018A00104	| 24X12X15G
|F003	| FRUTIE SOBREM TORTA DE LIMAO	| 113018A00104	| 24X12X15G

Podemos observar o comportamento de muitos para muitos nessa tabela retornada pela query, notando que um **weight_id** pode estar associado a mais de um doce e um **product_id** pode estar associado a mais de um peso.

### 2. [PRODUCT_WEIGHT] 1:N [BOX_REGISTER] (Um para N):

#### Definição de tabelas

Como vimos no tópico anterior, a entidade **PRODUCT_WEIGHT** armazena os valores de peso que um produto pode assumir. Introduzo agora a entidade **BOX_REGISTER**, que armazena especificações de todas as embalagens disponíveis pela logística:

| box_id | box_desc |
|---|---|
| ME0BR257681	| CX PAP COLOR 12X100G V0 |
| ME0BR257703	| CX COLOR DP AUT 24X12X15 V0 | 

#### O relacionamento ideal:

Como um tipo de peso pode assumir N tipos de caixas, o relacionamento escolhido é um para N. Em modelagens **star schema — muito utilizadas no Power BI —**, a cardinalidade ideal é um para muitos, por se tratar do modelo que mais se aproxima da lógica de bancos de dados relacionais, onde, em um modelo tabular, o que se espera é que um elemento se comunique com N valores ou apenas um.

### 3. [LOT] N:1 [LOT_CATEGORY] (Muitos para um)

#### Definição de tabelas

Produtos são organizados em categorias e a **tabela LOT reflete essa regra**:
| lot_id | lot_number | Lot_category_id| 
|---|---|---|
|1|446|2|
|2|450|1|
|3|514|3|
|4|994|3|

A entidade **LOT_CATEGORY** é responsável por **armazenar essas categorias**:

|Lot_category_id| lot_cod|
|---|---|
|1|GOMA|
|2|GEL|
|3|CHICLETE|  

#### Regra de negócio garantida: um lote pode ter apenas uma categoria.
Como um lote pode pertencer a apenas **um tipo de lote**, a chave primária de **LOT_CATEGORY** entra como chave estrangeira na tabela **LOT**.
```SQL
  LOT(Lot_id[PK], Lot_category_id[FK], Lot_number) 
```

### 4. [PRODUCT_MOVEMENT] 1:1 [LOT] 1:1 [PRODUCTION_ORDER] 

#### Definição de tabelas:

A entidade **PRODUCTION_ORDER**, como o nome diz, é a ordem de produção.

A entidade **PRODUCT_MOVEMENT** é responsável por armazenar toda a movimentação de um produto. Essa tabela é o que chamamos de **entidade fraca**, porque ela tem como **sua chave primária uma chave composta estrangeira formada pela chave de PRODUCT_WEIGHT_UNITY e a chave de LOT**.

#### A normalização até a 3NF força um número maior de joins: um trade-off entre performance e organização:

Uma ordem de produção compartilha um mesmo lote registrado na tabela **PRODUCT_MOVEMENT**. Portanto, a conexão entre **PRODUCTION_ORDER** e **PRODUCT_MOVEMENT** é feita pela tabela **LOT**.

Por conta da normalização até a Terceira Forma Normal, cada tabela é especialmente designada para armazenar um tipo de informação. Desse modo, para a formulação de uma query que detalhe todos os elementos associados a uma ordem de produção, são necessários diversos joins:

##### Query:
```SQL
--Production order generation:
SELECT
     PO.po_id            AS 'Product Order',
     PO.issue_date       AS 'Issue Date',
     PO.beginning_date   AS 'Start Date',
     PO.delivery_date    AS 'Delivery Date',
     PO.obs              AS 'Obs',
     PU.product_name     AS 'Product Name',
     PM.quantity         AS 'Quantity of Products',
     PW.weight_desc      AS 'Weight Description',
     BR.box_desc         AS 'Box Description',
     BM.quantity         AS 'Box Quantity',
     L.lot_number        AS 'Lot Number',
     LC.lot_cod          AS 'Lot Code'
FROM production_order AS PO
     INNER JOIN lot AS L                -- Lote
          ON PO.lot_id = L.lot_id
     INNER JOIN product_movement AS PM  -- Quantidade de produtos
          ON L.lot_id = PM.lot_id
     INNER JOIN product_unity AS PU     -- Descrição do produto
          ON PM.product_id = PU.product_id
     INNER JOIN product_weight AS PW    -- Descrição do peso
          ON PM.weight_id = PW.weight_id
     INNER JOIN box_movement AS BM      -- Quantidade de embalagens
          ON L.lot_id = BM.lot_id
     INNER JOIN box_register AS BR      -- Descrição da embalagem
          ON BM.box_id = BR.box_id
     INNER JOIN lot_category AS LC      -- Categoria do lote
          ON L.lot_category_id = LC.lot_category_id
```
##### Output:
| Product Order | Issue Date | Start Date | Delivery Date | Product Name | Quantity of Products | Weight Description | Box Description | Box Quantity | Lot Number | Lot Code |
|---|---|---|---|---|---|---|---|---|---|---|
| 61549 | 2024-12-12 | 2024-12-09 | 2024-12-15 | AMORAS | 180000 | 12X70G | CX PAP COLOR 12X100G V0 | 200 | 450 | GOMA |
| 61489 | 2026-05-10 | 2026-05-11 | 2026-05-15 | FRUTIE SOBREM PAVE DE ABACAXI | 1000 | 12X70G | CX PAP COLOR 12X100G V0 | 600 | 994 | GOMA |
| 61487 | 2024-11-08 | 2024-11-11 | 2024-11-17 | FRUTIE SOBREM TORTA DE LIMAO | 43200 | 24X12X15G | CX COLOR DP AUT 24X12X15 V0 | 150 | 446 | GOMA |
| 61488 | 2026-05-03 | 2026-05-06 | 2026-10-06 | FRUTIE SOBREM MORANGO CREMOSO | 1008 | 24X12X15G | CX COLOR DP AUT 24X12X15 V0 | 500 | 514 | GOMA |
| 61490 | 2026-05-10 | 2026-05-11 | 2026-05-17 | FRUTIE SOBREM PAVE DE ABACAXI | 1000 | 24X12X15G | CX COLOR DP AUT 24X12X15 V0 | 180 | 995 | GOMA |

## Criação do banco de dados:

```SQL
CREATE DATABASE PRODUCTION_ORDER_DATABASE
GO

USE PRODUCTION_ORDER_DATABASE
GO

--TABLES CREATION:

CREATE TABLE product_unity( 
	product_id	char(04),
	product_name		varchar(100),
	constraint pk_product primary key(product_id)
)
GO

CREATE TABLE idiom(
	idiom_id		int not null,
	idiom_name		varchar(15),
	constraint pk_idiom primary key (idiom_id)
)
GO

CREATE TABLE box_register(
	box_id		char(11),
	box_desc	varchar(100),	
	constraint pk_box primary key	(box_id)
)
GO

CREATE TABLE lot_category(
	lot_category_id	int not null,
	lot_cod		char(08),
	constraint pk_category_id primary key (lot_category_id)
)
GO

CREATE TABLE product_weight(
	weight_id	char(12),
	weight_desc	varchar(50),
	box_id		char(11),
	constraint pk_weight_id primary key (weight_id),
	constraint fk_box_id foreign key (box_id) references box_register(box_id)
)
GO

CREATE TABLE product_weight_unity(
	product_id	char(04),
	weight_id	char(12),
	constraint pk_product_weight primary key (product_id, weight_id),
	constraint fk_product_id foreign key (product_id) references product_unity(product_id),
	constraint fk_weight_id foreign key (weight_id) references product_weight(weight_id)
)
GO

CREATE TABLE translation_product(
	product_id			char(04),
	idiom_id			int not null,
	product_name_trans	varchar(50),
	constraint pk_idiom_weight primary key (product_id, idiom_id),
	constraint fk_product_trans	foreign key (product_id) references product_unity(product_id),
	constraint fk_idiom_id  foreign key (idiom_id)  references idiom(idiom_id)
)
GO

CREATE TABLE lot(
	lot_id	int not null,
	lot_number int not null,
	lot_category_id int not null,
	constraint pk_lot_id primary key (lot_id),
	constraint fk_lot_category_id foreign key (lot_category_id) references lot_category(lot_category_id)
)
GO

CREATE TABLE production_order(
	po_id			int not null,
	issue_date		date,
	beginning_date	date,
	delivery_date	date,
	obs				varchar(100),
	lot_id	int not null UNIQUE,
	constraint fk_lot_po foreign key (lot_id) references lot(lot_id),
	constraint pk_po_id primary key (po_id),
)		
GO

CREATE TABLE product_movement(
    product_id  char(04)    not null,
    weight_id   char(12)	not null,
    lot_id      int         not null UNIQUE,
    quantity    int         not null,
    constraint pk_pm primary key (product_id, weight_id, lot_id),
    constraint fk_pwu_pm foreign key (product_id, weight_id) 
        references product_weight_unity(product_id, weight_id),
    constraint fk_lot_pm foreign key (lot_id) references lot(lot_id)
)

CREATE TABLE box_movement(
	box_id	char(11),
	lot_id	int not null,
	quantity int not null,
	constraint fk_box_bm foreign key (box_id) 
		references box_register(box_id),
	constraint fk_lot_bm foreign key (lot_id) 
		references lot(lot_id),
	constraint pk_bm primary key (box_id, lot_id)
)
GO
```
## Inserção dos dados 
``` SQL
--DATA INSERTION
INSERT INTO product_unity (product_id, product_name)
	VALUES	('F001', 'FRUTIE SOBREM MORANGO CREMOSO'),
			('F002', 'FRUTIE SOBREM PAVE DE ABACAXI'),
			('F003', 'FRUTIE SOBREM TORTA DE LIMAO'),
			('A001', 'AMORAS'),
			('D001', 'DENTADURAS')
GO

INSERT INTO idiom(idiom_id, idiom_name)
	VALUES	(1, 'ES'),
			(2, 'EN')
GO

INSERT INTO box_register(box_id, box_desc)
	VALUES	('ME0BR257681', 'CX PAP COLOR 12X100G V0'),
			('ME0BR257703', 'CX COLOR DP AUT 24X12X15 V0')
GO

INSERT INTO lot_category(lot_category_id, lot_cod)
	VALUES	(1, 'GOMA'),
			(2,	'GEL'),
			(3, 'CHICLETE')
GO

INSERT INTO product_weight(weight_id, weight_desc, box_id)
	VALUES	('111070F07611', '12X70G', 'ME0BR257681'),
			('113018A00104', '24X12X15G', 'ME0BR257703')
GO

INSERT INTO product_weight_unity(product_id, weight_id)
	VALUES	('A001', '111070F07611'),
			('F001', '113018A00104'),
			('F001', '111070F07611'),
			('F002', '111070F07611'),
			('F002', '113018A00104'),
			('F003', '113018A00104')
GO

INSERT INTO translation_product(product_id, idiom_id, product_name_trans)
	VALUES	('F001', 1, 'FRUTIE POSTRE FRESA CREMOSA'),
			('F001', 2, 'FRUTIE DESSERT CREAMY STRAWBERRY'),
			('F002', 1, 'FRUTIE POSTRE PAVÉ DE PIÑA'),
			('F002', 2, 'FRUTIE DESSERT PINEAPPLE PAVÉ'),
			('F003', 1, 'FRUTIE POSTRE TARTA DE LIMÓN'),
			('F003', 2, 'FRUTIE DESSERT LEMON PIE')
GO

INSERT INTO lot(lot_id, lot_number, lot_category_id)
	VALUES	(1, 446, 1),
			(2, 450, 1),
			(3, 514, 1),
			(4, 994, 1),
			(5, 995, 1)
GO

INSERT INTO production_order(po_id, issue_date, beginning_date, delivery_date, obs, lot_id)
	VALUES	(61549, '12/12/2024', '12/09/2024', '12/15/2024', null, 2),
			(61487, '11/08/2024', '11/11/2024', '11/17/2024', null, 1),
			(61488, '05/03/2026', '05/06/2026', '10/06/2026', null, 3),
			(61489, '05/10/2026', '05/11/2026', '05/15/2026', null, 4),
			(61490, '05/10/2026', '05/11/2026', '05/17/2026', null, 5)
GO

INSERT INTO product_movement(product_id, weight_id, lot_id, quantity)
	VALUES	('A001','111070F07611', 2, 180000),
			('F003','113018A00104', 1, 43200),
			('F001', '113018A00104',3, 1008),
			('F002','111070F07611', 4, 1000),
			('F002','113018A00104', 5, 1000)
GO

INSERT INTO box_movement(box_id, lot_id, quantity)
	VALUES	('ME0BR257703', 3, 500),
			('ME0BR257681', 4, 600),
			('ME0BR257681', 2, 200),
			('ME0BR257703', 1, 150),
			('ME0BR257703', 5, 180)
GO
```