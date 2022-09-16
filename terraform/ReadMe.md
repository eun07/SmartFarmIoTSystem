# Terraform File Directory
```bash
├── terraform
│   ├── ReadMe.md
│   ├── alarm_lambda.tf
│   ├── kinesis.tf
│   ├── provider.tf
│   ├── trigger_lambda.tf
│   └── variables.tf

```
### provider.tf
It contains Terrafrom and AWS version.
### tirgger_lambda.tf
It contains lambda function for send sensor data to kinesis data stream.
### alarm_lambda.tf
It contains lambda function that send alarm on the discord when the value is over.
### kinesis.tf
It contains kinesis data stream, kinesis firehose, and open serach service.
### variables.tf
It contains the environment value for building infrastrcuture of architecture.
 
