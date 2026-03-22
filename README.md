# CHAT Extended Dataset

**Cross-country Historical Adoption of Technology (CHAT) merged with labor share, exchange rates, and economic variables.**

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX)

## Overview

This dataset extends the original CHAT dataset (Comin & Hobijn, 2010) by merging it with:
- **OECD labor share data** (compensation of employees / GDP)
- **Federal Reserve exchange rates** (bilateral rates vs USD)
- **Technology classifications** (sector, type, market segment)

The result is a unified country × year × technology panel for studying technology diffusion and distributional constraints.

## Dataset Summary

| Dimension | Coverage |
|-----------|----------|
| **Observations** | 208,310 |
| **Countries** | 161 |
| **Technologies** | 104 |
| **Years** | 1750–2004 |
| **With labor share** | 37,166 observations (66 OECD countries) |
| **With exchange rates** | 9,895 observations |

## Data Pipeline Summary

| Component | Source | Raw Size | Processed | Coverage |
|-----------|--------|----------|-----------|----------|
| CHAT | GitHub/Zenodo | 6.4 MB CSV | 208,310 rows | 161 countries, 1750–2004 |
| Labor Share | OECD National Accounts | 104 MB CSV | 11,838 rows | 66 countries, 1950–2025 |
| Exchange Rates | Fed H.10 | 26 MB XML | 3,335 rows | 69 currencies, 1971–2026 |

## Files

```
data/
├── chat_extended.csv          # Main merged dataset (34 MB)
├── raw/
│   ├── chat_github.csv        # Original CHAT from GitHub (41,700 rows)
│   └── hatch_v1.5.csv         # HATCH v1.5 from Zenodo (7,358 rows)
└── processed/
    ├── chat_diffusion.csv     # Long-format subset (8 key technologies, 44,806 rows)
    ├── labor_share_panel.csv  # OECD labor share data
    └── exchange_rates_annual.csv  # Fed H.10 annual averages
```

## Variables

### Identifiers
| Variable | Description |
|----------|-------------|
| `country_name` | Country name (161 unique) |
| `country_code` | ISO 3-letter code (where available via mapping) |
| `year` | Calendar year (1750–2004) |
| `technology` | Technology name (104 types) |

### Technology Classification
| Variable | Description | Values |
|----------|-------------|--------|
| `tech_sector` | Economic sector | communication, computing, media, transportation, energy, agriculture, finance, medical, other |
| `tech_type` | Good type | durable, service, infrastructure, consumable, unknown |
| `tech_market` | Primary market | consumer, enterprise, both, unknown |

### Adoption Measures
| Variable | Description | Notes |
|----------|-------------|-------|
| `adoption_level` | Raw adoption measure | Units vary by technology (see below) |
| `adoption_per_capita` | Adoption level / population | Comparable across countries |
| `log_adoption` | log(1 + adoption_level) | For regression analysis |

### Economic Variables (from CHAT)
| Variable | Description | Source |
|----------|-------------|--------|
| `population` | Population (thousands) | CHAT `xlpopulation` |
| `real_gdp` | Real GDP | CHAT `xlrealgdp` |
| `gdp_per_capita` | GDP / population | Calculated |
| `log_gdp_pc` | log(GDP per capita) | Calculated |
| `log_population` | log(population) | Calculated |

### Distribution Variables (from OECD)
| Variable | Description | Source |
|----------|-------------|--------|
| `labor_share` | Compensation / GDP | OECD D1/B1GQ |
| `profit_share` | 1 - labor_share | Calculated |
| `compensation` | D1: Compensation of employees | OECD National Accounts |
| `gdp` | B1GQ: Gross domestic product | OECD National Accounts |

### Exchange Rates (from Fed H.10)
| Variable | Description | Source |
|----------|-------------|--------|
| `usd_exchange_rate` | Annual average bilateral rate vs USD | Fed H.10 release |

## Key Technologies

The dataset includes 104 technologies. Key ones for diffusion analysis:

| Technology | Observations | Years | Description | Units |
|------------|--------------|-------|-------------|-------|
| `telephone` | 7,316 | 1876–2003 | Telephone subscribers | Total subscribers |
| `vehicle_car` | 6,793 | 1895–2003 | Passenger automobiles | Total vehicles |
| `tv` | 4,998 | 1946–2002 | Television sets | Total sets |
| `radio` | 10,473 | 1815–2000 | Radio receivers | Total receivers |
| `computer` | 1,414 | 1960–2003 | Personal computers | Total computers |
| `cellphone` | 4,177 | 1979–2003 | Mobile phone subscribers | Total subscribers |
| `internetuser` | 1,503 | 1990–2003 | Internet users | Total users |
| `elecprod` | 8,132 | 1750–2001 | Electricity production | kWh |

## Methodology

### CHAT Data Processing

- **Source**: Original CHAT dataset from [GitHub](https://github.com/datasets/historical-adoption-of-technology) (Comin & Hobijn)
- **Reshaping**: Wide format (technology columns) → Long format (technology rows)
- **Cleaning**: Converted all adoption values to numeric; dropped missing values
- **Technology columns identified**: 104 technologies (excluding metadata columns: `xlpopulation`, `xlrealgdp`, enrollment/investment variables)

### Labor Share Calculation

```
labor_share = D1 / B1GQ
```

Where:
- **D1** = Compensation of employees (OECD transaction code)
- **B1GQ** = Gross domestic product (OECD transaction code)

**Processing notes**:
- Filtered to total economy (ACTIVITY = `_T` for D1)
- Merged D1 (compensation) with B1GQ (GDP) by country-year
- Filtered to valid range: 0.20 < labor_share < 0.90 (excludes outliers from measurement issues)
- Multiple rows per country-year exist due to different unit measures (e.g., USD_EXC); we prefer USD_EXC where available

**Key insight**: `labor_share` represents the wage share of national income—a key variable for studying how income distribution affects technology adoption.

### Exchange Rate Processing

- **Source**: Federal Reserve H.10 release (Foreign Exchange Rates)
- **Format**: Parsed from SDMX XML format
- **Filtering**: Kept only RX* series (bilateral exchange rates vs USD); excluded indices and other derived series
- **Aggregation**: Daily rates aggregated to annual averages
- **Key series**: EU (Euro), UK (GBP), JP (JPY), KO (KRW), IN (INR), and 64 other currencies

### Country Matching

CHAT country names were matched to ISO 3-letter codes for merging with OECD data:
- 46 major countries mapped (USA, GBR, DEU, FRA, JPN, etc.)
- Countries without ISO mapping retain OECD data where country names match
- Smaller/historical countries may lack labor share data

## Variable Construction Guide

### For S-Curve Estimation (Historical Diffusion)

```
A(t) = K / (1 + exp(-r * (t - t0)))
```

Where:
- `A(t)` = adoption level at time t (use `adoption_per_capita`)
- `K` = saturation level (maximum adoption)
- `r` = diffusion speed (rate parameter)
- `t0` = inflection point (year of fastest adoption)

Estimate `r`, `K`, and `t0` from `chat_extended.csv` for each technology × country.

### For Distribution Analysis

```
# How does labor share affect adoption speed?
regression: adoption_growth ~ labor_share + gdp_per_capita + year_FE + country_FE
```

### For AI Affordability Analysis

```
AI_affordability = (1 / usd_exchange_rate) * AI_price_USD
```

Where `AI_price_USD` comes from external data (e.g., Epoch AI, GPU prices).

## Usage

### Python
```python
import pandas as pd

df = pd.read_csv('data/chat_extended.csv')

# Filter to key technologies
key_techs = ['telephone', 'vehicle_car', 'tv', 'computer', 'cellphone', 'internetuser']
df_key = df[df['technology'].isin(key_techs)]

# Analyze adoption by labor share
df_with_ls = df[df['labor_share'].notna()]
print(f"Observations with labor share: {len(df_with_ls):,}")
print(df_with_ls.groupby('technology')['labor_share'].describe())

# Calculate adoption growth rates
df_sorted = df.sort_values(['country_name', 'technology', 'year'])
df['adoption_growth'] = df.groupby(['country_name', 'technology'])['adoption_per_capita'].pct_change()
```

### R
```r
library(tidyverse)

df <- read_csv('data/chat_extended.csv')

# S-curve visualization
df %>%
  filter(technology == 'telephone') %>%
  ggplot(aes(x = year, y = adoption_per_capita, color = country_name)) +
  geom_line(alpha = 0.5) +
  scale_y_log10() +
  theme_minimal() +
  labs(title = "Telephone Diffusion Across Countries",
       y = "Adoption per capita (log scale)")

# Labor share relationship
df %>%
  filter(!is.na(labor_share)) %>%
  group_by(technology) %>%
  summarise(
    n = n(),
    mean_labor_share = mean(labor_share),
    correlation = cor(adoption_per_capita, labor_share, use = "complete.obs")
  )
```

### Stata
```stata
import delimited "data/chat_extended.csv", clear

* Keep key technologies
keep if inlist(technology, "telephone", "vehicle_car", "tv", "computer")

* Panel setup
encode country_name, gen(country_id)
encode technology, gen(tech_id)
xtset country_id year

* Fixed effects regression
xtreg adoption_per_capita labor_share log_gdp_pc i.year, fe cluster(country_id)
```

## Data Sources

| Source | URL | Raw File | Citation |
|--------|-----|----------|----------|
| CHAT Dataset | [GitHub](https://github.com/datasets/historical-adoption-of-technology) | `chat_github.csv` | Comin & Hobijn (2010) |
| HATCH v1.5 | [Zenodo](https://zenodo.org/records/7893867) | `hatch_v1.5.csv` | Extended CHAT |
| OECD National Accounts | [OECD Data Explorer](https://data-explorer.oecd.org/) | `oecd_gdp_income_approach.csv` | Income approach: D1, B1GQ |
| Fed H.10 | [Federal Reserve](https://www.federalreserve.gov/releases/h10/) | `H10_data.xml` | Foreign Exchange Rates |

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2026-03-21 | 1.0.0 | Initial data collection and processing |
| 2026-03-21 | 1.0.1 | Fixed labor share calculation (B1GQ not B1G) |
| 2026-03-21 | 1.0.2 | Fixed exchange rate parsing (RX* series filter) |
| 2026-03-22 | 1.1.0 | Created merged CHAT Extended dataset |

## Citation

If you use this dataset, please cite:

```bibtex
@misc{chat_extended_2026,
  author = {Strauss, Ilan},
  title = {CHAT Extended: Cross-country Technology Adoption with Labor Share and Exchange Rates},
  year = {2026},
  publisher = {GitHub},
  url = {https://github.com/IlanStrauss/chat-extended-dataset}
}
```

And the original CHAT paper:
```bibtex
@article{comin2010exploration,
  title={An Exploration of Technology Diffusion},
  author={Comin, Diego and Hobijn, Bart},
  journal={American Economic Review},
  volume={100},
  number={5},
  pages={2031--2059},
  year={2010}
}
```

## Related Research

This dataset was created for research on AI adoption and distributional constraints:

> **"AI Adoption and the Distributional Constraint: When Scale Economies Race Real Wages Along the S-Curve"**
>
> The distributional constraint on technology diffusion is not uniform along the adoption trajectory. For expensive durables, the binding constraint is the price-to-median-wage ratio before scale economies have driven costs down. For cheap goods, the constraint is bottom-quintile real income at the tail of diffusion. This dataset enables testing these hypotheses across 200+ years of technology diffusion.

## Known Limitations

1. **Labor share coverage**: Only 66 OECD countries have labor share data (1950–2025); non-OECD countries are missing this variable
2. **Exchange rate coverage**: Fed H.10 data begins in 1971; earlier years lack exchange rate data
3. **Country code matching**: Not all CHAT countries have ISO codes; some may fail to merge with OECD data
4. **Technology units**: Adoption units vary by technology and are not always comparable
5. **Historical GDP**: Real GDP figures for early years (pre-1900) should be treated with caution

## Code

The processing scripts are available in the main research repository:

| Script | Purpose |
|--------|---------|
| `build_chat_extended.py` | Creates the merged CHAT Extended dataset |
| `process_all_data.py` | Master processing script for all data sources |

## License

The merged dataset is provided under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Original data sources retain their respective licenses:
- CHAT: Public domain (via GitHub)
- OECD: [OECD Terms and Conditions](https://www.oecd.org/termsandconditions/)
- Fed H.10: Public domain (US Government)

## Contact

- **Author**: Ilan Strauss
- **Email**: [your email]
- **Institution**: Social Science Research Council / AI Disclosures Project

---

*Last updated: March 2026*
