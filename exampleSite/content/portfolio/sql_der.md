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


```SQL
  PRODUCTION_ORDER(Po_id[PK], Issue_Date, Beginning_date, Delivery_date, OBS, Lot_id[FK])

  PRODUCT_UNITY(Product_id[PK], product_name) 

  PRODUCT_WEIGHT_UNITY(Product_id[PK and FK], Weight_id[PK and FK]) 

  PRODUCT_WEIGHT(Weight_id[PK], Box_id[FK], Weight_desc)

  TRANSLATION_WEIGHT(Weight_id[PK and FK], Idiom_id[PK and FK], Weight_desc_trans)

  IDIOM(Idiom_id[PK], Idiom_name)

  BOX_REGISTER(Box_id[PK], Box_desc) 	

  LOT(Lot_id[PK], Lot_cat_id[FK], Lot_number) 

  LOT_CATEGORY(Lot_category_id[PK], Lot_cod)

  BOX_MOVEMENT(Box_id[PK AND FK], Po_id[PK AND FK], lot_id[FK], Quantity)

  PRODUCT_MOVEMENT(Product_id[PK and FK], Weight_id[PK and FK], Po_id[PK AND FK], Lot_id[FK], Quantity)
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

#### A solução para o relacionamento muitos para muitos (N:N): A entidade associativa

Como **muitos produtos podem ser associados a N pesos** e **muitos pesos podem ser associados a N produtos** é necessário a criação de uma **tabela associativa** para garantir que todas as combinações serão possíveis. Em tabelas desse tipo é criada uma **chave primaria composta** formada pelas duas chaves primarias das tabelas que precisam ser relacionadas. Desse modo, é possível registrar o valor produto-peso no banco de dados - Uma chave primaria não pode se repetir!

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

##### Querry:
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

Podemos observar o comportamento de muitos para muitos nessa tabela retornada pela querry notando que um **weight_id** pode estar associado a mais de um doce e um **product_id** pode estar associado a mais de um peso.

### 2. [PRODUCT_WEIGHT] 1:N [BOX_REGISTER] (Um para N):

#### Definição de tabelas

Como vimos no tópico anterior, a entidade **PRODUCT_WEIGHT** armazena os valores de peso que um produto pode assumir. Introduzo agora a entidade **BOX_REGISTER** que armazena especificações de todas as embagens disponíveis pela logistica:

| box_id | box_desc |
|---|---|
| ME0BR257681	| CX PAP COLOR 12X100G V0 |
| ME0BR257703	| CX COLOR DP AUT 24X12X15 V0 | 

#### O relacionamento ideal:

Como um tipo de peso pode assumir N tipos de caixas, o relacionamento escolhido é um para N. Em modelagens star squema - muito utilizada no Power BI -, a **cardinalidade ideal** é **um para muitos** por se tratar do modelo que mais se aproxima da lógica de bancos de dados relacionais.

### 3. [LOT] N:1 [LOT_CATEGORY] (Muitos para um)

#### Definição de tabelas

Produtos são organizados em categorias e a **tabela LOT reflete essa regra**:
| lot_id | lot_number | lot_cod_id| 
|---|---|---|
|1|446|2|
|2|450|1|
|3|514|3|
|4|994|3|

A entidade **LOT_CATEGORY** é responsável por **armazenar essas categorias**:

|lot_cod_id| lot_cod|
|---|---|
|1|GAMO|
|2|GFRS|
|3|CX|  

#### Regra de négocio garantida: Um lote pode ter apenas uma categória.
Como um lote pode pertecer a apenas **um tipo de lote**, a chave primária de **LOT_CODE** entra como chave estrangeira na tabela **LOT**.
```SQL
  LOT(Lot_id[PK], Lot_cod_id[FK], Lot_number) 
```

### 4. [PRODUCT_MOVEMENT] 1:1 [LOT] 1:1 [PRODUCTION_ORDER] 

#### Definição de tabelas

A entidade **PRODUCTION_ORDER**, como o nome diz é a ordem de produção.

A entidade **PRODUCT_MOVEMENT** é responsável por armazenar toda a movimentação de um produto. Essa tabela é o que chamamons de **entidade fraca**, porque ela tem como **sua chave primária uma chave composta estrangeira formada pela chave de PRODUCT_WEIGHT_UNITY e a chave da PRODUCTION_ORDER**

#### A normalização até a 3NF força um número maior de joins: Um trade-off sobre performance e organização

Uma ordem de produção compartilha um mesmo lote registrado na tabela de transação (**PRODUCT_MOVEMENT**). Portanto, a conexão entre **PRODUCTION_ORDER** e **PRODUCTION_MOVEMENT** é pela tabela **LOT**

Por conta da normalização até a terceira forma normal, cada tabela é especialmente designida para armazenar um tipo de informação. Desse modo, para a formulação de uma querry que detalha todos os elementos associados a uma ordem de produção, são nessários diversos joins:



## DDL (Data Definition Language) do banco

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

CREATE TABLE lot_code(
	lot_cod_id	int not null,
	lot_cod		char(04),
	constraint pk_lot_cod primary key (lot_cod_id)
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

CREATE TABLE translation_weight(
	weight_id			char(12),
	idiom_id			int not null,
	weight_desc_trans	varchar(50),
	constraint pk_idiom_weight primary key (weight_id, idiom_id),
	constraint fk_weight_trans	foreign key (weight_id) references product_weight(weight_id),
	constraint fk_idiom_id  foreign key (idiom_id)  references idiom(idiom_id)
)
GO

CREATE TABLE lot(
	lot_id	int not null,
	lot_number int not null,
	lot_cod_id int not null,
	constraint pk_lot_id primary key (lot_id),
	constraint fk_lot_cod_id foreign key (lot_cod_id) references lot_code(lot_cod_id)
)
GO

CREATE TABLE production_order(
	po_id			int not null,
	issue_date		date,
	beginning_date	date,
	delivery_date	date,
	total_2_to_one	int not null,
	obs				varchar(100),
	weight_id	char(12),
	lot_id	int not null,
	constraint pk_po_id primary key (po_id), 
	constraint fk_product_weight_po foreign key (weight_id) references product_weight(weight_id),
	constraint fk_lot_po foreign key (lot_id) references lot(lot_id)
)		
GO

CREATE TABLE product_movement(
	product_id	char(04),
	lot_id	int not null,
	quantity int not null,
	constraint fk_pro_pm foreign key (product_id) references product_unity(product_id),
	constraint fk_lot_pm foreign key (lot_id) references lot(lot_id),
	constraint pk_pk primary key (product_id)
)
GO

CREATE TABLE box_movement(
	box_id		char(11),
	lot_id	int not null,
	quantity int not null,
	po_id			int not null,
	constraint fk_box_bm foreign key (box_id) references box_register(box_id),
	constraint fk_lot_bm foreign key (lot_id) references lot(lot_id),
	constraint pk_bm primary key (box_id),
	constraint fk_po_bx foreign key (po_id) references production_order(po_id)
)
GO


--DATA INSERTION

INSERT INTO product_unity (product_id, product_name)
	VALUES	('F005', 'FRUTIE SOBREM MORANGO CREMOSO'),
			('F013', 'FRUTIE SOBREM PAVE DE ABACAXI'),
			('F014', 'FRUTIE SOBREM TORTA DE LIMAO'),
			('A008', 'AMORAS 1,55G'),
			('D017', 'DENTADURAS 5,5G M3')
GO

INSERT INTO idiom(idiom_id, idiom_name)
	VALUES	(1, 'ES'),
			(2, 'EN')
GO

INSERT INTO box_register(box_id, box_desc)
	VALUES	('ME0BR257681', 'CX PAP COLOR 12X100G V0'),
			('ME0BR257703', 'CX COLOR DP AUT 24X12X15 V0')
GO

INSERT INTO lot_code(lot_cod_id, lot_cod)
	VALUES	(1, 'GAMO'),
			(2,	'GFRS'),
			(3, 'CX')
GO

INSERT INTO product_weight(weight_id, weight_desc, box_id)
	VALUES	('111070F07611', 'FRUTIE SOBREMESAS 12X70G', 'ME0BR257681'),
			('113018A00104', 'AMORAS 24X12X15G', 'ME0BR257703')
GO

INSERT INTO product_weight_unity(product_id, weight_id)
	VALUES	('A008', '113018A00104'),
			('F005', '111070F07611'),
			('F013', '111070F07611'),
			('F014', '111070F07611')
GO

INSERT INTO translation_weight(weight_id, idiom_id, weight_desc_trans)
	VALUES	('113018A00104', 1, 'GOMA MORAS 15GX12X24'),
			('111070F07611', 1, 'POSTRE FRUTIE 70GX12'),
			('113018A00104', 2, 'BLACKBERRY GUMMIES 15G 12CT X 24PK'),
			('111070F07611', 2, 'FRUTIE DESSERTS 70G 12CT')
GO

INSERT INTO lot(lot_id, lot_number, lot_cod_id)
	VALUES	(1, 446, 2),
			(2, 450, 1),
			(3, 514, 3),
			(4, 994, 3)
GO

INSERT INTO production_order(po_id, issue_date, beginning_date, delivery_date, total_2_to_one, obs, weight_id, lot_id)
	VALUES	(61549, '12/12/2024', '12/09/2024', '12/15/2024', 180000, null, '113018A00104', 2),
			(61487, '11/08/2024', '11/11/2024', '11/17/2024', 43200, null, '111070F07611', 1)
GO

INSERT INTO product_movement(product_id, lot_id, quantity)
	VALUES	('A008', 2, 32400),
			('F005', 1, 1008),
			('F013', 1, 1008),
			('F014', 1, 1008)
GO

INSERT INTO box_movement(box_id, lot_id, quantity, po_id)
	VALUES	('ME0BR257703', 3, 3600, 61549),
			('ME0BR257681', 4, 7500, 61487)
GO
```

