# AWS CLI Cheat Sheet


## Managing Multiple AWS Profiles

NOTE Environment variables override configuration files.

https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html


Create a new profile (IAM user/region/etc.)
```sh
$ aws configure --profile <profile_name>
```

Exec using a specific profile
```sh
$ aws <command> --profile <profile_name>
```

Permanently set a specific profile via environment variable
```sh
$ export AWS_PROFILE=user1
```

Query for region
```sh
$ aws configure get region --profile default
```

Change region
```sh
$ aws configure set region us-west-2 --profile default
```

Whoami
```sh
$ aws sts get-caller-identity
```


## Regions

|||
|---------------|---------------------------|
|us-east-1      | US East (N. Virginia)     |
|us-east-2      | US East (Ohio)            |
|us-west-1      | US West (N. California)   |
|us-west-2      | US West (Oregon)          |
|ca-central-1   | Canada (Central)          |
|eu-central-1   | EU (Frankfurt)            |
|eu-west-1      | EU (Ireland)              |
|eu-west-2      | EU (London)               |
|eu-west-3      | EU (Paris)                |
|eu-north-1     | EU (Stockholm)            |
|ap-east-1      | Asia Pacific (Hong Kong)  |
|ap-northeast-1 | Asia Pacific (Tokyo)      |
|ap-northeast-2 | Asia Pacific (Seoul)      |
|ap-northeast-3 | Asia Pacific (Osaka-Local)|
|ap-southeast-1 | Asia Pacific (Singapore)  |
|ap-southeast-2 | Asia Pacific (Sydney)     |
|ap-south-1     | Asia Pacific (Mumbai)     |
|me-south-1     | Middle East (Bahrain)     |
|sa-east-1      | South America (Sao Paulo) |


## IAM

Create role using role policy document similar to this one:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

```sh
$ aws iam create-role --role-name <value> --assume-role-policy-document file://role-policy.json
```

Get role info
```sh
$ aws iam get-role --role-name <value>
```

Add access policy to a role using a policy document similar to this one:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
      "Effect": "Allow",
      "Action": [
        "s3:Get*",
        "s3:List*"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket/shared/*"
      ]
    }
  ]
}
```

```sh
$ aws iam put-role-policy --role-name <value> --policy-name <value> --policy-document file://policy.json
```

Get role policy info
```sh
$ aws iam get-role-policy --role-name <value> --policy-name <value>
```

Delete role (must delete role policies first)
```sh
$ aws iam list-roles --query Roles[*].RoleName | grep .*-executor
```

```sh
$ aws iam list-role-policies --role-name <value>
```

```sh
$ aws iam delete-role-policy --role-name <value> --policy-name <value>
```

```sh
$ aws iam delete-role --role-name <value>
```

Change username (rename)
```sh
$ aws iam update-user --user-name <value> --new-user-name <value>
```


## S3

List all buckets
```sh
$ aws s3 ls
```

Make bucket
```sh
$ aws s3 mb s3://<bucket_name>
```

Remove empty bucket
```sh
$ aws s3 rb s3://<bucket_name>
```

Remove non-empty bucket
```sh
$ aws s3 rb --force s3://<bucket_name>
```

Upload file
```sh
$ aws s3 cp <local_file> s3://<bucket_name>/path
```

Upload file w/metadata
```sh
$ aws s3 cp <local_file> s3://<bucket_name>/path --metadata '{ "author": "John Doe" }'
```

List files
```sh
$ aws s3 ls --recursive s3://<bucket_name>/path
```

Get size of a bucket
```sh
aws s3 ls --recursive --human-readable --summarize s3://<bucket_name>
```

Setup a website
```sh
$ aws s3 website s3://<bucket_name>
```

Upload the whole directory
```sh
$ aws s3 sync . s3://<bucket_name>
```

Download the whole directory
```sh
$ aws s3 sync s3://<bucket_name> .
```

Sync between bucket in different regions
```sh
$ aws s3 sync s3://my-us-west-2-bucket s3://my-us-east-1-bucket --source-region us-west-2 --region us-east-1
```

Configure mime type (e.g. text/javascript)
```sh
$ aws s3 cp s3://<bucket_name> s3://<bucket_name> --exclude * --include *.js --no-guess-mime-type --content-type text/javascript --metadata-directive REPLACE --recursive
```

Rename a bucket (create-copy-delete)
```sh
$ aws s3 mb s3://<new_bucket>
```

```sh
$ aws s3 sync s3://<old_bucket> s3://<new_bucket>
```

```sh
$ aws s3 rb --force s3://<old_bucket>
```

Create a pre-signed url that's valid for 1 hour
```sh
$ aws s3 presign s3://<bucket_name>/<file_name> --expires-in 3600
```


## CloudFront

Get all distributions
```sh
$ aws cloudfront list-distributions
```

TODO Review start >>>>>>>>
Create a distribution
```sh
$ aws cloudfront create-distribution --origin-domain-name my-bucket.s3.amazonaws.com --default-root-object index.html
```

Force users to access your content using a CloudFront URL instead of the Amazon S3 URL
```sh
$ aws create-cloud-front-origin-access-identity --cloud-front-origin-access-identity-config <CallerReference=string,Comment=string>
   [--cli-input-json <value>]
   [--generate-cli-skeleton <value>]
```
TODO Review end <<<<<<<<

Invalidate distribution path ("/*" for the whole distribution)
```sh
$ aws cloudfront create-invalidation --distribution-id <value> --paths /*
```

Wait for distribution to finish deploying (including behaviors)
```sh
$ aws cloudfront wait distribution-deployed --id <value>
```

Wait for cache invalidation to complete
```sh
$ aws cloudfront wait invalidation-completed --distribution-id <value> --id <invalidation_id>
```

Delete a distribution
NOTE Distribution must be disabled first, either manually from the console or via CLI `$ aws cloudfront update-distribution`. The etag from this operation is then used to delete a distribution.
TODO Disabling a distribution from CLI is cumbersome. Consider looking into AWS CDK or AWS SDK to codify this.
```sh
$ aws cloudfront delete-distribution --id <value> --if-match <etag>
```


## Logs

Get the log group names
```sh
$ aws logs describe-log-groups
```

Show everything
```sh
$ aws logs filter-log-events --log-group-name <value> --no-paginate
```

Find out which keys are used, so we can query on them when outputing in text format
```sh
$ aws logs get-log-group-fields --log-group-name <value>
```

Query for specific keys and output in text format
```sh
$ aws logs filter-log-events --log-group-name <value> --query events[*].[timestamp,message] --output text --no-paginate
```

Filter errors
```sh
$ aws logs filter-log-events --log-group-name <value> --query events[*].[message] --filter "error" --output text --no-paginate
```

Delete log group
```sh
$ aws logs delete-log-group --log-group-name <value>
```


## Awslogs (third-party lib)

See https://github.com/jorgebastida/awslogs

List existing log groups
```sh
$ awslogs groups
```

List existing log groups in a specific region
```sh
$ awslogs groups --aws-region <region>
```

List existing streams
```sh
$ awslogs streams <group-name>
```

Get events from any stream in the /var/logs/syslog group generated in the last day
```sh
$ awslogs get <group-name> ALL -s1d
```

Tail logs via "awslogs" third party lib (https://github.com/jorgebastida/awslogs)
```sh
$ awslogs get <group-name> ALL --watch
```

```sh
$ awslogs get /var/log/syslog ip-10-1.* --start='2d ago' --end='1h ago' | grep ERROR
```

```sh
$ awslogs get my_lambda_group --filter-pattern="[r=REPORT,...]"
```

```sh
$ awslogs get my_lambda_group --query=message
```


## Apilogs (third-party lib)

https://github.com/rpgreen/apilogs

Stream logs for your Serverless API
```sh
$ apilogs get --api-id xyz123 --stage prod --watch
```

Grep for errors one hour ago using credentials from AWS CLI profile "myprofile"
```sh
$ apilogs get --api-id xyz123 --stage test2 --profile myprofile --aws-region us-east-1 --start='2h ago' --end='1h ago' | grep "ERROR"
```


## Lambda

- Free usage tier: 
    - 1M requests/mo and
    - 400,000 GB-seconds of compute time/mo.
- Billing granularity: 1 MS
- Maximum concurrent executions: 1000
- Function and layer storage: 75 GB
- Elastic network interfaces per VPC: 250
- Function memory allocation: 128 MB to 3008 MB (in 64 MB increments)
- Function timeout: 900 seconds (15 minutes)
- Function environment variables: 4 KB
- Function resource-based policy: 20 KB
- Function layers: 5 layers
- Function burst concurrency: 500 - 3000 (varies per region)
- Invocation frequency (requests per second):
    - 10 x concurrent executions
    - Unlimited asynchronous executions
- Invocation payload (request and response):
    - 6 MB (synchronous)
    - 256 KB (asynchronous)
- Deployment package size:
    - 50 MB (zipped, for direct upload)
    - 250 MB (unzipped, including layers)
    - 3 MB (console editor)
- Test events (console editor): 10
- /tmp directory storage: 512 MB
- File descriptors: 1024
- Execution processes/threads: 1024
- To enable functions to scale without fluctuations in latency, use provisioned concurrency. For functions that take a long time to initialize, or require extremely low latency for all invocations, provisioned concurrency enables you to pre-initialize instances of your function and keep them running at all times. Lambda integrates with Application Auto Scaling to support autoscaling for provisioned concurrency based on utilization.

Get all function names
```sh
$ aws lambda list-functions --query=Functions[*].[FunctionName,Timeout,MemorySize,LastModified] --output table
```

Get function aliases ($LATEST/dev/prod/etc.)
```sh
$ aws lambda list-aliases --function-name <value>
```

Get function info (configuration & code)
```sh
$ aws lambda get-function --function-name <value>
```

Get function configuration (including environment varibales)
```sh
$ aws lambda get-function-configuration --function-name <value>
```

Create a new function
```sh
$ aws lambda create-function
    --function-name <value>
    --runtime nodejs
    --role <role_arn>
    --handler index.handler
    --zip-file fileb://<deployment_package>.zip
    --timeout 10
    --region <region>
```

Update function - see help
```sh
$ aws lambda update-function-code help
$ aws lambda update-function-configuration help
```

List function environment variables
```sh
$ aws lambda list-functions --query=Functions[*].[FunctionName,Timeout,MemorySize,LastModified] --output table
```

Update function environment varibales.
NOTE This API only supports unqualified ARN orLATEST.
```sh
$ aws lambda update-function-configuration --function-name <value> --environment Variables={KeyName1=string,KeyName2=string}
```

Invoke a function
```sh
$ aws lambda invoke
    --invocation-type RequestResponse
    --function-name <value>[:alias]
    --log-type Tail
    --payload '{"key1":"Lambda","key2":"is","key3":"awesome!"}'
    output.txt
```

Example
```sh
$ aws lambda invoke --invocation-type RequestResponse --function-name stage-test:prod output.txt
```

```sh
$ aws lambda invoke --function-name <value> out --log-type Tail
```

Delete a function (all versions)
```sh
$ aws lambda delete-function --function-name <value>
```

List function versions
```sh
$ aws lambda list-versions-by-function --function-name <value>
```

Delete specific function version
```sh
$ aws lambda delete-function --function-name <name:version>
```
or
```sh
$ aws lambda delete-function --function-name <name> --qualifier <version>
```

Add invoke permission (See "AWS Service Principals" bellow for a list of AWS principal ids)
```sh
$ aws lambda add-permission
    --function-name <value>
    --statement-id <custom_id>
    --principal <aws_service_principal>
    --source-arn <value>
    --action lambda:InvokeFunction
```

Show all available permissions for a function
```sh
$ aws lambda get-policy --function-name <value>
```

Remove permission
```sh
$ aws lambda remove-permission --function-name <value> --statement-id <custom_id>
```


## Lambda@Edge

See
- https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-requirements-limits.html
- https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-generating-http-responses-in-requests.html

Response object

```javascript
const response = {
    body: 'content',
    bodyEncoding: 'text' | 'base64',
    headers: {
        'header name in lowercase': [{
            key: 'header name in standard case',
            value: 'header value'
         }],
    },
    status: 'HTTP status code',
    statusDescription: 'status description'
};
callback(null, response);
```


## API Gateway

See https://docs.aws.amazon.com/apigateway/latest/developerguide/create-api-using-awscli.html

Create API
```sh
$ aws apigateway create-rest-api --name 'Simple PetStore (AWS CLI)'
```

Get parent (root) resource id of the REST API
```sh
$ aws apigateway get-resources --rest-api-id <value>
```

Append child resource to the parent resource
```sh
$ aws apigateway create-resource --rest-api-id <api_id> --parent-id <parent_api_id> --path-part pets
```

Append a path param to the parent resource
```sh
$ aws apigateway create-resource --rest-api-id <api_id> --parent-id <parent_api_id> --path-part '{petId}'
```

Add a GET HTTP method on the /pets resource
```sh
$ aws apigateway put-method --rest-api-id <api_id> --resource-id <res_id> --http-method GET
```


## Scheduled Events (CloudWatch)

Schedule events w/rate
```sh
$ aws events put-rule --schedule-expression "rate(1 minute)" --name <value>
```

```sh
$ aws events put-rule --schedule-expression "rate(5 minutes)" --name <value>
```

```sh
$ aws events put-rule --schedule-expression "rate(1 hour)" --name <value>
```

```sh
$ aws events put-rule --schedule-expression "rate(1 day)" --name <value>
```

Schedule events w/cron
```sh
$ aws events put-rule --schedule-expression "cron(0 12 * * ? *)" --name <value>
```

```sh
$ aws events put-rule --schedule-expression "cron(5,35 14 * * ? *)" --name <value>
```

```sh
$ aws events put-rule --schedule-expression "cron(15 10 ? * 6L 2002-2005)" --name <value>
```

Delete scheduled event
```sh
$aws events delete-rule --name <value>
```

Disable scheduled event
```sh
$aws events disable-rule --name <value>
```

Enable scheduled event
```sh
$aws events enable-rule --name <value>
```


## SES

See
- https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html
- https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-domain-procedure.html

Verify domain (add output to a TXT DNS record)
```sh
$ aws ses verify-domain-identity --domain <value>
```

Verify a real, existing email address (will receive a verification email)
```sh
$ aws ses verify-email-identity --email-address <value>
```


## SNS

Create a topic
```sh
$ aws sns create-topic --name <value>
```

List topics
```sh
$ aws sns list-topics
```

Delete topic
```sh
$ aws sns delete-topic --topic-arn <value>
```

Create subscribtion
```sh
$ aws sns subscribe --topic-arn <value> --protocol <value> --notification-endpoint <value>
```

| protocol   | endpoint      |
|------------|---------------|
| http       | http://...    |
| https      | https://...   |
| email      | email address |
| email-json | email address |
| sms        | phone number  |
| sqs        | SQS ARN       |
| application| mobile app ARN|
| lambda     | lambda ARN    |

Example
```sh
$ aws sns subscribe --topic-arn arn:aws:sns:us-east-1:0123456789012:my-topic --protocol email --notification-endpoint my-email@example.com
```

```sh
$ aws sns subscribe --topic-arn arn:aws:sns:us-east-1:0123456789012:my-topic --protocol lambda --notification-endpoint arn:aws:lambda:us-east-1:0123456789012:function:my-lambda
```

Confirm subscribtion
"token" - Short-lived token sent to an endpoint during the "Subscribe" action.
```sh
$ aws sns confirm-subscription --topic-arn <value> --token <value>
```

List subscribtions
```sh
$ aws sns list-subscriptions-by-topic --topic-arn <value>
```

Delete subscribtion
```sh
$ aws sns unsubscribe --subscription-arn <value>
```


## SQS

Get all queue urls
```sh
$ aws sqs list-queues
```

Get queue attributes, including approximate message count
```sh
$ aws sqs get-queue-attributes --attribute-names All --queue-url <value>
```

Get up to 10 messages from a queue (default 1 message)
```sh
$ aws sqs receive-message --attribute-names All --queue-url <value>
$ aws sqs receive-message --attribute-names All --max-number-of-messages 10 --queue-url <value>
```

Remove all messages from a queue
```sh
$ aws sqs purge-queue --queue-url <value>
```

Send one message to a queue
```sh
$ aws sqs send-message --queue-url <value> --message-body <value> --message-attributes file://send-message.json
```


## Cognito

Create user pool
```sh
$ aws cognito-idp create-user-pool --pool-name <value> --username-attributes email
```

Create a client for the user pool
```sh
$ aws cognito-idp create-user-pool-client --user-pool-id <value> --client-name <value> --no-generate-secret
```

Create identity pool
NOTE It will automatically create two roles:
    1. authenticated role
    2. unauthenticated role
```sh
$ aws cognito-identity create-identity-pool --identity-pool-name <value> --allow-unauthenticated-identities --supported-login-providers <value> --cognito-identity-providers <value>
```

TODO Create authenticated and unauthenticated roles

Add roles to identity pool (using role ARNs)
```sh
$ aws cognito-identity set-identity-pool-roles --identity-pool-id <value> --roles authenticated=<arn_auth>,unauthenticated=<arn_unauth>
```

Customize Hosted UI (register/login)
https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-app-ui-customization.html
```sh
$ aws cognito-idp set-ui-customization --user-pool-id <your-user-pool-id> --client-id <your-app-client-id> --image-file <path-to-logo-image-file> --css ".label-customizable{ color: <color>;}"
```

Add Cognito Lambda trigger (callback)
```sh
$ aws cognito-idp update-user-pool
    --user-pool-id <value>
    --lambda-config <cognito_trigger_name>=<lambda_name_or_arn>[:alias]
```


## DynamoDB

- Reference:
    - Example policies: 
        - https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html#policy_library_DynamoDB
    - Dynamo client:    
        - https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html
        - https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#constructor-property
    - Sort key example:
        - https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-sort-keys.html
    - Dynamo localhost:
        - https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html
    - Reserved words:
        - https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ReservedWords.html
    - Limits:
        - https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Limits.html
    - Transactions:
        - https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html
- Terminology:
  | SQL      | NoSQL      |
  |----------|------------|
  | Table    | Collection |
  | Record   | Item       |
  | Column   | Property   |
- Billing: The AWS Free Tier includes 25 WCUs and 25 RCUs (reducing your monthly bill by14.04).
- CPU optimized (CPU is expensive) - not storage optimized (storage is cheap)
- There are **no** joins because joins are expensive and thus relational databases don't scale well
- Supports nested attributes up to 32 levels deep
- Data types:
    - Scalars: Number (N), String (S), Boolean (BOOL), null (NULL), Binary (B)
    - Binary = Buffer, File, Blob, ArrayBuffer, DataView, and JavaScript typed arrays
    - Documents: Array (L), Object (M)
    - Sets: String Set (SS), Number Set (NS), Binary Set (BS)
- Key types:
    - Partition key, aka "hash key" - used do distribute data across DynamoDB partitions
    - Sort key, aka "range key":
        - Gathers related information together where it can be queried efficiently using range queries (see bellow)
        - Lets you define hierarchical (one-to-many) relationships. e.g. [country]#[region]#[state]#[county]#[city]
- Consistency
    - Eventual consistency - read data from any replica partition
    - Strong consistency - read data only from the master partition (x2 as expensive$, and may be throttled)
- Secondary index:
    - Global (GSI):
        - Has partition key and sort key different from those on the table
        - GSIs are eventually consistent
    - Local (LSI):
        - Has the same partition key as the table, but a different sort key
        - LSIs are strongly consistent
        - Created when the table is created and cannot be removed
- Limits:
    - 1000 WCUs/sec per-partition
    - 3000 RCUs/sec per-partition
    - 20 GSIs/table (default - can be increased)
    - 5 LSIs/table
    - 100 projected secondary index attributes
    - Transaction:
        - 25 items per-transaction
        - 4mb per-transaction
    - Max partition key length: 2048 bytes
    - Max sort key length: 1024 bytes
    - 10gb partition size
    - 400kb string size (strings are utf-8 encoded)
    - 400kb binary size
    - 400kb item size
    - 32 levels nested attributes
    - 4kb expression length
    - 300 operators or functions in an "UpdateExpression"
    - Batch:
        - 100 items/16mb retrieved in a single "BatchGetItem" operation
        - 25 "PutItem" or "DeleteItem" requests in a single "BatchWriteItem" operation
    - Query: 1mb result set. (Use "LastEvaluatedKey" from the query response to retrieve more results)
    - Scan: 1mb result set. (Use "LastEvaluatedKey" from the query response to retrieve more results)
- Stream is the changelog of the table. Good for stored-procedures type operations and also for performing computed aggregations.
- Stream records have a lifetime of 24 hours; after that, they are removed automatically
- Transactions: ACID transactions - Atomicity, Consistency, Isolation, Durability
- To fully use all the throughput capacity that is provisioned for a table, you must distribute your workload across your partition key values
- Relationships:
    - 1:1, one-to-one or key-value
        - Model using a table or GSI with a partition key
        - User "GetItem" or "BatchGetItem" API
    - 1:N, one-to-many
        - Model using ta table or GSI with partition and sort key
        - Use "Query" API to get multiple items
    - M:N, many-to-many
        - Model using a table and inverted GSI (with partition and sort key elements switched)
        - Use "Query" API to get multiple items
- Design patterns:
    - Hierarchical (composite) sort key: [country]#[region]#[state]#[county]#[city]
    - Sparse index: Conditionally populating the SK, i.e. not all items have a value for a specific attribute (e.g. "STATUS_WARN")
    - Tables & indexes partitioning
    - GSI overloading: Storing different types of data in the same attribute
    - GSI write sharding:
        - Add a random number to the partition key values (e.g. GUID)
            - Pro: Improves write throughput
            - Con: Difficult to read a specific item
        - Add a calculated hash suffix, based on something that you are querying on
    - Adjacency list: Representing many-to-many relationships
    - Materialized aggregations
    - Write sharding: Adding "salt" to the partition key for better data distribution
        - Random prefix/suffix:
            - _RAND(0..N) where N is the number of shards (partitions)
            - When you don't need per-item access
            - Query all shards and merge results
        - Calculated prefix/suffix:
            - _HASH(OrderStatus) % N
            - When you need per-item access
            - Calculate salt from a known value
- Query / Scan filter:
    - BEGINS_WITH
    - BETWEEN
    - CONTAINS
    - EQ
    - GE
    - GT
    - IN
    - LE
    - LT
    - NE
    - NOT_CONTAINS
    - NOT_NULL
    - NULL
- Partition key operators:
    - =
- Sort key operators:
    - =
    - <
    - <=
    - >
    - >=
    - A BETWEEN B AND C
    - BEGINS_WITH(a, substr)
- Save operations ("ReturnValues" key options):
    - NONE
    - ALL_OLD
    - UPDATED_OLD
    - ALL_NEW
    - UPDATED_NEW

List tables
```sh
$ aws dynamodb list-tables
```

List local DynamoDB tables
```sh
$ aws dynamodb list-tables --endpoint-url http://localhost:8000
```

Show table properties
```sh
$ aws dynamodb describe-table --table-name <value>
```

Provision a new table with a composite primary key, i.e. a partition key (hash) and a sort key (range)
```sh
$ aws dynamodb create-table
    --table-name <value>
    --attribute-definitions AttributeName=<value>,AttributeType=S AttributeName=<value>,AttributeType=S
    --key-schema AttributeName=<value>,KeyType=HASH AttributeName=<value>,KeyType=RANGE
    --billing-mode PROVISIONED
    --provisioned-throughput ReadCapacityUnits=10,WriteCapacityUnits=5
```

Change billing mode to "on-demand"
```sh
$ aws dynamodb update-table
    --table-name <value>
    --billing-mode PAY_PER_REQUEST
```

Create global secondary index (consult documentation)
```sh
$ aws dynamodb update-table
    --table-name Music
    --attribute-definitions AttributeName=AlbumTitle,AttributeType=S
    --global-secondary-index-updates "[{\"Create\":{\"IndexName\":\"AlbumTitle-index\",\"KeySchema\":[{\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"HASH\"}],\"ProvisionedThroughput\":{\"ReadCapacityUnits\":10,\"WriteCapacityUnits\":5},\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```

Setup table streams
```sh
$ aws dynamodb update-table
    --table-name <value>
    --stream-specification StreamEnabled=true,StreamViewType=KEYS_ONLY|NEW_IMAGE|OLD_IMAGE|NEW_AND_OLD_IMAGES
```

Delete table
```sh
$ aws dynamodb delete-table --table-name <value>
```

Insert an item
```sh
$ aws dynamodb put-item --table-name <value> --item "<value>"
```

Insert an item from "data.json" file
```sh
$ aws dynamodb put-item --table-name <value> --item file://data.json
```

Populate table from input file
```sh
$ aws dynamodb batch-write-item --request-items file://data.json
```

Get all data from a table
```sh
$ aws dynamodb scan --table-name <value>
```

Count data in a table
```sh
$ aws dynamodb scan --table-name <value> --select "COUNT"
```

Get item from table
```sh
$ aws dynamodb get-item --table-name <value> --key "<value>"
```

Search data in a table
```sh
$ aws dynamodb query
    --table-name <value>
    --key-condition-expression "myAttr=:val"
    --expression-attribute-values '{":val": <value>}'
```

Search data by global secondary index
```sh
$ aws dynamodb query
    --table-name <value>
    --index-name <value>
    --key-condition-expression "myAttr=:val"
    --expression-attribute-values '{":val":<value>}'
```

Update data
```sh
$ aws dynamodb update-item
    --table-name <value>
    --key "<value>"
    --update-expression "SET myAttr=:neval"
    --expression-attribute-values '{":neval":<value>}'
    --return-values ALL_NEW
```

Delete item
```sh
$ aws dynamodb delete-item --table-name <value> --key "<value>"
```


## ECS

List clusters
```sh
$ aws ecs list-clusters
```

List container instances
```sh
$ aws ecs list-container-instances --cluster <value>
```

List services
```sh
$ aws ecs list-services --cluster <value>
```

List tasks
```sh
$ aws ecs list-tasks --cluster <value>
```

Deregister task definition (it is currently impossibe to delete task definitions)
```sh
$ aws ecs deregister-task-definition --task-definition <value>
```

Delete service
```sh
$ aws ecs delete-service --cluster <value> --service <value>
```

Delete cluster
```sh
$ aws ecs delete-cluster --cluster <value>
```


## CloudFormation

List stacks
```sh
$ aws cloudformation describe-stacks --query Stacks[*].[StackName,Outputs]
```

Get template
```sh
$ aws cloudformation get-template --stack-name <value>
```

List resources in a stack
```sh
$ aws cloudformation list-stack-resources --stack-name <value>
```

Create stack from local file
```sh
$ aws cloudformation create-stack
    --stack-name <name>
    --capabilities CAPABILITY_IAM
    --template-body file://mytemplate.yaml
```

Delete stack
```sh
$ aws cloudformation delete-stack --stack-name <value>
```

## SSM (AWS System Manager)

### Parameter Store

Create encrypted parameter
```sh
aws ssm put-parameter
    --name <param_name>
    --value <param_value>
    --type "SecureString"
    --key-id <kms_key_uuid>
    --overwrite
```

List parameters
```sh
aws ssm describe-parameters
```

Decrypt parameter
```sh
aws ssm get-parameter --name <param_name> --with-decryption
```

Delete parameter
```sh
aws ssm delete-parameter --name <param_name>
```
