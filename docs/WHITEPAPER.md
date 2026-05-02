# AIMEM — AI Memory Price Index

## Abstract

AIMEM is the world's first on-chain memory semiconductor price index, designed to be traded as a perpetual contract on Hyperliquid via HIP-3. It tracks the actual spot prices of DRAM, HBM, and NAND memory chips — the critical bottleneck of AI infrastructure.

## 1. Motivation

Memory semiconductors are the most cyclical and supply-constrained component of the AI compute stack. In 2026:
- DRAM contract prices surged 58-63% QoQ in Q2
- HBM is sold out through 2026, with 20% price increases
- NAND contract prices rose 70-75% QoQ

Yet there exists **no tradeable instrument** that directly tracks memory chip prices:
- No DRAM futures on any exchange (CME, SGX attempts failed)
- No on-chain memory price oracle
- The closest proxy — Roundhill Memory ETF (DRAM) — tracks stock prices, not chip prices

AIMEM fills this gap.

## 2. Index Methodology

### 2.1 Components & Weights

| Component | Weight | What It Tracks | Data Source |
|---|---|---|---|
| DDR5 16Gb Spot | 35% | Mainstream server/PC memory | TrendForce DRAMeXchange |
| HBM Proxy | 30% | AI memory (via stock basket) | Exchange data |
| NAND 512Gb TLC Spot | 20% | Flash storage | TrendForce |
| DDR4 16Gb Spot | 15% | Legacy memory | TrendForce |

### 2.2 HBM Proxy Construction

Since HBM chips have no public spot price, we construct a proxy:

```
HBM_proxy = 50% × (SK_Hynix / SK_Hynix_base) 
          + 25% × (Samsung / Samsung_base)
          + 25% × (Micron / Micron_base)
```

Weights reflect HBM market share: SK Hynix 62%, Samsung 17%, Micron 21%.

### 2.3 Index Calculation

```
AIMEM_t = 100 × Σ(w_i × P_i_t / P_i_base)
```

- Base date: January 1, 2026
- Base value: 100.00
- Update frequency: Daily (with intraday updates for HBM proxy)

### 2.4 Rebalancing

- Component weights reviewed quarterly
- HBM proxy stock weights adjusted when market share data updates
- New components may be added (e.g., direct HBM pricing if available)

## 3. Data Sources

### 3.1 TrendForce DRAMeXchange (Primary)
- URL: trendforce.com/price/dram
- Coverage: DDR3/DDR4/DDR5/GDDR spot prices
- Frequency: Daily (business days)
- Method: Automated scraping

### 3.2 Stock Exchanges (HBM Proxy)
- SK Hynix (000660.KS): Korea Exchange
- Samsung (005930.KS): Korea Exchange  
- Micron (MU): NASDAQ
- Source: IBKR API / Pyth Network

### 3.3 Silicon Data RAM Index (Supplementary)
- GDDR6 per GB pricing
- Daily update at 16:00 UTC

## 4. Oracle Architecture

```
Data Layer          Computation Layer       On-Chain Layer
┌──────────┐       ┌──────────────┐        ┌──────────────┐
│TrendForce│──┐    │              │        │  Hyperliquid  │
│  Scraper │  │    │   AIMEM      │        │  HIP-3        │
├──────────┤  ├───→│   Index      ├───────→│  vntl:AIMEM   │
│Stock API │  │    │   Engine     │        │  Perpetual    │
│(IBKR/Pyth│──┘    │              │        │  Contract     │
├──────────┤       └──────────────┘        └──────────────┘
│Silicon   │              │
│Data      │──────────────┘
└──────────┘

Update cadence: Every 3 seconds (on-chain)
Data refresh: Daily for chip prices, real-time for stocks
```

## 5. Contract Specifications

| Parameter | Value |
|---|---|
| Ticker | vntl:AIMEM |
| Underlying | AIMEM Index |
| Settlement | USDH |
| Max Leverage | 10x |
| OI Cap | $10,000,000 (initial) |
| Mark Band | ±20% of oracle |
| Funding | Hyperliquid standard formula |
| Trading Hours | 24/7/365 |

## 6. Use Cases

### 6.1 Hedging
- Memory chip buyers (OEMs, cloud providers) hedge procurement costs
- Memory manufacturers hedge revenue exposure
- Semiconductor investors hedge portfolio risk

### 6.2 Speculation
- Trade memory supercycles (historically 3-4 year cycles)
- Express views on AI compute demand
- Bet on supply shortages / oversupply

### 6.3 Research
- Academic study of commodity price discovery on-chain
- Cross-asset correlation analysis (AIMEM vs NVDA, AI tokens)
- Market microstructure research

## 7. Risk Factors

1. **Data source risk**: TrendForce website changes could break scraper
2. **Liquidity risk**: Cold start period with thin orderbook
3. **Oracle manipulation**: Low-liquidity HBM proxy could be manipulated
4. **Regulatory risk**: On-chain commodity derivatives in gray area
5. **Component obsolescence**: DDR4 declining, new memory types emerging

## 8. Team

- **Qihong Ruan** — Cornell PhD, Quantitative Finance
- Research infrastructure: Cornell JCB Research servers + BioHPC
- Data: Kaiko, TAQ, WRDS, TrendForce

## 9. Current Index Value

As of May 2, 2026:

**AIMEM = 119.08** (+19.08% YTD)

| Component | Base | Current | Change |
|---|---|---|---|
| DDR5 16Gb | $28.00 | $37.20 | +32.9% |
| DDR4 16Gb | $34.00 | $32.50 | -4.4% |
| HBM Proxy | 100.00 | 131.72 | +31.7% |
| NAND 512Gb | $22.00 | $20.60 | -6.4% |

The index is being driven by DDR5 and HBM strength, partially offset by NAND spot weakness.

---

*AIMEM is provided for informational and research purposes. It does not constitute financial advice.*
