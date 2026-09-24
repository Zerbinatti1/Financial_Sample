# Modelagem Dimensional — Star Schema com Financial Sample

## 🎯 Objetivo
Neste projeto eu transformei a base **Financial Sample** (uma única tabela, no formato relacional) em um modelo dimensional no padrão **Star Schema**, dentro do Power BI.
O objetivo foi sair de uma tabela só, com tudo misturado, e chegar em um modelo separado em tabela fato e tabelas dimensão, deixando as análises de vendas mais rápidas, mais organizadas e mais fáceis de explorar.

## 🧠 Sobre o projeto
Este é o meu desafio de projeto da trilha de **Power BI**, com foco em:

- Modelagem dimensional (fato x dimensão)
- Transformações no Power Query (ETL)
- Criação de chaves substitutas
- Uso de DAX para a dimensão de datas

A ideia central foi pegar dados de venda "crus" e organizá-los de um jeito que qualquer pessoa consiga montar gráficos por produto, por país, por desconto ou por período sem esbarrar em uma tabela gigante e confusa.

## ⭐ Estrutura do modelo
O modelo segue o formato de estrela: uma tabela fato no centro, recebendo informações de várias tabelas dimensão ao redor.

### 🟡 Tabela fato — `F_Vendas`
É a tabela central, com uma linha para cada venda registrada:

- `SK_ID` (chave substituta)
- `ID_Produto`
- `Produto`
- `Units Sold`
- `Sales Price`
- `Discount Band`
- `Segment`
- `Country`
- `Sales Value`
- `Profit`
- `Date`

Com ela dá pra responder perguntas como:
- vendas por produto
- desempenho por país
- resultado por segmento
- evolução das vendas ao longo do tempo

### 🔵 Tabelas dimensão

**`D_Produtos`**
- `ID_Produto`
- `Produto`
- `Contagem` (soma total de unidades vendidas do produto)
- `Valor Minimo de Venda`
- `Valor Maximo de Venda`
- `Media do Valor de Vendas`
- `Mediana do Valor de Vendas`
- `Media da Manufatura`

**`D_Produtos_Detalhes`**
- `SK_ID`
- `ID_Produto`
- `Discount Band`
- `Sale Price`
- `Units Sold`
- `Manufactoring Price`

**`D_Descontos`**
- `ID_Desconto`
- `ID_Produto`
- `Discount` (formatado em porcentagem)
- `Discount Band`

**`D_Detalhes`**
Tabela que reúne as informações que não se encaixavam em nenhuma outra dimensão, mas que ainda ajudam a entender a venda:
- `SK_ID`
- `Gross Sales`
- `Discount Amount`
- `COGS`

## 🔑 Chave substituta (SK_ID)
Para conectar as tabelas de detalhe (`D_Produtos_Detalhes`, `D_Detalhes`) com a fato sem depender de textos, criei uma chave numérica sequencial em cada uma delas, usando a coluna de índice do Power Query:

1. Parti sempre da consulta de origem (`Financials_origem`)
2. Selecionei só as colunas que interessavam para aquela tabela
3. Adicionei uma coluna de índice (0, 1, 2, 3...) para gerar o `SK_ID`
4. Usei esse mesmo índice para relacionar a tabela com a `F_Vendas` na view de modelo

Assim, os relacionamentos ficaram todos no formato:

```
D_Dimensão (1) → (N) F_Vendas
```

## 📅 Dimensão de datas (DAX)
A única parte do modelo feita com DAX foi a tabela de calendário, criada direto no Power BI a partir do menor e do maior valor de data encontrados na `F_Vendas`:

```dax
D_Calendario = 
ADDCOLUMNS(
    CALENDAR(MIN(F_Vendas[Date]), MAX(F_Vendas[Date])),
    "Ano", YEAR([Date]),
    "NumeroMes", MONTH([Date]),
    "NomeMes", FORMAT([Date], "MMMM"),
    "Trimestre", "T" & FORMAT([Date], "Q"),
    "DiaDaSemana", FORMAT([Date], "dddd"),
    "AnoMes", FORMAT([Date], "YYYY-MM")
)
```

Com essa tabela marcada como "Tabela de Datas", consigo filtrar e comparar as vendas por ano, mês, trimestre ou dia da semana.

## ⚙️ Etapas do desenvolvimento
1. Importei a base original e criei a consulta `Financials_origem`, deixada oculta no relatório
2. Criei a coluna condicional `ID_Produto`, transformando o nome de cada produto em um número
3. A partir da origem, gerei cada tabela dimensão com Selecionar Colunas, Renomear Colunas e Agrupar Por
4. Montei a tabela fato `F_Vendas`, com a chave substituta `SK_ID`
5. Criei a dimensão de datas `D_Calendario` com DAX
6. Fui até a view de Modelo e conectei todas as dimensões à tabela fato
7. Ajustei o formato de cada coluna (moeda, porcentagem e data) para o relatório já nascer com os números certos

## 🛠️ Tecnologias utilizadas
- Power BI
- Power Query (M)
- DAX (apenas na dimensão de datas)
- Modelagem dimensional (Star Schema)

## 🖥️ Redes Sociais
www.linkedin.com/in/matheus-zerbinatti-a991b7204/
