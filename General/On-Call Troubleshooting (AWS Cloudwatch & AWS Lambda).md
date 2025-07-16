# AWS CloudWatch & AWS Lambda On-Call Troubleshooting

## Monitoring and Metrics Troubleshooting

1. **Critical application metric not appearing in CloudWatch**
   - **Solution**: Verify metric publishing, IAM permissions (`cloudwatch:PutMetricData`), namespace, and CloudWatch agent/SDK configuration.
     ```python
     import boto3
     client = boto3.client('cloudwatch', region_name='us-east-1')
     client.put_metric_data(
         Namespace='MyApp',
         MetricData=[{
             'MetricName': 'CriticalMetric',
             'Value': 100.0,
             'Unit': 'Count',
             'Dimensions': [{'Name': 'InstanceId', 'Value': 'i-1234567890abcdef0'}]
         }]
     )
     ```
   - **UI Steps**:
     1. CloudWatch > Metrics > All metrics > Search `MyApp`/`CriticalMetric`.
     2. If missing, check Systems Manager > Parameter Store or EC2 for agent config.
     3. Verify IAM role permissions in IAM > Roles.

2. **Delayed CloudWatch metrics for EC2 instance**
   - **Solution**: Check agent configuration, network latency, and collection interval.
     ```json
     {
       "metrics": {
         "namespace": "CWAgent",
         "metrics_collected": {
           "cpu": {
             "measurement": ["cpu_usage_active"],
             "metrics_collection_interval": 10
           }
         }
       }
     }
     ```
   - **UI Steps**:
     1. Systems Manager > Parameter Store > Update `AmazonCloudWatch-agent` config to lower `metrics_collection_interval`.
     2. Restart agent: `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a restart`.

3. **Custom metric showing incomplete data**
   - **Solution**: Check publishing frequency, application logs, throttling, and timestamp accuracy.
     ```python
     import boto3
     client = boto3.client('cloudwatch')
     try:
         client.put_metric_data(
             Namespace='MyApp',
             MetricData=[{'MetricName': 'CustomMetric', 'Value': 50.0, 'Unit': 'Count'}]
         )
     except client.exceptions.InvalidParameterValueException as e:
         print(f"Error: {e}")
     ```
   - **UI Steps**:
     1. CloudWatch > Metrics > Check for gaps in `CustomMetric`.
     2. CloudWatch > Logs > Log groups > Review application logs.
     3. Verify IAM permissions for `cloudwatch:PutMetricData`.

4. **No metrics for newly launched resource**
   - **Solution**: Ensure CloudWatch agent installed, IAM permissions, correct namespace/region.
     ```bash
     sudo yum install amazon-cloudwatch-agent
     sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
     sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a start
     ```
   - **UI Steps**:
     1. CloudWatch > Metrics > Check namespace (e.g., `AWS/EC2`).
     2. Systems Manager > Run Command > Run `AWS-ConfigureCloudWatchAgent`.
     3. IAM > Roles > Attach `CloudWatchAgentServerPolicy`.

5. **Metric data missing for certain periods**
   - **Solution**: Check application downtime, metric publishing, API throttling, retention period.
     ```bash
     aws cloudwatch list-metrics --namespace MyApp
     aws cloudwatch get-metric-statistics --namespace MyApp --metric-name MyMetric --start-time 2025-07-01T00:00:00Z --end-time 2025-07-02T00:00:00Z --period 300 --statistics Average
     ```
   - **UI Steps**:
     1. CloudWatch > Metrics > Adjust time range for gaps.
     2. CloudWatch > Logs > Check application logs for downtime.
     3. Adjust retention in CloudWatch > Metrics > Settings.

## Alarms and Notifications

6. **CloudWatch alarm not triggering despite threshold breach**
   - **Solution**: Verify threshold, evaluation period, namespace, and SNS permissions.
     ```bash
     aws cloudwatch describe-alarms --alarm-names MyAlarm
     aws cloudwatch get-metric-statistics --namespace MyApp --metric-name MyMetric --start-time 2025-07-01T00:00:00Z --end-time 2025-07-02T00:00:00Z --period 60 --statistics Average
     ```
   - **UI Steps**:
     1. CloudWatch > Alarms > `MyAlarm` > Check “Details” for metric/threshold.
     2. CloudWatch > Metrics > Verify data crosses threshold.
     3. Ensure SNS topic has `sns:Publish` in IAM > Roles.

7. **Frequent false positives from CloudWatch alarm**
   - **Solution**: Increase evaluation period, adjust threshold, or use anomaly detection.
     ```bash
     aws cloudwatch put-metric-alarm --alarm-name MyAlarm --metric-name CPUUtilization --namespace AWS/EC2 --threshold 80.0 --comparison-operator GreaterThanThreshold --evaluation-periods 3 --period 300 --statistic Average --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic
     ```
   - **UI Steps**:
     1. CloudWatch > Alarms > `MyAlarm` > Modify > Increase “Evaluation Periods” (e.g., to 3).
     2. Adjust “Threshold” if needed.

8. **Alarm stuck in "INSUFFICIENT_DATA" state**
   - **Solution**: Verify metric existence, data points, namespace, and dimensions.
     ```bash
     aws cloudwatch get-metric-statistics --namespace MyApp --metric-name MyMetric --start-time 2025-07-01T00:00:00Z --end-time 2025-07-02T00:00:00Z --period 60 --statistics Average
     ```
   - **UI Steps**:
     1. CloudWatch > Alarms > Check metric in “Details”.
     2. CloudWatch > Metrics > Verify data points.
     3. Publish test data via SDK/CLI.

9. **Frequent alarm notifications**
   - **Solution**: Increase evaluation period or period length, adjust threshold.
     ```bash
     aws cloudwatch put-metric-alarm --alarm-name MyAlarm --metric-name ErrorRate --namespace MyApp --threshold 10.0 --comparison-operator GreaterThanThreshold --evaluation-periods 5 --period 300 --statistic Average --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic
     ```
   - **UI Steps**:
     1. CloudWatch > Alarms > `MyAlarm` > Modify > Set “Evaluation Periods” to 5, “Period” to 300s.
     2. Adjust “Threshold” if needed.

10. **SNS notifications not reaching recipients**
    - **Solution**: Verify SNS topic subscription, IAM permissions, endpoint validity.
      ```bash
      aws sns list-subscriptions-by-topic --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic
      aws sns publish --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic --message "Test message"
      ```
    - **UI Steps**:
      1. SNS > Topics > `MyTopic` > Check “Subscriptions” for endpoint confirmation.
      2. CloudWatch > Alarms > Verify SNS topic ARN.
      3. Test via SNS > Topics > Publish message.

## Logs and Log Insights

11. **Application logs not appearing in CloudWatch Logs**
    - **Solution**: Verify agent configuration, IAM permissions, log group/stream.
      ```bash
      sudo cat /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
      aws logs describe-log-groups --log-group-name-prefix my-app-logs
      ```
      ```json
      {
        "logs": {
          "logs_collected": {
            "files": {
              "collect_list": [
                {
                  "file_path": "/var/log/application.log",
                  "log_group_name": "my-app-logs",
                  "log_stream_name": "{instance_id}"
                }
              ]
            }
          }
        }
      }
      ```
      ```json
      {
        "Effect": "Allow",
        "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
        "Resource": "arn:aws:logs:region:account-id:log-group:my-app-logs:*"
      }
      ```
    - **UI Steps**:
      1. CloudWatch > Logs > Log groups > Verify `my-app-logs`.
      2. IAM > Roles > Confirm `logs:PutLogEvents` permissions.
      3. Check agent: `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status`.

12. **Duplicate log entries in CloudWatch Logs**
    - **Solution**: Check for multiple log writers, ensure single stream.
      ```bash
      sudo ps aux | grep amazon-cloudwatch-agent
      sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
      ```
      ```javascript
      const winston = require('winston');
      const { CloudWatchLogs } = require('winston-cloudwatch');
      const logger = winston.createLogger({
        transports: [
          new CloudWatchLogs({
            logGroupName: 'my-app-logs',
            logStreamName: 'app-stream',
            awsRegion: 'us-east-1'
          })
        ]
      });
      ```
    - **UI Steps**:
      1. CloudWatch > Logs > `my-app-logs` > Check for duplicate streams.
      2. Review application code or agent config.

13. **Delayed log data in CloudWatch Logs**
    - **Solution**: Check agent metrics, increase buffer size.
      ```bash
      aws cloudwatch get-metric-statistics --namespace AWS/Logs --metric-name IncomingLogEvents --dimensions Name=LogGroupName,Value=my-app-logs --start-time $(date -u -Iseconds -d '-1 hour') --end-time $(date -u -Iseconds) --period 300 --statistics Sum
      ```
      ```json
      {
        "logs": {
          "logs_collected": {
            "files": {
              "collect_list": [
                {
                  "file_path": "/var/log/application.log",
                  "log_group_name": "my-app-logs",
                  "log_stream_name": "{instance_id}",
                  "buffer_duration": 5000
                }
              ]
            }
          }
        }
      }
      ```
      ```bash
      sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start
      ```
    - **UI Steps**:
      1. CloudWatch > Metrics > AWS/Logs > Check `IncomingLogEvents`.
      2. Update agent config on EC2 instance.

14. **CloudWatch Logs Insights query returning incomplete results**
    - **Solution**: Verify query syntax, time range, optimize filters.
      ```logs
      fields @timestamp, @message
      | filter @message like /ERROR/
      | sort @timestamp desc
      | limit 100
      ```
      ```bash
      aws logs start-query --log-group-name my-app-logs --start-time $(date -d "-1 hour" +%s) --end-time $(date +%s) --query-string 'fields @timestamp, @message | filter @message like /ERROR/'
      ```
    - **UI Steps**:
      1. CloudWatch > Logs > Logs Insights > Select `my-app-logs`.
      2. Adjust time range, refine query with specific filters.

15. **Log groups causing storage issues**
    - **Solution**: Set retention, export to S3.
      ```bash
      aws logs put-retention-policy --log-group-name my-app-logs --retention-in-days 7
      aws logs create-export-task --log-group-name my-app-logs --from $(date -d "-30 days" +%s000) --to $(date +%s000) --destination my-s3-bucket --destination-prefix logs
      ```
    - **UI Steps**:
      1. CloudWatch > Logs > `my-app-logs` > Actions > Edit retention > Set to 7 days.
      2. Actions > Export data to S3 > Specify time range and bucket.

## Permissions and Configuration Issues

16. **User cannot view CloudWatch metrics**
    - **Solution**: Verify IAM policy for `cloudwatch:GetMetricData`, `cloudwatch:ListMetrics`.
      ```json
      {
        "Effect": "Allow",
        "Action": ["cloudwatch:GetMetricData", "cloudwatch:ListMetrics", "cloudwatch:DescribeAlarms"],
        "Resource": "*"
      }
      ```
      ```bash
      aws iam get-user-policy --user-name user-name
      ```
    - **UI Steps**:
      1. IAM > Users > user-name > Permissions > Add `cloudwatch:GetMetricData`.
      2. Test access in CloudWatch > Metrics.

17. **CloudWatch agent not sending metrics**
    - **Solution**: Check agent status, configuration, restart.
      ```bash
      sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
      ```
      ```json
      {
        "metrics": {
          "namespace": "CustomMetrics",
          "metrics_collected": {
            "cpu": {
              "measurement": ["cpu_usage_active"]
            }
          }
        }
      }
      ```
      ```bash
      sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start
      ```
    - **UI Steps**:
      1. SSH to EC2 > Check agent status.
      2. Edit `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`.
      3. CloudWatch > Metrics > Verify `CustomMetrics`.

18. **Quota limit preventing new CloudWatch alarm**
    - **Solution**: Check quota, request increase.
      ```bash
      aws service-quotas list-service-quotas --service-code cloudwatch
      aws service-quotas request-service-quota-increase --service-code cloudwatch --quota-code L-12345678 --desired-value 1000
      ```
    - **UI Steps**:
      1. Service Quotas > AWS services > Amazon CloudWatch > “Alarms” > Request quota increase.

## Performance and Scalability

19. **Slow CloudWatch dashboards**
    - **Solution**: Reduce widget complexity, use higher periods.
      ```json
      {
        "widgets": [
          {
            "type": "metric",
            "properties": {
              "metrics": [["AWS/EC2", "CPUUtilization", "InstanceId", "i-1234567890abcdef0"]],
              "period": 300,
              "stat": "Average"
            }
          }
        ]
      }
      ```
    - **UI Steps**:
      1. CloudWatch > Dashboards > Edit widgets > Reduce metrics, increase period (e.g., 300s).

20. **High CloudWatch API request latency**
    - **Solution**: Check throttling, enable metric streams.
      ```bash
      aws cloudwatch get-metric-statistics --namespace AWS/CloudWatch --metric-name ThrottledRequests --start-time $(date -u -Iseconds -d '-1 hour') --end-time $(date -u -Iseconds) --period 300 --statistics Sum
      aws cloudwatch create-metric-stream --name my-metric-stream --role-arn arn:aws:iam::account-id:role/my-stream-role --firehose-arn arn:aws:firehose:region:account-id:deliverystream/my-stream --output-format opentelemetry
      ```
    - **UI Steps**:
      1. CloudWatch > Metrics > AWS/CloudWatch > Check `ThrottledRequests`.
      2. CloudWatch > Metric streams > Create stream with Kinesis Firehose.

21. **High CloudWatch metric costs**
    - **Solution**: Reduce frequency, delete unused metrics.
      ```json
      {
        "metrics": {
          "namespace": "CustomMetrics",
          "metrics_collected": {
            "cpu": {
              "measurement": ["cpu_usage_active"],
              "metrics_collection_interval": 300
            }
          }
        }
      }
      ```
      ```bash
      aws cloudwatch delete-metrics --namespace CustomMetrics --metric-name UnusedMetric
      ```
    - **UI Steps**:
      1. CloudWatch > Metrics > `CustomMetrics` > Identify high-frequency metrics.
      2. Update agent config on EC2.

## Lambda Invocation and Execution Issues

22. **Lambda function not invoked by event source**
    - **Solution**: Check event source mapping, permissions, CloudWatch Logs.
      ```bash
      aws lambda get-event-source-mapping --uuid <event-source-mapping-uuid>
      aws lambda get-policy --function-name <function-name>
      aws logs describe-log-streams --log-group-name /aws/lambda/<function-name>
      ```
    - **UI Steps**:
      1. Lambda > Functions > Select function > Triggers > Verify event source (e.g., SQS).
      2. Permissions > Check execution role.
      3. Monitor > Logs > View CloudWatch Logs.

23. **Lambda function timing out**
    - **Solution**: Increase timeout, optimize code, check logs.
      ```bash
      aws lambda update-function-configuration --function-name <function-name> --timeout 60
      ```
    - **UI Steps**:
      1. Lambda > Functions > Select function > Configuration > General configuration > Edit > Increase timeout (e.g., 60s).
      2. Monitor > Logs > Check for bottlenecks.

24. **Lambda function not processing SQS queue events**
    - **Solution**: Verify SQS trigger, permissions, visibility timeout.
      ```bash
      aws lambda list-event-source-mappings --function-name <function-name>
      aws lambda get-policy --function-name <function-name>
      aws sqs get-queue-attributes --queue-url <queue-url> --attribute-names VisibilityTimeout
      ```
    - **UI Steps**:
      1. Lambda > Functions > Triggers > Verify SQS ARN.
      2. Permissions > Ensure `sqs:ReceiveMessage`.
      3. SQS > Select queue > Check “Visibility timeout”.

25. **Multiple invocations for a single Lambda event**
    - **Solution**: Check for duplicate sources, add idempotency, adjust retries.
      ```bash
      aws lambda list-event-source-mappings --function-name <function-name>
      ```
      ```python
      import json
      def lambda_handler(event, context):
          event_id = event.get('id')
          # Check event_id in DynamoDB
          return {"statusCode": 200, "body": json.dumps("Processed")}
      ```
    - **UI Steps**:
      1. Lambda > Triggers > Check for duplicate sources.
      2. Code > Add idempotency logic.
      3. SQS > Adjust “Delivery delay” or “Visibility timeout”.

26. **Lambda runtime error**
    - **Solution**: Enable logging, review logs, fix code.
      ```bash
      aws logs describe-log-streams --log-group-name /aws/lambda/<function-name>
      ```
      ```python
      import logging
      logger = logging.getLogger()
      logger.setLevel(logging.INFO)
      def lambda_handler(event, context):
          try:
              logger.info("Processing event")
          except Exception as e:
              logger.error(f"Error: {str(e)}")
              raise e
      ```
    - **UI Steps**:
      1. Lambda > Monitor > Logs > View CloudWatch Logs.
      2. Code > Add try-catch and logging.
      3. Test > Reproduce error.

## Lambda Permissions and IAM Issues

27. **Lambda "Permission denied" for S3**
    - **Solution**: Verify execution role, add S3 permissions.
      ```bash
      aws iam put-role-policy --role-name <lambda-role-name> --policy-name s3-access --policy-document '{
          "Version": "2012-10-17",
          "Statement": [{"Effect": "Allow", "Action": ["s3:GetObject", "s3:PutObject"], "Resource": "arn:aws:s3:::<bucket-name>/*"}]
      }'
      ```
    - **UI Steps**:
      1. Lambda > Permissions > Execution role > IAM Console.
      2. Add inline policy with `s3:GetObject`, `s3:PutObject`.

28. **Lambda cannot assume execution role**
    - **Solution**: Check trust policy, update for Lambda service.
      ```bash
      aws iam update-assume-role-policy --role-name <lambda-role-name> --policy-document '{
          "Version": "2012-10-17",
          "Statement": [{"Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]
      }'
      ```
    - **UI Steps**:
      1. IAM > Roles > Lambda role > Trust relationships > Edit > Add `lambda.amazonaws.com`.

29. **Lambda cannot publish to SNS**
    - **Solution**: Add SNS permissions to role.
      ```bash
      aws iam put-role-policy --role-name <lambda-role-name> --policy-name sns-access --policy-document '{
          "Version": "2012-10-17",
          "Statement": [{"Effect": "Allow", "Action": "sns:Publish", "Resource": "arn:aws:sns:<region>:<account-id>:<topic-name>"}]
      }'
      ```
    - **UI Steps**:
      1. Lambda > Permissions > Execution role > IAM Console.
      2. Add inline policy with `sns:Publish`.

## Lambda Performance and Optimization

30. **High Lambda latency**
    - **Solution**: Enable X-Ray, optimize code, increase memory.
      ```bash
      aws lambda update-function-configuration --function-name <function-name> --tracing-config Mode=Active
      aws lambda update-function-configuration --function-name <function-name> --memory-size 512
      ```
    - **UI Steps**:
      1. Lambda > Configuration > Monitoring and operations tools > Enable X-Ray.
      2. General configuration > Edit > Increase memory (e.g., 512 MB).
      3. CloudWatch > X-Ray > Traces > Analyze bottlenecks.

31. **Lambda hitting concurrency limit**
    - **Solution**: Check metrics, set reserved concurrency, optimize execution.
      ```bash
      aws lambda put-function-concurrency --function-name <function-name> --reserved-concurrent-executions 100
      ```
    - **UI Steps**:
      1. Lambda > Configuration > Concurrency > Set “Reserved concurrency” (e.g., 100).
      2. Monitor > Metrics > Check concurrency usage.

32. **High Lambda cold start times**
    - **Solution**: Enable Provisioned Concurrency, use lightweight runtime, optimize code.
      ```bash
      aws lambda put-provisioned-concurrency-config --function-name <function-name> --qualifier <alias-or-version> --provisioned-concurrent-executions 5
      ```
    - **UI Steps**:
      1. Lambda > Configuration > Concurrency > Enable Provisioned Concurrency (e.g., 5).
      2. Code > Optimize initialization.

## Lambda Error Handling and Retries

33. **Lambda errors not logged**
    - **Solution**: Ensure IAM permissions, add logging.
      ```python
      import logging
      logging.getLogger().setLevel(logging.INFO)
      def lambda_handler(event, context):
          try:
              result = some_operation()
              logging.info(f"Result: {result}")
              return {"statusCode": 200, "body": json.dumps(result)}
          except Exception as e:
              logging.error(f"Error: {str(e)}", exc_info=True)
              raise
      ```
      ```json
      {
          "Version": "2012-10-17",
          "Statement": [{"Effect": "Allow", "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"], "Resource": "*"}]
      }
      ```
    - **UI Steps**:
      1. Lambda > Permissions > Ensure CloudWatch Logs permissions.
      2. Monitor > Logs > Verify logs.

34. **Lambda retrying indefinitely**
    - **Solution**: Set retry limit, configure DLQ.
      ```bash
      aws lambda update-function-configuration --function-name my-function --maximum-retry-attempts 2 --dead-letter-config TargetArn=arn:aws:sqs:us-east-1:123456789012:my-dlq-queue
      ```
    - **UI Steps**:
      1. Lambda > Configuration > Asynchronous invocation > Set “Maximum retry attempts” to 2, add SQS/SNS DLQ.

35. **Unexpected DLQ messages in Lambda**
    - **Solution**: Check logs, verify retries, inspect event source.
      ```python
      import json
      import logging
      logging.getLogger().setLevel(logging.INFO)
      def lambda_handler(event, context):
          logging.info(f"Event: {json.dumps(event)}")
          try:
              result = process_event(event)
              return result
          except Exception as e:
              logging.error(f"Error: {str(e)}", exc_info=True)
              raise
      ```
    - **UI Steps**:
      1. Lambda > Monitor > Logs > Check error details.
      2. Configuration > Asynchronous invocation > Verify retries/DLQ.
      3. SQS > [DLQ] > Inspect messages.

## Lambda Integration with Other Services

36. **Lambda not receiving EventBridge events**
    - **Solution**: Verify rule, permissions, logs.
      ```bash
      aws events describe-rule --name my-rule
      aws lambda get-policy --function-name my-function
      ```
      ```json
      {
          "Version": "2012-10-17",
          "Statement": [{"Effect": "Allow", "Action": "lambda:InvokeFunction", "Resource": "arn:aws:lambda:us-east-1:123456789012:function:my-function", "Principal": "events.amazonaws.com"}]
      }
      ```
    - **UI Steps**:
      1. EventBridge > Rules > Verify rule targets Lambda.
      2. Lambda > Permissions > Confirm EventBridge policy.
      3. Monitor > Logs > Check invocation errors.

37. **Lambda failing to write to Kinesis**
    - **Solution**: Verify permissions, stream name, code.
      ```python
      import json
      import boto3
      kinesis = boto3.client("kinesis")
      def lambda_handler(event, context):
          try:
              response = kinesis.put_record(
                  StreamName="my-stream",
                  Data=json.dumps({"data": "example"}),
                  PartitionKey="partition-key"
              )
              return {"statusCode": 200, "body": json.dumps(response)}
          except Exception as e:
              return {"statusCode": 500, "body": json.dumps({"error": str(e)})}
      ```
      ```json
      {
          "Version": "2012-10-17",
          "Statement": [{"Effect": "Allow", "Action": ["kinesis:PutRecord", "kinesis:PutRecords"], "Resource": "arn:aws:kinesis:us-east-1:123456789012:stream/my-stream"}]
      }
      ```
    - **UI Steps**:
      1. Lambda > Permissions > Add Kinesis policy.
      2. Kinesis > Data Streams > Verify stream name.
      3. Test with sample event.

38. **Lambda not processing S3 events**
    - **Solution**: Verify S3 event configuration, permissions, logs.
      ```bash
      aws s3api put-bucket-notification-configuration --bucket my-bucket --notification-configuration '{
          "LambdaFunctionConfigurations": [{"LambdaFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-function", "Events": ["s3:ObjectCreated:*"]}]
      }'
      ```
      ```json
      {
          "Version": "2012-10-17",
          "Statement": [{"Effect": "Allow", "Action": "lambda:InvokeFunction", "Resource": "arn:aws:lambda:us-east-1:123456789012:function:my-function", "Principal": "s3.amazonaws.com"}]
      }
      ```
    - **UI Steps**:
      1. S3 > [Bucket] > Properties > Event notifications > Add Lambda trigger.
      2. Lambda > Permissions > Verify S3 policy.
      3. Monitor > Logs > Check errors.

39. **Lambda failing to access RDS**
    - **Solution**: Verify VPC config, security groups, IAM permissions.
      ```python
      import pymysql
      import os
      def lambda_handler(event, context):
          try:
              conn = pymysql.connect(
                  host=os.environ["RDS_HOST"],
                  user=os.environ["RDS_USER"],
                  password=os.environ["RDS_PASSWORD"],
                  database="my_database"
              )
              with conn.cursor() as cursor:
                  cursor.execute("SELECT * FROM my_table")
                  result = cursor.fetchall()
              conn.close()
              return {"statusCode": 200, "body": json.dumps(result)}
          except Exception as e:
              return {"statusCode": 500, "body": json.dumps({"error": str(e)})}
      ```
      ```json
      {
          "Version": "2012-10-17",
          "Statement": [
              {"Effect": "Allow", "Action": ["rds-db:connect"], "Resource": "arn:aws:rds:us-east-1:123456789012:db:my-database"},
              {"Effect": "Allow", "Action": ["ec2:CreateNetworkInterface", "ec2:DescribeNetworkInterfaces", "ec2:DeleteNetworkInterface"], "Resource": "*"}
          ]
      }
      ```
    - **UI Steps**:
      1. Lambda > Configuration > VPC > Select correct VPC/subnets/security groups.
      2. Security Groups > Allow RDS port (e.g., 3306).
      3. Permissions > Add RDS policy.