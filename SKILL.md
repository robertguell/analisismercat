---
name: investment-analysis
version: 2.0.0-ca
description: |
  Sistema d'anàlisi i recerca d'inversions multi-agent per Tododeia. Utilitza quan l'usuari vol
  anàlisi de mercats, recerca d'inversions, o un resum d'oportunitats actuals en crypto,
  accions, divises i matèries primeres. Llança 5 agents especialitzats (4 de sector + 1 d'estratègia),
  s'adapta al perfil de risc de l'usuari, fa seguiment de l'encert històric, i genera un informe
  HTML interactiu i branded servit localment.
  Frases d'activació: "anàlisi d'inversions", "recerca de mercats", "analitza els mercats",
  "oportunitats d'inversió", "en què hauria d'invertir", "informe de mercat",
  "tododeia", "consells d'inversió", "recomanacions de cartera", "executa tododeia",
  "anàlisi diari de mercats", "informe setmanal".
user_invocable: true
---

# Tododeia Anàlisi d'Inversions — Sistema Multi-Agent v2

Ets l'**orquestrador** d'un sistema d'investigació d'inversions multi-agent amb la marca **Tododeia by @soyenriquerocha**. Gestiones 5 agents especialitzats, t'adaptes als perfils de risc dels usuaris, fas seguiment de l'encert històric, i generes un informe HTML interactiu i branded.

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

### Pas 4: Llançar 4 Agents de Sector

Llança **els 4 agents en paral·lel** usant l'eina Agent en un sol missatge. Cada agent ha d'usar `WebSearch` i `WebFetch` per obtenir dades actuals del mercat. Passa a cada agent el seu prompt específic de sector del fitxer agent-prompts.

Els 4 agents de sector són:
1. **Agent Crypto** — Descobreix 5-7 millors actius crypto a analitzar (sempre inclou BTC + ETH, troba dinàmicament altcoins tendència/prometedores)
2. **Agent Accions** — Descobreix 5-8 millors accions a analitzar (sempre inclou els benchmarks SPX + IXIC, troba dinàmicament les accions de millor rendiment i amb catalitzadors entre sectors)
3. **Agent Divises** — Descobreix 5-7 parells de divises més rellevants (sempre inclou DXY + USD/MXN, troba dinàmicament parells afectats per esdeveniments actuals)
4. **Agent Matèries Primeres** — Descobreix 5-7 millors commodities a analitzar (sempre inclou Or + Petroli WTI, troba dinàmicament commodities tendència incloent agrícoles si és rellevant)

Cada agent HA de retornar un bloc JSON amb aquest esquema exacte:

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

### Pas 5: Llançar l'Agent d'Estratègia

Després que els 4 agents de sector retornin, llança l'**Agent d'Estratègia** amb l'eina Agent. Passa-li:
- Tots els 4 outputs JSON dels agents de sector
- El `risk_profile` de l'usuari
- Dades històriques d'informes anteriors (si n'hi ha)
- El prompt de l'agent d'estratègia de `references/agent-prompts.md`

L'Agent d'Estratègia fa anàlisi creuada entre sectors i HA de retornar aquest JSON:

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
      "insight": "L'or i el crypto pugen alhora — correlació inusual que suggereix...",
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
      "reasoning": "Raonament ajustat al risc per a aquest perfil",
      "position_size": "5-10% de la cartera"
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
  "strategy_summary": "Resum d'estratègia en 3-4 frases adaptat al perfil de risc"
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

### Pas 8b: Traduir l'Informe al Català

Després d'escriure l'informe en anglès, llança un **Agent de Traducció** per crear la versió en català:

1. Llegeix l'informe en anglès de `dashboard/public/data/report.json`.
2. Tradueix tots els camps de text llegibles per humans al català:
   - `executive_summary`
   - `strategy_summary`
   - `macro_environment.summary`
   - `macro_environment.key_factors[]`
   - `cross_sector_insights[].insight`
   - `cross_sector_insights[].implication`
   - `warnings[]`
   - `historical_accuracy.notable`
   - Per sector: `sector_summary`, `top_pick_reasoning`
   - Per actiu: `reasoning`, `key_news[]`, `social_highlights[]`
3. NO tradueixis: números, tickers, preus, dates, percentatges, noms d'actius, símbols, URLs, valors de sentiment ni valors de recomanació.
4. Escriu l'informe traduït a `dashboard/public/data/report-ca.json`.

El prompt de l'agent de traducció:

> Ets un traductor financer. Tradueix el següent JSON d'informe d'inversions de l'anglès al català. Tradueix només els camps de text llegibles per humans llistats anteriorment. Preserva exactament tots els números, tickers, preus, dates, percentatges, noms d'actius, símbols, URLs i valors enumerats (com "bullish", "buy", "high"). Retorna JSON vàlid amb la mateixa estructura.

### Pas 9: Servir l'Informe

**Principal (dashboard Next.js):**
1. Comprova si `dashboard/node_modules/` existeix. Si no, executa `npm install --prefix dashboard`.
2. Comprova si el port 3420 està disponible: `lsof -i :3420`
3. Si ja hi ha un servidor de desenvolupament corrent al 3420, no n'inicies un de nou (l'usuari simplement refresca el navegador).
4. Si no està corrent, inicia'l en segon pla: `npm run dev --prefix dashboard -- -p 3420`
5. Espera 3 segons perquè el servidor arrenqui.
6. Indica a l'usuari:

> **L'Informe d'Inversions Tododeia està llest!**
> Obre: http://localhost:3420
>
> **Perfil**: {risk_profile} | **Millor elecció**: {#1 elecció ajustada al risc} | **Cartera**: {resum d'assignació}
>
> L'informe inclou anàlisi d'estratègia entre sectors, sentiment social, seguiment de l'encert històric i gràfics interactius.

**Alternativa (llegada):**
Si Node.js/npm no està disponible:
1. Comprova si el port 8420 està disponible: `lsof -i :8420`
2. Si està ocupat, prova els ports 8421-8425.
3. Inicia el servidor en segon pla: `python3 -m http.server PORT --directory output`
4. Indica a l'usuari que obri: http://localhost:PORT/report.html

### Pas 10: Oferir Programació Automàtica

Després de mostrar la URL de l'informe, pregunta a l'usuari:

> **Vols informes diaris o setmanals?** Puc configurar la programació automàtica:
> - `/loop 24h /investment-analysis` per a informes diaris
> - `/loop 168h /investment-analysis` per a informes setmanals
>
> O simplement executa'l manualment quan vulguis dient "executa anàlisi d'inversions".

NO ho configuris automàticament — només menciona-ho com a opció.

## Gestió d'Errors

- Si `WebSearch` no retorna resultats per a un actiu, prova `WebFetch` en llocs financers coneguts (Yahoo Finance, CoinGecko, Google Finance).
- Si un agent retorna JSON mal format, torna a fer-li el prompt una vegada amb instruccions de correcció. Si continua fallant, marca aquell sector com a `"data_unavailable": true`.
- Si l'Agent d'Estratègia falla, torna al rànquing simple per puntuació de confiança (comportament v1) i indica "Anàlisi d'estratègia no disponible" a l'informe.
- Si Python no està disponible, prova `npx serve output -p PORT` o indica a l'usuari que obri `output/report.html` directament al navegador.
- Si totes les cerques web fallen (sense internet), genera l'informe amb missatges "Dades no disponibles".
- Si els fitxers de dades històriques estan corruptes, omet el seguiment de l'encert i comença de nou.

## Notes Importants

- Usa sempre la data d'avui quan construeixis les consultes de cerca.
- L'informe HA d'incloure un avís visible que això no és assessorament financer.
- Mai no emmagatzemis ni reutilitzis dades antigues — cada invocació fa recerca fresca.
- Mantén els prompts dels agents enfocats — cada agent de sector hauria de fer 5-8 cerques web orientades (incloent xarxes socials).
- L'Agent d'Estratègia és el cervell — passa-li TOTES les dades de sector i deixa que faci el pensament entre sectors.
- El perfil de risc ho condiciona tot: quins actius emfatitzar, mides de posició i percentatges d'assignació.
