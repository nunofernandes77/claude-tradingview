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

## Ouro Intermarket Robot — `gold_intermarket_robot.pine` + `gold_intermarket_panel.pine`

Estratégia (robot) de compra e venda de **ouro (XAUUSD)** que confirma cada sinal com a **prata (XAGUSD)**
e o **US Dollar Index (DXY)**, mais um painel em sub-janela para ver os três mercados ao mesmo tempo.

**Como pôr no gráfico:** abrir `OANDA:XAUUSD` (ou `TVC:GOLD`, `COMEX:GC1!`) → *Pine Editor* → colar
`gold_intermarket_robot.pine` → *Add to chart*. Repetir com `gold_intermarket_panel.pine` (abre por baixo).
Timeframes sugeridos: 1H / 4H (intradiário) ou 1D (swing).

### Score de confluência (-8 a +8)

| # | Componente | Comprador (+1) | Vendedor (-1) |
|---|---|---|---|
| 1 | Tendência do ouro | preço > EMA200 e EMA21 > EMA50 | preço < EMA200 e EMA21 < EMA50 |
| 2 | Momento do ouro | RSI > 50 e MACD hist. > 0 | RSI < 50 e MACD hist. < 0 |
| 3 | Estrutura | Supertrend em alta | Supertrend em baixa |
| 4 | Tendência da prata | prata > EMA50 | prata < EMA50 |
| 5 | Momento da prata | ROC(10) > 0 | ROC(10) < 0 |
| 6 | Tendência do DXY* | DXY < EMA50 (dólar fraco) | DXY > EMA50 |
| 7 | Momento do DXY* | ROC(10) < 0 | ROC(10) > 0 |
| 8 | Rácio Ouro/Prata | abaixo da EMA50 (metais fortes) | acima da EMA50 |

\* O DXY só vota quando a correlação móvel (50 barras, sobre retornos) entre ouro e dólar é negativa (< -0,2);
quando a relação inversa se quebra, o dólar é ignorado.

### Regras do robot

- **Entrada**: score ≥ 6 (compra) ou ≤ -6 (venda), ADX ≥ 20, tendência a favor no timeframe superior (EMA50 diária),
  sessão Londres/NY (07:00–17:00 UTC), fora da janela do Payroll e 3 barras de espera após cada saída.
- **Risco**: 1% do capital por trade; stop inicial 1,5×ATR; alvo 3R; break-even a 1R; depois trailing
  Chandelier 3×ATR; fecha também se o score inverter (≤ -2 numa compra); perda máxima diária de 3%.
- **Sem repaint**: os dados da prata/DXY usam `lookahead_off` e o timeframe superior usa a barra anterior já fechada;
  os sinais são calculados no fecho da barra.

### Automação (webhook)

Criar um alerta na estratégia com a condição **"alert() function calls only"** e ativar *Webhook URL*.
O robot envia JSON como:

```json
{"robot":"gold_intermarket","action":"buy","symbol":"XAUUSD","price":2650.15,"qty":0.66,"sl":2635.10,"tp":2695.30,"score":7,"tf":"60"}
```

`action` pode ser `buy`, `sell`, `move_sl` (novo stop do break-even/trailing) ou `close` (score inverteu).
O servidor/bridge (PineConnector, ponte para MT5, bot próprio) deve abrir a ordem já com SL/TP e atualizar o stop a cada `move_sl`.

### Painel intermercados

- **Preços normalizados**: z-score do ouro, prata e DXY invertido — três linhas juntas a subir = vento a favor do ouro.
- **Score de confluência**: histograma com os limiares de compra/venda.
- **Correlações**: ouro/DXY e ouro/prata móveis.
- Marca **divergências** (ouro faz máximo/mínimo sem confirmação da prata e com o dólar contra) e tem `alertcondition`s.

Ajuste os parâmetros e valide no *Strategy Tester* (e em conta demo) antes de ligar o robot a uma conta real.
Não é recomendação de investimento.

## Pin Bar Scanner — `pin_bar_scanner.pine`

Procura **pin bars** em tempo real em **40 gráficos ao mesmo tempo** (28 pares forex + ouro, prata, platina,
paládio, cobre, WTI, Brent, gás natural, milho, trigo, soja e açúcar) e envia **um alerta por e-mail e
notificação na app** a dizer em que gráfico apareceu, **assim que a vela de 15 minutos fecha**.

**Instalar (uma vez):**
1. Abrir `OANDA:EURUSD` em **15 minutos** → *Pine Editor* → colar `pin_bar_scanner.pine` → *Add to chart*.
2. Criar alerta (relógio ⏰): **Condição** = *PinBar Scanner* → *Any alert() function call*;
   **Expiração** = sem expiração; **Notificações** = ✅ *Notificar na app* + ✅ *Enviar e-mail*.
3. Pode fechar o TradingView: o alerta corre nos servidores e cobre os 40 gráficos.

Exemplo da mensagem recebida:

```
PIN BAR 15m — vela fechada às 14:15
▲ XAUUSD — PIN BAR ALTISTA (compra) 4155.2 | pavio 74%
▼ GBPJPY — PIN BAR BAIXISTA (venda) 198.412 | pavio 66%
```

**Regras (ajustáveis):** pavio ≥ 60% da vela, corpo ≤ 33%, pavio oposto ≤ 20%, vela ≥ 1×ATR(14)
e o pavio tem de fazer o mínimo (altista) ou máximo (baixista) das últimas 5 velas, como uma rejeição de nível.
Os símbolos podem ser trocados nas definições do indicador. O gráfico aberto também mostra as suas pin bars
e um painel com as pin bars recentes de todos os gráficos.
