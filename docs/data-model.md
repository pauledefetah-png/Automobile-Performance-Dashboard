# Data Model

The report uses a star schema built in Power BI, rather than a single flat table. The raw CSV is loaded once, then split into a fact table and five dimension tables connected by relationships:

**Fact table**
- `Automobile` — one row per vehicle, holding the numeric measures (price, horsepower, engine size, city/highway MPG, curb weight, etc.) and foreign keys to each dimension.

**Dimension tables**
- `Dim_Vehicle_Brand` — the 22 manufacturers in the dataset (Alfa Romeo, Audi, BMW, Chevrolet, ... Volvo).
- `Dim_BodyStyle` — convertible, hatchback, sedan, wagon, hardtop.
- `Dim_Drive_Wheels` — fwd, rwd, 4wd.
- `Dim_Fuel_Type` — gas, diesel.
- `Dim_Price_Band` — price bucketed into ranges, used to slice the treemap and cards by market segment.

Splitting the flat CSV into a fact/dimension structure (rather than reporting directly off the raw table) keeps the model normalized, makes the slicers faster, and mirrors how a real BI model would be built on top of a wider transactional export.

## Renamed / derived fields

The raw CSV headers are terse (`make`, `horsepower`, `city-mpg`, ...). In the model they were renamed to business-friendly names and one field was derived:

| Raw CSV column | Model field | Notes |
|---|---|---|
| `make` | `Vehicle_Brand` | dimension key |
| `horsepower` | `Horse_Power` | numeric measure |
| `body-style` | `BodyStyle` | dimension key |
| `city-mpg`, `highway-mpg` | `City_MPG`, `Highway_MPG` | kept separate, and combined into... |
| *(derived)* | `AvgMPG_Of_City&Highway` | calculated field averaging city + highway MPG, used in the "Average Miles Per Gallon" KPI card |
| *(row id)* | `Vehicle_ID` | added to give each row a unique key for the scatterplots |

## Report pages and visuals

**Page 1 — Overview**

| Visual | Type | Fields |
|---|---|---|
| Average Price by Vehicle Brand | Bar chart | `Vehicle_Brand` x `SUM(Price)` |
| Average Horsepower by Vehicle Brand | Bar chart | `Vehicle_Brand` x `SUM(Horse_Power)` |
| Average Price by Body Style | Column chart | `BodyStyle` x `SUM(Price)` |
| Horsepower & Price correlation | Scatter chart | `SUM(Horse_Power)` vs `SUM(Price)`, by `Vehicle_ID` |
| Engine Size vs. MPG | Scatter chart | `Highway_MPG` vs `Engine_Size`, by `Vehicle_ID` |
| MPG efficiency per brand | Line chart | `City_MPG` & `Highway_MPG` by `Vehicle_Brand` |
| Total Value of Vehicles | KPI card | `SUM(Price)` |
| Average Miles per Gallon | KPI card | `AvgMPG_Of_City&Highway` |
| Total Vehicle Brands | KPI card | `MIN(Vehicle_Brand)` (count context) |
| Dominating Vehicle Brands | Treemap | `Vehicle_Brand` sized by `COUNTNONNULL(BodyStyle)` |

**Page 2 — Engine detail**

| Visual | Type | Fields |
|---|---|---|
| Average Horsepower by Number of Cylinders | Scatter chart | `Num_Of_Cylinders` vs `SUM(Horse_Power)` |
