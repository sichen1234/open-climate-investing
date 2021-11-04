#Steps to Replicate the Paper's Results

These are steps to replicate the paper "Constructing Equity Model Factors from ESG".  Please also see [the video](https://youtu.be/Dr3G14ogceU) which shows you the steps.

## R Scripts

### Correlation matrix

Run `R/correlation_script.R`, changing `carbon_data` to your BMG factors file.  Get the `corr_matrix` result to compare with the Correlations and the `fit` summary to compare with the BMG Regressed vs Other Factors result.

### Regressions

Run predictive power for data/msci_sector_returns.csv up to 5 to get sector market weighted returns
Run predictive power for data/msci_constituent_returns.csv 1-4 and 8 to get equal weighted results

## Python Scripts

Initialize your database with the steps from [README.md](README.md)  Then load the stock prices of the MSCI World index using the command

```
python get_stocks.py -f data/stock_tickers_msci_world.csv
```

You only need to do this once to populate the stock history data.

Then to run it for other BMG series:
- Run SQL `delete from stock_stats`
- Run SQL `delete from carbon_risk_factor`
- Reload carbon_risk_factor with command:
```
psql open_climate_investing  -c 'COPY carbon_risk_factor FROM STDIN WITH (FORMAT CSV, HEADER);' < new-bmg-series.csv
```
- Run all regressions
```
python get_regressions.py -f data/stock_tickers_msci_world.csv -e 2021-09-30
```
- Get the stocks with significant BMG results:
```
select SS.thru_date, S.ticker, S.name, S.sector, SS.bmg, SS.bmg_t_stat 
from stock_stats as SS join stocks as S on S.ticker = SS.ticker 
where SS.thru_date > '2018-12-01'
and SS.bmg_p_gt_abs_t < 0.05
order by S.sector, bmg
```
- Get a count of the stocks with significant BMG results by sector:
```
select distinct S.sector, count(SS.ticker)
from stock_stats as SS 
left outer join stocks as S on S.ticker = SS.ticker
where SS.thru_date > '2021-09-01'
and SS.bmg_p_gt_abs_t < 0.05
group by S.sector
```
Substitute '2021-09-01' for the last date of your regression results.


### Fixing Missing Stock Tickers

You probably won't encounter this, but if the SQL queries from above

If there are missing tickers, update msci_constituent_details.csv and msci_sector_breakdowns.csv with the correct tickers

Delete stocks

Reload stocks from msci_sector_breakdowns.csv

## Construct your own BMG

It's easy and fun!  First get the tickers for the stocks you want to use.  Put them in a CSV file such as `data/my-bmg-tickers.csv`.

Then use `get_stocks.py` to get the historical prices of those stocks.

Finally use this SQL:
```
select SD1.date, SD1.ticker, SD1.return, SD2.ticker, SD2.return, SD1.return-SD2.return as BMG
from stock_data as SD1
join stock_data as SD2 on SD1.date = SD2.date
where SD1.ticker = 'XOP' and SD1.date > '2010-01-01'
and SD2.ticker = 'SMOG'
```
and substitute `XOP` and `SMOG` with your tickers.  If the results look good, only select the date and BMG columns into a CSV file.