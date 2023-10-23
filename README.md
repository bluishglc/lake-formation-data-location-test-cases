# lake-formation-data-location-test-cases



# Test Case 1 

## Setup Infrastructures

```batch
aws cloudformation delete-stack --stack-name lf-test-case-1
aws cloudformation deploy --stack-name lf-test-case-1 --capabilities CAPABILITY_NAMED_IAM --template-file lf-test-case-1.yaml
```

## Create Table

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS ORDERS (
	ORD_NUM STRING, 
	ORD_AMOUNT STRING, 
	ADVANCE_AMOUNT STRING, 
	ORD_DATE DATE, 
	CUST_CODE STRING, 
	AGENT_CODE STRING, 
	ORD_DESCRIPTION STRING
)
STORED AS PARQUET
LOCATION 's3://366020657890-lf-test-db-1/orders/'
```

