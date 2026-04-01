# Otimização de Programa de Cashback

Análise end-to-end de um programa de cashback usando Python, DuckDB e Power BI.
A partir de dados reais de e-commerce, o projeto simula três janelas de validade
(30, 45 e 60 dias), segmenta clientes por comportamento de compra via matriz R×F
e quantifica o impacto incremental de cada extensão, entregando uma recomendação
de política por segmento sustentada por dados.

---

## Problema de negócio

Programas de cashback geram bônus a cada compra, mas a eficácia depende criticamente
do prazo de validade: muito curto, e o bônus expira antes do cliente voltar; muito longo,
e o custo do programa cresce sem contrapartida em retenção.

A análise simula três cenários de validade (30, 45 e 60 dias) sobre dados reais de
transações, segmenta os clientes por comportamento de compra e quantifica o impacto
incremental de cada extensão, separando os segmentos que respondem à mudança dos que
não respondem.

---

## Metodologia

Adaptação do framework CRISP-DM em 7 etapas:

1. **Coleta e entendimento dos dados** - inspeção do dataset bruto, identificação de anomalias e definição do escopo analítico
2. **Limpeza** - remoção de registros sem Customer ID, filtragem para UK, isolamento de devoluções, criação de `TotalPrice`
3. **Feature Engineering** - duas camadas:
   - **SQL/DuckDB**: agregação item → pedido, numeração sequencial por cliente, contexto temporal via `LAG` e `DATEDIFF`
   - **Python/Pandas**: simulação stateful de cashback com saldo carregado entre iterações, classificação em 5 status, parametrização por janela de validade
4. **EDA** - três eixos de análise com segmentação RFM como variável de corte:
   - Distribuição de intervalos entre compras vs. janela de validade
   - Impacto financeiro da expiração por segmento
   - Comparação de aproveitamento entre os três cenários
5. **Segmentação RFM** - matriz R×F com buffer temporal de 60 dias para evitar viés de clientes novos; três segmentos: Recorrentes, Ocasionais e Dormentes
6. **Simulação de cenários** - tabela de trade-off, cálculo de impacto incremental e eficiência marginal por salto de janela
7. **Dashboard Power BI** - três páginas com progressão narrativa: Visão Geral → Diagnóstico → Recomendações

---

## Principais findings

**A expiração é o status dominante em todos os segmentos com a janela de 30 dias.**

A causa, no entanto, é diferente por grupo:

| Segmento | Causa da expiração | Responde à extensão? | Recomendação |
|---|---|---|---|
| Recorrentes | Intervalo natural concentrado entre 30–45 dias | ✓ Sim - eficiência crescente nos dois saltos | Estender para 60d se houver orçamento; 45d como mínimo |
| Ocasionais | Frequência irregular, mas intervalo aceitável | ✓ Sim - melhor custo-benefício do programa | Estender para 45d; 60d tem retorno decrescente |
| Dormentes | Intervalo estruturalmente longo, além de qualquer janela | ✗ Não - cashback ineficaz independente do prazo | Intervenção de reativação, não extensão de validade |

**Eficiência marginal por salto:**

| Segmento | 30d → 45d | 45d → 60d |
|---|---|---|
| Recorrentes | 47.72 | 57.63 ↑ |
| Ocasionais | 23.33 | 30.84 ↑ |
| Dormentes | 17.65 | 22.28 |

*Eficiência = pp de expiração eliminados por cada 1% de custo adicional*

Recorrentes e Ocasionais apresentam eficiência **crescente** entre os dois saltos,
comportamento incomum que indica que o intervalo natural desses clientes está concentrado
justamente na faixa 45–60 dias, tornando a segunda extensão ainda mais eficiente que a primeira.

---

## Estrutura do repositório
```
cashback-optimization/
│
├── README.md
│
├── data/
│   ├── raw/
│   │   └── .gitkeep
│   └── processed/
│       ├── cenarios_completo.csv
│       ├── cenarios.csv
│       ├── df_pedidos.csv
│       ├── deltas.csv
│       ├── df_risco.csv
│       └── rfm.csv
│
├── notebooks/
│   ├── cashback_analysis.ipynb
│   └── project_decomposition.md
│
├── dashboard/
│   └── cashback_dashboard.pbix
│
└── presentation/
    ├── cashback_presentation.pptx
    └── cashback_presentation.pdf
```

---

## Como reproduzir

**1. Clonar o repositório**
```bash
git clone https://github.com/wspdata/cashback-optimization
cd cashback-optimization
```

**2. Instalar dependências**
```bash
pip install pandas numpy duckdb plotly
```

**3. Baixar o dataset**

O dataset bruto não está versionado por tamanho. Baixar o arquivo
`online_retail_II.csv` no [Kaggle](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci)
e salvar em `data/raw/`.

**4. Executar o notebook**

Abrir `notebooks/cashback_analysis.ipynb` e executar todas as células em ordem.
As tabelas processadas serão geradas automaticamente em `data/processed/`.

**5. Abrir o dashboard**

Abrir `dashboard/cashback_dashboard.pbix` no Power BI Desktop e atualizar
a fonte de dados apontando para `data/processed/`.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Extração e agregação | SQL / DuckDB (CTEs, window functions) |
| Transformação e simulação | Python - Pandas, Numpy |
| Visualização exploratória | Plotly |
| Dashboard | Power BI (DAX, slicer desconectado, coluna calculada) |
| Ambiente | Jupyter Notebook |

---

## Dataset

**Online Retail II** - UCI Machine Learning Repository  
Transações de e-commerce do Reino Unido entre 2009 e 2011.  
Disponível em: https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci

Após limpeza e filtro para UK: **700.388 transações → 33.541 pedidos → 4.315 clientes identificados**

---

## Contato

Willian De Souza Pereira — ws13292@gmail.com

LinkedIn: https://linkedin.com/in/willian-de-souza-pereira-b69109202

## Licença

Este repositório está disponível para estudo e demonstração. Sinta-se à vontade para clonar, adaptar e abrir *issues* com dúvidas ou sugestões.
