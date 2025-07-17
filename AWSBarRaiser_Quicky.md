
---

## **Jeevan Sai Kanaparthi - Interview Preparation Notes**

**Current Role**: DevOps and Data Engineering professional at NielsenIQ, specializing in AWS, Kubernetes, Terraform, Ansible, GitHub Actions, Python, and Snowflake SQL. Focus on cloud infrastructure, CI/CD pipelines, monitoring, and data processing.

**Key Commands**:
```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image myapp:latest
sudo fsck -y /dev/sdb1
```

---

### **Role and Responsibilities**
- **Cloud Infrastructure**: Manage AWS resources (EC2, EKS, S3, RDS) using Terraform and Ansible for provisioning and configuration.
- **CI/CD Pipelines**: Automate build and deployment workflows with GitHub Actions, reducing deployment time by 25% and misalignments by 30%.
- **Kubernetes**: Orchestrate containerized applications, ensuring smooth deployments and eliminating errors via Docker.
- **Monitoring**: Configure AWS CloudWatch for custom metrics and alarms, improving system reliability by 25%.
- **Data Engineering**: Develop Snowflake SQL models, optimize queries, and validate data with Python, boosting processing efficiency by 20% and reporting accuracy by 45%.
- **Automation**: Write Python scripts (Boto3) for infrastructure and monitoring automation, reducing provisioning time by 20%.

**Leadership Experience**:
- Led a team of three on a Snowflake SQL and data modeling project based on Data Science Specification.
- Coordinated tasks, conducted sync-ups, and mentored team on logic and requirements.
- Validated data with Python, guided model development, and drove end-to-end process (development to deployment), enhancing leadership and mentoring skills.

**Introduction**:
DevOps and Data Engineering professional with expertise in AWS, Kubernetes, Terraform, Ansible, GitHub Actions, Python, and Snowflake SQL. Led model development and deployment, achieving 45% improved reporting accuracy and 20% faster data processing.

---

### **Amazon’s Leadership Principles**
1. **Customer Obsession**: Prioritize customer needs (e.g., 45% improved reporting accuracy for client analytics).
2. **Ownership**: Act for the company’s benefit (e.g., led end-to-end model development).
3. **Invent and Simplify**: Streamlined CI/CD pipelines and Snowflake queries.
4. **Are Right, A Lot**: Made data-driven decisions in query optimization.
5. **Learn and Be Curious**: Pursued AWS and Kubernetes expertise.
6. **Hire and Develop the Best**: Mentored team, boosting productivity by 15%.
7. **Insist on Highest Standards**: Achieved error-free deployments.
8. **Think Big**: Designed scalable data aggregation systems.
9. **Bias for Action**: Quickly resolved CPU spikes in production.
10. **Frugality**: Reduced infrastructure costs by 25%.
11. **Earn Trust**: Built trust through transparent communication.
12. **Dive Deep**: Analyzed logs to fix deployment issues.
13. **Have Backbone; Disagree and Commit**: Guided team on model logic.
14. **Deliver Results**: Improved reliability by 25% and processing by 20%.
15. **Strive to Be Earth’s Best Employer**: Fostered team growth.
16. **Success and Scale**: Took responsibility for scalable solutions.

---

### **Measuring Success**
- **Customer-Centric Metrics**: 45% improved model consistency, faster report delivery.
- **Technical Performance**: 20% faster Snowflake queries, 30% fewer deployment errors.
- **Operational Efficiency**: 15% team productivity increase, 25% cost reduction.
- **Client Feedback**: Ensured satisfaction via reliable analytics.
- **Continuous Improvement**: Set up CloudWatch for proactive monitoring.

### **Blue-Green Deployment Process**
- **Code Commit**: Developers push to GitHub/Bitbucket.
- **CI/CD Trigger**: GitHub Actions builds/tests Docker containers.
- **Green Deployment**: New version deployed to Green environment (Kubernetes/EKS).
- **Validation**: Test Green environment with CloudWatch monitoring.
- **Traffic Switch**: Route traffic to Green via AWS ELB/Kubernetes Ingress.
- **Monitoring**: Track metrics (CPU, latency) in CloudWatch.
- **Rollback**: Switch back to Blue if issues arise.

**Metrics Impact**:
- **Deployment Frequency (DF)**: Faster via automation.
- **Lead Time for Changes (LTTC)**: Reduced by 25%.
- **Change Failure Rate (CFR)**: Eliminated errors.
- **Mean Time to Restore (MTTR)**: Minimized via rollback.
- **Post-Level Availability**: Ensured via zero-downtime switch.

---

### **Career Goals (5 Years)**
- **Vision**: Lead Cloud Architect, driving innovative DevOps solutions.
- **Plan**: Master serverless computing, AI-driven automation, and earn AWS Solutions Architect Professional certification.
- **Impact**: Optimize infrastructure costs, enhance user experiences, and mentor teams, building on 15% productivity gains at NielsenIQ.

---

### **Why Leave Current Role?**
- Seeking complex challenges to grow expertise in large-scale cloud solutions.
- Delivered 25% faster deployments and 25% cost reductions at NielsenIQ but want broader impact.
- Maintain trust through transparency about career goals.

---

### **Why Amazon?**
- Aligns with **Customer Obsession** via reliable solutions (e.g., 45% reporting accuracy).
- Inspires **Think Big** with global-scale systems using AWS, Kubernetes, and Python.
- Salary expectation: 18 LPA (within 24 LPA budget).
- Opportunity to drive efficiency and customer satisfaction.

---

### **Primary Metrics**
- **Deployment Frequency (DF)**: Track via GitHub Actions for rapid delivery.
- **Lead Time for Changes (LTTC)**: Monitor via Git logs (hours).
- **Change Failure Rate (CFR)**: Low (0–15%) via incident tools.
- **Mean Time to Restore (MTTR)**: <1 hour via CloudWatch.
- **Service Failure Rate**: Low via Prometheus/Grafana.
- **Post-Level Availability**: 99.99% via Grafana SLOs.
- **Request Latency**: <200ms via CloudWatch.
- **Error Rate**: Low via ELK/Splunk.
- **CSAT/NPS**: High via client surveys.

**Secondary Metrics**:
- **Pile Up**: Minimal via CI/CD queue tracking.
- **Backup Rate**: Reliable via backup logs.
- **Clearance Sale**: Resolve technical debt via Jira.
- **Canary/Non-Canary**: Compare via Grafana.
- **Deployment for Zone**: Consistent via CloudWatch.
- **Identify the Pattern**: Detect trends via Grafana/Prometheus.
- **Code Review Time**: Fast via GitHub.
- **Test Coverage**: >80% via Codecov.
- **Infrastructure Utilization**: Optimize via Grafana (CPU, memory).
- **CPU Spiking Up**: Detect via CloudWatch/Grafana.
- **Ring-Based Deployments**: Stable via Kubernetes.
- **Last Successful Control**: Track via Grafana.
- **Incident Response Time**: Fast via OpsGenie.
- **Build Failure Rate**: Low via GitHub Actions.

---

### **Improving Operations**
- **CI/CD Optimization**: Reduced deployment time by 25% and misalignments by 30%.
- **Containerization**: Eliminated errors via Docker/Kubernetes.
- **Infrastructure Automation**: Cut provisioning time by 20% and costs by 25% with Terraform/Ansible.
- **Monitoring**: Improved reliability by 25% with CloudWatch.
- **Data Processing**: Boosted efficiency by 20% and accuracy by 45% with Snowflake/Python.
- **Team Mentoring**: Increased productivity by 15%.
- **API Development**: Enhanced communication with Flask APIs.

---

### **Deployment Strategies**
1. **Blue-Green (Used)**:
   - Two environments (Blue: current, Green: new).
   - Zero downtime, low MTTR, high availability.
   - Tools: AWS ELB, Kubernetes, Terraform, CloudWatch.
2. **Canary**:
   - Gradual rollout to small user subset.
   - Reduces CFR via early issue detection.
3. **Rolling**:
   - Incremental pod updates in Kubernetes.
   - Resource-efficient but risks compatibility issues.
4. **Feature Flag**:
   - Hide features behind toggles.
   - Flexible, reduces CFR via selective rollout.
5. **A/B Testing**:
   - Compare versions for user feedback.
   - Improves CSAT/NPS.
6. **Shadow**:
   - Test new version with duplicated traffic.
   - Safe, reduces CFR without user impact.
7. **Recreate**:
   - Terminate old version, deploy new.
   - High risk, not used due to downtime.

**Comparison with Blue-Green**:
- **Canary**: Safer for testing, slower LTTC.
- **Rolling**: Resource-efficient, risks disruptions.
- **Feature Flag**: Ideal for feature releases.
- **A/B**: User-focused, complements data systems.
- **Shadow**: Safe testing, high resource use.
- **Recreate**: Unsuitable for high availability.

---

### **Production Issue 1: Kubernetes ImagePullBackOff**

**Scenario**: A Python Flask API pod in AWS EKS fails to start, showing `ImagePullBackOff` due to issues pulling from a private AWS ECR repository, disrupting real-time data aggregation.

**Symptoms**:
- `kubectl get pods` shows `ImagePullBackOff` or `ErrImagePull`.
- Logs indicate failure to pull from ECR.
- CloudWatch alerts triggered for pod health.

**Step-by-Step Solution**:

1. **Verify Pod Status**:
   ```bash
   kubectl get pods -n <namespace>
   kubectl describe pod <pod-name> -n <namespace>
   ```
   **Error Example**:
   ```
   Failed to pull image "<aws-ecr-repo>/app:latest": pull access denied, repository does not exist or may require 'docker login'
   ```

2. **Check Image Name/Tag**:
   ```yaml
   spec:
     containers:
     - name: app-container
       image: <aws-account-id>.dkr.ecr.<region>.amazonaws.com/app:latest
   ```
   **Fix**:
   - Verify image in ECR:
     ```bash
     aws ecr describe-images --repository-name app --region <region>
     ```
   - Correct image name/tag and reapply:
     ```bash
     kubectl apply -f <deployment-file>.yaml
     ```

3. **Verify Registry Authentication**:
   - Check for `imagePullSecrets`:
     ```yaml
     spec:
       imagePullSecrets:
       - name: ecr-secret
     ```
   - Create/update ECR secret:
     ```bash
     aws ecr get-login-password --region <region> | kubectl create secret docker-registry ecr-secret \
       --docker-server=<aws-account-id>.dkr.ecr.<region>.amazonaws.com \
       --docker-username=AWS \
       --docker-password=$(aws ecr get-login-password --region <region>) \
       --namespace <namespace>
     ```
   - Automate token refresh with Python:
     ```python
     import boto3
     import subprocess
     import base64

     def update_ecr_secret():
         ecr_client = boto3.client('ecr', region_name='<region>')
         token = ecr_client.get_authorization_token()['authorizationData'][0]
         password = base64.b64decode(token['authorizationToken']).decode('utf-8').split(':')[1]
         cmd = f"""kubectl create secret docker-registry ecr-secret \
             --docker-server={token['proxyEndpoint']} \
             --docker-username=AWS \
             --docker-password={password} \
             --namespace <namespace> --dry-run=client -o yaml | kubectl apply -f -"""
         subprocess.run(cmd, shell=True, check=True)

     update_ecr_secret()
     ```

4. **Validate Network Connectivity**:
   - Test connectivity:
     ```bash
     kubectl run -it test-pod --image=alpine --namespace <namespace> -- sh
     wget <aws-account-id>.dkr.ecr.<region>.amazonaws.com
     ```
   - Check EKS security groups and VPC endpoints:
     ```bash
     aws ec2 describe-vpc-endpoints --region <region>
     ```

5. **Check Resource Limits**:
   - Verify node resources:
     ```bash
     kubectl get nodes
     kubectl describe node <node-name>
     kubectl top nodes
     ```
   - Adjust pod limits:
     ```yaml
     resources:
       limits:
         cpu: "500m"
         memory: "512Mi"
       requests:
         cpu: "200m"
         memory: "256Mi"
     ```

6. **Monitor and Validate**:
   - Reapply deployment:
     ```bash
     kubectl apply -f <deployment-file>.yaml
     ```
   - Monitor pod status:
     ```bash
     kubectl get pods -n <namespace> -w
     kubectl logs <pod-name> -n <namespace>
     ```
   - Verify CloudWatch metrics for pod health.

7. **Prevent Future Issues**:
   - Add image validation to CI/CD:
     ```yaml
     name: Validate ECR Image
     on: [push]
     jobs:
       validate-image:
         runs-on: ubuntu-latest
         steps:
           - name: Configure AWS Credentials
             uses: aws-actions/configure-aws-credentials@v1
             with:
               aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
               aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
               aws-region: <region>
           - name: Check Image Existence
             run: |
               aws ecr describe-images --repository-name app --image-ids imageTag=latest || exit 1
     ```

**Outcome**:
- Pod transitions to `Running`, restoring data aggregation.
- Automation scripts prevent future issues.
- Team upskilled via documentation and training.

---

### **Production Issue 2: Pod Stuck in Pending/ContainerCreating**

**Scenario**: A pod in AWS EKS is stuck in `Pending` or `ContainerCreating` due to resource limits or image pull issues, causing deployment delays.

**Symptoms**:
- `kubectl get pods` shows `Pending` or `ContainerCreating`.
- Events indicate:
  ```
  FailedScheduling: 0/10 nodes are available: 10 Insufficient cpu, 10 Insufficient memory.
  ```
  or
  ```
  Failed to pull image: no space left on device.
  ```

**Step-by-Step Solution**:

1. **Verify Pod Status**:
   ```bash
   kubectl get pods -n <namespace>
   kubectl describe pod <pod-name> -n <namespace>
   ```

2. **Check Resource Requests/Limits**:
   ```yaml
   spec:
     containers:
     - name: app-container
       image: my-app:latest
       resources:
         requests:
           cpu: "250m"
           memory: "256Mi"
         limits:
           cpu: "500m"
           memory: "512Mi"
   ```
   **Fix**:
   - Adjust to match cluster capacity and reapply:
     ```bash
     kubectl apply -f <pod-file>.yaml
     ```

3. **Check Cluster Resources**:
   ```bash
   kubectl get nodes -o wide
   kubectl describe nodes
   kubectl top nodes
   ```
   **Fix**:
   - Scale node group:
     ```bash
     eksctl scale nodegroup --cluster=<cluster-name> --nodes=<desired-nodes> --name=<nodegroup-name>
     ```

4. **Investigate ContainerCreating**:
   - Check disk usage:
     ```bash
     kubectl exec -it <node-name> -- df -h
     ```
   - Clean up:
     ```bash
     kubectl exec -it <node-name> -- docker system prune -f
     ```
   - Optimize Dockerfile:
     ```dockerfile
     FROM python:3.9-slim AS builder
     FROM python:3.9-slim
     COPY --from=builder /app /app
     ```

5. **Check Node Taints**:
   ```bash
   kubectl describe nodes | grep -i taint
   ```
   **Fix**:
   - Add tolerations:
     ```yaml
     spec:
       tolerations:
       - key: "key1"
         operator: "Equal"
         value: "value1"
         effect: "NoSchedule"
     ```

6. **Verify Image Pull**:
   - Check for `ErrImagePull` and ensure `imagePullSecrets` is set.
   - Test manually:
     ```bash
     kubectl exec -it <node-name> -- docker pull my-app:latest
     ```

7. **Monitor and Validate**:
   ```bash
   kubectl get pods -n <namespace> -w
   aws cloudwatch get-metric-data --metric-data-queries file://metrics.json
   ```

8. **Prevent Future Issues**:
   - Set resource quotas:
     ```yaml
     apiVersion: v1
     kind: ResourceQuota
     metadata:
       name: compute-quota
       namespace: <namespace>
     spec:
       hard:
         requests.cpu: "4"
         requests.memory: "8Gi"
         limits.cpu: "8"
         limits.memory: "16Gi"
     ```
   - Use Horizontal Pod Autoscaler (HPA):
     ```yaml
     apiVersion: autoscaling/v2
     kind: HorizontalPodAutoscaler
     metadata:
       name: app-hpa
     spec:
       scaleTargetRef:
         apiVersion: apps/v1
         kind: Deployment
         name: app-deployment
       minReplicas: 2
       maxReplicas: 10
       metrics:
       - type: Resource
         resource:
           name: cpu
           target:
             type: Utilization
             averageUtilization: 70
     ```

**Outcome**:
- Pod transitions to `Running`, resolving delays.
- Cluster optimized with quotas and HPA, improving reliability by 25%.

---

### **AWS CloudWatch Monitoring**

**Setup**:
- **IAM Role**:
  ```hcl
  resource "aws_iam_role" "cloudwatch_role" {
    name = "CloudWatchRole"
    assume_role_policy = jsonencode({
      Version = "2012-10-17"
      Statement = [{
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = { Service = "ec2.amazonaws.com" }
      }]
    })
  }
  resource "aws_iam_role_policy_attachment" "cloudwatch_policy" {
    role       = aws_iam_role.cloudwatch_role.name
    policy_arn = "arn:aws:iam::aws:policy/CloudWatchFullAccess"
  }
  ```

- **Log Group**:
  ```hcl
  resource "aws_cloudwatch_log_group" "app_logs" {
    name              = "/nielseniq/data-aggregation"
    retention_in_days = 7
  }
  ```

- **CloudWatch Agent**:
  ```hcl
  resource "aws_ssm_document" "install_cloudwatch_agent" {
    name          = "InstallCloudWatchAgent"
    document_type = "Command"
    content = jsonencode({
      schemaVersion = "2.2"
      description   = "Install and configure CloudWatch Agent"
      mainSteps = [{
        action = "aws:runShellScript"
        name   = "installAgent"
        inputs = {
          runCommand = [
            "sudo yum install -y amazon-cloudwatch-agent",
            "sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard"
          ]
        }
      }]
    })
  }
  ```

- **Metric Alarm**:
  ```hcl
  resource "aws_cloudwatch_metric_alarm" "high_cpu" {
    alarm_name          = "HighCPUUtilization"
    comparison_operator = "GreaterThanThreshold"
    evaluation_periods  = 2
    metric_name         = "CPUUtilization"
    namespace           = "AWS/EC2"
    period              = 300
    statistic           = "Average"
    threshold           = 80
    alarm_description   = "Triggers when EC2 CPU exceeds 80% for 10 minutes"
    dimensions = {
      InstanceId = module.ec2_instance.instance_id
    }
    alarm_actions = [aws_sns_topic.alerts.arn]
  }
  ```

- **Custom Metric**:
  ```hcl
  resource "aws_cloudwatch_log_metric_filter" "query_latency" {
    name           = "QueryLatency"
    pattern        = "query_duration_ms"
    log_group_name = aws_cloudwatch_log_group.app_logs.name
    metric_transformation {
      name      = "QueryLatency"
      namespace = "NielsenIQ/App"
      value     = "1"
    }
  }
  ```

- **Link to EC2**:
  ```hcl
  resource "aws_iam_instance_profile" "ec2_profile" {
    name = "EC2CloudWatchProfile"
    role = aws_iam_role.cloudwatch_role.name
  }
  module "ec2_instance" {
    source              = "./modules/ec2"
    ami                 = "ami-0abcdef1234567890"
    instance_type       = "t3.medium"
    subnet_id           = aws_subnet.main.id
    iam_instance_profile = aws_iam_instance_profile.ec2_profile.name
  }
  ```

- **Log Configuration**:
  ```json
  {
    "logs": {
      "logs_collected": {
        "files": {
          "collect_list": [
            {
              "file_path": "/var/log/data-aggregation/app.log",
              "log_group_name": "/nielseniq/data-aggregation",
              "log_stream_name": "{instance_id}"
            }
          ]
        }
      }
    }
  }
  ```

- **SNS Alerts**:
  ```hcl
  resource "aws_sns_topic" "alerts" {
    name = "nielseniq-alerts"
  }
  resource "aws_sns_topic_subscription" "email" {
    topic_arn = aws_sns_topic.alerts.arn
    protocol  = "email"
    endpoint  = "team@nielseniq.com"
  }
  ```

**Accessing Logs**:
- **Console**: Navigate to CloudWatch > Log Groups > `/nielseniq/data-aggregation`. Filter logs (e.g., `ERROR` or `query_duration_ms > 1000`).
- **CLI**:
  ```bash
  aws logs filter-log-events --log-group-name /nielseniq/data-aggregation --filter-pattern "ERROR"
  aws cloudwatch get-metric-data \
    --metric-data-queries '[{
        "Id": "m1",
        "MetricStat": {
            "Metric": {
                "Namespace": "AWS/EC2",
                "MetricName": "CPUUtilization",
                "Dimensions": [{"Name": "InstanceId", "Value": "i-1234567890abcdef0"}]
            },
            "Period": 300,
            "Stat": "Average"
        }
    }]' \
    --start-time $(date -u -d "1 hour ago" +%s) \
    --end-time $(date -u +%s)
  ```

**Custom Metrics/Alarms**:
- **Pod Crashes**:
  ```hcl
  resource "aws_cloudwatch_log_metric_filter" "pod_crashes" {
    name           = "EKSPodCrashes"
    pattern        = "CrashLoopBackOff"
    log_group_name = aws_cloudwatch_log_group.deployment_logs.name
    metric_transformation {
      name      = "PodCrashCount"
      namespace = "NielsenIQ/EKS"
      value     = "1"
    }
  }
  ```

- **Snowflake Query Latency**:
  ```hcl
  resource "aws_cloudwatch_log_metric_filter" "query_latency" {
    name           = "SnowflakeQueryLatency"
    pattern        = "query_duration_ms"
    log_group_name = aws_cloudwatch_log_group.deployment_logs.name
    metric_transformation {
      name      = "QueryLatency"
      namespace = "NielsenIQ/Snowflake"
      value     = "$query_duration_ms"
    }
  }
  ```

- **ALB Latency Alarm**:
  ```hcl
  resource "aws_cloudwatch_metric_alarm" "high_alb_latency" {
    alarm_name          = "HighALBLatency"
    comparison_operator = "GreaterThanThreshold"
    evaluation_periods  = 2
    metric_name         = "RequestLatency"
    namespace           = "AWS/ApplicationELB"
    period              = 300
    statistic           = "Average"
    threshold           = 1000
    alarm_description   = "Triggers if ALB latency exceeds 1 second"
    dimensions = {
      LoadBalancer = aws_lb.app_alb.arn_suffix
    }
    alarm_actions = [aws_sns_topic.alerts.arn]
  }
  ```

- **Snowflake Query Failure Alarm**:
  ```hcl
  resource "aws_cloudwatch_metric_alarm" "query_failure" {
    alarm_name          = "SnowflakeQueryFailure"
    comparison_operator = "GreaterThanThreshold"
    evaluation_periods  = 1
    metric_name         = "QueryLatency"
    namespace           = "NielsenIQ/Snowflake"
    period              = 300
    statistic           = "Average"
    threshold           = 2000
    alarm_description   = "Triggers if Snowflake query latency exceeds 2 seconds"
    alarm_actions       = [aws_sns_topic.alerts.arn]
  }
  ```

- **Dashboard**:
  ```hcl
  resource "aws_cloudwatch_dashboard" "deployment_health" {
    dashboard_name = "NielsenIQ-Deployment-Health"
    dashboard_body = jsonencode({
      widgets = [
        {
          type   = "metric"
          x      = 0
          y      = 0
          width  = 12
          height = 6
          properties = {
            metrics = [
              ["AWS/ApplicationELB", "RequestLatency", "LoadBalancer", aws_lb.app_alb.arn_suffix],
              ["NielsenIQ/EKS", "PodCrashCount"],
              ["NielsenIQ/Snowflake", "QueryLatency"]
            ]
            view = "timeSeries"
            title = "Deployment and Application Health"
          }
        }
      ]
    })
  }
  ```

**CLI for Custom Metrics**:
```bash
aws cloudwatch put-metric-data \
  --namespace "PodMetrics" \
  --metric-name "CPUUtilization" \
  --dimensions PodName=my-pod \
  --value 65.5 \
  --unit Percent

aws cloudwatch put-metric-data \
  --namespace "PodMetrics" \
  --metric-name "MemoryUtilization" \
  --dimensions PodName=my-pod \
  --value 78.2 \
  --unit Percent

aws cloudwatch put-metric-alarm \
  --alarm-name "CPUUtilizationAlarm" \
  --metric-name "CPUUtilization" \
  --namespace "PodMetrics" \
  --statistic "Average" \
  --period 300 \
  --threshold 80 \
  --comparison-operator "GreaterThanThreshold" \
  --evaluation-periods 1 \
  --dimensions Name=PodName,Value=my-pod \
  --alarm-actions <SNS-ARN> \
  --actions-enabled

aws cloudwatch put-metric-alarm \
  --alarm-name "MemoryUtilizationAlarm" \
  --metric-name "MemoryUtilization" \
  --namespace "PodMetrics" \
  --statistic "Average" \
  --period 300 \
  --threshold 90 \
  --comparison-operator "GreaterThanThreshold" \
  --evaluation-periods 1 \
  --dimensions Name=PodName,Value=my-pod \
  --alarm-actions <SNS-ARN> \
  --actions-enabled
```

---

### **Snowflake SQL Techniques**

1. **Window Functions**:
   - **ROW_NUMBER**:
     ```sql
     SELECT order_id, customer_id, order_date, 
            ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS order_sequence 
     FROM sales_data;
     ```
     **Benefit**: Tracks order sequences, reducing query time for purchase pattern analysis.
   - **RANK**:
     ```sql
     SELECT product_id, sales_amount, 
            RANK() OVER (PARTITION BY region ORDER BY sales_amount DESC) AS sales_rank 
     FROM product_sales;
     ```
     **Benefit**: Identifies top products per region, simplifying leaderboards.
   - **LAG**:
     ```sql
     SELECT order_id, order_date, sales_amount, 
            LAG(sales_amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS previous_sale 
     FROM sales_data;
     ```
     **Benefit**: Enables sales growth calculations, optimizing time-series analysis.
   - **Outcome**: Reduced query time by ~15%.

2. **Query Optimization with Clustering Keys**:
   ```sql
   ALTER TABLE sales_data CLUSTER BY (order_date, region_id);
   ```
   **Process**: Identified slow queries via Snowflake’s query profiling, applied clustering keys, and monitored performance.
   **Outcome**: Reduced query time from 10 to 3 seconds (~20% improvement).

3. **Dynamic SQL with Freemarker Templates**:
   ```python
   import freemarker
   import snowflake.connector

   parameters = {
       'start_date': '2025-01-01',
       'end_date': '2025-03-31',
       'region': 'APAC',
       'product_category': 'Electronics'
   }

   fm_config = freemarker.Configuration()
   fm_config.set_directory_for_template_loading('templates/')
   template = fm_config.get_template('sales_report.ftl')
   rendered_sql = template.process(parameters)

   conn = snowflake.connector.connect(user='username', password='password', account='account', 
                                     warehouse='warehouse', database='database', schema='schema')
   cursor = conn.cursor()
   try:
       cursor.execute(rendered_sql)
       results = cursor.fetchall()
       print(results)
   finally:
       cursor.close()
       conn.close()
   ```

   **Template (sales_report.ftl)**:
   ```sql
   WITH base_sales AS (
       SELECT product_id, region_id, order_date, sales_amount
       FROM sales_data
       WHERE order_date BETWEEN '${start_date}' AND '${end_date}'
       <#if region??>AND region_id = '${region}'</#if>
       <#if product_category??>AND product_category = '${product_category}'</#if>
   ),
   <#if region??>
   region_summary AS (
       SELECT r.region_name, SUM(s.sales_amount) AS total_sales
       FROM base_sales s
       JOIN regions r ON s.region_id = r.region_id
       GROUP BY r.region_name
   ),
   </#if>
   <#if product_category??>
   category_summary AS (
       SELECT p.product_category, SUM(s.sales_amount) AS category_sales
       FROM base_sales s
       JOIN products p ON s.product_id = p.product_id
       GROUP BY p.product_category
   ),
   </#if>
   final_summary AS (
       SELECT 
           <#if region??>r.region_name,</#if>
           <#if product_category??>c.product_category,</#if>
           COALESCE(SUM(s.sales_amount), 0) AS total_sales
       FROM base_sales s
       <#if region??>JOIN region_summary r ON s.region_id = r.region_id</#if>
       <#if product_category??>JOIN category_summary c ON s.product_id = c.product_id</#if>
       GROUP BY 
           <#if region??>r.region_name,</#if>
           <#if product_category??>c.product_category</#if>
   )
   SELECT * FROM final_summary;
   ```

   **Outcome**: Reduced report generation time by 45% and improved accuracy.

4. **Performance Tuning with Query Profiling**:
   ```sql
   WITH pre_agg AS (
       SELECT region_id, product_id, SUM(sales_amount) AS total_sales
       FROM sales_data
       GROUP BY region_id, product_id
   )
   SELECT p.product_id, r.region_name, a.total_sales
   FROM pre_agg a
   JOIN products p ON a.product_id = p.product_id
   JOIN regions r ON a.region_id = r.region_id;
   ```
   **Outcome**: Reduced query time from 12 to 4 seconds (~25% improvement).

**Real-Time Example (Q1 2025 Sales Dashboard)**:
- **Problem**: Slow queries (~10 seconds).
- **Steps**: Used RANK(), LAG(), clustering keys, Freemarker templates, and query profiling.
- **Outcome**: Dashboard load time reduced to ~3 seconds, 20% faster processing, 45% improved accuracy.

---

**Team Management**:
- Set clear goals (45% accuracy, 20% query speed improvement).
- Delegated tasks based on strengths, mentored juniors.
- Automated workflows with GitHub Actions and Terraform.
- Monitored system health with CloudWatch dashboards.
- Held regular check-ins for collaboration and risk management.

---

---

### **Deployment Support Example**
- **Scenario**: Supported Blue-Green deployment for real-time data aggregation in AWS EKS.
- **Challenge**: CPU spikes in Green environment risked performance.
- **Actions**:
  - Configured CloudWatch Agent on EC2 via SSM:
    ```hcl
    resource "aws_ssm_document" "install_cloudwatch_agent" { ... }
    ```
  - Enabled Container Insights for EKS pods with IAM role:
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: aws-load-balancer-controller
      namespace: kube-system
      annotations:
        eks.amazonaws.com/role-arn: arn:aws:iam::<account_id>:role/AmazonEKSLoadBalancerControllerRole
    ```
  - Created CloudWatch dashboards, alarms, and SNS notifications:
    ```hcl
    resource "aws_cloudwatch_metric_alarm" "high_cpu" { ... }
    resource "aws_sns_topic" "alerts" { ... }
    ```
  - Optimized Snowflake queries with Python, reducing execution time by 20%.
- **Outcome**: Zero-downtime deployment, 30% fewer misalignments, 25% improved reliability.

---

### **Debugging Connection Refused**
1. **Verify Service Status**:
   ```bash
   netstat -tlnp | grep <port_number>
   ss -tlnp | grep <port_number>
   ```

2. **Check Configuration**: Ensure correct ports/protocols in config files.
3. **Inspect Firewall**:
   ```bash
   iptables -n -L
   ufw status
   ```

4. **Test Connectivity**:
   ```bash
   telnet <service_ip> <port_number>
   nc -vz <service_ip> <port_number>
   ```

5. **Check Logs**:
   ```bash
   docker logs -f <container_id>
   kubectl logs -f <pod_name>
   ```

6. **Verify DNS**:
   ```bash
   dig <service_dns_name>
   nslookup <service_dns_name>
   ```

**Common Causes**:
- Service not listening, firewall blocks, misconfigured ports, network issues, DNS failures.

---

### **Auto Scaling in AWS and Kubernetes**

**EC2 Auto Scaling**:
1. **Launch Template**:
   ```hcl
   resource "aws_launch_template" "web_template" {
     name_prefix   = "web-lt"
     image_id      = "ami-0abcdef1234567890"
     instance_type = "t3.micro"
     key_name      = "my-key"
     network_interfaces {
       associate_public_ip_address = true
       security_groups             = [aws_security_group.web_sg.id]
     }
     user_data = base64encode(<<-EOF
                 #!/bin/bash
                 echo "Hello from AutoScaling EC2" > index.html
                 nohup python3 -m http.server 80 &
                 EOF
               )
   }
   ```

2. **Auto Scaling Group**:
   ```hcl
   resource "aws_autoscaling_group" "web_asg" {
     desired_capacity     = 2
     max_size             = 4
     min_size             = 1
     vpc_zone_identifier  = [aws_subnet.public1.id, aws_subnet.public2.id]
     launch_template {
       id      = aws_launch_template.web_template.id
       version = "$Latest"
     }
     target_group_arns = [aws_lb_target_group.web_tg.arn]
   }
   ```

3. **ALB**:
   ```hcl
   resource "aws_lb" "web_alb" {
     name               = "web-alb"
     internal           = false
     load_balancer_type = "application"
     subnets            = [aws_subnet.public1.id, aws_subnet.public2.id]
     security_groups    = [aws_security_group.alb_sg.id]
   }
   ```

4. **Target Group and Listener**:
   ```hcl
   resource "aws_lb_target_group" "web_tg" {
     name     = "web-tg"
     port     = 80
     protocol = "HTTP"
     vpc_id   = aws_vpc.main.id
   }
   resource "aws_lb_listener" "web_listener" {
     load_balancer_arn = aws_lb.web_alb.arn
     port              = 80
     protocol          = "HTTP"
     default_action {
       type             = "forward"
       target_group_arn = aws_lb_target_group.web_tg.arn
     }
   }
   ```

5. **CloudWatch Alarm**:
   ```hcl
   resource "aws_cloudwatch_metric_alarm" "high_cpu" {
     alarm_name          = "high-cpu-alarm"
     comparison_operator = "GreaterThanThreshold"
     evaluation_periods  = 2
     metric_name         = "CPUUtilization"
     namespace           = "AWS/EC2"
     period              = 60
     statistic           = "Average"
     threshold           = 70
     alarm_actions       = [aws_autoscaling_policy.scale_out.arn]
     dimensions = {
       AutoScalingGroupName = aws_autoscaling_group.web_asg.name
     }
   }
   ```

6. **Scaling Policy**:
   ```hcl
   resource "aws_autoscaling_policy" "scale_out" {
     name                   = "scale-out"
     scaling_adjustment     = 1
     adjustment_type        = "ChangeInCapacity"
     cooldown               = 300
     autoscaling_group_name = aws_autoscaling_group.web_asg.name
   }
   ```

**Kubernetes Auto Scaling**:
1. **IAM Policy for ALB Controller**:
   ```hcl
   resource "aws_iam_policy" "alb_controller" {
     name   = "AWSLoadBalancerControllerIAMPolicy"
     policy = file("iam_policy_alb_controller.json")
   }
   ```

2. **IAM Role and Service Account**:
   ```hcl
   data "aws_eks_cluster" "eks" {
     name = "your-cluster-name"
   }
   data "aws_iam_policy_document" "assume_role" {
     statement {
       effect = "Allow"
       principals {
         type        = "Federated"
         identifiers = [data.aws_eks_cluster.eks.identity[0].oidc.issuer]
       }
       actions = ["sts:AssumeRoleWithWebIdentity"]
       condition {
         test     = "StringEquals"
         variable = "${replace(data.aws_eks_cluster.eks.identity[0].oidc.issuer, "https://", "")}:sub"
         values   = ["system:serviceaccount:kube-system:aws-load-balancer-controller"]
       }
     }
   }
   resource "aws_iam_role" "alb_sa_role" {
     name               = "AmazonEKSLoadBalancerControllerRole"
     assume_role_policy = data.aws_iam_policy_document.assume_role.json
   }
   resource "aws_iam_role_policy_attachment" "alb_attach" {
     role       = aws_iam_role.alb_sa_role.name
     policy_arn = aws_iam_policy.alb_controller.arn
   }
   ```

3. **Service Account**:
   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: aws-load-balancer-controller
     namespace: kube-system
     annotations:
       eks.amazonaws.com/role-arn: arn:aws:iam::<account_id>:role/AmazonEKSLoadBalancerControllerRole
   ```

4. **Deploy ALB Controller**:
   ```bash
   kubectl apply -f v2_7_1_full.yaml
   ```

5. **App Deployment**:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: web-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: web
       spec:
         containers:
         - name: nginx
           image: nginx
           ports:
           - containerPort: 80
           resources:
             requests:
               cpu: "100m"
             limits:
               cpu: "200m"
   ```

6. **ClusterIP Service**:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-service
   spec:
     type: ClusterIP
     selector:
       app: web
     ports:
       - port: 80
         targetPort: 80
   ```

7. **ALB Ingress**:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: web-ingress
     annotations:
       kubernetes.io/ingress.class: alb
       alb.ingress.kubernetes.io/scheme: internet-facing
       alb.ingress.kubernetes.io/target-type: ip
   spec:
     rules:
       - http:
           paths:
             - path: /
               pathType: Prefix
               backend:
                 service:
                   name: web-service
                   port:
                     number: 80
   ```

8. **HPA**:
   ```yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: web-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: web-app
     minReplicas: 2
     maxReplicas: 5
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 60
   ```

9. **Metrics Server**:
   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

**Flow**:
- ALB routes traffic to pods via Ingress.
- HPA scales pods based on CPU (Metrics Server).
- CloudWatch triggers EC2 scaling.

---

### **Terraform Module: ALB and Target Group**
```hcl
resource "aws_lb" "web_alb" {
  name               = "web-alb"
  internal           = false
  load_balancer_type = "application"
  subnets            = [aws_subnet.public1.id, aws_subnet.public2.id]
  security_groups    = [aws_security_group.alb_sg.id]
}
resource "aws_lb_target_group" "web_tg" {
  name     = "web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
}
resource "aws_lb_target_group_attachment" "example" {
  target_group_arn = aws_lb_target_group.web_tg.arn
  target_id        = aws_instance.example.id
  port             = 80
}
```

**Workflow**:
- ALB created in specified subnets, internet-facing.
- Target group routes traffic to EC2 instances.
- Attachments link instances to target group.

**Request Flow**:
1. User request hits ALB.
2. ALB evaluates rules, selects target group.
3. Target group routes to healthy EC2 instance.
4. Instance processes request, returns response via ALB.

**Benefits**:
- Scalability: Distributes traffic across instances.
- High Availability: Routes to healthy instances.
- Flexibility: Supports various targets.

---

### **EKS Cluster with Single Node Group**
```hcl
resource "aws_eks_cluster" "example" {
  name     = "my-eks-cluster"
  role_arn = aws_iam_role.eks_role.arn
  vpc_config {
    subnet_ids = [aws_subnet.private1.id, aws_subnet.private2.id]
  }
}
resource "aws_eks_node_group" "example" {
  cluster_name    = aws_eks_cluster.example.name
  node_group_name = "my-node-group"
  node_role_arn   = aws_iam_role.node_role.arn
  subnet_ids      = [aws_subnet.private1.id, aws_subnet.private2.id]
  scaling_config {
    desired_size = 1
    max_size     = 2
    min_size     = 1
  }
}
resource "aws_iam_role" "eks_role" {
  name = "eks-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = { Service = "eks.amazonaws.com" }
    }]
  })
}
resource "aws_iam_role" "node_role" {
  name = "eks-node-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}
```

---

### **VPC Peering**
```hcl
resource "aws_vpc" "vpc_a" { cidr_block = "10.0.0.0/16" }
resource "aws_vpc" "vpc_b" { cidr_block = "10.1.0.0/16" }
resource "aws_vpc_peering_connection" "peer" {
  vpc_id        = aws_vpc.vpc_a.id
  peer_vpc_id   = aws_vpc.vpc_b.id
  auto_accept   = true
}
resource "aws_route" "vpc_a_route" {
  route_table_id            = aws_vpc.vpc_a.main_route_table_id
  destination_cidr_block    = aws_vpc.vpc_b.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}
resource "aws_route" "vpc_b_route" {
  route_table_id            = aws_vpc.vpc_b.main_route_table_id
  destination_cidr_block    = aws_vpc.vpc_a.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}
```

---

### **S3 Bucket with Encryption**
```hcl
resource "aws_s3_bucket" "bucket" { bucket = "my-encrypted-bucket" }
resource "aws_s3_bucket_server_side_encryption_configuration" "encryption" {
  bucket = aws_s3_bucket.bucket.bucket
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
resource "aws_s3_bucket_versioning" "versioning" {
  bucket = aws_s3_bucket.bucket.bucket
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_public_access_block" "block" {
  bucket                  = aws_s3_bucket.bucket.bucket
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

---

### **Update EC2 Security Group for HTTPS**
```bash
aws ec2 authorize-security-group-ingress --group-id <security-group-id> --protocol tcp --port 443 --cidr 0.0.0.0/0
```

---

### **Troubleshoot CloudWatch Alarm**
```bash
aws cloudwatch describe-alarms --alarm-names <alarm-name>
aws cloudwatch describe-alarm-history --alarm-name <alarm-name> --history-item-type StateUpdate
```

---

### **RDS Instance**
```hcl
resource "aws_db_instance" "example" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0.28"
  instance_class       = "db.t2.micro"
  db_name              = "exampledb"
  username             = "exampleuser"
  password             = "examplepassword"
  parameter_group_name = "default.mysql8.0"
  skip_final_snapshot  = true
  vpc_security_group_ids = [aws_security_group.rds.id]
}
resource "aws_security_group" "rds" {
  name        = "rds-sg"
  description = "Security group for RDS instance"
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

### **Add IAM Policy to Role**
```hcl
resource "aws_iam_role" "existing_role" {
  name = "my-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}
resource "aws_iam_policy" "new_policy" {
  name = "new-policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "s3:GetObject"
      Effect = "Allow"
      Resource = "arn:aws:s3:::my-bucket/*"
    }]
  })
}
resource "aws_iam_role_policy_attachment" "attach" {
  role       = aws_iam_role.existing_role.name
  policy_arn = aws_iam_policy.new_policy.arn
}
```

---

### **Scale Kubernetes Deployment**
```bash
kubectl scale deployment web-app --replicas=5 -n <namespace>
```

---
