# Build Data Pipeline of Smart Farm IoT System with monitoring 

# Overview
## Scenario
>Collect lots of data from sensor that installed on the IoT Device, based on these data send streaming data to log storage through data pipeline. These data can be monitored and visualized based on timeline.
## Requirements
1. Farmer can monitor real time data from smart farm sensor. <br>
2. The logs that generated from application should be delivered on real-time through data pipeline on the log storage.<br>
3. Logs can be visualized based on time-series.<br>
4. All of the services should be formed by serverless resource.
## Resource
### AWS
- API Gateway, Lambda, Kinesis Data Stream, Kinesis Firehose, Open Search Service, S3, Open Search dashboard
### IaC
- Terraform
## Architecture
![image](https://user-images.githubusercontent.com/55527712/183580369-8380c9fa-651c-4b16-ac1e-3e759c647074.png)

### Step 1: Data Production
***✔ API Gateway to Lambda***

When **Lambda** is triggered by **API Gateway** , **Lambda** put the data on the **Kinesis Data Stream** record that collected from sensor.
 

### Step 2: Data Pipeline
***✔ Kiensis Data Stream to Kiensis Firehose***

The data that is streaming on the **Kinesis Data Stream** send the streaming data to **Kinesis Firehose**. 
The data on the **Kinesis Firehose** arrived at the **Open Search Service** and **S3** for back up.

### Step 3: Storing on S3 to lambda

***✔ S3 to Lambda***
 
 When the data are stored at **S3** and if the speicific data value excceds the threshold,
  **Lambda** would send the discord webhook alarm.


![Image](https://user-images.githubusercontent.com/55527712/183611090-fd18ea7b-59b3-4716-a7eb-c150f2f465f6.png)

### Step 4: Visualization
***✔ Firehose to Open Search Service & Open Search Dashboard***

Since the data is arrived at open search service from **Kinesis Firehose**, the new document is created on the open search index,
 which is mapping based on data field type. The mapping data on the **Open Search Service** can be visualized on the 
 **Open Search Dashboard**.

<br>

<img width="696" alt="image" src="https://cdn.discordapp.com/attachments/779649485680279552/1020238517344944178/2022-08-11_1.51.26.png">

## Install Requirements
- AWS Root Account 
- AWS CLI Install and AWS Credintial Configure setting
- Git Install
- Terraform Install

## Deployment
1. Make and move the new directory. Git Clone Github Repository on the directory.
    ```
    git clone https://github.com/cs-devops-bootcamp/devops-02-Final-TeamD-scenario1.git
    ```
2. Change Directory to terraform Dierctory.
    ```
    cd terraform
    ```
3. On the Command line, run it thorugh with a command such as :
    ```
    terraform init
    ```

4. On the command line, deploy on the AWS with a command such as :
   <br>
  It will take about 15 minutes to deploy on the AWS.
    ```
    terraform apply
    ```
5. When the Deployment is completed, go to the Amazon Open Search Service on the AWS Management Console.<br> ID: admin PW: #Devops02

6. Enter the OpenSearch Dash board URL with deployed domain.

7. Make a role for user accessing on the open search cluster and index.
![role](https://user-images.githubusercontent.com/78151046/184048274-9f26076b-eedb-4afe-8376-2c7e23b0b6da.jpg)

    - Create Role 
    
    - Make a Name(whatever) 
    
    - Cluster permissions - `cluster_all`, `indices_all` assign 
    
    - Index permissions - index insert `*` , on the Index permissions assign same roles with Cluster permissons
    - Ignore the other stuff
1. When the role is created, role should be mapping on the user.
![mapping](https://user-images.githubusercontent.com/78151046/184048300-30622a82-343b-481c-b2f7-8bcd390bea64.jpg)
    - Select created Role name -> Select Mapped users -> Select Manage mapping
    - Insert the AWS own user ARN on the Users. (Also, insert the IAM user ARN on the Users) 
    - For the accessing the open serach service, make the new role and assign `AmazonOpenSearchFullAccess` policy on the AWS IAM.
    - Insert the previous created role on the Backend roles.

## Indexing
Move the OpenSearch Dashboard to Dev tool. Copy and Paste the below code and exeucte. This mapping process can enable to change `geo_point` proper data type. 

```
PUT /weather
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  },
  
  "mappings": {
      "properties" : {
        "co2" : {
          "type" : "long"
        },
        "coord" : {
          "type": "geo_point"
        },
        "device_id" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "error_code" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "humidity" : {
          "type" : "long"
        },
        "pressure" : {
          "type" : "long"
        },
        "result" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "server_time" : {
          "type" : "date"
        },
        "temperature" : {
          "type" : "long"
        }
      }
    }
}
```
If the process is over, enter the OpenSearch Dashboards - Management - Stack Management - Index patterns.

Select the speific index to create index pattern and make index.

Check the proper data type on the index pattern.

## How it works
>Send the GET Request on the API Gateway Endpoint, lambda is triggered and send log data to Kinesis Data Stream. 
Kinesis Data Stream deliver to the Kinesis Firehose, which push the data on open search service and s3 for back  up. When the data is stored at S3, lambda is called. The speific data value exceeds the threshold, lambda function will automatically send alarm message on the discord. Open Search Service can consume data from Kinesis Firehose and visualize on the Open Search Dashboard.

## Testing
![test](https://user-images.githubusercontent.com/78151046/184051204-38895242-8d08-4f33-8cdd-56a3d4f5bdc8.jpg)

When the API Gateway got the GET Requst from endpoint, sensor data can be viewed on the Open Search Dashboard after 1~2 minutes later.

## Clean up
Delete the data on the S3 Bucket.

On the terraform directory exeucte command such as : <br>
It will take about 10 mintues to clean up resource.
```
terraform destroy
```



## Architecture Resource 
**💡 Real-time**
>The sensor information installed in the smart farm sensor is frequently viewed, and it need to handle the data on real time. To meet these requirements, we chose **kinesis data stream** and **kinesis firehose**, which can handle real time data and manage streaming. Real-time log data was delivered streaming format to **kinesis data stream**. Configuring efficient of data pipeline we have designed **kinesis data stream** to **kinesis firehose**, which can be attached with other resources.

**💡 Timestamp**
>Smart farm data from sensor should be viewed time-series for client. We used the open search to classify data based on timeline. When the data is directly delivered to open search from **kinesis firehose**, the classified data is shown based on data types with time.

**💡 Real-time**
>We chose the **open search dashboard** to provide client monitoring service. When the new document is created in open search index, which is indexing with data classification. The data is classified based on metrics and the classified results are viewed on the open search dashboard.<br>
On the **open search dashboard**, we can customize the dashboard depending on needs of client. Basically, our **open search dashboard** provides client with data visualization that indicates smart farm’s status. According to the client requirements, we would query easily from **open search service** and visualize specific metric.<br>
When the **kinesis firehose** send data to **S3** back up, the **lambda** function is called to analysis the log data. If the specific sensor data value exceeded the standard then **lambda** function send to discord alarm immediately. For end client, that’s always critical to be alerted immediately.
<!-- > **AWS Lambda**
<br>Lambda는 서버리스 서비스로 서버에 대한 Managed Service를 제공하기 때문에 말 그대로 서버를 신경쓰지 않아도 되기 때문에 서버리스라고 부른다.Lambda를 통해서 아키텍처를 구성함에 있어어 비용과 서버에 대한 관리를 줄이고자 작은 단위의 함수를 실행할 수 있는 Lambda를 선택해서 요청이 들어올때마다 처리해주도록 구현했다. 
  
> **AWS Kiensis Data Stream, Kinesis Firehose**
> <br>Kinesis는 실시간 데이터 스트리밍을 다루는데 유용하고 보다 안정적인 IoT 디바이스에서 오는 정보의 경우 실시간으로 정보가 계속 전달되기 때문에 실시간으로 데이터를 전달 할 수 있는 Kinesis data stream을 사용함으로서 데이터의 안정성을 보장하고 실시간 데이터를 수집 및 저장할 수 있는 리소스를 선택했다.<br>
> Kinesis Firehose는 Kinesis Data Stream으로 부터 데이터를 전달 받아서 처리하고 필요한 destination에 보내주기 위해 Firehose를 사용하였다. Firehose는 데이터를 받아서 별다른 처리 과정 없이 바로 원하는 서비스로 transfer 할 수 있기 때문에 선택했습니다. <br>
> 이 두 리소스를 사용함으로서 실시간으로 메세지를 data stream 에서 firehose로 전달할 수 있고 firehose에서 전달 받은 로그들은 s3에 백업함과 동시에 다른 과정을 거치지 않고 destination인 open search service로 갈 수 있도록 리소스를 구성했습니다.

> **AWS Open Search Service**
> <br> Open Search Service는 실시간으로 유저가 센서의 정보를 보여주고 그 데이터를 시각화 시키기 위한 툴로 Open Serach Service를 사용했습니다. 거의 실시간 대용량 데이터를 시계열 기반으로 원하는 데이터를 시각화 할 수 있고 직관적인 시각화를 제공하기 때문에 이 리소스를 구성했습니다. -->


## The purpose of Architecture
Hence, through this architecture can handle real time data and can be viewed by time-series and alert to farmer immediately.Using Serverless resource allow us to decouple dependency of system.

