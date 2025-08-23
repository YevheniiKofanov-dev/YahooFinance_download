The gold layer contains four levels of reporting data:
1. Financial data with one hour granularity - finance_gold fact table. This is the lowest and most detailed level od data. Metrics' values are not aggregated at this level.
2. Financial data with one day granularity - finance_gold_daily fact table. Metrics' values are aggregated on this level.
3. Financial data with one week granularity - finance_gold_weekly fact table. Metrics' values are aggregated on this level.
4. Financial data with one month granularity - finance_gold_monthly fact table. Metrics' values are aggregated on this level.

finance_gold fact table has the following dimensional keys and metrics:
  ticker_id - surrogate key of a ticker (company). Related dimension - gold.ticker_gold.
  date_id - surrogate key of a calendar row with day granularity. Related dimension - gold.date_gold.
  datetime - timestamp value of the moment when the financial metricks were captured by Yahoo Finance.
  close - Price at market close.
  high - Highest price during the trading session.
  low - Lowest price during the trading session.
  open - Price at market open.
  volume - Number of shares traded.

Aggregated fact tables have the following dimensional keys and metrics:
  ticker_id - surrogate key of a ticker (company). Related dimension - gold.ticker_gold.
  date_id - surrogate key of a calendar row with day granularity. Related dimension - gold.date_gold.
  year - year value corresponding to date_id.
  month - month name corresponding to date_id.
  week - week mumber corresponding to date_id. If Sunday is not the first day of the year the days before the first Sunday of the year are related to week 00.
  min_of_open - minimal value of Open price during the time period.
  max_of_close - maximum value of Close price during the time period.
  max_of_high - maximum value of High price during the time period.
  min_of_low - minimal value of Low price during the time period.
  avg_of_close - average value of Close price during the time period.
  stddev_of_close - standard deviation of Close price during the time period.
  median_of_close - median of Close price during the time period.
  sum_of_volume - summed volume during the time period.

The bronze layer gets data from Yahoo Finance and saves columns in string format. The start date for the first load is hard coded as 2025-01-01. For next loads for the same ticker start date is defined as the latest loaded date for that ticker. The end date for load is current date. Data is filtered during saving to prevent saving of rows which were loaded earlier.

The silver layer reads data from the bronze layer and converts fact columns to the needed data type. The silver layer finds input rows which were not loaded yet into silver layer. The silver layer finds the tickers which were loaded in new portion of data and do not exist yet in the gold ticker dimension.

The gold layer inserts new tickers into gold.ticker_gold table. The gold layer resolves ticker names to ticker IDs and finds input rows which were not loaded yet into gold layer. Those new fact rows are inserted into gold layer table finance_gold. Then the gold layer aggregates fact data and populates finance_gold_daily, finance_gold_weekly and finance_gold_monthly tables.

yahoo_finance.processrunlogs.processrunlog table tracks the latest timestamp of input data which was loaded to each layer. In the bronze layer tracking is done separately for each ticker. In the silver and gold layer tracking does not have ticker granularity. Tracking also includes start and end time of processing and status of processing - Running, Completed, Failed.

