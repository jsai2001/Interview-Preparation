# AWS CloudWatch & AWS Lambda On-Call Troubleshooting and Scenario-Based Interview Questions

## Monitoring and Metrics Troubleshooting

1. **A critical application metric is not appearing in CloudWatch. How would you troubleshoot this issue?**
   - **Steps**: Check if the metric is being published correctly, verify IAM permissions, ensure the correct namespace, and confirm the CloudWatch agent or SDK is configured properly.
   - **Code** (Example to publish a custom metric using AWS SDK for Python):
     ```python
     import boto3
     client = boto3.client('cloudwatch', region_name='us-east-1')
     response = client.put_metric_data(
         Namespace='MyApp',
         MetricData=[
             {
                 'MetricName': 'CriticalMetric',
                 'Value': 100.0,
                 'Unit': 'Count',
                 'Dimensions': [{'Name': 'InstanceId', 'Value': 'i-1234567890abcdef0'}]
             }
         ]
     )
     ```
   - **UI Steps**:
     1. Go to CloudWatch > Metrics > All metrics.
     2. Search for the namespace (e.g., `MyApp`) and metric name (`CriticalMetric`).
     3. If missing, check the CloudWatch agent configuration in Systems Manager > Parameter Store or EC2 instance.
     4. Verify IAM role permissions for `cloudwatch:PutMetricData` using IAM > Roles.

2. **You notice that CloudWatch metrics for an EC2 instance are delayed by several minutes. What could be causing this, and how would you resolve it?**
   - **Steps**: Check CloudWatch agent configuration, network latency, and metric collection interval. Adjust the agent’s reporting frequency.
   - **Code** (Update CloudWatch agent config to reduce interval):
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
     1. Go to Systems Manager > Parameter Store > Find `AmazonCloudWatch-agent` config.
     2. Edit the JSON to set `metrics_collection_interval` to a lower value (e.g., 10 seconds).
     3. Restart the CloudWatch agent on the EC2 instance: `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a restart`.

3. **A custom metric you configured is showing incomplete data. What steps would you take to diagnose and fix this?**
   - **Steps**: Verify the metric publishing frequency, check for errors in the application logs, ensure no throttling, and confirm the correct timestamp in the metric data.
   - **Code** (Check for errors in metric publishing):
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
     1. Go to CloudWatch > Metrics > Search for the custom metric.
     2. Check the metric’s data points for gaps.
     3. Go to CloudWatch > Logs > Log groups > Check application logs for errors.
     4. Verify IAM permissions for `cloudwatch:PutMetricData`.

4. **CloudWatch shows no metrics for a newly launched resource. What could be the reasons, and how would you address them?**
   - **Steps**: Confirm the resource is configured to send metrics (e.g., CloudWatch agent for EC2), check IAM permissions, and verify the namespace and region.
   - **Code** (Install and configure CloudWatch agent on EC2):
     ```bash
     sudo yum install amazon-cloudwatch-agent
     sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
     sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a start
     ```
   - **UI Steps**:
     1. Go to CloudWatch > Metrics > Check for the resource’s namespace (e.g., `AWS/EC2` or custom).
     2. If missing, go to Systems Manager > Run Command > Run `AWS-ConfigureCloudWatchAgent`.
     3. Verify IAM role has `CloudWatchAgentServerPolicy` attached in IAM > Roles.

5. **You observe that metric data is missing for certain time periods in CloudWatch. How would you investigate and mitigate this?**
   - **Steps**: Check for application downtime, verify metric publishing code, ensure no API throttling, and confirm the retention period.
   - **Code** (Verify metric retention with AWS CLI):
     ```bash
     aws cloudwatch list-metrics --namespace MyApp
     aws cloudwatch get-metric-statistics --namespace MyApp --metric-name MyMetric --start-time 2025-07-01T00:00:00Z --end-time 2025-07-02T00:00:00Z --period 300 --statistics Average
     ```
   - **UI Steps**:
     1. Go to CloudWatch > Metrics > Select the metric and adjust the time range.
     2. Check for gaps in data points.
     3. Go to CloudWatch > Logs > Application logs to identify downtime or errors.
     4. Adjust retention in CloudWatch > Metrics > Settings if needed.

6. **A CloudWatch metric is reporting incorrect values for an application. How would you identify the root cause?**
   - **Steps**: Validate the metric publishing logic, check for data aggregation issues, and review application logs for errors.
   - **Code** (Debug metric publishing):
     ```python
     import boto3
     client = boto3.client('cloudwatch')
     client.put_metric_data(
         Namespace='MyApp',
         MetricData=[{'MetricName': 'ErrorCount', 'Value': 10.0, 'Unit': 'Count'}]
     )
     print("Metric published successfully")
     ```
   - **UI Steps**:
     1. Go to CloudWatch > Metrics > Select the metric and review data points.
     2. Go to CloudWatch > Logs > Log groups > Check application logs for incorrect metric calculations.
     3. Validate the metric’s unit and dimensions in the metric publishing code.

7. **You receive a report that a metric is not updating in real-time as expected. What troubleshooting steps would you follow?**
   - **Steps**: Check the metric collection interval, verify network connectivity, and ensure no delays in the CloudWatch agent or SDK.
   - **Code** (Reduce collection interval in CloudWatch agent):
     ```json
     {
       "metrics": {
         "namespace": "MyApp",
         "metrics_collected": {
           "mem": {
             "measurement": ["mem_used_percent"],
             "metrics_collection_interval": 5
           }
         }
       }
     }
     ```
   - **UI Steps**:
     1. Go to CloudWatch > Metrics > Check the metric’s last updated timestamp.
     2. Go to Systems Manager > Parameter Store > Update `AmazonCloudWatch-agent` config to lower `metrics_collection_interval`.
     3. Restart the agent: `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a restart`.

8. **A team reports that a custom metric is not visible in the CloudWatch console. What could be the issue, and how would you fix it?**
   - **Steps**: Verify the namespace, check IAM permissions, ensure the metric is published, and confirm the correct region.
   - **Code** (Publish and verify custom metric):
     ```python
     import boto3
     client = boto3.client('cloudwatch')
     client.put_metric_data(
         Namespace='CustomNamespace',
         MetricData=[{'MetricName': 'AppMetric', 'Value': 1.0, 'Unit': 'Count'}]
     )
     metrics = client.list_metrics(Namespace='CustomNamespace')
     print(metrics)
     ```
   - **UI Steps**:
     1. Go to CloudWatch > Metrics > All metrics > Search for `CustomNamespace`.
     2. If not visible, verify the region in the AWS console (top-right corner).
     3. Check IAM permissions for `cloudwatch:PutMetricData` in IAM > Roles.
     4. Publish a test metric and refresh the console.

## Alarms and Notifications

9. **A CloudWatch alarm is not triggering despite the metric crossing the threshold. How would you troubleshoot this?**
   - **Steps**: Verify the alarm’s threshold, evaluation period, metric namespace, and IAM permissions for SNS.
   - **Code** (Check alarm configuration with AWS CLI):
     ```bash
     aws cloudwatch describe-alarms --alarm-names MyAlarm
     aws cloudwatch get-metric-statistics --namespace MyApp --metric-name MyMetric --start-time 2025-07-01T00:00:00Z --end-time 2025-07-02T00:00:00Z --period 60 --statistics Average
     ```
   - **UI Steps**:
     1. Go to CloudWatch > Alarms > Select `MyAlarm`.
     2. Check the metric, threshold, and evaluation period under “Details”.
     3. Go to CloudWatch > Metrics > Verify the metric data crosses the threshold.
     4. Ensure the SNS topic has `sns:Publish` permissions in IAM > Roles.

10. **You receive multiple false positives from a CloudWatch alarm. What steps would you take to refine the alarm configuration?**
    - **Steps**: Increase the evaluation period, adjust the threshold, or use anomaly detection.
    - **Code** (Update alarm to increase evaluation periods):
      ```bash
      aws cloudwatch put-metric-alarm \
        --alarm-name MyAlarm \
        --metric-name CPUUtilization \
        --namespace AWS/EC2 \
        --threshold 80.0 \
        --comparison-operator GreaterThanThreshold \
        --evaluation-periods 3 \
        --period 300 \
        --statistic Average \
        --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic
      ```
    - **UI Steps**:
      1. Go to CloudWatch > Alarms > Select `MyAlarm` > Modify.
      2. Increase “Evaluation Periods” (e.g., from 1 to 3).
      3. Adjust “Threshold” to a higher value if needed.
      4. Save changes and test with sample metric data.

11. **An alarm is stuck in the "INSUFFICIENT_DATA" state. What does this indicate, and how would you resolve it?**
    - **Steps**: Check if the metric exists, verify data points are being published, and ensure the correct namespace and dimensions.
    - **Code** (Verify metric data availability):
      ```bash
      aws cloudwatch get-metric-statistics --namespace MyApp --metric-name MyMetric --start-time 2025-07-01T00:00:00Z --end-time 2025-07-02T00:00:00Z --period 60 --statistics Average
      ```
    - **UI Steps**:
      1. Go to CloudWatch > Alarms > Select the alarm.
      2. Check the metric under “Details” and go to CloudWatch > Metrics to verify data points.
      3. If no data, check the application or CloudWatch agent for publishing issues.
      4. Publish test data using the SDK or CLI to trigger the alarm.

12. **A CloudWatch alarm is sending notifications too frequently. How would you adjust it to reduce noise?**
    - **Steps**: Increase the evaluation period, adjust the threshold, or use a longer period for metric evaluation.
    - **Code** (Modify alarm to reduce notifications):
      ```bash
      aws cloudwatch put-metric-alarm \
        --alarm-name MyAlarm \
        --metric-name ErrorRate \
        --namespace MyApp \
        --threshold 10.0 \
        --comparison-operator GreaterThanThreshold \
        --evaluation-periods 5 \
        --period 300 \
        --statistic Average \
        --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic
      ```
    - **UI Steps**:
      1. Go to CloudWatch > Alarms > Select `MyAlarm` > Modify.
      2. Increase “Evaluation Periods” (e.g., to 5) and “Period” (e.g., to 300 seconds).
      3. Adjust “Threshold” to a higher value if appropriate.
      4. Save changes and monitor notification frequency.

13. **You notice that an SNS notification from a CloudWatch alarm is not reaching the intended recipients. How would you diagnose this issue?**
    - **Steps**: Verify the SNS topic subscription, check IAM permissions, and confirm the endpoint (e.g., email, SMS) is valid.
    - **Code** (Check SNS topic subscriptions):
      ```bash
      aws sns list-subscriptions-by-topic --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic
      aws sns publish --topic-arn arn:aws:sns:us-east-1:123456789012:MyTopic --message "Test message"
      ```
    - **UI Steps**:
      1. Go to SNS > Topics > Select `MyTopic` > Check “Subscriptions” tab.
      2. Verify the endpoint (e.g., email) is confirmed.
      3. Go to CloudWatch > Alarms > Ensure the alarm’s action points to the correct SNS topic ARN.
      4. Test by manually publishing a message in SNS > Topics > Publish message.

14. **A CloudWatch alarm is not transitioning to the "OK" state after the issue is resolved. What could be the cause, and how would you fix it?**
    - **Steps**: Check the alarm’s evaluation period, verify metric data is below the threshold, and ensure no missing data points.
    - **Code** (Verify metric data for alarm state):
      ```bash
      aws cloudwatch get-metric-statistics --namespace MyApp --metric-name MyMetric --start-time 2025-07-01T00:00:00Z --end-time 2025-07-02T00:00:00Z --period 60 --statistics Average
      ```
    - **UI Steps**:
      1. Go to CloudWatch > Alarms > Select the alarm > Check “History” tab for state changes.
      2. Go to CloudWatch > Metrics > Verify the metric is below the threshold.
      3. If stuck, modify the alarm to reset the evaluation period or force a state change by publishing test data.

15. **An alarm is triggering unexpectedly due to a metric spike. How would you determine if it’s a legitimate issue or an anomaly?**
    - **Steps**: Analyze the metric data for patterns, check application logs, and consider using anomaly detection.
    - **Code** (Enable anomaly detection for the metric):
      ```bash
      aws cloudwatch put-anomaly-detector \
        --namespace MyApp \
        --metric-name MyMetric \
        --stat Average \
        --configuration '{"ExcludedTimeRanges": []}'
      ```
    - **UI Steps**:
      1. Go to CloudWatch > Metrics > Select the metric > Graph the data to inspect the spike.
      2. Go to CloudWatch > Logs > Check application logs for errors during the spike.
      3. Go to CloudWatch > Alarms > Create an anomaly detection alarm: Select “Anomaly detection” and configure the model.
      4. Compare the spike against the anomaly detection band to determine if it’s expected.

16. **You’re tasked with setting up an alarm for a mission-critical application. How would you determine the appropriate threshold and evaluation period?**
    - **Steps**: Analyze historical metric data, set a threshold based on normal behavior, and choose an evaluation period to balance sensitivity and noise.
    - **Code** (Create an alarm with calculated threshold):
      ```bash
      aws cloudwatch put-metric-alarm \
        --alarm-name CriticalAppAlarm \
        --metric-name AppLatency \
        --namespace MyApp \
        --threshold 200.0 \
        --comparison-operator GreaterThanThreshold \
        --evaluation-periods 2 \
        --period 60 \
        --statistic Average \
        --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic
      ```
    - **UI Steps**:
      1. Go to CloudWatch > Metrics > Select `AppLatency` in `MyApp` namespace.
      2. Graph the metric and analyze historical data to determine a threshold (e.g., 200ms).
      3. Go to CloudWatch > Alarms > Create alarm > Select the metric > Set threshold and evaluation periods (e.g., 2 periods of 60 seconds).
      4. Add an SNS topic for notifications and save the alarm.
---

## Logs and Log Insights

17. **A team reports that application logs are not appearing in CloudWatch Logs. How would you troubleshoot this issue?**

- **Check CloudWatch Agent Configuration** (if used):
  ```bash
  sudo cat /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
  ```
  Ensure the log group and log stream names match the configuration, e.g.:
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
- **Verify IAM Permissions**:
  ```json
  {
    "Effect": "Allow",
    "Action": [
      "logs:CreateLogGroup",
      "logs:CreateLogStream",
      "logs:PutLogEvents"
    ],
    "Resource": "arn:aws:logs:region:account-id:log-group:my-app-logs:*"
  }
  ```
- **Check Log Group Exists**:
  ```bash
  aws logs describe-log-groups --log-group-name-prefix my-app-logs
  ```
- **UI Steps**:
  1. Go to CloudWatch > Logs > Log groups.
  2. Verify the log group `my-app-logs` exists.
  3. Check Log streams for recent events.
  4. Navigate to IAM > Roles, verify the role has `logs:PutLogEvents` permissions.
  5. Check CloudWatch Agent status: `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status`.

18. **CloudWatch Logs are showing duplicate entries for a single event. What might be causing this, and how would you resolve it?**

- **Check for Multiple Log Writers**:
  ```bash
  sudo ps aux | grep amazon-cloudwatch-agent
  ```
  Stop duplicate agents:
  ```bash
  sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop
  ```
- **Verify Application Log Configuration**:
  Ensure the application isn’t logging to multiple streams. Example for a Node.js app:
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
  Remove duplicate transport configurations.
- **UI Steps**:
  1. Go to CloudWatch > Logs > Log groups > my-app-logs.
  2. Check Log streams for duplicate streams with similar events.
  3. Review application code or CloudWatch Agent configuration files in the EC2 instance or container.

19. **You notice that log data is delayed in CloudWatch Logs. What steps would you take to investigate and fix this?**

- **Check CloudWatch Agent Metrics**:
  ```bash
  aws cloudwatch get-metric-statistics --namespace AWS/Logs --metric-name IncomingLogEvents --dimensions Name=LogGroupName,Value=my-app-logs --start-time $(date -u -Iseconds -d '-1 hour') --end-time $(date -u -Iseconds) --period 300 --statistics Sum
  ```
- **Increase Agent Buffer Size**:
  Edit `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:
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
  Restart agent:
  ```bash
  sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start
  ```
- **UI Steps**:
  1. Go to CloudWatch > Metrics > AWS/Logs.
  2. Check `IncomingLogEvents` for delays.
  3. Navigate to EC2 > Instances, access the instance, and update the CloudWatch Agent configuration file.

20. **A CloudWatch Logs Insights query is returning incomplete results. How would you troubleshoot and optimize the query?**

- **Check Query Syntax**:
  Example query:
  ```logs
  fields @timestamp, @message
  | filter @message like /ERROR/
  | sort @timestamp desc
  | limit 100
  ```
  Ensure correct log group and time range.
- **Verify Time Range**:
  ```bash
  aws logs start-query --log-group-name my-app-logs --start-time $(date -d "-1 hour" +%s) --end-time $(date +%s) --query-string 'fields @timestamp, @message | filter @message like /ERROR/'
  ```
- **Optimize Query**:
  Use specific fields and filters:
  ```logs
  fields @timestamp, @message, errorCode
  | filter errorCode = "500"
  | stats count(*) by errorCode
  ```
- **UI Steps**:
  1. Go to CloudWatch > Logs > Logs Insights.
  2. Select the correct log group (`my-app-logs`).
  3. Adjust the time range (e.g., Last 1 Hour).
  4. Run the query and check for syntax errors or missing fields.
  5. Refine the query by adding specific filters or reducing the result limit.

21. **Log groups are filling up quickly, causing storage issues. How would you manage and optimize log storage in CloudWatch?**

- **Set Log Retention**:
  ```bash
  aws logs put-retention-policy --log-group-name my-app-logs --retention-in-days 7
  ```
- **Export Logs to S3**:
  ```bash
  aws logs create-export-task --log-group-name my-app-logs --from $(date -d "-30 days" +%s000) --to $(date +%s000) --destination my-s3-bucket --destination-prefix logs
  ```
- **UI Steps**:
  1. Go to CloudWatch > Logs > Log groups > my-app-logs.
  2. Click Actions > Edit retention.
  3. Set retention to 7 days.
  4. To export, click Actions > Export data to Amazon S3.
  5. Specify the time range and S3 bucket, then submit.

22. **You’re unable to find specific log events using CloudWatch Logs Insights. What could be the issue, and how would you resolve it?**

- **Verify Log Group and Stream**:
  ```bash
  aws logs describe-log-streams --log-group-name my-app-logs
  ```
- **Check Query Filters**:
  Example query:
  ```logs
  fields @timestamp, @message
  | filter @message like /specific-event/
  | sort @timestamp desc
  ```
  Ensure the filter matches the log format.
- **UI Steps**:
  1. Go to CloudWatch > Logs > Logs Insights.
  2. Select the log group `my-app-logs`.
  3. Run a broad query: `fields @timestamp, @message`.
  4. Refine with filters like `| filter @message like /specific-event/`.
  5. Verify the time range covers the event period.

23. **A CloudWatch Logs subscription filter is not delivering logs to the configured destination. How would you diagnose this?**

- **Check Subscription Filter**:
  ```bash
  aws logs describe-subscription-filters --log-group-name my-app-logs
  ```
- **Verify Destination Permissions**:
  Example IAM policy for Lambda destination:
  ```json
  {
    "Effect": "Allow",
    "Action": "logs:PutSubscriptionFilter",
    "Resource": "arn:aws:logs:region:account-id:log-group:my-app-logs:*"
  }
  ```
- **Test Filter Pattern**:
  ```bash
  aws logs test-metric-filter --log-group-name my-app-logs --filter-pattern "[error=ERROR]"
  ```
- **UI Steps**:
  1. Go to CloudWatch > Logs > Log groups > my-app-logs.
  2. Click Subscription filters tab.
  3. Verify the filter pattern and destination (e.g., Lambda, Kinesis).
  4. Check IAM roles for the destination in IAM > Roles.
  5. Test the filter pattern in the Console’s filter pattern tester.

24. **You need to troubleshoot an application issue using CloudWatch Logs but find that logs are not structured. How would you improve log management?**

- **Implement Structured Logging**:
  Example for a Python app:
  ```python
  import logging
  import json
  logger = logging.getLogger()
  logger.setLevel(logging.INFO)
  def log_event(message, details):
      log_entry = {"message": message, "details": details}
      logger.info(json.dumps(log_entry))
  ```
- **Create Metric Filter**:
  ```bash
  aws logs put-metric-filter --log-group-name my-app-logs --filter-name ErrorCount --filter-pattern "{ $.message = \"ERROR\" }" --metric-transformations metricName=ErrorCount,metricNamespace=AppMetrics,metricValue=1
  ```
- **UI Steps**:
  1. Go to CloudWatch > Logs > Log groups > my-app-logs.
  2. Click Create metric filter.
  3. Define a filter pattern: `{ $.message = "ERROR" }`.
  4. Assign metric name (`ErrorCount`) and namespace (`AppMetrics`).
  5. Update application code to log in JSON format.

## Permissions and Configuration Issues

25. **A user reports they cannot view CloudWatch metrics despite having permissions. How would you troubleshoot their access issue?**

- **Check IAM Policy**:
  ```json
  {
    "Effect": "Allow",
    "Action": [
      "cloudwatch:GetMetricData",
      "cloudwatch:ListMetrics",
      "cloudwatch:DescribeAlarms"
    ],
    "Resource": "*"
  }
  ```
- **Verify User Permissions**:
  ```bash
  aws iam get-user-policy --user-name user-name
  ```
- **UI Steps**:
  1. Go to IAM > Users > user-name.
  2. Check Permissions tab for CloudWatch policies.
  3. Ensure `cloudwatch:GetMetricData` and `cloudwatch:ListMetrics` are included.
  4. Test access in CloudWatch > Metrics.

26. **CloudWatch is not receiving data from a third-party integration. What configuration issues might be causing this, and how would you fix them?**

- **Check Third-Party Role**:
  ```json
  {
    "Effect": "Allow",
    "Action": [
      "logs:PutLogEvents",
      "logs:CreateLogStream"
    ],
    "Resource": "arn:aws:logs:region:account-id:log-group:third-party-logs:*"
  }
  ```
- **Verify Integration Configuration**:
  Example for a third-party tool like Datadog:
  ```bash
  aws logs describe-log-groups --log-group-name-prefix third-party-logs
  ```
- **UI Steps**:
  1. Go to CloudWatch > Logs > Log groups.
  2. Verify the log group `third-party-logs` exists.
  3. Check IAM > Roles for the third-party integration role.
  4. Validate the third-party tool’s configuration (e.g., correct AWS region, log group).

27. **A CloudWatch dashboard is not displaying data for certain resources. What could be the cause, and how would you resolve it?**

- **Check Widget Configuration**:
  Example dashboard JSON:
  ```json
  {
    "widgets": [
      {
        "type": "metric",
        "properties": {
          "metrics": [
            ["AWS/EC2", "CPUUtilization", "InstanceId", "i-1234567890abcdef0"]
          ],
          "region": "us-east-1"
        }
      }
    ]
  }
  ```
  Ensure correct resource IDs and region.
- **UI Steps**:
  1. Go to CloudWatch > Dashboards > dashboard-name.
  2. Edit the widget with missing data.
  3. Verify the metric namespace, dimensions, and region.
  4. Check if the resource (e.g., EC2 instance) is sending metrics (CloudWatch > Metrics).

28. **You receive an error that a CloudWatch Logs group cannot be created due to permissions. How would you diagnose and resolve this?**

- **Check IAM Policy**:
  ```json
  {
    "Effect": "Allow",
    "Action": "logs:CreateLogGroup",
    "Resource": "arn:aws:logs:region:account-id:log-group:*"
  }
  ```
- **Create Log Group**:
  ```bash
  aws logs create-log-group --log-group-name my-app-logs
  ```
- **UI Steps**:
  1. Go to IAM > Roles > role-name.
  2. Add `logs:CreateLogGroup` to the policy.
  3. Navigate to CloudWatch > Logs > Log groups.
  4. Click Create log group and enter `my-app-logs`.

29. **A CloudWatch agent is not sending metrics to CloudWatch. How would you troubleshoot the agent’s configuration?**

- **Check Agent Status**:
  ```bash
  sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
  ```
- **Verify Configuration**:
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
- **Restart Agent**:
  ```bash
  sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start
  ```
- **UI Steps**:
  1. SSH into the EC2 instance.
  2. Check agent status using the above command.
  3. Edit `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`.
  4. Verify metrics in CloudWatch > Metrics > CustomMetrics.

30. **A team reports that they cannot create a new CloudWatch alarm due to a quota limit. How would you handle this situation?**

- **Check Quota**:
  ```bash
  aws service-quotas list-service-quotas --service-code cloudwatch
  ```
- **Request Quota Increase**:
  ```bash
  aws service-quotas request-service-quota-increase --service-code cloudwatch --quota-code L-12345678 --desired-value 1000
  ```
- **UI Steps**:
  1. Go to Service Quotas > AWS services > Amazon CloudWatch.
  2. Find the “Alarms” quota.
  3. Click Request quota increase.
  4. Enter the desired value (e.g., 1000) and submit.

## Performance and Scalability

31. **CloudWatch dashboards are loading slowly for a team. What could be causing this, and how would you optimize performance?**

- **Reduce Widget Complexity**:
  Edit dashboard JSON:
  ```json
  {
    "widgets": [
      {
        "type": "metric",
        "properties": {
          "metrics": [
            ["AWS/EC2", "CPUUtilization", "InstanceId", "i-1234567890abcdef0"]
          ],
          "period": 300,
          "stat": "Average"
        }
      }
    ]
  }
  ```
  Use higher periods (e.g., 300s) and fewer metrics.
- **UI Steps**:
  1. Go to CloudWatch > Dashboards > dashboard-name.
  2. Edit widgets to reduce the number of metrics or increase the period.
  3. Remove unnecessary widgets or use simpler visualizations (e.g., line instead of stack).

32. **You notice high API request latency when querying CloudWatch metrics. How would you investigate and mitigate this?**

- **Check API Throttling**:
  ```bash
  aws cloudwatch get-metric-statistics --namespace AWS/CloudWatch --metric-name ThrottledRequests --start-time $(date -u -Iseconds -d '-1 hour') --end-time $(date -u -Iseconds) --period 300 --statistics Sum
  ```
- **Enable Metric Streams**:
  ```bash
  aws cloudwatch create-metric-stream --name my-metric-stream --role-arn arn:aws:iam::account-id:role/my-stream-role --firehose-arn arn:aws:firehose:region:account-id:deliverystream/my-stream --output-format opentelemetry
  ```
- **UI Steps**:
  1. Go to CloudWatch > Metrics > AWS/CloudWatch.
  2. Check `ThrottledRequests` metric.
  3. Navigate to CloudWatch > Metric streams.
  4. Create a new stream with a Kinesis Data Firehose destination.

33. **A large volume of custom metrics is causing cost spikes in CloudWatch. How would you optimize metric usage to reduce costs?**

- **Reduce Metric Frequency**:
  Example CloudWatch Agent config:
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
- **Delete Unused Metrics**:
  ```bash
  aws cloudwatch delete-metrics --namespace CustomMetrics --metric-name UnusedMetric
  ```
- **UI Steps**:
  1. Go to CloudWatch > Metrics > CustomMetrics.
  2. Identify high-frequency or unused metrics.
  3. Update CloudWatch Agent configuration on the EC2 instance.
  4. Delete unused metrics via AWS CLI or SDK (not available in UI).

34. **CloudWatch Logs Insights queries are timing out for a large dataset. How would you optimize the query or environment?**

- **Optimize Query**:
  ```logs
  fields @timestamp, @message
  | filter @message like /ERROR/
  | limit 100
  ```
  Reduce time range and use specific filters.
- **Increase Timeout (if using SDK)**:
  ```python
  import boto3
  client = boto3.client('logs', config=Config(connect_timeout=5, read_timeout=900))
  response = client.start_query(
      logGroupName='my-app-logs',
      startTime=int((datetime.now() - timedelta(hours=1)).timestamp()),
      endTime=int(datetime.now().timestamp()),
      queryString='fields @timestamp, @message | filter @message like /ERROR/'
  )
  ```
- **UI Steps**:
  1. Go to CloudWatch > Logs > Logs Insights.
  2. Select a shorter time range (e.g., 1 hour).
  3. Refine the query with specific filters (e.g., `| filter @message like /ERROR/`).
  4. Limit results (e.g., `| limit 100`).

35. **You’re tasked with scaling CloudWatch monitoring for a rapidly growing application. What best practices would you follow?**

- **Use Metric Streams**:
  ```bash
  aws cloudwatch create-metric-stream --name app-metric-stream --role-arn arn:aws:iam::account-id:role/my-stream-role --firehose-arn arn:aws:firehose:region:account-id:deliverystream/app-stream --output-format opentelemetry
  ```
- **Set Retention Policies**:
  ```bash
  aws logs put-retention-policy --log-group-name my-app-logs --retention-in-days 14
  ```
- **UI Steps**:
  1. Go to CloudWatch > Metric streams > Create metric stream.
  2. Configure a Kinesis Data Firehose destination.
  3. Navigate to CloudWatch > Logs > Log groups > my-app-logs.
  4. Set retention to 14 days via Actions > Edit retention.

--- 

## Invocation and Execution Issues

36. **A Lambda function is not being invoked as expected by an event source. How would you troubleshoot this issue?**  
   - **Steps**: Check event source mapping, verify event source configuration, ensure Lambda permissions, and review CloudWatch Logs for errors.  
   - **Code (CLI)**:  
     ```bash
     aws lambda get-event-source-mapping --uuid <event-source-mapping-uuid>
     aws lambda get-policy --function-name <function-name>
     aws logs describe-log-streams --log-group-name /aws/lambda/<function-name>
     ```
   - **UI Steps**:  
     1. Go to AWS Lambda Console > Functions > Select function.  
     2. Check "Triggers" tab for event source (e.g., SQS, S3).  
     3. Verify event source configuration (e.g., SQS queue ARN, enabled status).  
     4. Navigate to "Permissions" tab to confirm execution role has access.  
     5. Go to "Monitor" > "Logs" to view CloudWatch Logs for invocation errors.

37. **You notice that a Lambda function is timing out frequently. What steps would you take to diagnose and resolve this?**  
   - **Steps**: Check function timeout setting, review CloudWatch Logs for bottlenecks, optimize code, and increase timeout if needed.  
   - **Code (CLI)**:  
     ```bash
     aws lambda get-function-configuration --function-name <function-name>
     aws lambda update-function-configuration --function-name <function-name> --timeout 60
     ```
   - **UI Steps**:  
     1. Go to AWS Lambda Console > Functions > Select function.  
     2. Under "Configuration" > "General configuration," click "Edit."  
     3. Check current timeout (default 3s) and increase (e.g., to 60s).  
     4. Go to "Monitor" > "Logs" to analyze CloudWatch Logs for timeout causes.  
     5. Optimize code in "Code" tab if bottlenecks are identified.

38. **A Lambda function is failing with a "Task timed out" error. What could be causing this, and how would you fix it?**  
   - **Steps**: Identify long-running operations in code, increase timeout, optimize code, or scale resources.  
   - **Code (CLI)**:  
     ```bash
     aws lambda update-function-configuration --function-name <function-name> --timeout 120
     aws lambda update-function-configuration --function-name <function-name> --memory-size 256
     ```
   - **UI Steps**:  
     1. Go to AWS Lambda Console > Functions > Select function.  
     2. Under "Configuration" > "General configuration," click "Edit."  
     3. Increase timeout (e.g., to 120s) and memory (e.g., to 256 MB).  
     4. Check CloudWatch Logs under "Monitor" > "Logs" for timeout details.  
     5. Edit code in "Code" tab to optimize long-running operations.

39. **A Lambda function is not processing events from an SQS queue. How would you investigate and resolve this?**  
   - **Steps**: Verify SQS trigger configuration, check Lambda permissions, ensure queue visibility timeout, and review CloudWatch Logs.  
   - **Code (CLI)**:  
     ```bash
     aws lambda list-event-source-mappings --function-name <function-name>
     aws lambda get-policy --function-name <function-name>
     aws sqs get-queue-attributes --queue-url <queue-url> --attribute-names VisibilityTimeout
     ```
   - **UI Steps**:  
     1. Go to AWS Lambda Console > Functions > Select function.  
     2. Check "Triggers" tab for SQS trigger and verify queue ARN.  
     3. Go to "Permissions" tab to ensure execution role has `sqs:ReceiveMessage` permission.  
     4. Navigate to SQS Console > Select queue > Check "Visibility timeout."  
     5. Go to Lambda "Monitor" > "Logs" to check for errors.

40. **You observe that a Lambda function is being invoked multiple times for a single event. What might be causing this, and how would you address it?**  
   - **Steps**: Check for duplicate event sources, verify idempotency in code, configure retry settings, and review event source settings.  
   - **Code (CLI)**:  
     ```bash
     aws lambda list-event-source-mappings --function-name <function-name>
     # Example: Add idempotency logic in code
     import json
     def lambda_handler(event, context):
         event_id = event.get('id')
         # Check if event_id processed (e.g., in DynamoDB)
         # Process only if not processed
         return {"statusCode": 200, "body": json.dumps("Processed")}
     ```
   - **UI Steps**:  
     1. Go to AWS Lambda Console > Functions > Select function.  
     2. Check "Triggers" tab for duplicate event sources (e.g., multiple SQS triggers).  
     3. Edit code in "Code" tab to add idempotency (e.g., store event IDs in DynamoDB).  
     4. For SQS, go to SQS Console > Select queue > Adjust "Delivery delay" or "Visibility timeout."  
     5. Check "Monitor" > "Logs" for duplicate invocation logs.

41. **A Lambda function is throwing a runtime error during execution. How would you debug and resolve this issue?**  
   - **Steps**: Enable detailed logging, review CloudWatch Logs, fix code errors, and test locally.  
   - **Code (CLI)**:  
     ```bash
     aws logs describe-log-streams --log-group-name /aws/lambda/<function-name>
     # Example: Add error logging
     import logging
     logger = logging.getLogger()
     logger.setLevel(logging.INFO)
     def lambda_handler(event, context):
         try:
             # Function logic
             logger.info("Processing event")
         except Exception as e:
             logger.error(f"Error: {str(e)}")
             raise e
     ```
   - **UI Steps**:  
     1. Go to AWS Lambda Console > Functions > Select function.  
     2. Go to "Monitor" > "Logs" to view CloudWatch Logs for error details.  
     3. Edit code in "Code" tab to add try-catch and logging (as shown above).  
     4. Test function in "Test" tab with sample events to reproduce the error.  
     5. Deploy updated code and retest.

42. **A Lambda function is not triggering on a scheduled CloudWatch Events rule. How would you troubleshoot this?**  
   - **Steps**: Verify CloudWatch Events rule, check Lambda permissions, ensure rule is enabled, and review logs.  
   - **Code (CLI)**:  
     ```bash
     aws events describe-rule --name <rule-name>
     aws lambda get-policy --function-name <function-name>
     aws events list-targets-by-rule --rule <rule-name>
     ```
   - **UI Steps**:  
     1. Go to CloudWatch Console > Events > Rules > Select rule.  
     2. Verify rule is enabled and targets include the Lambda function.  
     3. Go to Lambda Console > Functions > Select function > "Permissions" tab.  
     4. Ensure execution role has `events:PutEvents` permission.  
     5. Check "Monitor" > "Logs" for invocation logs.

43. **A Lambda function is intermittently failing to process API Gateway requests. What steps would you take to diagnose the issue?**  
   - **Steps**: Check API Gateway configuration, review Lambda logs, verify throttling limits, and test integration.  
   - **Code (CLI)**:  
     ```bash
     aws apigateway get-stage --rest-api-id <api-id> --stage-name <stage-name>
     aws logs describe-log-streams --log-group-name /aws/lambda/<function-name>
     ```
   - **UI Steps**:  
     1. Go to API Gateway Console > Select API > Stages > Check throttling settings.  
     2. Navigate to Lambda Console > Functions > Select function > "Monitor" > "Logs."  
     3. Review CloudWatch Logs for errors (e.g., timeouts, payload issues).  
     4. Test API in API Gateway Console > Resources > Test endpoint.  
     5. Adjust API Gateway throttling or Lambda timeout in respective consoles if needed.

## Permissions and IAM Issues

44. **A Lambda function is failing with a "Permission denied" error when accessing an S3 bucket. How would you troubleshoot and fix this?**  
   - **Steps**: Verify Lambda execution role, add S3 permissions, and test access.  
   - **Code (CLI)**:  
     ```bash
     aws iam get-role-policy --role-name <lambda-role-name> --policy-name <policy-name>
     aws iam put-role-policy --role-name <lambda-role-name> --policy-name s3-access --policy-document '{
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": ["s3:GetObject", "s3:PutObject"],
                 "Resource": "arn:aws:s3:::<bucket-name>/*"
             }
         ]
     }'
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Permissions" tab.  
     2. Click execution role to open IAM Console.  
     3. Add inline policy under "Permissions" > "Add permissions" > Create policy.  
     4. Add `s3:GetObject`, `s3:PutObject` for the bucket ARN.  
     5. Save and test Lambda function.

45. **You receive an error that a Lambda function cannot assume its execution role. What could be the cause, and how would you resolve it?**  
   - **Steps**: Check trust policy of the IAM role, ensure Lambda service trust, and update if needed.  
   - **Code (CLI)**:  
     ```bash
     aws iam get-role --role-name <lambda-role-name>
     aws iam update-assume-role-policy --role-name <lambda-role-name> --policy-document '{
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Principal": {"Service": "lambda.amazonaws.com"},
                 "Action": "sts:AssumeRole"
             }
         ]
     }'
     ```
   - **UI Steps**:  
     1. Go to IAM Console > Roles > Select Lambda role.  
     2. Under "Trust relationships," click "Edit trust policy."  
     3. Ensure `lambda.amazonaws.com` is listed as a trusted service.  
     4. Save and retest Lambda function.

46. **A Lambda function is unable to publish to an SNS topic. How would you diagnose and fix the permissions issue?**  
   - **Steps**: Verify Lambda role permissions for SNS, add necessary policy, and test.  
   - **Code (CLI)**:  
     ```bash
     aws iam put-role-policy --role-name <lambda-role-name> --policy-name sns-access --policy-document '{
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": "sns:Publish",
                 "Resource": "arn:aws:sns:<region>:<account-id>:<topic-name>"
             }
         ]
     }'
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Permissions" tab.  
     2. Click execution role to open IAM Console.  
     3. Add inline policy under "Permissions" > "Add permissions" > Create policy.  
     4. Add `sns:Publish` for the SNS topic ARN.  
     5. Save and test Lambda function.

47. **A user reports they cannot invoke a Lambda function due to access issues. How would you troubleshoot their permissions?**  
   - **Steps**: Check Lambda resource policy, verify user IAM permissions, and update if needed.  
   - **Code (CLI)**:  
     ```bash
     aws lambda get-policy --function-name <function-name>
     aws iam get-user-policy --user-name <user-name> --policy-name <policy-name>
     aws lambda add-permission --function-name <function-name> --statement-id allow-user --action lambda:InvokeFunction --principal <user-arn>
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Permissions" tab.  
     2. Check "Resource-based policy" for user permissions.  
     3. Go to IAM Console > Users > Select user > "Permissions" tab.  
     4. Add `lambda:InvokeFunction` for the function ARN.  
     5. Save and test user access.

48. **A Lambda function is failing to access a DynamoDB table. What IAM configuration might be incorrect, and how would you fix it?**  
   - **Steps**: Verify Lambda role permissions for DynamoDB, add necessary policy, and test.  
   - **Code (CLI)**:  
     ```bash
     aws iam put-role-policy --role-name <lambda-role-name> --policy-name dynamodb-access --policy-document '{
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": ["dynamodb:GetItem", "dynamodb:PutItem"],
                 "Resource": "arn:aws:dynamodb:<region>:<account-id>:table/<table-name>"
             }
         ]
     }'
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Permissions" tab.  
     2. Click execution role to open IAM Console.  
     3. Add inline policy under "Permissions" > "Add permissions" > Create policy.  
     4. Add `dynamodb:GetItem`, `dynamodb:PutItem` for the table ARN.  
     5. Save and test Lambda function.

49. **You notice that a Lambda function’s role has excessive permissions. How would you audit and refine the IAM policy?**  
   - **Steps**: Use IAM Access Analyzer, review policy, remove unnecessary permissions, and test.  
   - **Code (CLI)**:  
     ```bash
     aws accessanalyzer list-analyzed-resources --analyzer-arn <analyzer-arn>
     aws iam put-role-policy --role-name <lambda-role-name> --policy-name refined-policy --policy-document '{
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": ["s3:GetObject"],
                 "Resource": "arn:aws:s3:::<bucket-name>/*"
             }
         ]
     }'
     ```
   - **UI Steps**:  
     1. Go to IAM Console > Access Analyzer > Create analyzer for account.  
     2. Review findings for excessive permissions on Lambda role.  
     3. Go to IAM Console > Roles > Select Lambda role > "Permissions" tab.  
     4. Edit policy to remove unnecessary actions (e.g., replace `*` with specific actions).  
     5. Save and test Lambda function.

## Performance and Optimization

50. **A Lambda function is experiencing high latency during execution. How would you identify the bottleneck and optimize performance?**  
   - **Steps**: Enable X-Ray tracing, review CloudWatch Logs, optimize code, and adjust memory.  
   - **Code (CLI)**:  
     ```bash
     aws lambda update-function-configuration --function-name <function-name> --tracing-config Mode=Active
     aws lambda update-function-configuration --function-name <function-name> --memory-size 512
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Configuration" > "Monitoring and operations tools."  
     2. Enable AWS X-Ray under "Tracing."  
     3. Go to "General configuration" > "Edit" > Increase memory (e.g., to 512 MB).  
     4. Check X-Ray traces in CloudWatch Console > X-Ray > Traces.  
     5. Optimize code in "Code" tab based on trace insights.

51. **You observe that a Lambda function is hitting its concurrency limit. How would you troubleshoot and mitigate this issue?**  
   - **Steps**: Check concurrency metrics, set reserved concurrency, and optimize function execution.  
   - **Code (CLI)**:  
     ```bash
     aws lambda get-function-concurrency --function-name <function-name>
     aws lambda put-function-concurrency --function-name <function-name> --reserved-concurrent-executions 100
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Configuration" > "Concurrency."  
     2. Set "Reserved concurrency" (e.g., to 100).  
     3. Go to "Monitor" > "Metrics" to check concurrency usage.  
     4. Optimize code in "Code" tab to reduce execution time.

52. **A Lambda function is consuming excessive memory. How would you analyze and optimize its resource usage?**  
   - **Steps**: Check CloudWatch metrics, enable X-Ray, optimize code, and adjust memory allocation.  
   - **Code (CLI)**:  
     ```bash
     aws lambda update-function-configuration --function-name <function-name> --memory-size 256
     aws lambda update-function-configuration --function-name <function-name> --tracing-config Mode=Active
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Monitor" > "Metrics."  
     2. Check memory usage metrics.  
     3. Go to "Configuration" > "Monitoring and operations tools" > Enable X-Ray.  
     4. Adjust memory in "General configuration" > "Edit" (e.g., to 256 MB).  
     5. Optimize code in "Code" tab based on X-Ray insights.

53. **Cold start times for a Lambda function are impacting application performance. What strategies would you use to reduce cold starts?**  
   - **Steps**: Enable Provisioned Concurrency, use lightweight runtime, and optimize code.  
   - **Code (CLI)**:  
     ```bash
     aws lambda put-provisioned-concurrency-config --function-name <function-name> --qualifier <alias-or-version> --provisioned-concurrent-executions 5
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Configuration" > "Concurrency."  
     2. Enable "Provisioned Concurrency" and set a value (e.g., 5).  
     3. Use a lightweight runtime (e.g., Python/Node.js) in "Code" tab.  
     4. Optimize initialization code in "Code" tab to reduce startup time.

54. **A Lambda function is processing events slower than expected. How would you optimize its execution time?**  
   - **Steps**: Enable X-Ray, optimize code, increase memory, and batch process events.  
   - **Code (CLI)**:  
     ```bash
     aws lambda update-function-configuration --function-name <function-name> --memory-size 512
     aws lambda update-function-configuration --function-name <function-name> --tracing-config Mode=Active
     # Example: Batch processing
     def lambda_handler(event, context):
         for record in event['Records']:
             # Process each record
             pass
         return {"statusCode": 200}
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Configuration" > "Monitoring and operations tools."  
     2. Enable AWS X-Ray.  
     3. Go to "General configuration" > "Edit" > Increase memory (e.g., to 512 MB).  
     4. Edit code in "Code" tab to process events in batches.  
     5. Check X-Ray traces in CloudWatch Console for bottlenecks.

55. **You need to scale a Lambda function to handle a sudden spike in traffic. What configurations would you adjust?**  
   - **Steps**: Increase reserved concurrency, enable Provisioned Concurrency, and optimize code.  
   - **Code (CLI)**:  
     ```bash
     aws lambda put-function-concurrency --function-name <function-name> --reserved-concurrent-executions 200
     aws lambda put-provisioned-concurrency-config --function-name <function-name> --qualifier <alias-or-version> --provisioned-concurrent-executions 10
     ```
   - **UI Steps**:  
     1. Go to Lambda Console > Functions > Select function > "Configuration" > "Concurrency."  
     2. Set "Reserved concurrency" (e.g., to 200).  
     3. Enable "Provisioned Concurrency" and set a value (e.g., 10).  
     4. Optimize code in "Code" tab for faster execution.  
     5. Monitor scaling in "Monitor" > "Metrics."

--- 

## Error Handling and Retries

56. **A Lambda function is failing, but the error is not being captured in logs. How would you troubleshoot and ensure proper error logging?**

**Solution:**
- Check if the Lambda function has the necessary IAM permissions to write to CloudWatch Logs.
- Verify that logging is enabled and the execution role includes `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents`.
- Add explicit logging in the function code to capture errors.

**Code (Python):**
```python
import logging
import json

# Configure logging
logging.getLogger().setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Your function logic here
        result = some_operation()
        logging.info(f"Operation result: {result}")
        return {"statusCode": 200, "body": json.dumps(result)}
    except Exception as e:
        logging.error(f"Error occurred: {str(e)}", exc_info=True)
        raise
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function].
2. Check the "Execution role" under the Configuration tab > Permissions.
3. Ensure the role has a policy like:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "logs:CreateLogGroup",
                   "logs:CreateLogStream",
                   "logs:PutLogEvents"
               ],
               "Resource": "*"
           }
       ]
   }
   ```
4. Go to the Monitoring tab > View logs in CloudWatch to verify logs are being written.

---

57. **A Lambda function is retrying indefinitely on failure, causing duplicate processing. How would you configure it to handle failures properly?**

**Solution:**
- Set a maximum retry limit and configure a Dead Letter Queue (DLQ) to handle failed events.
- Use an SNS topic or SQS queue as the DLQ.

**Code (AWS CLI to configure retries and DLQ):**
```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --maximum-retry-attempts 2 \
  --dead-letter-config TargetArn=arn:aws:sqs:us-east-1:123456789012:my-dlq-queue
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function].
2. Under Configuration > Asynchronous invocation:
   - Set "Maximum retry attempts" to 2.
   - Under "Dead Letter Queue," select an SQS queue or SNS topic and provide its ARN.
3. Save changes and test by invoking the function asynchronously.

---

58. **You notice that a Lambda function’s dead-letter queue (DLQ) is receiving messages unexpectedly. How would you investigate the root cause?**

**Solution:**
- Check CloudWatch Logs for error details.
- Verify the function’s retry attempts and DLQ configuration.
- Inspect the event source to ensure valid inputs.

**Code (Python to log events for debugging):**
```python
import json
import logging

logging.getLogger().setLevel(logging.INFO)

def lambda_handler(event, context):
    logging.info(f"Received event: {json.dumps(event)}")
    try:
        # Function logic
        result = process_event(event)
        return result
    except Exception as e:
        logging.error(f"Error processing event: {str(e)}", exc_info=True)
        raise
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Monitoring > View logs in CloudWatch.
2. Review logs for errors or invalid event data.
3. Check Configuration > Asynchronous invocation to confirm retry attempts and DLQ settings.
4. Go to SQS > [Your DLQ] to inspect messages and identify event patterns.

---

59. **A Lambda function is failing due to an unhandled exception. How would you modify the function to handle errors gracefully?**

**Solution:**
- Add try-catch blocks to handle exceptions and return meaningful responses.
- Log errors for debugging.

**Code (Python):**
```python
import json
import logging

logging.getLogger().setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Function logic
        result = perform_operation(event)
        return {"statusCode": 200, "body": json.dumps({"message": "Success", "result": result})}
    except ValueError as ve:
        logging.error(f"ValueError: {str(ve)}")
        return {"statusCode": 400, "body": json.dumps({"error": str(ve)})}
    except Exception as e:
        logging.error(f"Unexpected error: {str(e)}", exc_info=True)
        return {"statusCode": 500, "body": json.dumps({"error": "Internal server error"})}
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Code.
2. Update the code with try-catch blocks as shown above.
3. Deploy the function and test with sample events to verify error handling.

---

60. **A Lambda function is not retrying failed events from an event source. How would you troubleshoot and enable retries?**

**Solution:**
- Verify the event source (e.g., SQS, Kinesis) retry configuration.
- Check IAM permissions for the event source.
- Enable retries by configuring the event source mapping.

**Code (AWS CLI to configure retries for SQS):**
```bash
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-queue \
  --batch-size 10 \
  --maximum-retry-attempts 5
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Configuration > Triggers.
2. Select the event source (e.g., SQS).
3. Check "Batch size" and "Maximum retry attempts" settings.
4. If not set, edit the trigger and configure retries (e.g., 5 attempts).
5. Save and test by sending events to the source.

---

61. **You’re tasked with setting up error handling for a new Lambda function. What best practices would you implement?**

**Solution:**
- Use try-catch blocks for error handling.
- Log errors to CloudWatch.
- Configure a DLQ for asynchronous invocations.
- Set appropriate retry limits.

**Code (Python with best practices):**
```python
import json
import logging

logging.getLogger().setLevel(logging.INFO)

def lambda_handler(event, context):
    try:
        # Validate input
        if not event.get("key"):
            raise ValueError("Missing required key")
        
        # Function logic
        result = process_event(event)
        logging.info(f"Processed event: {json.dumps(result)}")
        return {"statusCode": 200, "body": json.dumps({"message": "Success", "result": result})}
    except ValueError as ve:
        logging.error(f"Validation error: {str(ve)}")
        return {"statusCode": 400, "body": json.dumps({"error": str(ve)})}
    except Exception as e:
        logging.error(f"Unexpected error: {str(e)}", exc_info=True)
        raise  # Send to DLQ for async invocations
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > Create function.
2. Add the above code in the Code tab.
3. Under Configuration > Permissions, ensure the execution role includes CloudWatch Logs permissions.
4. Go to Configuration > Asynchronous invocation, set "Maximum retry attempts" to 2, and configure an SQS or SNS DLQ.
5. Deploy and test the function.

---

## Deployment and Configuration

62. **A newly deployed Lambda function is not working as expected. How would you troubleshoot the deployment process?**

**Solution:**
- Check CloudWatch Logs for errors.
- Verify the deployment package (e.g., correct runtime, dependencies).
- Confirm IAM role permissions and environment variables.

**Code (AWS CLI to check function configuration):**
```bash
aws lambda get-function --function-name my-function
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Monitoring > View logs in CloudWatch.
2. Review logs for runtime or dependency errors.
3. Check Configuration > General configuration for runtime and environment variables.
4. Verify Permissions tab for IAM role settings.
5. Redeploy if needed via Code > Deploy.

---

63. **A Lambda function is failing due to a missing dependency. How would you diagnose and resolve this issue?**

**Solution:**
- Check CloudWatch Logs for import/module errors.
- Add the missing dependency to the deployment package.

**Code (Example for Python with missing `requests` library):**
```bash
# Create deployment package
mkdir lambda-package
cd lambda-package
pip install requests -t .
cp ../lambda_function.py .
zip -r ../function.zip .
```

**AWS CLI to update function:**
```bash
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Monitoring > View logs in CloudWatch to identify the missing module.
2. Locally, install the dependency in a folder (e.g., `pip install requests -t .`).
3. Zip the function code and dependencies.
4. Go to Code > Upload from > .zip file, upload the new package, and deploy.

---

64. **You notice that a Lambda function’s environment variables are not being applied correctly. How would you troubleshoot this?**

**Solution:**
- Verify environment variables in the function configuration.
- Check code to ensure variables are accessed correctly.

**Code (Python to access environment variables):**
```python
import os
import json

def lambda_handler(event, context):
    try:
        my_var = os.environ["MY_VARIABLE"]
        return {"statusCode": 200, "body": json.dumps({"message": f"Value: {my_var}"})}
    except KeyError as e:
        return {"statusCode": 500, "body": json.dumps({"error": f"Environment variable {str(e)} not found"})}
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Configuration > Environment variables.
2. Verify the key-value pairs are correct.
3. If incorrect, edit and save the variables.
4. Test the function to ensure variables are read correctly.

---

65. **A Lambda function is using an outdated version after deployment. How would you ensure the latest version is invoked?**

**Solution:**
- Publish a new version and update the alias to point to it.
- Verify the event source is using the correct alias or version.

**Code (AWS CLI to publish and update alias):**
```bash
aws lambda publish-version --function-name my-function
aws lambda update-alias \
  --function-name my-function \
  --name PROD \
  --function-version LATEST
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Code > Deploy.
2. After deployment, go to Versions > Publish new version.
3. Go to Aliases, create or update an alias (e.g., PROD) to point to the new version.
4. Check Triggers to ensure the alias is used.

---

66. **A Lambda function’s timeout setting is causing failures for long-running tasks. How would you adjust the configuration?**

**Solution:**
- Increase the timeout setting in the function configuration.

**Code (AWS CLI to update timeout):**
```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 300
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Configuration > General configuration.
2. Click Edit, set Timeout to a higher value (e.g., 5 minutes or 300 seconds).
3. Save and test the function.

---

67. **You’re tasked with troubleshooting a Lambda function that fails to connect to a VPC resource. What steps would you take?**

**Solution:**
- Verify VPC configuration (subnets, security groups).
- Ensure the Lambda execution role has VPC access permissions.
- Check security group rules and route tables.

**Code (AWS CLI to update VPC config):**
```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --vpc-config SubnetIds=subnet-12345678,subnet-87654321,SecurityGroupIds=sg-12345678
```

**IAM Role Policy:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateNetworkInterface",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DeleteNetworkInterface"
            ],
            "Resource": "*"
        }
    ]
}
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Configuration > VPC.
2. Click Edit, select the correct VPC, subnets, and security groups.
3. Go to Permissions, ensure the execution role includes the above policy.
4. Check VPC > Security Groups and Route Tables for connectivity.

---

## Integration with Other Services

68. **A Lambda function is not receiving events from an EventBridge rule. How would you troubleshoot this issue?**

**Solution:**
- Verify the EventBridge rule’s target and permissions.
- Check CloudWatch Logs for invocation errors.

**Code (AWS CLI to check rule):**
```bash
aws events describe-rule --name my-rule
aws lambda get-policy --function-name my-function
```

**IAM Policy for EventBridge:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:us-east-1:123456789012:function:my-function",
            "Principal": "events.amazonaws.com"
        }
    ]
}
```

**UI Steps:**
1. Go to AWS Management Console > EventBridge > Rules > [Your Rule].
2. Verify the target is set to the correct Lambda function.
3. Go to Lambda > Functions > [Your Function] > Permissions, ensure the EventBridge resource-based policy is present.
4. Check Monitoring > View logs in CloudWatch for errors.

---

69. **A Lambda function is failing to write to a Kinesis stream. What could be causing this, and how would you resolve it?**

**Solution:**
- Check IAM permissions for Kinesis write actions.
- Verify the stream name and function code.

**Code (Python to write to Kinesis):**
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

**IAM Policy:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kinesis:PutRecord",
                "kinesis:PutRecords"
            ],
            "Resource": "arn:aws:kinesis:us-east-1:123456789012:stream/my-stream"
        }
    ]
}
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Permissions.
2. Attach the above IAM policy to the execution role.
3. Verify the stream name in Kinesis > Data Streams.
4. Test the function with a sample event.

---

70. **A Lambda function triggered by S3 events is not processing new uploads. How would you diagnose and fix this?**

**Solution:**
- Verify the S3 event notification configuration.
- Check Lambda permissions and CloudWatch Logs.

**Code (AWS CLI to configure S3 event):**
```bash
aws s3api put-bucket-notification-configuration \
  --bucket my-bucket \
  --notification-configuration '{
      "LambdaFunctionConfigurations": [
          {
              "LambdaFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:my-function",
              "Events": ["s3:ObjectCreated:*"]
          }
      ]
  }'
```

**IAM Policy for S3 Trigger:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:us-east-1:123456789012:function:my-function",
            "Principal": "s3.amazonaws.com"
        }
    ]
}
```

**UI Steps:**
1. Go to AWS Management Console > S3 > [Your Bucket] > Properties > Event notifications.
2. Create or edit an event notification, select the Lambda function, and set events (e.g., All object create events).
3. Go to Lambda > Functions > [Your Function] > Permissions, ensure the S3 resource-based policy is present.
4. Check Monitoring > View logs in CloudWatch for errors.

---

71. **A Lambda function is not interacting correctly with an RDS database. What troubleshooting steps would you follow?**

**Solution:**
- Verify VPC configuration, security group rules, and IAM permissions.
- Check database connection details in the code.

**Code (Python to connect to RDS):**
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

**IAM Policy for RDS:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "rds-db:connect"
            ],
            "Resource": "arn:aws:rds:us-east-1:123456789012:db:my-database"
        }
    ]
}
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Configuration > VPC.
2. Ensure the function is in the same VPC as the RDS instance, with correct subnets and security groups.
3. Go to Security Groups, allow inbound traffic on the RDS port (e.g., 3306 for MySQL).
4. Check Configuration > Environment variables for correct RDS connection details.
5. Verify Permissions for the RDS IAM policy.

---

72. **A Lambda function is unable to send metrics to CloudWatch. How would you troubleshoot and resolve this issue?**

**Solution:**
- Verify IAM permissions for CloudWatch metrics.
- Check the function code for correct metric publishing.

**Code (Python to send custom metrics):**
```python
import boto3
import json

cloudwatch = boto3.client("cloudwatch")

def lambda_handler(event, context):
    try:
        cloudwatch.put_metric_data(
            Namespace="MyApp",
            MetricData=[
                {
                    "MetricName": "CustomMetric",
                    "Value": 1.0,
                    "Unit": "Count"
                }
            ]
        )
        return {"statusCode": 200, "body": json.dumps({"message": "Metric sent"})}
    except Exception as e:
        return {"statusCode": 500, "body": json.dumps({"error": str(e)})}
```

**IAM Policy:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData"
            ],
            "Resource": "*"
        }
    ]
}
```

**UI Steps:**
1. Go to AWS Management Console > Lambda > Functions > [Your Function] > Permissions.
2. Attach the above IAM policy to the execution role.
3. Check Monitoring > View logs in CloudWatch for errors.
4. Go to CloudWatch > Metrics, search for the custom namespace (e.g., MyApp) to verify metrics.

---

73. **You need to integrate a Lambda function with a new AWS service. What steps would you take to ensure successful integration?**

**Solution:**
- Identify the service’s event structure and required IAM permissions.
- Configure the event source and test the integration.

**Code (Example for DynamoDB Streams):**
```python
import json

def lambda_handler(event, context):
    try:
        for record in event["Records"]:
            print(f"Processing record: {json.dumps(record)}")
        return {"statusCode": 200, "body": json.dumps({"message": "Processed"})}
    except Exception as e:
        return {"statusCode": 500, "body": json.dumps({"error": str(e)})}
```

**IAM Policy for DynamoDB Streams:**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetRecords",
                "dynamodb:GetShardIterator",
                "dynamodb:DescribeStream",
                "dynamodb:ListStreams"
            ],
            "Resource": "arn:aws:dynamodb:us-east-1:123456789012:table/my-table/stream/*"
        },
        {
            "Effect": "Allow",
            "Action": "lambda:InvokeFunction",
            "Resource": "arn:aws:lambda:us-east-1:123456789012:function:my-function",
            "Principal": "dynamodb.amazonaws.com"
        }
    ]
}
```

**UI Steps:**
1. Go to AWS Management Console > [Service] (e.g., DynamoDB > Tables > [Your Table] > Exports and streams).
2. Enable the stream and note the ARN.
3. Go to Lambda > Functions > [Your Function] > Configuration > Triggers, add the service (e.g., DynamoDB).
4. Select the stream and configure batch size.
5. Go to Permissions, ensure the execution role has the required permissions.
6. Test the integration by generating events in the source service.