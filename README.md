# Lake Formation Data Location Test Cases

# 1. Test Case 1 

## 1.1. Setup Environment

Tips: If you re-run this test case, you must revoke data location permissions and remove data location on lake formation console manually.

```bash

TEST_CASE_NUM=1
AWS_ACCOUNT_ID=366020657890

# revoke data location access permissions
cat << EOF > revoke-permissions.json
{
    "CatalogId": "${AWS_ACCOUNT_ID}",
    "Principal": {
        "DataLakePrincipalIdentifier": "arn:aws:iam::${AWS_ACCOUNT_ID}:user/lf-test-user-${TEST_CASE_NUM}"
    },
    "Resource": {
        "DataLocation": {
            "CatalogId": "${AWS_ACCOUNT_ID}",
            "ResourceArn": "arn:aws:s3:::${AWS_ACCOUNT_ID}-lf-test-db-${TEST_CASE_NUM}"
        }
    },
    "Permissions": [ "DATA_LOCATION_ACCESS" ],
    "PermissionsWithGrantOption": []
}
EOF
aws lakeformation revoke-permissions --cli-input-json file://revoke-permissions.json

# drop datbase
aws glue delete-database --name "lf_test_db_${TEST_CASE_NUM}"

# deregister data location
aws lakeformation deregister-resource --resource-arn "arn:aws:s3:::${AWS_ACCOUNT_ID}-lf-test-db-${TEST_CASE_NUM}"

aws s3 rm --recursive s3://${AWS_ACCOUNT_ID}-lf-test-db-${TEST_CASE_NUM}
aws s3 rm --recursive s3://${AWS_ACCOUNT_ID}-lf-test-user-${TEST_CASE_NUM}
aws s3 rb s3://${AWS_ACCOUNT_ID}-lf-test-db-${TEST_CASE_NUM}
aws s3 rb s3://${AWS_ACCOUNT_ID}-lf-test-user-${TEST_CASE_NUM}
aws cloudformation delete-stack --stack-name lf-test-case-${TEST_CASE_NUM}
aws cloudformation wait stack-delete-complete --stack-name lf-test-case-${TEST_CASE_NUM}
aws cloudformation deploy --stack-name lf-test-case-${TEST_CASE_NUM} --capabilities CAPABILITY_NAMED_IAM --template-file lf-test-case-${TEST_CASE_NUM}.yaml

```

```batch

set TEST_CASE_NUM=1
set AWS_ACCOUNT_ID=366020657890
aws s3 rm --recursive s3://%AWS_ACCOUNT_ID%-lf-test-db-%TEST_CASE_NUM%
aws s3 rm --recursive s3://%AWS_ACCOUNT_ID%-lf-test-user-%TEST_CASE_NUM%
aws s3 rb s3://%AWS_ACCOUNT_ID%-lf-test-db-%TEST_CASE_NUM%
aws s3 rb s3://%AWS_ACCOUNT_ID%-lf-test-user-%TEST_CASE_NUM%
aws cloudformation delete-stack --stack-name lf-test-case-%TEST_CASE_NUM%
aws cloudformation wait stack-delete-complete --stack-name lf-test-case-%TEST_CASE_NUM%
aws cloudformation deploy --stack-name lf-test-case-%TEST_CASE_NUM% --capabilities CAPABILITY_NAMED_IAM --template-file lf-test-case-%TEST_CASE_NUM%.yaml

```
## 1.2. Create Database

```sql
CREATE DATABASE IF NOT EXISTS lf_test_db_1
LOCATION 's3://366020657890-lf-test-db-1';
```

## 1.2. Create Table

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS lf_test_table (
    ID BIGINT
)
STORED AS PARQUET
LOCATION 's3://366020657890-lf-test-db-1/lf-test-table/';
```

## 1.3. Insert Data

```sql
INSERT INTO lf_test_table VALUES (1), (2), (3);
```

```bash
# list all permissions on a data location
cat << EOF > list-permissions.json
{
    "CatalogId": "${AWS_ACCOUNT_ID}",
    "Resource": {
        "DataLocation": {
            "CatalogId": "${AWS_ACCOUNT_ID}",
            "ResourceArn": "arn:aws:s3:::${AWS_ACCOUNT_ID}-lf-test-db-${TEST_CASE_NUM}"
        }
    }
}
EOF
aws lakeformation list-permissions \
    --no-paginate --no-cli-pager \
    --cli-input-json file://list-permissions.json
```