# project
```
-- First, update existing records
UPDATE deposit.new_td_rollover tgt
SET
    investment_type = src.investment_type,
    start_date = CAST(src.start_date AS DATE),
    maturity_date = CAST(src.maturity_date AS DATE),
    tenor = CAST(src.tenor AS INTEGER),
    currency = src.currency,
    interest_rate = CAST(src.interest_rate AS DECIMAL(20,2)),
    maturity_status = CAST(src.maturity_status AS VARCHAR),
    reference_number = CAST(src.reference_number AS INTEGER),
    old_reference_number = src.old_reference_number,
    time_deposit_amount = CAST(REPLACE(src.time_deposit_amount, ',', '') AS DECIMAL(20,2)),
    time_deposit_account_number = src.time_deposit_account_number,
    settlement_account_number = src.settlement_account_number,
    frequency = src.frequency,
    interest_accrued_till_date = CAST(src.interest_accrued_till_date AS DECIMAL(20,2)),
    interest_at_maturity = CAST(REPLACE(src.interest_at_maturity, '''', '') AS DECIMAL(20,2)),
    branch = src.branch,
    trade_type = src.trade_type,
    funding_source = src.funding_source,
    obs_code = src.obs_code,
    sun_id = src.sun_id,
    client_name = src.client_name,
    account_official_name = src.account_official_name,
    done_time = CAST(src.done_time AS TIMESTAMP),
    update_time = CAST(src.update_time AS TIMESTAMP)
FROM deposit.stg_td_rollover src
WHERE tgt.trade_number = src.trade_number;

-- Next, insert new records
INSERT INTO deposit.new_td_rollover (
    trade_number, investment_type, start_date, maturity_date, tenor, currency,
    interest_rate, maturity_status, reference_number, old_reference_number,
    time_deposit_amount, time_deposit_account_number, settlement_account_number, frequency,
    interest_accrued_till_date, interest_at_maturity, branch, trade_type, funding_source,
    obs_code, sun_id, client_name, account_official_name, done_time, update_time
)
SELECT 
    src.trade_number, src.investment_type,
    CAST(src.start_date AS DATE), CAST(src.maturity_date AS DATE), CAST(src.tenor AS INTEGER), src.currency,
    CAST(src.interest_rate AS DECIMAL(20,2)), CAST(src.maturity_status AS VARCHAR), 
    CAST(src.reference_number AS INTEGER), src.old_reference_number,
    CAST(REPLACE(src.time_deposit_amount, ',', '') AS DECIMAL(20,2)), src.time_deposit_account_number, 
    src.settlement_account_number, src.frequency,
    CAST(src.interest_accrued_till_date AS DECIMAL(20,2)), 
    CAST(REPLACE(src.interest_at_maturity, '''', '') AS DECIMAL(20,2)), 
    src.branch, src.trade_type, src.funding_source, 
    src.obs_code, src.sun_id, src.client_name, src.account_official_name,
    CAST(src.done_time AS TIMESTAMP), CAST(src.update_time AS TIMESTAMP)
FROM deposit.stg_td_rollover src
WHERE NOT EXISTS (
    SELECT 1 FROM deposit.new_td_rollover tgt
    WHERE tgt.trade_number = src.trade_number
);

```


```
INSERT INTO deposit.new_td_rollover AS tgt (
    trade_number, investment_type, start_date, maturity_date, tenor, currency,
    interest_rate, maturity_status, reference_number, old_reference_number,
    time_deposit_amount, time_deposit_account_number, settlement_account_number, frequency,
    interest_accrued_till_date, interest_at_maturity, branch, trade_type, funding_source,
    obs_code, sun_id, client_name, account_official_name, done_time, update_time
)
SELECT 
    src.trade_number, src.investment_type,
    CAST(src.start_date AS DATE), CAST(src.maturity_date AS DATE), CAST(src.tenor AS INTEGER), src.currency,
    CAST(src.interest_rate AS DECIMAL(20,2)), CAST(src.maturity_status AS VARCHAR), 
    CAST(src.reference_number AS INTEGER), src.old_reference_number,
    CAST(REPLACE(src.time_deposit_amount, ',', '') AS DECIMAL(20,2)), src.time_deposit_account_number, 
    src.settlement_account_number, src.frequency,
    CAST(src.interest_accrued_till_date AS DECIMAL(20,2)), 
    CAST(REPLACE(src.interest_at_maturity, '''', '') AS DECIMAL(20,2)), 
    src.branch, src.trade_type, src.funding_source, 
    src.obs_code, src.sun_id, src.client_name, src.account_official_name,
    CAST(src.done_time AS TIMESTAMP), CAST(src.update_time AS TIMESTAMP)
FROM deposit.stg_td_rollover src
ON CONFLICT (trade_number) 
DO UPDATE SET
    investment_type = EXCLUDED.investment_type,
    start_date = EXCLUDED.start_date,
    maturity_date = EXCLUDED.maturity_date,
    tenor = EXCLUDED.tenor,
    currency = EXCLUDED.currency,
    interest_rate = EXCLUDED.interest_rate,
    maturity_status = EXCLUDED.maturity_status,
    reference_number = EXCLUDED.reference_number,
    old_reference_number = EXCLUDED.old_reference_number,
    time_deposit_amount = EXCLUDED.time_deposit_amount,
    time_deposit_account_number = EXCLUDED.time_deposit_account_number,
    settlement_account_number = EXCLUDED.settlement_account_number,
    frequency = EXCLUDED.frequency,
    interest_accrued_till_date = EXCLUDED.interest_accrued_till_date,
    interest_at_maturity = EXCLUDED.interest_at_maturity,
    branch = EXCLUDED.branch,
    trade_type = EXCLUDED.trade_type,
    funding_source = EXCLUDED.funding_source,
    obs_code = EXCLUDED.obs_code,
    sun_id = EXCLUDED.sun_id,
    client_name = EXCLUDED.client_name,
    account_official_name = EXCLUDED.account_official_name,
    done_time = EXCLUDED.done_time,
    update_time = EXCLUDED.update_time;



```
```
UPDATE deposit.new_td_rollover
SET investment_type = 'New Value'  -- Replace 'New Value' with the desired investment type
WHERE trade_number = 123;
```

```
-- Step 1: Delete existing records with the same trade_number
DELETE FROM deposit.new_td_rollover
WHERE trade_number IN (SELECT trade_number FROM deposit.stg_td_rollover);

-- Step 2: Insert new data from the staging table
INSERT INTO deposit.new_td_rollover (
    trade_number, investment_type, start_date, maturity_date, tenor, currency,
    interest_rate, maturity_status, reference_number, old_reference_number,
    time_deposit_amount, time_deposit_account_number, settlement_account_number, frequency,
    interest_accrued_till_date, interest_at_maturity, branch, trade_type, funding_source,
    obs_code, sun_id, client_name, account_official_name, done_time, update_time
)
SELECT 
    trade_number, investment_type,
    CAST(start_date AS DATE), CAST(maturity_date AS DATE), CAST(tenor AS INTEGER), currency,
    CAST(interest_rate AS DECIMAL(20,2)), CAST(maturity_status AS VARCHAR), 
    CAST(reference_number AS INTEGER), old_reference_number,
    CAST(REPLACE(time_deposit_amount, ',', '') AS DECIMAL(20,2)), time_deposit_account_number, 
    settlement_account_number, frequency,
    CAST(interest_accrued_till_date AS DECIMAL(20,2)), 
    CAST(REPLACE(interest_at_maturity, '''', '') AS DECIMAL(20,2)), 
    branch, trade_type, funding_source, 
    obs_code, sun_id, client_name, account_official_name,
    CAST(done_time AS TIMESTAMP), CAST(update_time AS TIMESTAMP)
FROM deposit.stg_td_rollover;

```
