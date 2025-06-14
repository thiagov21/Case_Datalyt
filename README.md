README: Projeto de Análise de "Redos" - Datalyt
1. Visão Geral do Projeto
Este projeto foi desenvolvido para atender à necessidade de uma empresa de controle de pragas de entender e mitigar os altos custos e a frequência de "redos" (retrabalhos) em seus serviços. O objetivo central do dashboard é identificar os funcionários responsáveis pela causa raiz dos "redos", analisar o impacto financeiro e de clientes, e fornecer insights para a melhoria contínua dos processos.

Link para o projeto: https://app.powerbi.com/view?r=eyJrIjoiZWFmNDFmNDctMzQzZS00N2RmLThiODktNjViNDNhNjJmZGVhIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9

O dashboard foi estruturado para responder às seguintes perguntas de negócio:

Quais funcionários estão causando o maior prejuízo com "redos"?

O problema dos "redos" está melhorando ou piorando ao longo do tempo?

Qual é o ranking mensal de performance dos funcionários em relação aos "redos"?

Qual o impacto financeiro real (lucratividade e perdas) do problema?

Existem clientes ou áreas geográficas com maior concentração de problemas?

2. Fonte de Dados
A análise foi baseada em uma única fonte de dados, conforme especificado no desafio:

Sistema Gerenciador: PostgreSQL

Servidor: ep-sweet-hall-a81n9c4c.eastus2.azure.neon.tech

Banco de Dados: neondb

Schema: public

Tabela: jobs

A conexão no Power BI foi feita utilizando o conector nativo do PostgreSQL, em modo Importação (Import Mode), para garantir a melhor performance durante a análise e a possibilidade de realizar transformações complexas no Power Query.

3. Tratamento e Transformação de Dados (Power Query)
A etapa de transformação foi a mais crítica do projeto. A lógica principal foi desenvolvida para atribuir corretamente a responsabilidade de um "redo" ao funcionário que realizou a visita anterior no mesmo serviço.

As seguintes etapas foram aplicadas no Editor do Power Query:

Divisão da Coluna job_number: A coluna foi dividida pelo delimitador "-" para criar ID_Servico e Num_Visita, permitindo a ordenação correta das visitas.

Ordenação Cronológica: A tabela foi ordenada primeiramente por ID_Servico (crescente) e depois por Num_Visita (crescente). Esta etapa é essencial para garantir que a lógica de "olhar para a linha anterior" funcione corretamente.

Adição de Coluna de Índice: Uma coluna de índice (iniciando em 0) foi adicionada para permitir a referência a linhas anteriores na tabela.

Criação da Coluna Responsavel_Redo: Utilizando uma Coluna Personalizada, a seguinte lógica M foi implementada para identificar o verdadeiro responsável:

// Para cada linha, verifica se o tipo é "redo" e se a visita anterior pertence ao mesmo serviço.
// Se sim, retorna o nome do funcionário da visita anterior.
try if [visit_type] = "redo" and #"EtapaAnterior"{[Índice]-1}[ID_Servico] = [ID_Servico] then #"EtapaAnterior"{[Índice]-1}[employee_name] else null otherwise null

Criação da Coluna Custo_Redo: Uma coluna condicional foi criada para isolar o custo associado apenas às visitas de retrabalho, facilitando as medidas DAX posteriores.

if [visit_type] = "redo" then [visit_cost_dollars] else 0

Ajuste de Tipos de Dados: Todos os tipos de dados foram revisados e ajustados (Datas, Números Decimais, Texto) para garantir a integridade dos cálculos.

4. Modelo de Dados e Medidas DAX
O modelo de dados é composto pela tabela jobs (tratada) e uma tabela Calendario desconectada, criada com CALENDARAUTO() para facilitar as análises de tempo.

As seguintes medidas DAX foram criadas para alimentar os visuais:

Medida

Fórmula DAX

Propósito

Custo Total Redos

SUM('jobs'[Custo_Redo])

Calcula o prejuízo financeiro direto causado pelos "redos".

Qtd Redos Causados

COUNT('jobs'[Responsavel_Redo])

Conta o volume total de retrabalhos causados.

Taxa de Redo

DIVIDE([Qtd Redos Causados], COUNTROWS('jobs'))

Mede a eficiência operacional (percentual de visitas que viram redo).

Ranking Redo

RANKX(ALLSELECTED('jobs'[Responsavel_Redo]), CALCULATE([Qtd Redos Causados]), , DESC)

Cria o ranking dinâmico de funcionários, essencial para a matriz mensal.

Lucro Total

[Receita Total] - [Custo Total]

Apresenta a saúde financeira real do negócio, considerando o impacto dos custos.

Nº de Clientes

DISTINCTCOUNT('jobs'[customer_name])

Conta o número de clientes únicos atendidos.

5. Estrutura e Respostas do Dashboard
O dashboard foi dividido em duas páginas para contar uma história coesa.

Página 1: Análise da Causa Raiz
Objetivo: Diagnosticar o problema dos "redos".

Respostas Fornecidas:

Quem causa mais redos? O gráfico de barras "Funcionários com Maior Custo de Redo" aponta diretamente os principais responsáveis.

O problema está melhorando? O gráfico de linhas "Evolução do Custo com Redos" mostra a tendência histórica, permitindo avaliar a eficácia de ações corretivas.

Qual a performance mensal? A Matriz de Ranking Mensal detalha a performance de cada funcionário, mês a mês.

Página 2: Análise de Impacto (Clientes e Finanças)
Objetivo: Analisar as consequências do problema.

Respostas Fornecidas:

Qual o impacto na lucratividade? O gráfico "Lucro Gerado por Tipo de Visita" prova visualmente o prejuízo do "redo". O KPI "Lucro Total" mostra o resultado final.

Quais clientes são mais afetados? O ranking de "Clientes Menos Lucrativos" pode estar correlacionado com clientes que sofrem múltiplos "redos".

Existe concentração geográfica? O Mapa de "Concentração de Custo com Redos" ajuda a identificar se o problema é mais intenso em certas regiões.

6. Consultas SQL (Para Fins de Consulta)
Conforme solicitado, seguem algumas consultas em PostgreSQL que podem ser usadas para exploração e validação dos dados diretamente no banco.

1. Consulta para listar todas as visitas de um serviço específico (ex: '1001'):

SELECT *
FROM public.jobs
WHERE job_number LIKE '1001-%'
ORDER BY job_number;

2. Consulta para contar o número de "redos" por funcionário que realizou a visita:
(Nota: Esta consulta não atribui a culpa ao funcionário anterior como fizemos no Power BI, ela apenas conta quem executou a visita de "redo").

SELECT
    employee_name,
    COUNT(*) AS total_redos_executados
FROM public.jobs
WHERE visit_type = 'redo'
GROUP BY employee_name
ORDER BY total_redos_executados DESC;

3. Consulta para ver o custo total e receita por cliente:

SELECT
    customer_name,
    SUM(visit_cost_dollars) AS custo_total,
    SUM(visit_revenue_dollars) AS receita_total,
    (SUM(visit_revenue_dollars) - SUM(visit_cost_dollars)) AS lucro_total
FROM public.jobs
GROUP BY customer_name
ORDER BY lucro_total DESC;
