# Troubleshooting: DAGs, Operators, Connections, and other issues in Apache Airflow v2<a name="t-apache-airflow-202"></a>

The topics on this page describe resolutions to Apache Airflow v2 Python dependencies, custom plugins, DAGs, Operators, Connections, tasks, and *Web server* issues you may encounter on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Connections](#troubleshooting-conn-202)
  + [I can't connect to Secrets Manager](#access-secrets-manager-202)
  + [How do I configure `secretsmanager:ResourceTag/<tag-key>` secrets manager conditions or a resource restriction in my execution role policy?](#access-secrets-manager-condition-keys-202)
  + [I can't connect to Snowflake](#missing-snowflake)
  + [I can't see my connection in the Airflow UI](#connection-type-missing)
+ [Web server](#troubleshooting-webserver-202)
  + [I see a 5xx error accessing the web server](#5xx-webserver-202)
  + [I see a 'The scheduler does not appear to be running' error](#error-scheduler-202)
+ [Tasks](#troubleshooting-tasks-202)
  + [I see my tasks stuck or not completing](#stranded-tasks-202)
+ [CLI](#troubleshooting-cli-202)
  + [I see a '503' error when triggering a DAG in the CLI](#cli-toomany-202)
  + [Why does the `dags backfill` Apache Airflow CLI command fail? Is there a workaround?](#troubleshooting-cli-backfill)
+ [Operators](#troubleshooting-operators-202)
  + [I received a `PermissionError: [Errno 13] Permission denied` error using the S3Transform operator](#op-s3-transform)

## Connections<a name="troubleshooting-conn-202"></a>

The following topic describes the errors you may receive when using an Apache Airflow connection, or using another AWS database\.

### I can't connect to Secrets Manager<a name="access-secrets-manager-202"></a>

We recommend the following steps:

1. Learn how to create secret keys for your Apache Airflow connection and variables in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

1. Learn how to use the secret key for an Apache Airflow variable \(`test-variable`\) in [Using a secret key in AWS Secrets Manager for an Apache Airflow variable](samples-secrets-manager-var.md)\.

1. Learn how to use the secret key for an Apache Airflow connection \(`myconn`\) in [Using a secret key in AWS Secrets Manager for an Apache Airflow connection](samples-secrets-manager.md)\.

### How do I configure `secretsmanager:ResourceTag/<tag-key>` secrets manager conditions or a resource restriction in my execution role policy?<a name="access-secrets-manager-condition-keys-202"></a>

**Note**  
 Applies to Apache Airflow version 2\.0 and earlier\. 

 Currently, you cannot limit access to Secrets Manager secrets by using condition keys or other resource restrictions in your environment's execution role, due to a known issue in Apache Airflow\. 

### I can't connect to Snowflake<a name="missing-snowflake"></a>

We recommend the following steps:

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following entries to the requirements\.txt for your environment\.

   ```
   apache-airflow-providers-snowflake==1.3.0
   ```

1. Add the following imports to your DAG:

   ```
   from airflow.providers.snowflake.operators.snowflake import SnowflakeOperator
   ```

Ensure the Apache Airflow connection object includes the following key\-value pairs:

1. **Conn Id: **snowflake\_conn

1. **Conn Type: **Snowflake

1. **Host: **<my account>\.<my region if not us\-west\-2>\.snowflakecomputing\.com

1. **Schema: **<my schema>

1. **Login: **<my user name>

1. **Password: **\*\*\*\*\*\*\*\*

1. **Port: ** <port, if any>

1. **Extra: **

   ```
   {
       "account": "<my account>",
       "warehouse": "<my warehouse>",
       "database": "<my database>",
       "region": "<my region if not using us-west-2 otherwise omit this line>"
   }
   ```

For example:

```
>>> import json
>>> from airflow.models.connection import Connection
>>> myconn = Connection(
...    conn_id='snowflake_conn',
...    conn_type='Snowflake',
...    host='YOUR_ACCOUNT.YOUR_REGION.snowflakecomputing.com',
...    schema='YOUR_SCHEMA'
...    login='YOUR_USERNAME',
...    password='YOUR_PASSWORD',
...    port='YOUR_PORT'
...    extra=json.dumps(dict(account='YOUR_ACCOUNT', warehouse='YOUR_WAREHOUSE', database='YOUR_DB_OPTION', region='YOUR_REGION')),
... )
```

### I can't see my connection in the Airflow UI<a name="connection-type-missing"></a>

Apache Airflow provides connection templates in the Apache Airflow UI\. It uses this to generate the connection URI string, regardless of the connection type\. If a connection template is not available in the Apache Airflow UI, an alternate connection template can be used to generate a connection URI string, such as using the HTTP connection template\.

We recommend the following steps:

1. View the connection types Amazon MWAA's providing in the Apache Airflow UI at [Apache Airflow v2 provider packages installed on Amazon MWAA environments](connections-packages.md)\.

1. View the commands to create an Apache Airflow connection in the CLI at [Apache Airflow CLI command reference](airflow-cli-command-reference.md)\.

1. Learn how to use connection templates in the Apache Airflow UI interchangeably for connection types that aren't available in the Apache Airflow UI on Amazon MWAA at [Overview of connection types](manage-connection-types.md)\.

## Web server<a name="troubleshooting-webserver-202"></a>

The following topic describes the errors you may receive for your Apache Airflow *Web server* on Amazon MWAA\.

### I see a 5xx error accessing the web server<a name="5xx-webserver-202"></a>

We recommend the following steps:

1. Check Apache Airflow configuration options\. Verify that the key\-value pairs you specified as an Apache Airflow configuration option, such as AWS Secrets Manager, were configured correctly\. To learn more, see [I can't connect to Secrets Manager](t-apache-airflow-11012.md#access-secrets-manager)\.

1. Check the `requirements.txt`\. Verify the Airflow "extras" package and other libraries listed in your `requirements.txt` are compatible with your Apache Airflow version\.

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I see a 'The scheduler does not appear to be running' error<a name="error-scheduler-202"></a>

If the scheduler doesn't appear to be running, or the last "heart beat" was received several hours ago, your DAGs may not appear in Apache Airflow, and new tasks will not be scheduled\.

We recommend the following steps:

1. Confirm that your VPC security group allows inbound access to port `5432`\. This port is needed to connect to the Amazon Aurora PostgreSQL metadata database for your environment\. After this rule is added, give Amazon MWAA a few minutes, and the error should disappear\. To learn more, see [Security in your VPC on Amazon MWAA](vpc-security.md)\. 
**Note**  
 The Aurora PostgreSQL metadatabase is part of the [Amazon MWAA service architecture](what-is-mwaa.md#architecture-mwaa) and is not visible in your AWS account\. 
 Database\-related errors are usually a symptom of scheduler failure and not the root cause\. 

1.  If the scheduler is not running, it might be due to a number of factors such as [dependency installation failures](best-practices-dependencies.md), or an [overloaded scheduler](best-practices-tuning.md)\. Confirm that your DAGs, plugins, and requirements are working correctly by viewing the corresponding log groups in CloudWatch Logs\. To learn more, see [Monitoring and metrics for Amazon Managed Workflows for Apache Airflow \(MWAA\)](cw-metrics.md)\.

## Tasks<a name="troubleshooting-tasks-202"></a>

The following topic describes the errors you may receive for Apache Airflow tasks in an environment\.

### I see my tasks stuck or not completing<a name="stranded-tasks-202"></a>

If your Apache Airflow tasks are "stuck" or not completing, we recommend the following steps:

1. There may be a large number of DAGs defined\. Reduce the number of DAGs and perform an update of the environment \(such as changing a log level\) to force a reset\.

   1. Airflow parses DAGs whether they are enabled or not\. If you're using greater than 50% of your environment's capacity you may start overwhelming the Apache Airflow *Scheduler*\. This leads to large *Total Parse Time* in CloudWatch Metrics or long DAG processing times in CloudWatch Logs\. There are other ways to optimize Apache Airflow configurations which are outside the scope of this guide\.

   1. To learn more about the best practices we recommend to tune the performance of your environment, see [Performance tuning for Apache Airflow on Amazon MWAA](best-practices-tuning.md)\.

1. There may be a large number of tasks in the queue\. This often appears as a large—and growing—number of tasks in the "None" state, or as a large number in *Queued Tasks* and/or *Tasks Pending* in CloudWatch\. This can occur for the following reasons:

   1. If there are more tasks to run than the environment has the capacity to run, and/or a large number of tasks that were queued before autoscaling has time to detect the tasks and deploy additional *Workers*\. 

   1. If there are more tasks to run than an environment has the capacity to run, we recommend **reducing** the number of tasks that your DAGs run concurrently, and/or increasing the minimum Apache Airflow *Workers*\.

   1. If there are a large number of tasks that were queued before autoscaling has had time to detect and deploy additional workers, we recommend **staggering** task deployment and/or increasing the minimum Apache Airflow *Workers*\.

   1. You can use the [update\-environment](https://docs.aws.amazon.com/cli/latest/reference/mwaa/update-environment.html) command in the AWS Command Line Interface \(AWS CLI\) to change the minimum or maximum number of *Workers* that run on your environment\.

      ```
      aws mwaa update-environment --name MyEnvironmentName --min-workers 2 --max-workers 10
      ```

   1. To learn more about the best practices we recommend to tune the performance of your environment, see [Performance tuning for Apache Airflow on Amazon MWAA](best-practices-tuning.md)\.

1. There may be tasks being deleted mid\-execution that appear as task logs which stop with no further indication in Apache Airflow\. This can occur for the following reasons:

   1. If there is a brief moment where 1\) the current tasks exceed current environment capacity, followed by 2\) a few minutes of no tasks executing or being queued, then 3\) new tasks being queued\. 

   1. Amazon MWAA autoscaling reacts to the first scenario by adding additional workers\. In the second scenario, it removes the additional workers\. Some of the tasks being queued may result with the workers in the process of being removed, and will end when the container is deleted\.

   1. We recommend increasing the minimum number of workers on your environment\. Another option is to adjust the timing of your DAGs and tasks to ensure that that these scenarios don't occur\.

   1. You can also set the minimum workers equal to the maximum workers on your environment, effectively disabling autoscaling\. Use the [update\-environment](https://docs.aws.amazon.com/cli/latest/reference/mwaa/update-environment.html) command in the AWS Command Line Interface \(AWS CLI\) to **disable autoscaling** by setting the minimum and maximum number of workers to be the same\.

      ```
      aws mwaa update-environment --name MyEnvironmentName --min-workers 5 --max-workers 5
      ```

   1. To learn more about the best practices we recommend to tune the performance of your environment, see [Performance tuning for Apache Airflow on Amazon MWAA](best-practices-tuning.md)\.

1. If your tasks are stuck in the "running" state, you can also clear the tasks or mark them as succeeded or failed\. This allows the autoscaling component for your environment to scale down the number of workers running on your environment\. The following image shows an example of a stranded task\.  
![\[This is an image with a stranded task.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling.png)

   1. Choose the circle for the stranded task, and then select **Clear** \(as shown\)\. This allows Amazon MWAA to scale down workers; otherwise, Amazon MWAA can't determine which DAGs are enabled or disabled, and can't scale down, if there are still queued tasks\.  
![\[Apache Airflow Actions\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling-menu.png)

1. Learn more about the Apache Airflow task lifecycle at [Concepts](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#task-lifecycle) in the *Apache Airflow reference guide*\.

## CLI<a name="troubleshooting-cli-202"></a>

The following topic describes the errors you may receive when running Airflow CLI commands in the AWS Command Line Interface\.

### I see a '503' error when triggering a DAG in the CLI<a name="cli-toomany-202"></a>

The Airflow CLI runs on the Apache Airflow *Web server*, which has limited concurrency\. Typically a maximum of 4 CLI commands can run simultaneously\.

### Why does the `dags backfill` Apache Airflow CLI command fail? Is there a workaround?<a name="troubleshooting-cli-backfill"></a>

**Note**  
 The following applies only to Apache Airflow v2\.0\.2 environments\. 

 The `backfill` command, like other Apache Airflow CLI commands, parses all DAGs locally before any DAGs are processed, regardless of which DAG the CLI operation applies to\. In Amazon MWAA environments using Apache Airflow v2\.0\.2, because plugins and requirements are not yet installed on the web server by the time the CLI command runs, the parsing operation fails, and the `backfill` operation is not invoked\. If you did not have any requirements nor plugins in your environment, the `backfill` operation would succeed\. 

 In order to be able to run the `backfill` CLI command, we recommend invoking it in a bash operator\. In a bash operator, `backfill` is initiated from the worker, allowing the DAGs to parse successfully as all necessary requirements and plguins are available and installed\. The following example shows how you can create a DAG with a `BashOperator` to run `backfill`\. 

```
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago

with DAG(dag_id="backfill_dag", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
    cli_command = BashOperator(
        task_id="bash_command",
        bash_command="airflow dags backfill my_dag_id"
    )
```

## Operators<a name="troubleshooting-operators-202"></a>

The following topic describes the errors you may receive when using Operators\.

### I received a `PermissionError: [Errno 13] Permission denied` error using the S3Transform operator<a name="op-s3-transform"></a>

We recommend the following steps if you're trying to run a shell script with the *S3Transform* operator and you're receiving a `PermissionError: [Errno 13] Permission denied` error\. The following steps assume you have an existing plugins\.zip file\. If you're creating a *new* plugins\.zip, see [Installing custom plugins](configuring-dag-import-plugins.md)\.

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Create your "transform" script\.

   ```
   #!/bin/bash
   cp $1 $2
   ```

1. \(optional\) macOS and Linux users may need to run the following command to ensure the script is executable\.

   ```
   chmod 777 transform_test.sh
   ```

1. Add the script to your plugins\.zip\. 

   ```
   zip plugins.zip transform_test.sh
   ```

1. Follow the steps in [Upload the plugins\.zip to Amazon S3](configuring-dag-import-plugins.md#configuring-dag-plugins-upload)\.

1. Follow the steps in [Specifying the plugins\.zip version on the Amazon MWAA console](configuring-dag-import-plugins.md#configuring-dag-plugins-s3-mwaaconsole)\.

1. Create the following DAG\.

   ```
   from airflow import DAG
   from airflow.providers.amazon.aws.operators.s3_file_transform import S3FileTransformOperator
   from airflow.utils.dates import days_ago
   import os
   
   DAG_ID = os.path.basename(__file__).replace(".py", "")
   
   with DAG (dag_id=DAG_ID, schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
       file_transform = S3FileTransformOperator(
           task_id='file_transform',
           transform_script='/usr/local/airflow/plugins/transform_test.sh',
           source_s3_key='s3://YOUR_S3_BUCKET/files/input.txt',
           dest_s3_key='s3://YOUR_S3_BUCKET/files/output.txt'
       )
   ```

1. Follow the steps in [Uploading DAG code to Amazon S3](configuring-dag-folder.md#configuring-dag-folder-uploading)\.