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
