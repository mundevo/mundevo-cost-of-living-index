# Mundevo Cost of Living Index — 104 cities, sourced line by line

A cost-of-living index (New York = 100) for 104 cities in 49 countries, built
from a fixed one-person monthly basket — rent, food, transport, electricity and
heating, leisure, out-of-pocket healthcare — where **every line of every city
names its publisher, link, the period it describes and how it was computed**.
Alongside it: a rent index, the payroll-tax burden at the country's average
wage, and three quality-of-life indices (lethal violence, health-service
coverage, air quality), each shipped with the raw published figure it was
computed from.

- Live page: <https://www.mundevo.com/cost-of-living-index>
- Method: <https://www.mundevo.com/methodology> (basket definitions:
  [#monthly-basket](https://www.mundevo.com/methodology#monthly-basket))
- Sources: <https://www.mundevo.com/sources>
- License: [CC BY 4.0](LICENSE)

## Files

| File | Rows | What it is |
|---|---|---|
| `data/cost-of-living-index.csv` | 104 (one per city) | The index, ranked cheapest first |
| `data/cost-of-living-index-sources.csv` | 961 (city × component) | The provenance of every figure in the index: tier, publisher, URL, period, method, value |

Both are UTF-8, comma-separated, RFC 4180 quoting, `\n` line endings, one header
row. The same files are served at
<https://www.mundevo.com/data/cost-of-living-index.csv> and
<https://www.mundevo.com/data/cost-of-living-index-sources.csv>; this repository
is a versioned snapshot of them.

## `cost-of-living-index.csv` — columns

| # | Column | Type | Unit / definition |
|---|---|---|---|
| 1 | `rank` | integer | 1 = cheapest. Sorted by `cost_index` ascending, ties by city name |
| 2 | `city` | string | City name in English |
| 3 | `country` | string | Country or territory name in English |
| 4 | `iso2` | string | ISO 3166-1 alpha-2 code (includes `HK`, `TW`) |
| 5 | `cost_index` | integer | Monthly basket (six lines, below) ÷ New York's basket × 100, both converted to EUR with the exchange-rate table of the basket refresh. New York = 100 |
| 6 | `rent_index` | integer | The rent line alone ÷ New York's rent × 100. New York's rent is EUR 500 per m² per year, so `rent_index / 100 × 500` = the city's rent in EUR per m² per year |
| 7 | `tax_burden_pct` | integer, % | Income tax + employee social-security contributions as a share of gross pay, single filer **at the country's average wage**. Not the rate at any other salary |
| 8 | `rent_source` | enum | Evidence tier of the rent line: `official`, `official-adjusted`, `contrasted`, `estimated` (definitions below) |
| 9 | `data_as_of` | `YYYY-MM` | The most recent period among the city's six basket lines. Individual lines can be older — each line's own period is in the sources file |
| 10 | `safety_index` | integer 0–100 | From the homicide rate on a log scale: 0.5 per 100,000 or less = 100; each doubling of the rate costs 15 points |
| 11 | `homicides_per_100k` | number | Intentional homicides per 100,000 inhabitants, mean of up to five reported years (UNODC via World Bank WDI) |
| 12 | `homicide_scope` | enum | What the rate measures: `national`, `regional` or `city` |
| 13 | `health_coverage_index` | integer 0–100 | UHC service coverage rescaled against the world: the 10th-percentile country (44 in 2023) = 0, the best-covered country (92) = 100; anchors computed from 193 countries |
| 14 | `uhc_service_coverage` | integer 0–100 | The WHO / World Bank UHC service coverage index (SDG 3.8.1), national, as published |
| 15 | `air_quality_index` | integer 0–100 | From `pm25_ugm3`, piecewise-linear between the WHO 2021 guideline and interim targets: 5 µg/m³ = 100, 10 = 80, 15 = 65, 25 = 45, 35 = 30, 50 = 15, 75 or more = 0 |
| 16 | `pm25_ugm3` | number, µg/m³ | Annual mean surface PM2.5: median of the 2023–2025 annual means from the Copernicus CAMS global model, or the city's own monitoring network where one publishes (Beijing, Delhi, Shanghai) |
| 17 | `quality` | integer 0–100 | Weighted composite: 0.40 × safety + 0.35 × health coverage + 0.25 × air quality |
| 18 | `tax_source` | enum | `oecd-taxing-wages` (refreshed from OECD Taxing Wages) or `hand-calibrated` (non-OECD countries; see limitations) |
| 19 | `currency` | string | ISO 4217 code of the local currency |
| 20 | `monthly_basket_local` | integer | The six-line monthly basket in local currency, rounded — the figure `cost_index` is the ratio of |
| 21 | `slug` | string | Stable city identifier; joins to `city_slug` in the sources file |
| 22 | `url` | URL | The city's page, with the same breakdown and sources in prose |

### The basket

One person, an unfurnished one-bedroom flat of about 50 m², per month:

- **Rent** — an unfurnished one-bedroom (≈ 50 m²) on a **new** lease, typical
  for the whole municipality, not the city centre. Per-m² sources × 50.
- **Food** — what a one-person US household spends on groceries (BLS Consumer
  Expenditure Survey), repriced in each country with World Bank ICP 2021 price
  levels carried forward with the Bank's PPPs; BEA Regional Price Parities inside
  the US, the Statistics Bureau's regional index in Japan.
- **Leisure** — the same method for restaurants and recreation.
- **Utilities** — the official household electricity price per kWh × 200 kWh of
  base load plus heating/cooling for 50 m² from the city's degree days. Water,
  building charges and broadband are not included.
- **Transport** — the adult monthly public-transport pass, or single fare × 44
  trips where no pass exists.
- **Healthcare cost** — national out-of-pocket health spending per person ÷ 12
  (World Bank / WHO), plus the employee premium share in the US and the
  mandatory basic-insurance premium in Switzerland and the Netherlands.

### Evidence tiers

| Tier | Meaning |
|---|---|
| `official` | A public body (statistics office, ministry, rent board, central bank, transport authority) measured the figure for that city |
| `official-adjusted` | Public-body data, but national or regional, or run through the published method above |
| `contrasted` | No public body publishes it for that city; a named press or market report (a property portal's or consultancy's own published index) |
| `estimated` | Not even a published report exists for the city; derived from a same-country city, with the method stated |

## `cost-of-living-index-sources.csv` — columns

One row per city and component. Components: `rent`, `food`, `transport`,
`utilities`, `leisure`, `healthcare_cost` (the six basket lines, 104 rows each),
`rent_sitting_tenants` (25 cities: an official rent of existing leases shown
beside a market figure; **not** used in the index), `safety`, `health_coverage`,
`air_quality` (104 rows each).

| Column | Definition |
|---|---|
| `city_slug`, `city`, `iso2` | The city; `city_slug` joins to `slug` in the index file |
| `component` | One of the components above |
| `tier` | Evidence tier (above) |
| `scope` | For quality components: `city`, `national` or `regional`. Empty for basket lines |
| `publisher` | Who published the figure |
| `source_url` | Where it was published |
| `data_as_of` | `YYYY-MM`: the period the figure describes, not when it was fetched |
| `method` | How the published figure became the value in this row |
| `value`, `unit` | Basket lines: the monthly cost in local currency. Quality components: the published measurement the index is computed from (homicides per 100,000; UHC index; µg/m³ PM2.5) |
| `vs_new_york` | Basket lines: this line ÷ New York's same line, in EUR at the refresh's exchange rates |
| `caveat` | A known weakness of this specific figure, where there is one |

## Coverage today

- 104 cities in 49 countries; one row per city, no empty cells in either file
  except `scope`, `vs_new_york` and `caveat` where they do not apply.
- Rent tier: 31 `official`, 19 `official-adjusted`, 43 `contrasted`,
  11 `estimated`.
- Basket line periods range from 2023 to 2026-11 (see `data_as_of` per row).
- Tax: 68 cities in 30 OECD countries from OECD Taxing Wages; 36 cities in 19
  countries hand-calibrated.

These counts describe this release; the files themselves are the reference.

## Known limitations

Read these before you compare two cities.

1. **Half the basket is rent, and 51 cities' rent has no public-body source.**
   40 are `contrasted` (a named market report) and 11 are `estimated` from a
   same-country city (Arequipa, Bali, Busan, Casablanca, Cebu, Chiang Mai,
   Jakarta, Marrakech, Medellin, Penang, Valparaiso). When we compared earlier
   estimates against later sourced figures, the estimates were off by a median
   12.5 % and a 75th percentile of 25.6 %. Mundevo does not state a gap between
   two cities as fact when either has a non-official rent and the gap is under
   25 %; you should not either.
2. **Rent definitions are harmonised, not identical.** The target is a new-lease
   one-bedroom for the whole municipality, but sources differ: some publish
   rent per m² of all sizes, some medians of registered leases, some average
   asking rents of listings, some cover the urban area rather than the city.
   The `method` column says which, row by row. Asking rents usually run above
   signed rents.
3. **Food and leisure are modelled.** They reprice one US consumption basket
   with national price levels (ICP 2021 carried forward), so they vary by
   country, not by city, outside the US and Japan.
4. **Several quality figures are national.** 103 of 104 homicide rates are
   national averages applied to the city, and health coverage is national
   everywhere. Some homicide series are old: Thailand and Vietnam have reported
   none since 2011, Egypt since 2017. Taiwan's homicide rate, and Taiwan's and
   Hong Kong's health coverage, are estimated from regional peers because the
   World Bank / WHO do not score them. Each case is spelled out in `caveat`.
5. **Air quality is mostly modelled.** CAMS runs on a ~45 km grid and smooths
   urban peaks: good for ordering cities, less so for absolute levels.
6. **Tax for 19 non-OECD countries is hand-calibrated**, not refreshed from a
   statistical series, and every tax figure applies only at the average wage.
7. **Exchange rates.** `cost_index` uses the exchange-rate table of the basket
   refresh. Recomputing it from `monthly_basket_local` with today's rates will
   not reproduce it exactly.
8. **Series break, November 2026.** Before then the basket came from Numbeo
   (crowd-sourced, under terms that do not allow reuse). Copies of this file from
   before November 2026 are a different series and are not comparable.

## Provenance policy

Every number traces to a published source: a statistics office, ministry,
regulator, transport authority, central bank or international body where one
exists; otherwise a named publication, labelled `contrasted`; otherwise a stated
derivation, labelled `estimated`. Nothing is crowd-sourced and nothing is typed
in without a source. The files are generated by code from the same tables that
render <https://www.mundevo.com>, so the dataset and the site cannot disagree.

## License and attribution

Released under **Creative Commons Attribution 4.0 International** (CC BY 4.0) —
full legal code in [`LICENSE`](LICENSE). You may share and adapt it for any
purpose, including commercial, if you give attribution:

> Mundevo Cost of Living Index (https://www.mundevo.com/cost-of-living-index), CC BY 4.0

The license covers Mundevo's compilation, harmonisation and derived indices.
Underlying figures come from the publishers named in the sources file, who keep
their own terms; several require attribution of their own — World Bank (WDI,
ICP) and Eurostat data are CC BY 4.0, and the air figures contain modified
Copernicus Atmosphere Monitoring Service information. If you redistribute
individual source figures, credit their publisher too.

## How to cite

See [`CITATION.cff`](CITATION.cff) (GitHub shows a "Cite this repository"
button). Plain text:

> Mundevo (2026). *Mundevo Cost of Living Index* [Data set]. https://www.mundevo.com/cost-of-living-index. CC BY 4.0.

## Updates

Each source family has its own refresh script in the Mundevo codebase (rents and
basket lines, quality indices, OECD tax rates, ECB exchange rates, World Bank
series); the published CSVs are regenerated on every site build. This snapshot
is updated when a refresh moves the data — there is no fixed schedule, and each
line's own period is in `data_as_of`. Releases are dated (in `CITATION.cff` on
GitHub); the live files at mundevo.com/data are always the latest.

Errors and better sources are welcome as issues on this repository.
