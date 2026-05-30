+++
categories = ["rpa"]
coders = []
date = 2026-05-13T23:00:00Z
description = "Pipeline de dados utilizando M (Power Query) para geração de relatórios gerenciais com interação do usuário via Userform (VBA)"
#github = ["https://github.com/Brunorwell/Production_Order"]
image = "logos/vba.svg"
title = "Automação para relatórios gerenciais utilizando VBA e M"
type = "post"
[[tech]]
logo = "/logos/vba3.svg"
[[tech]]
logo = "/logos/m.png"
name = "Docker"


+++
Projeto de automação de relatórios de horas trabalhadas em projetos aplicáveis a dois programas governamentais ligados à inovação na indústria nacional **(Mover e Lei do Bem)** e relatórios internos para assegurar a qualidade de apontamentos, desenvolvido no meu estágio na SEG Automotive. 

Os relatórios que a ferramenta é capaz de gerar: 

1. **Mover e Lei do Bem:** Relatórios de horas enviados à ABGi - consultoria responsável pelo intermédio entre a SEG e o Governo - para captação de incentivos fiscais através de projetos aplicáveis ao Mover e (ou) à Lei do Bem.

2. **Grupo de projetos:** Relatório de horas interno de horas para controle. Seu objetivo é uma análise que divide projetos em diversas classificações do negócio.

3. **Apontamentos Vazios:** **Relatório interno de qualidade** que avalia apontamentos realizados sem um projeto informado. Seu intuito é documentar os erros e orientar usuários para que corrijam apontamentos, maximizando horas.

4. **Apontamentos Inválidos:** **Relatório interno de qualidade** que avalia apontamentos realizados em projetos inativos. Seu intuito é documentar os erros e orientar usuários para que corrijam apontamentos, maximizando horas.

## A ferramenta Gerador de Relatórios

O desenvolvimento da ferramenta **Gerador de Relatórios** foi pensado para que qualquer pessoa do setor de engenharia fosse capaz de gerar os relatórios Mover e Lei do Bem **(relatórios que têm impacto real no EBIT da empresa)**, democratizando assim o acesso à informação por meio de um pipeline automatizado com uma interface amigável. Além de facilitar o acesso à informação, a ferramenta reduz drasticamente o tempo de geração de relatórios e permite uma governança maior dos dados - por meio da aba **Análise de apontamentos inconsistentes e a geração de relatórios de qualidade de apontamento (Apontamentos Vazios e Apontamentos Inválidos)**

A ferramenta possui dois módulos: Gerador de relatórios e Análise de apontamentos inconsistentes. Abaixo eu descrevo melhor a funcionalidade de cada função:

### 1. Geração de relatórios
![](/images/periode.gif)

A primeira aba da ferramenta **Geração de relatórios** permite que o usuário escolha o mês e o ano que serão utilizados nos relatórios - esses dados são armazenados em uma tabela Excel e passados como parâmetros no pipeline M para que o usuário tenha total controle dos filtros temporais utilizados nos relatórios.

Quando o usuário clica no botão **Atualizar base de dados**, um evento é disparado para que o Power Query possa atualizar a base de dados que os relatórios utilizam. Logo em seguida, após a mensagem de confirmação de atualização, o usuário pode selecionar qual o relatório deseja gerar através dos quatro botões agrupados no canto inferior esquerdo. Cada botão de geração de relatório salva uma cópia de um arquivo contendo o relatório para histórico e cria um e-mail formatado para o gestor responsável.


### 2. Análise de apontamentos Inconsistentes
![](/images/Apontamento_analise.gif)

Em casos específicos, um apontamento de horas pode ser considerado como **não válido** por diversos motivos como, por exemplo, o apontamento ter sido realizado alguns **dias após o projeto ter sido arquivado ou cancelado**. Por conta de situações que fogem ao controle de um processo 100% automatizado, surgiu a necessidade da criação de uma interface que permita a gestão desse tipo de especificidade por meio de uma tabela que documente casos dessa natureza.

Por meio da segunda aba da ferramenta, o usuário pode documentar a decisão sobre um determinado apontamento:

- **SIM:** Significa que o apontamento deve ser considerado como válido.
- **NÃO:** Significa que o apontamento é de fato inválido.
- **PENDENTE:** Significa que o apontamento fica aguardando novas definições.

Seja qual for a decisão, o analista pode escrever uma observação. Ao clicar no botão **Salvar**, o registro da decisão é salvo na tabela **Apontamentos inconsistentes** com uma **chave primária composta pelos campos data, projeto e nome do usuário** para que seja possível a conexão com a base de dados dos relatórios. Quando a base de dados for atualizada novamente, isso deve acontecer:
1. Apontamentos marcados como **SIM passam a ser válidos**.
2. Apontamentos marcados como **NÃO são excluídos da base de dados que os relatórios utilizam**.
3. Apontamentos marcados como **PENDENTES continuam sendo classificados como NÃO** até que uma providência seja tomada.

## Pipeline
>>Essa seção se concentra nos aspectos mais cruciais para o tratamento dos dados. O pipeline real incluiu diversas tratativas, mas para fins de apresentação, apenas as tratativas mais fundamentais estão presentes nessa seção.

![](/images/pipeline_projeto_vba15.svg)

### 1. Parâmetros temporais:
Como foi apresentado anteriormente, a primeira aba do **Gerador de relatórios** é capaz de armazenar os parâmetros **mês** e **ano** dentro de uma tabela Excel **(Periode_Paramenter)**.

Esses valores são passados para duas variáveis do pipeline e utilizadas nos filtros das colunas **Year** e **Month**.

![](/images/paramentros_temporais_code.svg)

O resultado após a aplicação dos filtros é uma tabela filtrada somente pelo ano selecionado na ferramenta e meses menores ou iguais ao mês selecionado.

### 2. O campo projeto

O campo projeto é um dos campos que mais necessitavam de transformações. Os projetos podem ser classificados em dois grupos: **Clientes** e **Internos**. Para cada grupo, são aplicadas estratégias de transformações diferentes:

#### Projetos internos

Projetos internos podem sofrer mudanças em suas descrições. Para que os usuários não tenham de atualizar em suas planilhas todos os apontamentos de um projeto que passou por uma alteração de forma retroativa, a tabela **Gestão de Mudanças** armazena as novas descrições e é usada para se comunicar com a **tabela de apontamentos compilada**.

É realizado um join. A conexão é feita pela capitalização da descrição do projeto, de forma que apontamentos que não sigam o formato correto da descrição ainda consigam se conectar à tabela de Gestão de Mudanças. As chaves usadas para o join são **Projeto_Key** com **Antes_Key**. 

>>Quando comecei o meu estágio, notei que não havia validação de input de valores nas planilhas. Desenvolvi uma macro (VBA) para fazer essa validação. Porém, como nem todos os usuários possuíam essa macro, a medida de **capitalização da descrição do projeto foi necessária.**

#### Projetos de Clientes e flags de projetos aplicáveis ao Mover e Lei do Bem

Projetos de clientes sofrem mudanças frequentes em suas descrições. Para evitar que a base de dados utilizada em relatórios tenha mais de uma descrição para um mesmo projeto se fez necessário o join com a tabela **Dados Registro** que armazena a descrição mais recente do projeto.

A conexão é feita pela Projeto_Key2 que é formada com a seguinte regra: para projetos de internos é mantida a descrição do projeto, mas para **projetos de clientes é mantida a única parte imutável do projeto: o código numérico que inicia o projeto.**

Nessa tabela, coletamos os campos **Mover** e **Lei do Bem** que servem como indicadores para mostrar se um projeto é aplicável a esses programas governamentais.

![](/images/primeiro_segundo_join3.svg)

#### Enriquecimento de campos

##### Tabela Funcionário: Campos matrícula e setor

Por meio da tabela Funcionário, podemos coletar os campos **matrícula** e **setor**. A chave de conexão é o **Nome do Usuário**.

>>Importante salientar que essa não é a melhor forma de identificar um usuário em um banco de dados, porém, por limitações da ferramenta, essa foi a maneira escolhida pelos desenvolvedores da ferramenta de apontamentos de horas.

##### Tabela Contrato: Campo Contrato
Por meio da tabela Funcionário, podemos coletar o campo **contrato**. A chave de conexão é o **Nome do Usuário**. Logo em seguida, a base é filtrada para que somente as horas de **Mensalistas** sejam usadas.

![](/images/terceiro_quarto_join.svg)

#### Coluna de Apontamentos Inconsistentes

A tabela **Datas de inativação de projetos** armazena datas de inativação de projetos - sejam eles internos ou de clientes. Utilizamos a chave **Project_Key2** (Base de apontamentos) e a chave **Projeto** (Datas de inativação de projetos) para o join.

Como vimos na seção de apresentação da aplicação **Gerador de relatórios**, a tabela **Apontamentos Inconsistentes armazena a decisão do analista sobre apontamentos de horas que precisam de uma análise mais cuidadosa**

![](/images/sexto_join.drawio.svg)

Os dados obtidos através do merge são usados para a formulação da coluna **Apontamento Válido?**. A segunda estrutura condicional é usada:

![](/images/m_code_validos.svg)

## Orquestração com VBA 

![](/images/diagrama_funções_vba6.svg)

### Generalização do código: Código organizado e escalável

A ferramenta de geração de relatórios funciona da seguinte forma: O arquivo tem uma aba geral que contém os dados de apontamentos e abas com uma tabela dinâmica cada, que resume os dados da aba geral de forma apropriada para cada relatório.

Como descrito no início, a ferramenta permite a geração de dois tipos de relatórios: **Apontamentos de horas** e **Qualidade de apontamentos**. Consequentemente, o padrão de código para a automatização dos relatórios é parecido:

```Markdown
    Criar arquivo
        ↓   
    Copiar e colar tabela dinâmica
        ↓
    Salvar arquivo
        ↓ 
    Enviar E-mail formatado
```
Mudando apenas parâmetros como filtros a serem aplicados, abas a serem copiadas e e-mail. Por esse motivo, decidi criar funções que resumem o código e permitem que, se for necessário em algum momento, seja possível configurar um novo relatório apenas chamando as funções já criadas e aplicando parâmetros diferentes.

Como podemos ver no diagrama de Venn, as funções **CriarArquivoPorAno** e **AbirEmailtoComPasta** são usadas nos dois tipos de relatórios e elas fazem exatamente o que os nomes sugerem: criam um arquivo novo no diretório especificado e constroem o e-mail formatado.

A função **CopiarAbaEColarTabelaFiltrada** é usada nos relatórios de apontamentos. Para fins de auditoria, foi necessário salvar no arquivo do relatório também a tabela geral de apontamentos com filtros que variam de acordo com o relatório.

A função **CopiarSomenteAbas** é para uso em relatórios de qualidade de apontamentos. É uma função mais enxuta que apenas copia abas que contenham tabelas dinâmicas.

Abaixo seguem dois exemplos de chamadas das funções para a geração dos relatórios:

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
  src="/images/chamada_funcao1_2.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: none !important; max-height: none !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>

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
  src="/images/chamada_funcao2_2.svg"
  alt="SQL Query Diagram"
  class="sql-image-fullsize"
  style="max-width: none !important; max-height: none !important; width: auto !important; height: auto !important; display: block; margin: 0 auto;"
/>



