# Tododeia — Multi-Agent Investment Analysis Skill

**by @soyenriquerocha**

A Claude Code skill that spawns 5 specialized AI research agents to analyze investment opportunities across crypto, stocks, forex, and commodities. Adapts to your risk profile, tracks historical accuracy, and generates an interactive Next.js dashboard with full Spanish/English support.

## What Is This?

Tododeia is a **Claude Code skill** — a reusable prompt-and-tooling package that extends what Claude can do. When installed, Claude gains the ability to run a full multi-agent investment research workflow: 4 sector analysts research in parallel, a strategy agent synthesizes everything, and the result is served as an interactive dashboard in your browser.

## How It Works

```
                         ┌──────────────┐
                    ┌────┤  You trigger  ├────┐
                    │    │  "analyze     │    │
                    │    │   markets"    │    │
                    │    └──────────────┘    │
                    ▼                        ▼
            ┌──────────────┐        ┌──────────────┐
            │ Risk Profile │        │ Load History  │
            │   Question   │        │  (if exists)  │
            └──────┬───────┘        └──────┬───────┘
                   │                       │
                   ▼                       │
    ┌──────────────────────────────┐       │
    │     4 Sector Agents          │       │
    │  (parallel web research)     │       │
    │                              │       │
    │  ┌───────┐  ┌───────┐       │       │
    │  │Crypto │  │Stocks │       │       │
    │  └───────┘  └───────┘       │       │
    │  ┌───────┐  ┌──────────┐   │       │
    │  │ Forex │  │Materials │   │       │
    │  └───────┘  └──────────┘   │       │
    └──────────────┬───────────────┘       │
                   │                       │
                   ▼                       ▼
          ┌────────────────────────────────────┐
          │        Strategy Agent               │
          │  (cross-sector analysis,            │
          │   risk-adjusted ranking,            │
          │   portfolio allocation,             │
          │   historical accuracy check)        │
          └────────────────┬───────────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
            ┌────────────┐ ┌────────────┐
            │ report.json│ │report-es   │
            │ (English)  │ │  .json     │
            └──────┬─────┘ └──────┬─────┘
                   │              │
                   ▼              ▼
              ┌──────────────────────┐
              │   Next.js Dashboard  │
              │   localhost:3420     │
              │   EN / ES toggle    │
              └──────────────────────┘
```

## Features

- **5 AI Agents**: 4 sector researchers + 1 strategy synthesizer, all running in parallel
- **Dynamic Asset Discovery**: Agents don't follow a fixed list — they find the most relevant assets based on current market conditions
- **Risk Profiles**: Conservative, moderate, or aggressive — recommendations, position sizes, and allocations adapt to you
- **Social Sentiment**: Twitter/X and Reddit sentiment analysis per asset
- **Source Verification**: Cross-references prices from 2+ sources with agreement scoring
- **Historical Accuracy**: Compares past recommendations to actual outcomes over time
- **Portfolio Allocation**: Suggested % allocation across sectors based on your risk profile
- **Cross-Sector Insights**: Correlations and divergences that individual agents can't see
- **Spanish/English**: Full bilingual support — UI chrome AND report data translate when you toggle
- **Interactive Dashboard**: Charts, expandable sector analysis, top picks grid, risk warnings
- **Scheduling**: Option to run daily or weekly with automatic reports

## Sectors Covered

| Sector | Always Includes | Dynamically Discovers |
|--------|----------------|----------------------|
| Crypto | BTC, ETH | Trending altcoins, DeFi tokens, AI tokens |
| Stocks | S&P 500, NASDAQ | Top movers, catalyst-driven stocks across sectors |
| Forex | DXY, USD/MXN | Pairs affected by current events |
| Commodities | Gold, Oil WTI | Agricultural, energy, metals based on market conditions |

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI)
- Node.js 18+ (for the dashboard)
- Active internet connection (agents perform live web research)

## Installation

### Step 1: Clone the skill

```bash
git clone https://github.com/Hainrixz/jere-noticias-inver.git
```

### Step 2: Install the skill in Claude Code

From within the cloned repo, the skill is located at `.claude/skills/investment-analysis/`. Claude Code automatically detects skills in this directory structure when you work inside the project.

To use this skill from **any directory**, copy or symlink the skill folder into your global Claude Code skills:

```bash
mkdir -p ~/.claude/skills
ln -s "$(pwd)/.claude/skills/investment-analysis" ~/.claude/skills/investment-analysis
```

### Step 3: Install dashboard dependencies

```bash
npm install --prefix dashboard
```

That's it. The skill auto-activates when you mention investment-related topics in Claude Code.

## Usage

In any Claude Code conversation:

- "Run an investment analysis"
- "What are the best investment opportunities today?"
- "Analyze the markets"
- "Give me a market report"
- "Run tododeia"

You'll be asked your risk profile (conservative, moderate, or aggressive), then the 5 agents go to work. The report opens at `http://localhost:3420`.

### Language Toggle

When you first open the dashboard, you'll be asked to pick English or Spanish. All report content — summaries, reasoning, news headlines, insights, warnings — translates to your chosen language. Numbers, prices, tickers, and percentages stay as-is.

You can switch languages anytime from the toolbar.

### Scheduling (Optional)

After your first report, you can set up recurring analysis:

```
/loop 24h /investment-analysis    # Daily reports
/loop 168h /investment-analysis   # Weekly reports
```

## Project Structure

```
.claude/skills/investment-analysis/
  SKILL.md              # Skill definition (orchestrator instructions)
  assets/
    template.html       # Fallback HTML report (no Node.js)
  references/
    agent-prompts.md    # Prompts for all 5 agents
  dashboard/            # Next.js interactive dashboard
    src/
      app/              # App router pages
      components/       # Report UI components
      hooks/            # Language + data hooks
      lib/              # Translations, constants
      types/            # TypeScript types
    public/data/        # Generated report JSON (created at runtime)
```

## Customization

### Assets & Agent Behavior

Edit `references/agent-prompts.md` to change which assets each agent researches, how they search, or what data they prioritize.

### Risk Profiles

The Strategy Agent prompt in `references/agent-prompts.md` defines how each risk profile affects scoring and allocation. Customize the multipliers and allocation ranges there.

### Dashboard Styling

The dashboard uses Tailwind CSS. Edit components in `dashboard/src/components/report/` to change the look and feel.

### Translations

Add or modify UI translations in `dashboard/src/lib/translations.ts`. Report data translation happens automatically via the translation agent (Step 8b in SKILL.md).

## Dashboard Features

- **Executive summary** with macro environment assessment
- **Portfolio allocation** chart with recommended percentages
- **Risk-adjusted top picks** grid with confidence scores and position sizing
- **Cross-sector insights** highlighting correlations between markets
- **Expandable sector panels** with detailed per-asset analysis
- **Charts**: allocation breakdown, risk vs. confidence visualization
- **Historical accuracy** tracking past recommendation performance
- **Risk warnings** when the strategy agent detects concerns
- **Full bilingual support** — English and Spanish, including all data

## Fallback Mode

If Node.js is not available, the skill falls back to generating a standalone `output/report.html` file using the HTML template in `assets/`. This is served via Python's built-in HTTP server on port 8420.

## Disclaimer

This tool is for **informational and educational purposes only**. It does not constitute financial advice, investment recommendations, or solicitation to buy or sell any securities, cryptocurrencies, or commodities. AI-generated analysis may contain errors and should not be relied upon for investment decisions. Always consult a qualified financial advisor before making investment decisions. Past performance is not indicative of future results. Tododeia and its creators assume no liability for investment losses.

## License

MIT — see [LICENSE](LICENSE)

## Contributing

Open source. PRs welcome!
