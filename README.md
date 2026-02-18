# Projeto de Tesouraria — Olist Brazilian E-Commerce

Análise e projeção de fluxo de caixa aplicada ao dataset público da Olist, com modelagem de recebíveis, estimativa de saídas operacionais, análise de aging e forecast preditivo.

---

## Propósito

O projeto simula o trabalho de uma área de **tesouraria em um marketplace**, respondendo às principais perguntas do dia a dia financeiro:

- Quando e quanto vou receber de cada meio de pagamento?
- Qual é o meu saldo líquido após repasses e fretes?
- Quanto tenho a receber nos próximos 7, 15 e 30 dias?
- Qual será o inflow dos próximos 30 dias, com intervalo de confiança?

---

## Dataset

**Olist Brazilian E-Commerce** — dataset público disponível no Kaggle com ~100 mil pedidos de e-commerce brasileiro entre 2016 e 2018.

🔗 [Download no Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

Arquivos utilizados:

| Arquivo | Conteúdo |
|---|---|
| `olist_orders_dataset.csv` | Ciclo de vida dos pedidos e datas-chave |
| `olist_order_payments_dataset.csv` | Meios e valores de pagamento |
| `olist_order_items_dataset.csv` | Itens, preços e fretes (Módulo 4) |

---

## Estrutura do Projeto

```
Módulo 1  →  Ingestão e Exploração Inicial
Módulo 2  →  Tratamento de Tipos, Limpeza e Join
Módulo 3  →  Projeção de Recebíveis (Inflow)
Módulo 4  →  Estimativa de Saídas (Outflow)
Módulo 5  →  Consolidação do Vetor de Liquidez Diária
Módulo 6  →  Análise de Aging de Recebíveis
Módulo 7  →  Visualização Analítica da Série Temporal
Módulo 8  →  Modelagem Preditiva (Holt-Winters)
Módulo 9  →  Validação Estatística (Backtesting)
```

---

## Metodologia

### Módulo 3 — Projeção de Recebíveis (Inflow)

As regras de liquidação por meio de pagamento seguem as premissas abaixo. Os valores modelados são **brutos** — em produção, o MDR (Merchant Discount Rate) de cada operadora deve ser descontado.

| Meio de Pagamento | Liquidação | Observação |
|---|---|---|
| Boleto Bancário | D+2 | Compensação na rede bancária |
| Cartão de Débito | D+1 | Liquidação via rede de débito |
| Voucher | D+0 | Compensação imediata |
| Cartão de Crédito | D+30 × parcela | Cada parcela em múltiplos de 30 dias |

O desdobramento de parcelas utiliza `explode()` do pandas para expandir cada transação parcelada em linhas individuais, com reconciliação financeira ao final para garantir que o total projetado bate com o total transacionado.

### Módulo 4 — Estimativa de Saídas (Outflow)

Componentes modelados com base em `olist_order_items_dataset.csv`:

- **Frete**: valor integral de `freight_value`, saída no D+0 da aprovação
- **Repasse ao Seller**: 90% do valor dos itens (take rate de 10% do marketplace)

> Em ambiente produtivo, as saídas devem ser extraídas do ERP/financeiro da empresa. Devoluções, estornos e MDR não estão incluídos neste modelo.

### Módulo 6 — Aging de Recebíveis

Os recebíveis futuros são classificados em faixas de vencimento a partir de uma data de referência (31/07/2018 neste projeto), permitindo visualizar a concentração de liquidez em cada horizonte.

| Faixa | Horizonte |
|---|---|
| 1. Até 7 dias | Caixa imediato |
| 2. 8–15 dias | Curto prazo |
| 3. 16–30 dias | Médio prazo (1 mês) |
| 4. 31–60 dias | Médio prazo (2 meses) |
| 5. > 60 dias | Longo prazo / parcelas futuras |

### Módulo 8 — Forecast (Holt-Winters)

Modelo de **Suavização Exponencial Tripla** com:
- Tendência aditiva — captura crescimento ou queda linear da série
- Sazonalidade aditiva, período 7 — reflete o padrão semanal de compensação bancária

A data de corte simulada é **31/07/2018**, que corresponde ao pico operacional do dataset. O forecast projeta os próximos 30 dias com **intervalo de confiança de 90%** via bootstrap de resíduos, gerando:

- **Cenário central**: previsão pontual do modelo
- **Banda p10**: reserva mínima de capital de giro
- **Banda p90**: necessidade máxima de liquidez

> **Limitação conhecida**: a reindexação diária introduz zeros em dias sem movimentação, o que distorce a componente sazonal. Alternativas mais robustas incluem agregação semanal ou o modelo de Croston para séries intermitentes.

### Módulo 9 — Backtesting Out-of-Sample

Os dados de agosto/2018 foram retidos do treinamento e utilizados exclusivamente para validação. O modelo é avaliado pelas métricas MAE, RMSE e MAPE, e comparado com um **modelo Naive** (replica o dia anterior) como benchmark mínimo de referência.

---

## Dependências

```
pandas
numpy
matplotlib
seaborn
statsmodels
scikit-learn
```

---

## Autor

**Alexandre Souza** 
