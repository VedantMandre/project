```
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.4.xsd">

    <changeSet id="create-beneficiary-table" author="developer">
        <sql>
CREATE TABLE IF NOT EXISTS payments.beneficiary (
    beneficiary_id BIGINT NOT NULL PRIMARY KEY,
    sun_id VARCHAR UNIQUE NOT NULL,
    payment_id VARCHAR,
    branch_id VARCHAR,
    name VARCHAR NOT NULL,
    account_number VARCHAR NOT NULL,
    organization VARCHAR,
    department VARCHAR,
    street_name VARCHAR,
    building_number VARCHAR,
    building_name VARCHAR,
    floor VARCHAR,
    po_box VARCHAR,
    room VARCHAR,
    postal_code VARCHAR,
    city VARCHAR NOT NULL,
    town_location VARCHAR,
    district VARCHAR,
    state_province VARCHAR,
    country VARCHAR NOT NULL,
    approval_status VARCHAR DEFAULT 'Pending',
    is_active BOOLEAN DEFAULT TRUE,
    created_by VARCHAR,
    created_at TIMESTAMP DEFAULT now(),
    updated_by VARCHAR,
    updated_at TIMESTAMP DEFAULT now(),
    record_status VARCHAR NOT NULL CHECK (record_status IN ('Draft', 'Complete')),
    CONSTRAINT pk_payment_beneficiaries PRIMARY KEY (beneficiary_id) 
);
        </sql>
    </changeSet>

    <changeSet id="create-beneficiary-bank-table" author="developer">
        <sql>
CREATE TABLE IF NOT EXISTS payments.beneficiary_bank (
    bank_id VARCHAR NOT NULL,
    beneficiary_id BIGINT NOT NULL,
    identifier VARCHAR NOT NULL,
    bank_name VARCHAR NOT NULL,
    bank_department VARCHAR,
    bank_sub_department VARCHAR,
    bank_street_name VARCHAR,
    bank_building_number VARCHAR,
    bank_building_name VARCHAR,
    bank_floor VARCHAR,
    bank_po_box VARCHAR,
    bank_room VARCHAR,
    bank_postal_code VARCHAR,
    bank_city VARCHAR NOT NULL,
    bank_town_location VARCHAR,
    bank_district_name VARCHAR,
    bank_state_province VARCHAR,
    bank_country VARCHAR NOT NULL,
    approval_status VARCHAR DEFAULT 'Pending',
    is_active BOOLEAN DEFAULT TRUE,
    created_by VARCHAR,
    created_at TIMESTAMP DEFAULT now(),
    updated_by VARCHAR,
    updated_at TIMESTAMP DEFAULT now(),
    record_status VARCHAR NOT NULL CHECK (record_status IN ('Draft', 'Complete')),
    CONSTRAINT pk_payments_beneficiary_bank PRIMARY KEY (bank_id),
    CONSTRAINT fk_payments_beneficiary_bank_beneficiary_id FOREIGN KEY (beneficiary_id) REFERENCES payments.beneficiary(beneficiary_id)
);
        </sql>
    </changeSet>
</databaseChangeLog>
```
```
<changeSet id="create-beneficiary-table" author="developer">
    <sql>
CREATE TABLE IF NOT EXISTS payments.beneficiary (
    beneficiary_id BIGINT NOT NULL PRIMARY KEY,
    sun_id VARCHAR UNIQUE NOT NULL,
    payment_id VARCHAR,
    branch_id VARCHAR,
    name VARCHAR NOT NULL,
    account_number VARCHAR NOT NULL,
    organization VARCHAR,
    department VARCHAR,
    street_name VARCHAR,
    building_number VARCHAR,
    building_name VARCHAR,
    floor VARCHAR,
    po_box VARCHAR,
    room VARCHAR,
    postal_code VARCHAR,
    city VARCHAR NOT NULL,
    town_location VARCHAR,
    district VARCHAR,
    state_province VARCHAR,
    country VARCHAR NOT NULL,
    approval_status VARCHAR DEFAULT 'Pending',
    is_active BOOLEAN DEFAULT TRUE,
    created_by VARCHAR,
    created_at TIMESTAMP DEFAULT now(),
    updated_by VARCHAR,
    updated_at TIMESTAMP DEFAULT now(),
    record_status VARCHAR NOT NULL CHECK (record_status IN ('Draft', 'Complete')),
    CONSTRAINT pk_payment_beneficiaries PRIMARY KEY (beneficiary_id) 
);
    </sql>
</changeSet>
```
```
<changeSet id="create-beneficiary-bank-table" author="developer">
    <sql>
CREATE TABLE IF NOT EXISTS payments.beneficiary_bank (
    bank_id VARCHAR NOT NULL,
    beneficiary_id BIGINT NOT NULL,
    identifier VARCHAR NOT NULL,
    bank_name VARCHAR NOT NULL,
    bank_department VARCHAR,
    bank_sub_department VARCHAR,
    bank_street_name VARCHAR,
    bank_building_number VARCHAR,
    bank_building_name VARCHAR,
    bank_floor VARCHAR,
    bank_po_box VARCHAR,
    bank_room VARCHAR,
    bank_postal_code VARCHAR,
    bank_city VARCHAR NOT NULL,
    bank_town_location VARCHAR,
    bank_district_name VARCHAR,
    bank_state_province VARCHAR,
    bank_country VARCHAR NOT NULL,
    approval_status VARCHAR DEFAULT 'Pending',
    is_active BOOLEAN DEFAULT TRUE,
    created_by VARCHAR,
    created_at TIMESTAMP DEFAULT now(),
    updated_by VARCHAR,
    updated_at TIMESTAMP DEFAULT now(),
    record_status VARCHAR NOT NULL CHECK (record_status IN ('Draft', 'Complete')),
    CONSTRAINT pk_payment_beneficiary_bank PRIMARY KEY (bank_id), 
    CONSTRAINT fk_payment_beneficiary_bank_beneficiary_id 
         FOREIGN KEY (beneficiary_id) 
         REFERENCES payment.beneficiaries(beneficiary_id)
    </sql>
</changeSet>
```
```
CREATE OR REPLACE PROCEDURE deposit.sync_time_deposit_rollover()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Step 1: Debugging - Check which old_reference_numbers already exist
    RAISE NOTICE 'Checking existing references...';
    
    FOR rec IN 
        SELECT otd.old_reference_number, tdr.reference_number
        FROM deposit.test_recon_obs_time_deposit_data otd
        LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
        ON otd.old_reference_number = tdr.reference_number
        WHERE otd.old_reference_number IS NOT NULL
    LOOP
        RAISE NOTICE 'Existing: old_reference_number = %, reference_number = %', 
                     rec.old_reference_number, rec.reference_number;
    END LOOP;

    -- Step 2: Process records - Insert new rollovers and update existing matches
    RAISE NOTICE 'Processing rolled-over time deposits...';
    
    WITH rolled_over_data AS (
        SELECT 
            otd.trade_number, 
            otd.old_reference_number AS reference_number,  
            otd.time_deposit_amount AS principal_amount, 
            otd.maturity_date, 
            otd.currency AS currency_code, 
            otd.interest_accrued_till_date AS accrued_interest, 
            otd.interest_at_maturity AS interest_amount, 
            otd.branch AS branch_code, 
            otd.funding_source, 
            otd.obs_code AS obs_number, 
            otd.time_deposit_account_number AS account_number, 
            otd.settlement_account_number AS settlement_account, 
            otd.maturity_status, 
            'Finalized' AS status
        FROM deposit.test_recon_obs_time_deposit_data otd
        WHERE otd.old_reference_number IS NOT NULL
    )
    
    -- Insert new rollovers (where old_reference_number doesn't exist in target)
    INSERT INTO deposit.test_recon_time_deposit_rollover (
        trade_number, reference_number, principal_amount, 
        maturity_date, currency_code, accrued_interest, 
        interest_amount, branch_code, funding_source, 
        obs_number, account_number, settlement_account, 
        maturity_status, status
    )
    SELECT 
        rod.trade_number, rod.reference_number, rod.principal_amount, 
        rod.maturity_date, rod.currency_code, rod.accrued_interest, 
        rod.interest_amount, rod.branch_code, rod.funding_source, 
        rod.obs_number, rod.account_number, rod.settlement_account, 
        rod.maturity_status, rod.status
    FROM rolled_over_data rod
    WHERE NOT EXISTS (
        SELECT 1 
        FROM deposit.test_recon_time_deposit_rollover tdr
        WHERE tdr.reference_number = rod.reference_number
    )
    ON CONFLICT (reference_number) DO NOTHING;  -- Shouldn't happen due to WHERE NOT EXISTS

    -- Step 3: Update existing records where old_reference_number matches reference_number
    RAISE NOTICE 'Updating existing matched records...';

    UPDATE deposit.test_recon_time_deposit_rollover tdr
    SET 
        trade_number = rod.trade_number,
        principal_amount = rod.principal_amount,
        maturity_date = rod.maturity_date,
        currency_code = rod.currency_code,
        accrued_interest = rod.accrued_interest,
        interest_amount = rod.interest_amount,
        branch_code = rod.branch_code,
        funding_source = rod.funding_source,
        obs_number = rod.obs_number,
        account_number = rod.account_number,
        settlement_account = rod.settlement_account,
        maturity_status = rod.maturity_status,
        status = 'Finalized'
    FROM rolled_over_data rod
    WHERE tdr.reference_number = rod.reference_number;

    -- Confirmation Message
    RAISE NOTICE 'Sync completed successfully!';
END;
$$;
```

```
SELECT 
    otd.old_reference_number,
    tdr.reference_number,
    CASE 
        WHEN tdr.reference_number IS NULL THEN 'No match in rollover table'
        ELSE 'Match found in rollover table'
    END AS match_status,
    COUNT(*) OVER () AS total_rows
FROM deposit.test_recon_obs_time_deposit_data otd
LEFT JOIN deposit.test_recon_time_deposit_rollover tdr
    ON otd.old_reference_number = tdr.reference_number
WHERE otd.old_reference_number IS NOT NULL
ORDER BY otd.old_reference_number;
```
