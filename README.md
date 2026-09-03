# Desafio de Projeto: Processamento de Dados Simplificado com Power BI e MySQL na Azure 📊🚀

Este repositório contém a resolução prática do desafio de projeto focado em **ETL (Extração, Transformação e Carga)** utilizando o **Power BI Desktop**, integrado a um banco de dados relacional **MySQL hospedado na nuvem (Azure)**.

O objetivo principal foi sanar anomalias estruturais, validar regras de negócio, realizar junções relacionais e construir um Dashboard executivo otimizado.

---

## 🛠️ Etapas do Projeto

### 1. Criação da Infraestrutura e Carga de Dados
* Provisionamento de uma instância de banco de dados utilizando o **Azure Database for MySQL flexible server**.
* Execução do script SQL de criação e povoamento da base de dados corporativa `Company`.
* Integração nativa do Power BI Desktop ao banco de dados em nuvem através de strings de conexão seguras.

---

### 2. Processamento e Transformação de Dados (Power Query) 🧹

Durante a fase de tratamento no Power Query, as seguintes diretrizes foram rigorosamente aplicadas para garantir a qualidade analítica dos dados:

1. **Padronização de Cabeçalhos e Tipagem**: A primeira linha das tabelas foi promovida a cabeçalho. Colunas de identificação e chaves estrangeiras cruciais como `Ssn`, `Super_ssn` e `Mgr_ssn` foram configuradas explicitamente como **Texto** (`ABC`) para evitar a perda de zeros à esquerda e garantir a integridade dos relacionamentos.
2. **Alta Precisão Monetária (Double)**: A coluna de salários (`Salary`) na tabela `employee` foi alterada para o tipo **Número Decimal Fixo**, eliminando imprecisões matemáticas em cálculos futuros de soma e média.
3. **Análise de Valores Nulos e Regras de Negócio**: 
   * Na tabela `employee`, a coluna `Super_ssn` apresentou um registro nulo correspondente ao colaborador **James Borg**. A linha foi mantida intacta, validando a regra de negócio de que ele ocupa o topo da hierarquia corporativa (Diretor/Presidente), não possuindo supervisor.
   * Na tabela `department`, foi validado que todos os departamentos possuem gerentes associados na coluna `Mgr_ssn` (sem ocorrência de lacunas).
4. **Mesclagem de Consultas (Funcionário ↔ Departamento)**: Realizou-se a junção da tabela `employee` com a tabela `department` (Junção Externa Esquerda baseada no ID do departamento) para trazer o nome do departamento (`Dname`) diretamente para o registro do colaborador, eliminando chaves redundantes.
5. **Auto-Relacionamento (Funcionário ↔ Gerente)**: Para exibir nominalmente quem gerencia quem, realizou-se uma auto-mesclagem na tabela `employee`, vinculando a coluna `Super_ssn` à coluna `Ssn`. A coluna resultante foi expandida para exibir o primeiro nome do supervisor.
6. **Unificação Textual de Nomes**: As colunas de primeiro nome (`Fname`) e sobrenome (`Lname`) foram mescladas em uma única coluna chamada `Nome Completo`, utilizando o caractere de espaço como separador.
7. **Tratamento de Localizações**: Na tabela `dept_locations`, associou-se o nome do departamento à sua localização física criando o campo composto unificado `Depto_Localizacao`.
   * *Nota Teórica:* Utilizou-se o conceito de **Mesclar (Merge)** em vez de Acrescentar/Atribuir (Append). A mesclagem atua de forma *horizontal* (adicionando colunas com base em chaves em comum, similar ao `JOIN` do SQL). O comando Atribuir atua de forma *vertical* (empilhando linhas em tabelas de estruturas idênticas, similar ao `UNION`), o que desestruturaria o modelo de dados relacional deste cenário.
8. **Agrupamento de Dados**: Criou-se uma consulta de referência analítica agrupada pela coluna de supervisão para mensurar exatamente quantos colaboradores existem sob o comando de cada gerente técnico.
9. **Otimização de Performance**: Todas as colunas estruturais temporárias (como objetos aninhados `Table` gerados por chaves estrangeiras do MySQL) foram eliminadas do final das tabelas para reduzir a pegada de memória do modelo VertiPaq do Power BI.

---

### 🗄️ Consulta SQL para Mapeamento de Gerência

Caso a regra de negócio de auto-relacionamento (vincular o colaborador ao seu respectivo gerente) precisasse ser validada diretamente via código na origem do banco de dados MySQL, a query utilizada seria:

```sql
SELECT 
    CONCAT(e.Fname, ' ', e.Lname) AS Colaborador,
    CONCAT(g.Fname, ' ', g.Lname) AS Gerente
FROM employee e
LEFT JOIN employee g ON e.Super_ssn = g.Ssn;
```

---

## 📊 Dashboard de Visão Geral Orçamentária e RH

O relatório final foi construído utilizando um design corporativo premium (Azul Marinho com destaques em Laranja) para dar clareza de apresentação. O painel inclui:
* **Cartões de KPI**: Exibição por extenso do Headcount Total (**8 funcionários**), Custo Total de Folha (**R$ 281.000,00**) e Média Salarial da Empresa (**R$ 35.125,00**).
* **Gráficos de Distribuição**: Análise visual de funcionários e partição de orçamento financeiro consumido por departamento (*Research*, *Administration* e *Headquarters*).
* **Matriz de Liderança**: Tabela detalhada permitindo o rastreamento imediato de equipes por gerência.
