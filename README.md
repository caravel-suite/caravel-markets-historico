# Caravel Markets — histórico público de previsões (demonstração)

Cada linha de `historico/AAAA-MM.jsonl` é uma previsão do robô em demonstração da Caravel Markets,
acrescentada logo depois de ser feita e **antes** de o resultado ser conhecido. O ficheiro só cresce:
linhas antigas nunca são alteradas (o histórico do git mostra isso).

Campos: `hash`, `hora_utc`, `simbolo`, `direcao` (up = subida, down = descida, flat = estável, limiar ±0,5 %),
`modelo` (modelo que gerou a previsão), `horizonte_h`, `preco_referencia` (preço no momento da previsão),
`preco_previsto`, `dados_hash` (os dados exatos que entraram no hash).

## Como verificar um hash

```python
import hashlib, json
linha = json.loads(open("historico/2026-11.jsonl").readline())
calc = hashlib.sha256(json.dumps(linha["dados_hash"], sort_keys=True).encode()).hexdigest()[:16]
assert calc == linha["hash"]
```

Desde 07/11/2026 (~23 h UTC) há também `mercado` (`cripto` ou `bolsa`) e `oficial` (true = previsão do robô
mostrada no site, feita pela Caravel AI; false = previsão de um modelo isolado, usada na classificação dos modelos).
Na bolsa, `horizonte_h` = 1 quer dizer **1 sessão** (`dados_hash.unidade` = «sessao», `dados_hash.ref_data` = data da
sessão do preço de referência). Nas previsões da Caravel AI, `dados_hash.pesos` são os pesos de cada modelo.

## Modelos
- `caravel-ai`: conjunto que combina as previsões dos modelos abaixo, com pesos pelo acerto direcional recente de
  cada um (só resultados já conhecidos no momento da previsão).
- `chronos-bolt-small`: Chronos-Bolt small (amazon/chronos-bolt-small, Apache-2.0), CPU.
- `granite-ttm-r2`: Granite TimeSeries TTM r2.1 (ibm-granite/granite-timeseries-ttm-r2, Apache-2.0), CPU.
- `kronos-mini`: modelo aberto Kronos-mini (NeoQuasar/Kronos-mini, licença MIT), CPU.
- `combinada`: 60 % cruzamento de médias móveis (12/48 h) + 40 % momentum (24 h), sem aleatoriedade.
  Usada quando o Kronos não está disponível. Nas previsões anteriores a 7/11/2026 ~21:00 UTC o campo
  `dados_hash.strategy` diz «kronos» (nome interno do robô), mas o modelo usado foi sempre a `combinada`;
  o campo `modelo` é o que conta.

Demonstração com dinheiro fictício. Não é aconselhamento financeiro nem recomendação personalizada de investimento.
