# MSK Integration with Athena
In this lab, we will analyze real-time streaming data in Amazon MSK with Amazon Athena.


## Solution overview
First, we will show you how to get started with real-time SQL analytics using Athena and its connector for MSK. The process involves:

1. Registering the schema of your streaming data with [AWS Glue Schema Registry](https://docs.aws.amazon.com/glue/latest/dg/schema-registry.html). Schema Registry is a feature of [AWS Glue](https://aws.amazon.com/glue/) that allows you to validate and reliably evolve streaming data against JSON schemas. It can also serialize data into a compressed format, which helps you save on data transfer and storage costs.

2. Creating a new instance of the [Amazon Athena MSK Connector](https://docs.aws.amazon.com/athena/latest/ug/connectors-msk.html). Athena connectors are pre-built applications that run as serverless [AWS Lambda](https://aws.amazon.com/lambda/) applications, so there’s no need for standalone data export processes.

3. Using the Athena console to run interactive SQL queries on your Kafka topics.

## Get started with Athena’s connector for Amazon MSK
In this section, we’ll cover the steps necessary to set up your MSK cluster to work with Athena to run SQL queries on your Kafka topics.

### 1. Define the schema of your Kafka topics with AWS Glue Schema Registry
To run SQL queries on your Kafka topics, you’ll first need to define the schema of your topics as Athena uses this metadata for query planning. AWS Glue makes it easy to do this with its [Schema Registry feature](https://us-east-1.console.aws.amazon.com/glue/home?region=us-east-1#/v2/data-catalog/schemaRegistries) for streaming data sources.

![athena1](images/athena1.png)

#### Creating Glue Stream schema registries
1. In Stream schema registries, click on **Add registry**

![athena2](images/athena2.png)
2. Provide the below configuration:
>* Name: customer_schema
>* Description: {AthenaFederationMSK}

3.Click **Add registry**

![athena3](images/athena3.png)

Schema Registry allows you to centrally discover, control, and evolve streaming data schemas for use in analytics applications such as Athena. With AWS Glue Schema Registry, you can manage and enforce schemas on your data streaming applications using convenient integrations with Apache Kafka. To learn more, see [AWS Glue Schema Registry](https://docs.aws.amazon.com/glue/latest/dg/schema-registry-integrations.html) and [Getting started with Schema Registry](https://docs.aws.amazon.com/glue/latest/dg/schema-registry-gs.html).

If configured to do so, the producer of data can auto-register its schema and changes to it with AWS Glue. This is especially useful in use cases where the contents of the data is likely to change over time. However, you can also specify the schema manually, resembling the following JSON structure.

#### Creating Schema in Glue Stream schema registries

1. Choose the Stream schehma registry : **[customer_schema](https://us-east-1.console.aws.amazon.com/glue/home?region=us-east-1#/v2/data-catalog/schemaRegistries/view/customer_schema)**
2. Click **Create schema**

![athena4](images/athena4.png)
3. Provide the below configuration
>* Schema name: orders
>* Registry: customer_schema
>* Data format: JSON
>* Compatibility mode: Full - combination of 'Backward' and 'Forward'
>* Description: {AthenaFederationMSK}

![athena5](images/athena5.png)

    ##### First schema version
    
        {
          "topicName": "orders",
          "message": {
            "dataFormat": "json",
            "fields": [
              {
                "name": "customer_id",
                "mapping": "customer_id",
                "type": "VARCHAR"
              },
              {
                "name": "item_id",
                "mapping": "item_id",
                "type": "INTEGER"
              }
            ]
          }
        }

![athena6](images/athena6.png)    
4. Click **Create schema and version**

> For additional information on schema set up, see Schema examples for the [AWS Glue Schema Registry](https://docs.aws.amazon.com/athena/latest/ug/connectors-msk.html#connectors-msk-setup).

### 2. Configure the Athena connector for MSK
With your schema registered with Glue, the next step is to set up the Athena connector for MSK. We recommend using the Athena console for this step. For more background on the steps involved, see [Deploying a connector and connecting to a data source](https://docs.aws.amazon.com/athena/latest/ug/connect-to-a-data-source-lambda.html).

In Athena, federated data source connectors are applications that run on AWS Lambda and handle communication between your target data source and Athena. When a query runs on a federated source, Athena calls the Lambda function and tasks it with running the parts of your query that are specific to that source. To learn more about the query execution workflow, see [Using Amazon Athena Federated Query](https://docs.aws.amazon.com/athena/latest/ug/connect-to-a-data-source.html) in the Amazon Athena User Guide.

1. Navigate to [Athena console](https://us-east-1.console.aws.amazon.com/athena/home?region=us-east-1#/data-sources) and selecting Data sources on the left navigation, then choose Create data source

![athena7](images/athena7.png) <br>
2. Click on **Create Data sources**

![athena8](images/athena8.png)
3. Choose a data source: Type **MSK**

![athena9](images/athena9.png)
4. Choose the radio button beside the connector **Amazon MSK** and click **Next** <br>
5. Enter the following details in Data source details section:
>* Data source name: msk
>* Description - To connect to amazon msk cluster

![athena10](images/athena10.png)
6. Under Connection details section, select **Create Lambda function**. This will bring you to the AWS Lambda console where you’ll provide additional configuration properties

![athena11](images/athena11.png)
7. Scroll down to **Application settings** and provide below details 

![athena12](images/athena12.png)

    ##### Fetch MSK Configuration
>* To get some of the details you will have to navigate to [MSK console](https://us-east-1.console.aws.amazon.com/msk/home?region=us-east-1#/clusters) and click on cluster name as **MSKCluster-msk-labs-default**
>* In Cluster summary, Click on **View client Information**

![athena13](images/athena13.png)

>* Copy any one of the broker-ids(connection string for the private endpoint) from Authentication type - **Plaintext** and paste it in the Lambda configuration **"KafkaEndpoint"**
>> Note: You will get three endpoints for each of the brokers. You only need one broker endpoint for the following step. (Paste this in KafkaEndpoint) and click **Done**

![athena14](images/athena14.png)

>* Click on **Properties Tab** and scroll down to **Networking settings**

![athena16](images/athena16.png)

>* In **Networking settings**, under **Primary VPC configuration** look for **Subnets** and **Security groups** applied.

![athena17](images/athena17.png)

>* Copy **security-group-id** and one **subnet-id**, paste them in the below configuration accordingly
        
        KafkaEndpoint: <paste the broker-id here>
        LambdaFunctionName: msk-athena-lambda-function
        SecurityGroupIds: <paste the security-group-id here>
        SpillBucket: msk-athena-bucket-<paste the AWS-Account-Number>
        SpillPrefix: msk-athena-spill
        SubnetIds: <paste the subnet-id here >

![athena15](images/athena15.png)

![athena18](images/athena18.png) <br>
8. Click on **I acknowledge that this app creates custom IAM roles and resource policies**. <br>
9. Click **Deploy** (wait for a minute to auto-refresh the page)

![athena19](images/athena19.png)
10. Close the Lambda page and go back to previous tab (Athena Console - Enter data source details) <br>
11. In Connection details, use the refresh button and choose **Lambda function** which we created in the previous step ( **msk-athena-lambda-function** ) 

![athena20](images/athena20.png)

![athena21](images/athena21.png)
12. Click **Next** <br>
13. Review the details and click **Create data source**

![athena22](images/athena22.png)

### 3. Run queries on streaming data using Athena
With your MSK data connector set up, you can now run SQL queries on the data. Let’s explore a few use cases in more detail.

Use case: interactive analysis
If you want to run queries that aggregate, group, or filter your MSK data, you can run interactive queries using Athena. These queries will run against the current state of your Kafka topics at the time the query was submitted.

1. Navigate to [Athena Console](https://us-east-1.console.aws.amazon.com/athena/home?region=us-east-1#/query-editor)
2. Under **Editor** tab, in **Data** section, you will observe the following:

![athena23](images/athena23.png)
3. Choose **msk** from the drop down. <br>
4. Choose **customer_schema** from the database

![athena24](images/athena24.png)

Before running any queries, it may be helpful to validate the schema and data types available within your Kafka topics. To do this, run the **DESCRIBE** command on your Kafka topic, which appears in Athena as a table, as shown below. In this query, the **orders** table corresponds to the topic you specified in the Schema Registry **customer_schema**.

> DESCRIBE msk.customer_schema.orders

![athena25](images/athena25.png)

Now that you know the contents of your topic, you can begin to develop analytical queries. A sample query for querying all data in a topic is:

```
select * from msk.customer_schema.orders;
```
![athena26](images/athena26.png)

You can proceed to next lab!