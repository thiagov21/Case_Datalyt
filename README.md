
# 📊 Projeto de Análise de Redos - Datalyt

## 1. Visão Geral do Projeto

Este projeto foi desenvolvido para atender à necessidade de uma empresa de controle de pragas de entender e mitigar os altos custos e a frequência de **"redos"** (retrabalhos) em seus serviços.

O objetivo central do dashboard é:
- Identificar os funcionários responsáveis pela causa raiz dos "redos";
- Analisar o impacto financeiro e de clientes;
- Fornecer insights para a melhoria contínua dos processos.

🔗 [Acesse o Dashboard no Power BI](https://app.powerbi.com/view?r=eyJrIjoiZWFmNDFmNDctMzQzZS00N2RmLThiODktNjViNDNhNjJmZGVhIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)

### Perguntas de Negócio Respondidas:
- Quais funcionários estão causando o maior prejuízo com "redos"?
- O problema dos "redos" está melhorando ou piorando ao longo do tempo?
- Qual o ranking mensal de performance dos funcionários em relação aos "redos"?
- Qual o impacto financeiro real (lucratividade e perdas) do problema?
- Existem clientes ou áreas geográficas com maior concentração de problemas?

## 2. Fonte de Dados

- **Sistema Gerenciador:** PostgreSQL  
- **Servidor:** `ep-sweet-hall-a81n9c4c.eastus2.azure.neon.tech`  
- **Banco de Dados:** `neondb`  
- **Schema:** `public`  
- **Tabela:** `jobs`  

A conexão com o Power BI foi realizada via conector nativo PostgreSQL, em **modo Importação** (Import Mode), garantindo desempenho e capacidade de transformação no Power Query.

## 3. Tratamento e Transformação de Dados (Power Query)

A lógica central foi identificar corretamente o responsável por um "redo", considerando a visita anterior no mesmo serviço. As principais etapas foram:

### Etapas Realizadas:
- **Divisão da Coluna `job_number`:** Separação por "-" criando `ID_Servico` e `Num_Visita`.
- **Ordenação Cronológica:** Por `ID_Servico` e `Num_Visita` para garantir a sequência lógica das visitas.
- **Adição de Índice:** Para possibilitar o acesso à linha anterior.
- **Criação da Coluna `Responsavel_Redo`:**  
  ```m
  try if [visit_type] = "redo" and #"EtapaAnterior"{[Índice]-1}[ID_Servico] = [ID_Servico] 
  then #"EtapaAnterior"{[Índice]-1}[employee_name] 
  else null 
  otherwise null
  ```
- **Criação da Coluna `Custo_Redo`:**  
  ```m
  if [visit_type] = "redo" then [visit_cost_dollars] else 0
  ```
- **Ajuste de Tipos de Dados:** Datas, números e textos foram revisados e convertidos corretamente.

## 4. Modelo de Dados e Medidas DAX

O modelo possui:
- A tabela tratada `jobs`;
- Uma tabela calendário gerada com `CALENDARAUTO()`.

### Principais Medidas DAX:

| Medida                | Fórmula DAX                                                                                  | Propósito                                                                 |
|-----------------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| **Custo Total Redos** | `SUM('jobs'[Custo_Redo])`                                                                    | Calcula o prejuízo financeiro direto dos "redos".                        |
| **Qtd Redos Causados**| `COUNT('jobs'[Responsavel_Redo])`                                                            | Conta o número total de retrabalhos com responsável identificado.         |
| **Taxa de Redo**      | `DIVIDE([Qtd Redos Causados], COUNTROWS('jobs'))`                                           | Percentual de visitas que geraram retrabalho.                            |
| **Ranking Redo**      | `RANKX(ALLSELECTED('jobs'[Responsavel_Redo]), CALCULATE([Qtd Redos Causados]), , DESC)`     | Ranking dinâmico de funcionários.                                        |
| **Lucro Total**       | `[Receita Total] - [Custo Total Redos]`                                                      | Mede a lucratividade real do negócio.                                    |
| **Nº de Clientes**    | `DISTINCTCOUNT('jobs'[customer_name])`                                                       | Quantidade de clientes únicos atendidos.                                 |

## 5. Estrutura e Navegação do Dashboard

O dashboard foi dividido em **duas páginas** com foco em **diagnóstico** e **impacto**.

### 📌 Página 1: Análise da Causa Raiz
- **Objetivo:** Identificar quem está causando os redos e acompanhar tendências.
- **Visuais e Respostas:**
  - **Funcionários com Maior Custo de Redo** → Quem causa mais prejuízo.
  - **Evolução do Custo com Redos** → Problema está melhorando ou não.
  - **Matriz de Ranking Mensal** → Performance mensal por funcionário.

### 📌 Página 2: Análise de Impacto (Clientes e Finanças)
- **Objetivo:** Avaliar consequências dos redos para o negócio.
- **Visuais e Respostas:**
  - **Lucro por Tipo de Visita** → Mostra o impacto direto do redo no resultado.
  - **KPI Lucro Total** → Resultado final do negócio.
  - **Clientes Menos Lucrativos** → Relação com recorrência de redos.
  - **Mapa de Concentração de Redos** → Diagnóstico geográfico do problema.

## 6. Consultas SQL (Para Validação e Análises Adicionais)

### 🔍 1. Visitas de um serviço específico:
```sql
SELECT *
FROM public.jobs
WHERE job_number LIKE '1001-%'
ORDER BY job_number;
```

### 🔍 2. Redos executados por funcionário (sem atribuição de culpa):
```sql
SELECT
    employee_name,
    COUNT(*) AS total_redos_executados
FROM public.jobs
WHERE visit_type = 'redo'
GROUP BY employee_name
ORDER BY total_redos_executados DESC;
```

### 🔍 3. Custo, Receita e Lucro por Cliente:
```sql
SELECT
    customer_name,
    SUM(visit_cost_dollars) AS custo_total,
    SUM(visit_revenue_dollars) AS receita_total,
    (SUM(visit_revenue_dollars) - SUM(visit_cost_dollars)) AS lucro_total
FROM public.jobs
GROUP BY customer_name
ORDER BY lucro_total DESC;
```

## 📌 Conclusão

Este projeto fornece uma abordagem prática e visualmente clara para entender o impacto dos retrabalhos operacionais em uma empresa de serviços. A análise permite decisões orientadas por dados, com foco em melhoria contínua e performance financeira.
