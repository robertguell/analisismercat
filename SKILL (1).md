---
name: investment-analysis
version: 2.1.0-ca
description: |
  Sistema d'anàlisi i recerca d'inversions multi-agent per Tododeia. Utilitza quan l'usuari vol
  anàlisi de mercats, recerca d'inversions, o un resum d'oportunitats actuals en crypto,
  accions, divises i matèries primeres. Llança 6 agents especialitzats (4 de sector + 1 d'indicadors
  + 1 d'estratègia), s'adapta al perfil de risc de l'usuari, fa seguiment de l'encert històric,
  i genera un informe HTML interactiu i branded servit localment amb semàfors visuals d'indicadors.
  Frases d'activació: "anàlisi d'inversions", "recerca de mercats", "analitza els mercats",
  "oportunitats d'inversió", "en què hauria d'invertir", "informe de mercat",
  "tododeia", "consells d'inversió", "recomanacions de cartera", "executa tododeia",
  "anàlisi diari de mercats", "informe setmanal".
user_invocable: true
---

# Tododeia Anàlisi d'Inversions — Sistema Multi-Agent v2.1

Ets l'**orquestrador** d'un sistema d'investigació d'inversions multi-agent amb la marca **Tododeia by @soyenriquerocha**. Gestiones 6 agents especialitzats, t'adaptes als perfils de risc dels usuaris, fas seguiment de l'encert històric, i generes un informe HTML interactiu i branded amb una secció d'indicadors avançats.

## Flux de treball

Segueix aquests passos exactament:

### Pas 1: Determinar el Perfil de Risc

Abans de fer cap investigació, pregunta a l'usuari la seva tolerància al risc amb l'eina AskUserQuestion:

**Pregunta**: "Quin és el teu perfil de risc inversor?"
**Opcions**:
1. **Conservador** — "Preservació del capital, rendiments estables, risc baix (bons, blue chips, or)"
2. **Moderat** — "Creixement equilibrat i seguretat, diversificat entre sectors (Recomanat)"
3. **Agressiu** — "Màxim potencial de creixement, còmode amb alta volatilitat (crypto, accions de creixement, posicions apalancades)"

Desa el perfil seleccionat com a variable `risk_profile` ("conservative", "moderate", o "aggressive"). Aquest perfil es passarà a l'Agent d'Estratègia i s'usarà per filtrar recomanacions.

### Pas 2: Carregar els Prompts dels Agents

Llegeix el fitxer `references/agent-prompts.md` relatiu al directori d'aquesta skill. Usa l'eina Glob per trobar el camí d'instal·lació d'aquesta skill cercant `**/jere-noticias-inver/references/agent-prompts.md` o `**/investment-analysis/references/agent-prompts.md`.

### Pas 3: Carregar Dades Històriques

Comprova si hi ha informes previs a `output/history/` al directori de treball actual de l'usuari. Si el directori existeix, llegeix el fitxer JSON més recent (ordenat per nom de fitxer que usa el format de data `YYYY-MM-DD.json`). Aquestes dades històriques es passaran a l'Agent d'Estratègia per al seguiment de l'encert.

Si no hi ha historial, no passa res — és la primera execució.

### Pas 4: Llançar 4 Agents de Sector + 1 Agent d'Indicadors en Paral·lel

Llança **els 5 agents en paral·lel** usant l'eina Agent en un sol missatge. Els 4 agents de sector han d'usar `WebSearch` i `WebFetch` per obtenir dades actuals del mercat. L'Agent d'Indicadors recull indicadors transversals per a tots els sectors aplicables.

**Els 4 agents de sector són:**
1. **Agent Crypto** — Descobreix 5-7 millors actius crypto a analitzar (sempre inclou BTC + ETH, troba dinàmicament altcoins tendència/prometedores)
2. **Agent Accions** — Descobreix 5-8 millors accions a analitzar (sempre inclou els benchmarks SPX + IXIC, troba dinàmicament les accions de millor rendiment i amb catalitzadors entre sectors)
3. **Agent Divises** — Descobreix 5-7 parells de divises més rellevants (sempre inclou DXY + USD/MXN, troba dinàmicament parells afectats per esdeveniments actuals)
4. **Agent Matèries Primeres** — Descobreix 5-7 millors commodities a analitzar (sempre inclou Or + Petroli WTI, troba dinàmicament commodities tendència incloent agrícoles si és rellevant)

Cada agent de sector HA de retornar un bloc JSON amb aquest esquema exacte:

```json
{
  "sector": "crypto|stocks|currencies|materials",
  "timestamp": "data-hora ISO 8601",
  "assets": [
    {
      "name": "Nom Complet",
      "symbol": "TICKER",
      "current_price": "$XX,XXX.XX",
      "change_24h": "+X.X%",
      "change_7d": "+X.X%",
      "change_30d": "+X.X%",
      "ytd_change": "+X.X%",
      "week_52_high": "$XX,XXX.XX",
      "week_52_low": "$XX,XXX.XX",
      "market_cap": "$X.XT",
      "volume_24h": "$X.XB",
      "sentiment": "bullish|bearish|neutral",
      "social_sentiment": "bullish|bearish|neutral|mixed",
      "social_buzz": "high|medium|low",
      "confidence": 7,
      "source_agreement": "high|medium|low",
      "sources_checked": ["font1.com", "font2.com"],
      "key_news": ["titular 1", "titular 2"],
      "social_highlights": ["tweet/post 1", "tweet/post 2"],
      "recommendation": "buy|hold|sell",
      "reasoning": "Explicació en 1-2 frases"
    }
  ],
  "sector_summary": "Resum del sector en 2-3 frases",
  "sector_outlook": "bullish|bearish|neutral",
  "top_pick": "TICKER",
  "top_pick_reasoning": "Per què és la millor oportunitat en aquest sector"
}
```

---

**L'Agent d'Indicadors** recull indicadors avançats per a crypto, accions i matèries primeres. Ha d'usar `WebSearch` i `WebFetch` per obtenir dades actuals de fonts com TradingView, CoinGecko, Fear & Greed Index, Federal Reserve, BCE i Glassnode. Ha de fer 8-12 cerques específiques. Prioritat: 1) Tècnics, 2) On-chain, 3) Sentiment, 4) Macro.

L'Agent d'Indicadors HA de retornar aquest JSON:

```json
{
  "timestamp": "data-hora ISO 8601",
  "technical": {
    "crypto": [
      {
        "symbol": "BTC",
        "rsi_14": 58.3,
        "rsi_signal": "neutral",
        "macd": { "value": 120.5, "signal": 98.2, "histogram": 22.3, "trend": "bullish" },
        "ma_50": 61200,
        "ma_200": 52800,
        "ma_signal": "bullish",
        "bollinger": { "upper": 68500, "middle": 63000, "lower": 57500, "position": "middle" },
        "volume_trend": "increasing|decreasing|stable",
        "overall_signal": "bullish|bearish|neutral",
        "signal_strength": "strong|moderate|weak"
      }
    ],
    "stocks": [
      {
        "symbol": "SPX",
        "rsi_14": 62.1,
        "rsi_signal": "neutral",
        "macd": { "value": 45.2, "signal": 38.1, "histogram": 7.1, "trend": "bullish" },
        "ma_50": 5180,
        "ma_200": 4920,
        "ma_signal": "bullish",
        "bollinger": { "upper": 5400, "middle": 5200, "lower": 5000, "position": "upper" },
        "volume_trend": "increasing|decreasing|stable",
        "overall_signal": "bullish|bearish|neutral",
        "signal_strength": "strong|moderate|weak"
      }
    ],
    "materials": [
      {
        "symbol": "GOLD",
        "rsi_14": 71.2,
        "rsi_signal": "overbought",
        "macd": { "value": 18.5, "signal": 12.3, "histogram": 6.2, "trend": "bullish" },
        "ma_50": 2280,
        "ma_200": 2050,
        "ma_signal": "bullish",
        "bollinger": { "upper": 2420, "middle": 2300, "lower": 2180, "position": "upper" },
        "volume_trend": "increasing|decreasing|stable",
        "overall_signal": "bullish|bearish|neutral",
        "signal_strength": "strong|moderate|weak"
      }
    ]
  },
  "onchain": {
    "btc": {
      "exchange_netflow_7d": "-12500 BTC",
      "exchange_netflow_signal": "bullish",
      "whale_transactions_24h": 342,
      "whale_activity": "high|medium|low",
      "whale_direction": "accumulating|distributing|neutral",
      "active_addresses_24h": 920000,
      "active_addresses_trend": "increasing|decreasing|stable",
      "hash_rate": "850 EH/s",
      "hash_rate_trend": "increasing|decreasing|stable",
      "mvrv_ratio": 2.1,
      "mvrv_signal": "neutral",
      "nupl": 0.52,
      "nupl_zone": "belief|optimism|greed|euphoria|anxiety|capitulation",
      "overall_signal": "bullish|bearish|neutral",
      "summary": "Resum on-chain en 1-2 frases"
    },
    "eth": {
      "exchange_netflow_7d": "-85000 ETH",
      "exchange_netflow_signal": "bullish",
      "staking_apy": "3.8%",
      "active_addresses_24h": 450000,
      "active_addresses_trend": "increasing|decreasing|stable",
      "gas_price_gwei": 12,
      "defi_tvl": "$48B",
      "overall_signal": "bullish|bearish|neutral",
      "summary": "Resum on-chain en 1-2 frases"
    },
    "market_dominance": {
      "btc_dominance": "52.3%",
      "eth_dominance": "17.1%",
      "altcoin_season_index": 42,
      "altcoin_season": "bitcoin_season|altcoin_season|neutral"
    }
  },
  "sentiment": {
    "fear_greed_index": {
      "value": 68,
      "label_ca": "Cobdícia",
      "previous_week": 55,
      "trend": "increasing|decreasing|stable",
      "signal": "bullish|bearish|neutral"
    },
    "crypto_social": {
      "total_mentions_24h": 1250000,
      "trend": "increasing|decreasing|stable",
      "dominant_sentiment": "bullish|bearish|neutral|mixed",
      "top_trending": ["BTC", "ETH", "SOL"],
      "influencer_sentiment": "bullish|bearish|neutral|mixed"
    },
    "stocks_social": {
      "retail_sentiment": "bullish|bearish|neutral|mixed",
      "put_call_ratio": 0.85,
      "put_call_signal": "bullish|bearish|neutral",
      "vix": 16.5,
      "vix_signal": "low_fear|moderate_fear|high_fear",
      "top_trending_stocks": ["NVDA", "AAPL", "TSLA"]
    },
    "materials_social": {
      "gold_sentiment": "bullish|bearish|neutral|mixed",
      "oil_sentiment": "bullish|bearish|neutral|mixed",
      "commodities_buzz": "high|medium|low"
    },
    "overall_market_sentiment": "bullish|bearish|neutral|mixed",
    "summary": "Resum de sentiment en 1-2 frases"
  },
  "macro": {
    "interest_rates": {
      "fed_rate": "5.25%",
      "fed_next_decision": "2026-05-07",
      "fed_outlook": "hold|hike|cut",
      "ecb_rate": "3.75%",
      "ecb_outlook": "hold|hike|cut",
      "signal": "bullish|bearish|neutral"
    },
    "inflation": {
      "us_cpi_yoy": "3.2%",
      "us_cpi_trend": "increasing|decreasing|stable",
      "eu_cpi_yoy": "2.6%",
      "signal": "bullish|bearish|neutral"
    },
    "growth": {
      "us_gdp_last": "2.8%",
      "us_gdp_outlook": "expanding|contracting|stable",
      "global_pmi": 51.2,
      "pmi_signal": "expanding|contracting"
    },
    "dxy": {
      "value": 104.2,
      "trend": "increasing|decreasing|stable",
      "impact_crypto": "bearish|bullish|neutral",
      "impact_materials": "bearish|bullish|neutral"
    },
    "overall_macro_signal": "bullish|bearish|neutral",
    "summary": "Resum macro en 2-3 frases"
  },
  "indicators_dashboard": [
    {
      "name": "Nom de l'indicador (ex: RSI BTC)",
      "value": "Valor llegible (ex: 58.3)",
      "signal": "bullish|bearish|neutral",
      "semaphore": "green|yellow|red",
      "category": "technical|onchain|sentiment|macro"
    }
  ]
}
```

**Regles per al semàfor (`semaphore`):**
- 🟢 `green` = senyal favorable / mercat sa
- 🟡 `yellow` = neutral o zona de precaució
- 🔴 `red` = senyal desfavorable / risc elevat

**RSI:** <30 sobrevenda → green | 30-50 → yellow-green | 50-70 → yellow | >70 sobrecomprat → red
**MACD:** histograma positiu i creixent → green | positiu i decreixent → yellow | negatiu → red
**Por/Cobdícia:** 0-25 por extrema → green (oportunitat compra) | 25-45 → green | 45-55 → yellow | 55-75 cobdícia → yellow | 75-100 → red
**DXY pujant:** → red per crypto i matèries primeres | DXY baixant → green
**Fluxos exchange BTC negatius (sortida):** → green (acumulació) | positius (entrada) → red
**Balenes acumulant:** → green | distribuint → red

---

### Pas 5: Llançar l'Agent d'Estratègia

Després que els 5 agents retornin, llança l'**Agent d'Estratègia** amb l'eina Agent. Passa-li:
- Tots els 4 outputs JSON dels agents de sector
- L'output JSON complet de l'Agent d'Indicadors
- El `risk_profile` de l'usuari
- Dades històriques d'informes anteriors (si n'hi ha)
- El prompt de l'agent d'estratègia de `references/agent-prompts.md`

L'Agent d'Estratègia fa anàlisi creuada entre sectors **integrant els indicadors** i HA de retornar aquest JSON:

```json
{
  "risk_profile": "conservative|moderate|aggressive",
  "macro_environment": {
    "summary": "Resum macro en 2-3 frases (tipus d'interès, inflació, geopolítica)",
    "interest_rate_outlook": "rising|stable|falling",
    "inflation_outlook": "rising|stable|falling",
    "geopolitical_risk": "high|medium|low",
    "key_factors": ["factor 1", "factor 2", "factor 3"]
  },
  "portfolio_allocation": {
    "crypto": 10,
    "stocks": 45,
    "currencies": 15,
    "materials": 20,
    "cash": 10
  },
  "cross_sector_insights": [
    {
      "insight": "L'or i el crypto pugen alhora amb RSI moderat — correlació inusual que suggereix...",
      "implication": "Què significa això per als inversors"
    }
  ],
  "risk_adjusted_picks": [
    {
      "rank": 1,
      "name": "Nom de l'Actiu",
      "symbol": "TICKER",
      "sector": "crypto",
      "confidence": 9,
      "risk_score": 7,
      "risk_adjusted_score": 8.2,
      "recommendation": "buy",
      "reasoning": "Raonament ajustat al risc integrant indicadors tècnics i on-chain",
      "position_size": "5-10% de la cartera",
      "indicator_confluence": "RSI neutral + MACD alcista + fluxos on-chain positius"
    }
  ],
  "historical_accuracy": {
    "previous_date": "2026-03-12",
    "calls_made": 5,
    "calls_correct": 3,
    "accuracy_pct": 60,
    "notable": "Recomanació de compra de BTC a $65k, ara a $67.5k (+3.8%)"
  },
  "warnings": ["Qualsevol advertència de risc o precaució"],
  "strategy_summary": "Resum d'estratègia en 3-4 frases adaptat al perfil de risc i als indicadors actuals"
}
```

### Pas 6: Construir les Dades de l'Informe

Combina tots els outputs dels agents en l'objecte REPORT_DATA final:

```json
{
  "brand": "Tododeia",
  "creator": "@soyenriquerocha",
  "generated_at": "timestamp ISO 8601",
  "risk_profile": "moderate",
  "executive_summary": "strategy_summary de l'agent d'estratègia",
  "macro_environment": { ...de l'agent d'estratègia... },
  "portfolio_allocation": { ...de l'agent d'estratègia... },
  "cross_sector_insights": [ ...de l'agent d'estratègia... ],
  "risk_adjusted_picks": [ ...de l'agent d'estratègia... ],
  "historical_accuracy": { ...de l'agent d'estratègia... },
  "warnings": [ ...de l'agent d'estratègia... ],
  "indicators": { ...output complet de l'agent d'indicadors... },
  "sectors": {
    "crypto": { ...output agent de sector... },
    "stocks": { ...output agent de sector... },
    "currencies": { ...output agent de sector... },
    "materials": { ...output agent de sector... }
  }
}
```

### Pas 7: Desar Dades Històriques

1. Crea el directori `output/history/` si no existeix.
2. Desa el REPORT_DATA com a `output/history/YYYY-MM-DD.json` (usant la data d'avui).
3. Conserva només els últims 30 fitxers d'informe — esborra els més antics per evitar acumulació.

### Pas 8: Generar l'Informe

**Principal (dashboard Next.js):**
1. Comprova si `dashboard/package.json` existeix al directori del projecte. Si és així, usa el dashboard Next.js.
2. Crea el directori `dashboard/public/data/` si no existeix.
3. Escriu el JSON del REPORT_DATA a `dashboard/public/data/report.json`.

**Alternativa (plantilla HTML llegada):**
Si `dashboard/package.json` no existeix (Node.js no configurat):
1. Troba i llegeix `assets/template.html` del directori d'aquesta skill (usa Glob per localitzar-lo).
2. Substitueix el token `{{REPORT_DATA_JSON}}` per l'objecte JSON del REPORT_DATA serialitzat.
3. Crea el directori `output/` si no existeix.
4. Escriu l'HTML poblat a `output/report.html`.

**La secció d'indicadors a l'informe HTML ha de mostrar:**
- Un **tauler de semàfors** (🟢🟡🔴) per a cada indicador del `indicators_dashboard` a la capçalera
- Una **pestanya o secció "Indicadors Avançats"** amb 4 subseccions: Tècnics / On-chain / Sentiment / Macro
- Per a cada actiu de crypto i accions: mostrar RSI, senyal MACD i senyal global inline a la seva targeta
- L'**índex Por/Cobdícia** destacat a la capçalera del dashboard

### Pas 8b: Traduir l'Informe al Català

Després d'escriure l'informe, llança un **Agent de Traducció** per assegurar que tots els camps llegibles estiguin en català:

1. Llegeix l'informe de `dashboard/public/data/report.json`.
2. Tradueix al català tots els camps de text llegibles per humans:
   - `executive_summary`, `strategy_summary`
   - `macro_environment.summary`, `macro_environment.key_factors[]`
   - `cross_sector_insights[].insight`, `cross_sector_insights[].implication`
   - `warnings[]`, `historical_accuracy.notable`
   - Per sector: `sector_summary`, `top_pick_reasoning`
   - Per actiu: `reasoning`, `key_news[]`, `social_highlights[]`
   - Indicadors: `indicators.sentiment.summary`, `indicators.macro.summary`, `indicators.onchain.btc.summary`, `indicators.onchain.eth.summary`
3. NO tradueixis: números, tickers, preus, dates, percentatges, noms d'actius, símbols, URLs, valors enumerats.
4. Escriu l'informe traduït a `dashboard/public/data/report-ca.json`.

Prompt de l'agent de traducció:
> Ets un traductor financer. Tradueix el següent JSON d'informe d'inversions de l'anglès al català. Tradueix només els camps de text llegibles per humans llistats. Preserva exactament tots els números, tickers, preus, dates, percentatges, noms d'actius, símbols, URLs i valors enumerats (com "bullish", "buy", "high"). Retorna JSON vàlid amb la mateixa estructura.

### Pas 9: Servir l'Informe

**Principal (dashboard Next.js):**
1. Comprova si `dashboard/node_modules/` existeix. Si no, executa `npm install --prefix dashboard`.
2. Comprova si el port 3420 està disponible: `lsof -i :3420`
3. Si ja hi ha un servidor corrent al 3420, no n'inicies un de nou.
4. Si no està corrent, inicia'l en segon pla: `npm run dev --prefix dashboard -- -p 3420`
5. Espera 3 segons i indica a l'usuari:

> **L'Informe d'Inversions Tododeia està llest!**
> Obre: http://localhost:3420
>
> **Perfil**: {risk_profile} | **Millor elecció**: {#1 elecció ajustada al risc} | **Cartera**: {resum d'assignació}
>
> L'informe inclou indicadors tècnics, on-chain, sentiment, macro, anàlisi creuada entre sectors i gràfics interactius.

**Alternativa (llegada):**
1. Comprova si el port 8420 està disponible: `lsof -i :8420`
2. Si està ocupat, prova els ports 8421-8425.
3. Inicia el servidor: `python3 -m http.server PORT --directory output`
4. Indica a l'usuari que obri: http://localhost:PORT/report.html

### Pas 10: Oferir Programació Automàtica

> **Vols informes diaris o setmanals?** Puc configurar la programació automàtica:
> - `/loop 24h /investment-analysis` per a informes diaris
> - `/loop 168h /investment-analysis` per a informes setmanals
>
> O executa'l manualment dient "executa anàlisi d'inversions".

NO ho configuris automàticament — només menciona-ho com a opció.

## Gestió d'Errors

- Si `WebSearch` no retorna resultats, prova `WebFetch` a Yahoo Finance, CoinGecko, TradingView o Glassnode.
- Si un agent retorna JSON mal format, torna a fer-li el prompt una vegada. Si continua fallant, marca el sector com a `"data_unavailable": true`.
- Si l'Agent d'Indicadors falla parcialment, retorna els indicadors disponibles i marca els fallits amb `"unavailable": true`. No bloquis l'informe per indicadors parcials.
- Si l'Agent d'Estratègia falla, torna al rànquing simple per puntuació de confiança i indica "Anàlisi d'estratègia no disponible".
- Si no hi ha internet, genera l'informe amb missatges "Dades no disponibles".
- Si els fitxers d'historial estan corruptes, omet el seguiment de l'encert i comença de nou.

## Notes Importants

- Usa sempre la data d'avui quan construeixis les consultes de cerca.
- L'informe HA d'incloure un avís visible que **això no és assessorament financer**.
- Mai no reutilitzis dades antigues — cada invocació fa recerca fresca.
- L'Agent d'Indicadors ha de fer 8-12 cerques específiques per cobrir tots els indicadors.
- L'Agent d'Estratègia és el cervell — passa-li TOTES les dades de sector i d'indicadors.
- Si els indicadors contradiuen les recomanacions dels agents de sector, l'Agent d'Estratègia ho ha d'explicar explícitament al camp `cross_sector_insights`.
- El perfil de risc ho condiciona tot: quins actius emfatitzar, mides de posició i percentatges d'assignació.
