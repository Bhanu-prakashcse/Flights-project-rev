## **Bronze Layer – Raw Data Ingestion (Snowflake + AWS S3 + Snowpipe)**

This layer stores the raw, unmodified data exactly as received from the source systems.
Bronze is the source-of-truth in the Medallion Architecture and enables reproducibility, auditing, and reprocessing.

## **Objective**

To ingest the aviation datasets (Airlines, Airports, Historical Flights, Latest Flights) into Snowflake in raw format using:

AWS S3 (Landing Zone)

Snowflake External Stage

Snowpipe (Auto-ingestion)

Bronze Raw Tables in Snowflake

No cleaning or transformation is applied at this stage.

## **Datasets Used**

| Dataset            | Source                               | Bronze Table                |
| ------------------ | ------------------------------------ | --------------------------- |
| airlines.csv       | Static batch file                    | `bronze_airlines`           |
| airports.csv       | Static batch file                    | `bronze_airports`           |
| flights_sample.csv | Historical flight data               | `bronze_flights_historical` |
| flights_latest.csv | Latest streaming/near-real-time data | `bronze_flights_latest`     |


## **1. AWS S3 Bucket Setup**
A single S3 bucket is used as the raw landing zone.

aviation-data-bucket-2025/
    └── raw/
         ├── airlines/
         ├── airports/
         ├── flights_historical/
         └── flights_latest/
The four CSV datasets are uploaded into their respective folders.

## **2. Snowflake Storage Integration**
The integration allows Snowflake to read files from the S3 raw folder.

## **3. External Stage Creation**
A single stage pointing to the raw S3 directory:
CREATE STAGE aviation_raw_stage
  URL = 's3://aviation-data-bucket-2025/raw/'
  STORAGE_INTEGRATION = s3_int
  FILE_FORMAT = (TYPE = CSV ...);

## **4. Bronze Raw Tables**
Four raw tables created with no transformations and schema matching exactly the CSV structure.

bronze_airlines

bronze_airports

bronze_flights_historical

bronze_flights_latest

These tables store the raw CSV rows as-is.

## **5. Snowpipe (Continuous Auto-Ingestion)**
A dedicated Snowpipe is created for each dataset to load data automatically into the Bronze tables:
| Pipe Name                 | Source Folder              | Target Table                |
| ------------------------- | -------------------------- | --------------------------- |
| `pipe_airlines`           | `/raw/airlines/`           | `bronze_airlines`           |
| `pipe_airports`           | `/raw/airports/`           | `bronze_airports`           |
| `pipe_flights_historical` | `/raw/flights_historical/` | `bronze_flights_historical` |
| `pipe_flights_latest`     | `/raw/flights_latest/`     | `bronze_flights_latest`     |
Pipes watch their S3 folders and load new files automatically.

## **6. Verification**

LIST @aviation_raw_stage; to confirm S3 connectivity

SELECT COUNT(*) FROM bronze_*; to confirm the data loaded

SHOW PIPES; to check pipe status

Once counts were validated, the Bronze layer was successfully completed.

## **Outcome**

The Bronze Layer now contains:

Fully ingested raw flight data

Automated ingestion using Snowpipe

Schema-aligned raw tables for all datasets

A reliable foundation for Silver layer (cleaning & standardization)
