# MSK Integration with Redshift
In this lab, we will analyze real-time streaming data in Amazon MSK with Amazon Redshift.

## Solution overview

The purpose of Amazon Redshift streaming ingestion is to simplify the process for directly ingesting stream data from a streaming service into Amazon Redshift. This works with Amazon MSK and Amazon MSK Serverless. Amazon Redshift streaming ingestion removes the need to stage an Amazon MSK topic in Amazon S3 before ingesting the stream data into Amazon Redshift.

On a technical level, streaming ingestion, from Amazon Managed Streaming for Apache Kafka, provides low-latency, high-speed ingestion of stream or topic data into an Amazon Redshift materialized view. Following setup, using materialized view refresh, you can take in large data volumes.

Set up Amazon Redshift streaming ingestion for Amazon MSK by performing the following steps:

- Create an external schema that maps to the streaming data source.
- Create a materialized view that references the external schema.

> Streaming ingestion and Amazon Redshift Serverless - The configuration steps in this topic apply both to provisioned Amazon Redshift clusters and to Amazon Redshift Serverless. For more information, see Streaming ingestion considerations.


1. Navigate to [Redshift query editor v2](https://us-east-1.console.aws.amazon.com/sqlworkbench/home?region=us-east-1#?region=us-east-1)
2. In the left pane, choose the redshift cluster: redshift-cluster-1
3. Once it is explanded, go to editor section and create an external schema to map to the Amazon MSK cluster.

```
CREATE EXTERNAL SCHEMA MySchema
FROM MSK
IAM_ROLE default
AUTHENTICATION none 
CLUSTER_ARN 'msk-cluster-arn';
```

in Cluster identifier
redshift-cluster

