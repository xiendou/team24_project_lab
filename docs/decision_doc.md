# Decision log

The main choices Team 24 makes during the project, and why. Technical details of the data are in the [README](../README.md).

To add a decision, copy the template at the bottom and give it the next number. If a decision changes, don't delete it: mark it *Superseded by D-xx* and add the new one.

| # | Decision | Status | Date |
|---|---|---|---|
| D-01 | Research question and scope | Accepted | 2026-11-22 |
| D-02 | Which commodities to study | Accepted | 2026-11-22 |
| D-03 | Which data sources to use | Accepted | 2026-11-22 |
| D-04 | How to measure regional weather | Accepted | 2026-11-22 |
| D-05 | How to measure climate change | Accepted | 2026-11-22 |
| D-06 | Split the work into three stages | Accepted | 2026-11-22 |

---

## D-01. Research question and scope

- **Decision:** Study how weather and climate change affect commodity prices, using monthly data from January 1990 to October 2026.
- **Why:** Monthly is the finest frequency the free price data offers, and all the climate indices are monthly. 36+ years covers many El Niño / La Niña cycles and the recent warming.
- **Alternatives considered:** Daily or weekly data (would need paid price data). Annual data (too few observations; seasonal effects are lost).

## D-02. Which commodities to study

- **Decision:** Maize, US wheat, soybeans, arabica coffee, cocoa, sugar, and US and European natural gas. Brent crude oil is a control, not an outcome.
- **Why:**
  - The crops depend heavily on weather, and each comes from a few main producing regions.
  - Natural gas is driven by heating and cooling demand, which tests a different channel: demand rather than supply.
  - Brent captures general energy and economic moves that affect all commodities.
- **Alternatives considered:** Rice, palm oil, cotton, robusta coffee (possible extensions). Metals (little link to weather).

## D-03. Which data sources to use

- **Decision:**
  - prices from the World Bank Pink Sheet;
  - weather from ERA5 reanalysis (via Open-Meteo);
  - El Niño indices from NOAA;
  - global temperature from NASA GISTEMP.
- **Why:** All free, public and widely used in research, with long monthly histories and no gaps.
- **Alternatives considered:** Futures prices (paid, harder to work with). Weather station records (gaps, station changes).

## D-04. How to measure regional weather

- **Decision:** One representative point for each of 10 key regions:
  - the main producing areas for each crop (US Corn Belt, Kansas, Ukraine, Russia, Brazil, Ivory Coast, Ghana);
  - two big gas-demand cities (Chicago, New York).
- **Why:** Simple and transparent for a first version, and covers the regions that move world prices most.
- **Alternatives considered:** Country averages, or averages weighted by crop production (more accurate, much more work; can be a later improvement).

## D-05. How to measure climate change

- **Decision:** Two kinds of climate variable:
  - **ENSO** (El Niño / La Niña), the main year-to-year climate swing. We keep both of NOAA's indices: ONI and the newer RONI.
  - **The global temperature anomaly,** for the long-run warming trend.
- **Why:** ENSO shifts weather across the tropics and the Americas at the same time, so it can move several commodities at once. The global temperature captures slow change rather than short shocks. RONI has been NOAA's official ENSO index since 2026, and ONI is what most past studies used.
- **Alternatives considered:** Only local weather, with no global indices (misses the "climate change" part of the question).

## D-06. Split the work into three stages

- **Decision:** Data collection → cleaning → analysis, each with its own owners. Each stage hands over files and documents what it did.
- **Why:** Clear handovers and ownership. Each stage can start once the previous one is ready, and every change to the data is made in one known place.
- **Note:** The data collection stage gathers and reshapes the data but does not clean it (no adjusting, filling gaps or transforming), so that every cleaning choice is made, and recorded, by the cleaning stage.

## Template

```markdown
## D-xx. <short title>

- **Status:** Proposed | Accepted | Superseded by D-yy (YYYY-MM-DD)
- **Decision:** What we decided.
- **Why:** The reasons.
- **Alternatives considered:** What else we looked at.
```
