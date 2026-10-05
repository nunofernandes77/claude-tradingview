# claude-tradingview

## Turtle Trading (Richard Dennis) — `turtle_trading_strategy.pine`

Estratégia de compra e venda em Pine Script v5 baseada nas regras originais dos "Turtles"
(Richard Dennis / William Eckhardt, publicadas por Curtis Faith).

**Como pôr no gráfico:** TradingView → *Pine Editor* → colar o conteúdo do ficheiro → *Save* → *Add to chart*.
Os resultados aparecem no separador *Strategy Tester*.

| Regra | Sistema 1 | Sistema 2 |
|---|---|---|
| Entrada compra/venda | rompe máximo/mínimo de 20 barras | rompe máximo/mínimo de 55 barras |
| Saída | rompimento contrário de 10 barras | rompimento contrário de 20 barras |
| Filtro | ignora o sinal após um trade vencedor (exceto rompimento de 55) | — |

- **N** = ATR(20) · **Unidade** = 1% do capital por 1N · **Stop** = 2N da última entrada
- **Pirâmide**: +1 unidade a cada ½N a favor, máx. 4; o stop acompanha a última entrada

Funciona melhor em timeframe diário e em mercados com tendência (futuros, índices, forex, cripto).
Não é recomendação de investimento; faça backtest e use conta demo antes de arriscar dinheiro real.
