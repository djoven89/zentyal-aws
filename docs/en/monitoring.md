---

tags:
  - AWS

---

# Monitoring

On this page, we will configure a monitoring system for our Zentyal server using AWS [CloudWatch] service. Additionally, we will also make use of the AWS SSM [Parameter Store] service to host the CloudWatch agent configuration on our server and finally, the AWS [SNS] service for alert notifications.

!!! warning

    Implementing these services will have an additional monthly cost.

[Cloudwatch]: https://docs.aws.amazon.com/es_es/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html
[Parameter Store]: https://docs.aws.amazon.com/es_es/systems-manager/latest/userguide/systems-manager-parameter-store.html
[SNS]: https://docs.aws.amazon.com/es_es/sns/latest/dg/welcome.html

## SNS

To notify any alerts triggered in CloudWatch, we will use the SNS service, which will send an email to a specified email account. In my case, I will use the created account of `it.infra@icecrown.es`.

1. Go to `SNS` and create a topic called `Prod-Zentyal-Email-Alerting`:

    !["SNS topic creation"](assets/images/aws/monitoring-sns_topic.png "SNS topic creation")
    !["SNS topic creation"](assets/images/aws/monitoring-sns_topic-tags.png "SNS topic creation")

2. Create a `subscription` for the email account that will receive the notifications:

    !["SNS topic creation"](assets/images/aws/monitoring-sns_subscription.png "SNS topic creation")

3. Finally, wait for the email invitation to arrive and activate the subscription.

    !["SNS subscription confirmation"](assets/images/aws/monitoring-sns_confirmation.png "SNS subscription confirmation")

    !!! note

         Since we have the graylist enabled, it may take a few minutes for the invitation to arrive.

## SSM Parameter Store

To monitor the resources of the Zentyal server, we will use the AWS CloudWatch service, and we will store its configuration file in the SSM Parameter store.

The configuration I will specify is:

* The complete path to the parameter will be called `/zentyal/prod/cloudwatch-config`
* The CloudWatch namespace will be called `CWA-Prod-Zentyal`.
* The metrics interval will be `60` seconds.
* Additional metrics will be configured for:
    * RAM
    * Swap
    * Disk.
* The 3 EBS volumes will be included in the disk metrics.
* The log `/var/log/zentyal/zentyal.log` will also be monitored, with a retention of 7 days.
* The log group in CloudWatch will be called `CWAL-Prod-Zentyal`.

!!! info

    For additional configurations or questions, we have the configuration reference [here].

[here]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html#CloudWatch-Agent-Configuration-File-Metricssection

The following actions will be taken:

1. Go to the region where we have the instance, which in my case is Paris.
2. Go to `AWS Systems Manager -> Parameter Store -> Create parameter`.
3. Create the `parameter`:

    !["Parameter store configuration"](assets/images/aws/monitoring-parameter_config.png "Parameter store configuration")

4. Add the agent configuration to the `Value` section:

    ```json
    {
        "agent": {
            "metrics_collection_interval": 60,
            "run_as_user": "root"
        },
        "metrics": {
            "namespace": "CWA-Prod-Zentyal",
            "aggregation_dimensions": [
                [
                    "InstanceId"
                ]
            ],
            "append_dimensions": {
            "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
            "ImageId": "${aws:ImageId}",
            "InstanceId": "${aws:InstanceId}",
            "InstanceType": "${aws:InstanceType}"
            },
            "metrics_collected": {
                "mem": {
                    "measurement": [
                        "used_percent",
                        "used",
                        "free",
                        "total",
                        "cached",
                        "buffered"
                    ],
                    "metrics_collection_interval": 60
                },
                "swap": {
                    "measurement": [
                        "used_percent",
                        "used",
                        "free"
                    ],
                    "metrics_collection_interval": 60
                },
                "disk": {
                    "measurement": [
                        "used_percent",
                        "used",
                        "free",
                        "total",
                        "inodes_used",
                        "inodes_free",
                        "inodes_total"
                    ],
                    "metrics_collection_interval": 60,
                    "ignore_file_system_types": [
                        "tmpfs",
                        "vfat",
                        "devtmps"
                    ],
                    "resources": [
                    "/",
                    "/var/vmail",
                    "/home"
                    ]
                },
                "statsd": {
                    "metrics_aggregation_interval": 60,
                    "metrics_collection_interval": 60,
                    "service_address": ":8125"
                }
            }
        },
        "logs": {
            "logs_collected": {
                "files": {
                    "collect_list": [
                        {
                            "file_path": "/var/log/zentyal/zentyal.log",
                            "log_group_name": "CWAL-Prod-Zentyal",
                            "log_stream_name": "{instance_id}",
                            "retention_in_days": 7,
                            "timezone": "UTC"
                        }
                    ]
                }
            },
            "log_stream_name": "Stream-Prod-Zentyal",
            "force_flush_interval" : 15
        }
    }
    ```

5. With the parameter created, we will create an IAM policy that allows access from the EC2 instance to the newly created parameter. To do this, go to `IAM -> Policies`
6. Create a policy that has the following content:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "ParameterStoreZentyal1",
                "Effect": "Allow",
                "Action": [
                    "ssm:GetParameterHistory",
                    "ssm:GetParametersByPath",
                    "ssm:GetParameters",
                    "ssm:GetParameter"
                ],
                "Resource": [
                    "arn:aws:ssm:eu-west-3:*:parameter/zentyal/prod/cloudwatch-config"
                ]
            },
            {
                "Sid": "ParameterStoreZentyal2",
                "Effect": "Allow",
                "Action": "ssm:DescribeParameters",
                "Resource": "*"
            }
        ]
    }
    ```

    !["IAM policy for CloudWatch Agent"](assets/images/aws/monitoring-iam_cloudwatch.png "IAM policy for CloudWatch Agent")

7. Create another policy that allows uploading the log file to CloudWatch:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "ZentyalCloudWatchLogs",
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents",
                    "logs:DescribeLogStreams"
                ],
                "Resource": [
                    "arn:aws:logs:eu-west-3:*:*"
                ]
            }
        ]
    }
    ```

    !["IAM policy for CloudWatch Logs"](assets/images/aws/monitoring-iam_cloudwatch-logs.png "IAM policy for CloudWatch Logs")

8. Create a role where we will associate the newly created policies and also the existing one called `CloudWatchAgentServerPolicy`. To do this, go to `IAM -> Roles`:

    !["IAM role entity"](assets/images/aws/monitoring-iam_role-entity.png "IAM role entity")
    !["IAM role summary 1"](assets/images/aws/monitoring-iam_role-summary-1.png "IAM role summary 1")
    !["IAM role summary 2"](assets/images/aws/monitoring-iam_role-summary-2.png "IAM role summary 2")

9. Finally, associate the newly created role with the Zentyal instance. To do this, go to `EC2 -> Actions -> Security -> Modify IAM role`:

    !["IAM role assign to EC2"](assets/images/aws/monitoring-iam_role-ec2.png "IAM role assign to EC2")

## Cloudwatch

Once we have the AWS environment ready, we will proceed to install and configure the CloudWatch agent to monitor the server and Zentyal's main log file.

1. Download the CloudWatch agent `.deb` package to our Zentyal server:

    ```sh
    sudo curl "https://s3.amazonaws.com/amazoncloudwatch-agent/debian/amd64/latest/amazon-cloudwatch-agent.deb" -o "/opt/amazon-cloudwatch-agent.deb"
    ```

2. Install the package:

    ```sh
    sudo dpkg -i -E /opt/amazon-cloudwatch-agent.deb
    ```

3. Descargamos también el archivo comprimido que contiene el binario de AWS para la CLI:

    ```sh
    sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/opt/awscliv2.zip"
    ```

4. Install the unzip package to `unzip` the file:

    ```sh
    sudo apt update
    sudo apt install -y unzip
    ```

5. Unzip the file and install it:

    ```sh
    sudo unzip /opt/awscliv2.zip -d /opt/aws/
    sudo /opt/aws/aws/install
    ```

6. Configure the CloudWatch agent:

    ```sh
    sudo amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c ssm:/zentyal/prod/cloudwatch-config
    ```

7. Confirm that the service is active:

    ```sh
    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status
    ```

    The result I have obtained:

    ```json
    {
    "status": "running",
    "starttime": "2023-02-25T09:59:17+00:00",
    "configstatus": "configured",
    "version": "1.247357.0b252275"
    }
    ```

8. After waiting a couple of minutes, go to `CloudWatch -> All metrics` and check that the namespace with custom metrics has been created:

    !["CloudWatch namespace"](assets/images/aws/monitoring-check_metrics-1.png "CloudWatch namespace")
    !["CloudWatch metrics"](assets/images/aws/monitoring-check_metrics-2.png "CloudWatch metrics")

9. Finally, we also check that the Zentyal log file is being monitored. To do this, we go to `CloudWatch -> Log groups`:

    !["CloudWatch logs 1"](assets/images/aws/monitoring-check_logs-1.png "CloudWatch logs 1")
    !["CloudWatch logs 2"](assets/images/aws/monitoring-check_logs-2.png "CloudWatch logs 2")

### Logs

With the main Zentyal file monitored by CloudWatch, we create a metric filter that checks if the log file contains the event `ERROR>`. The purpose is to create an alert that notifies via email through AWS SNS when this type of event occurs.

1. Go to `CloudWatch -> Metric filters` and create the filter:

    !["CloudWatch filter log 1"](assets/images/aws/monitoring-logs_filter-1.png "CloudWatch filter log 1")
    !["CloudWatch filter log 2"](assets/images/aws/monitoring-logs_filter-2.png "CloudWatch filter log 2")

2. Once the filter is created and a couple of minutes have passed for CloudWatch to collect information.

3. Finally, verify that from `CloudWatch -> All metrics` we have the metric available:

    !["CloudWatch filter metric"](assets/images/aws/monitoring-logs_metric-check.png "CloudWatch filter metric")

    !!! note

        The type of metric shown in the image is of type `Number` as can be seen at the top.

### Dashboard

Once the monitoring system is confirmed to be working, we can create a dashboard that groups the most important metrics from `CloudWatch -> Dashboard`. Here's a simple example:

!["CloudWatch dashboard"](assets/images/aws/monitoring-dashboard.png "CloudWatch dashboard")

### Alerts

The last thing we will do on the monitoring system is to create alerts. All alerts we configure will be made from `CloudWatch -> All alarm` and will be as follows:

* **CPU:**
    * Check will be done every minute.
    * The alert value to trigger will be greater than 80%.
    * For a notification to be sent, the alert must occur 3 times in a row.
* **RAM:**
    * Check will be done every minute.
    * The alert value to trigger will be greater than 80%.
    * For a notification to be sent, the alert must occur 3 times in a row.
* **System disk:**
    * Check will be done every minute.
    * The alert value to trigger will be greater than 80%.
    * For a notification to be sent, the alert must occur 3 times in a row.
* **Mail disk:**
    * Check will be done every minute.
    * The alert value to trigger will be greater than 80%.
    * For a notification to be sent, the alert must occur 3 times in a row.
* **Shared resources disk:**
    * Check will be done every minute.
    * The alert value to trigger will be greater than 80%.
    * For a notification to be sent, the alert must occur 3 times in a row.
* **DLM for system:**
    * Check will be done once a day.
    * The alert value to trigger will be equal to or greater than 1.
    * For a notification to be sent, the alert must occur once.
* **DLM for mail:**
    * Check will be done once a day.
    * The alert value to trigger will be equal to or greater than 1.
    * For a notification to be sent, the alert must occur once.
* **DLM for shared resources:**
    * The check will be performed once a day.
    * The alert value to trigger it will be equal to or greater than 1.
    * For a notification to be sent, the alert must occur only once.
* **EC2 failed checks:**
    * The check will be performed every minute.
    * The alert value to trigger it will be greater than 80%.
    * For a notification to be sent, the alert must occur 3 times consecutively.
* **Instance failed checks:**
    * The check will be performed every minute.
    * The alert value to trigger it will be greater than 80%.
    * For a notification to be sent, the alert must occur 3 times consecutively.
* **Zentyal log errors:**
    * The check will be performed once a day.
    * The alert value to trigger it will be equal to or greater than 1.
    * For a notification to be sent, the alert must occur only once.

#### CPU

!["CloudWatch CPU alert 1"](assets/images/aws/monitoring-alert_cpu-1.png "CloudWatch CPU alert 1")
!["CloudWatch CPU alert 2"](assets/images/aws/monitoring-alert_cpu-2.png "CloudWatch CPU alert 2")

#### RAM

!["CloudWatch RAM alert 1"](assets/images/aws/monitoring-alert_ram-1.png "CloudWatch RAM alert 1")
!["CloudWatch RAM alert 2"](assets/images/aws/monitoring-alert_ram-2.png "CloudWatch RAM alert 2")

#### System Disk

!["CloudWatch System disk alert 1"](assets/images/aws/monitoring-alert_disk-system-1.png "CloudWatch System disk alert 1")
!["CloudWatch System disk alert 2"](assets/images/aws/monitoring-alert_disk-system-2.png "CloudWatch System disk alert 2")

#### Mail Disk

!["CloudWatch Mail disk alert 1"](assets/images/aws/monitoring-alert_disk-mail-1.png "CloudWatch Mail disk alert 1")
!["CloudWatch Mail disk alert 2"](assets/images/aws/monitoring-alert_disk-mail-2.png "CloudWatch Mail disk alert 2")

#### Shares Disk

!["CloudWatch Shares disk alert 1"](assets/images/aws/monitoring-alert_disk-shares-1.png "CloudWatch Shares disk alert 1")
!["CloudWatch Shares disk alert 2"](assets/images/aws/monitoring-alert_disk-shares-2.png "CloudWatch Shares disk alert 2")

#### DLM - System

!["CloudWatch DLM System alert 1"](assets/images/aws/monitoring-alert_dlm-system-1.png "CloudWatch DLM system alert 1")
!["CloudWatch DLM System alert 2"](assets/images/aws/monitoring-alert_dlm-system-2.png "CloudWatch DLM system alert 2")

#### DLM - Mail

!["CloudWatch DLM Mail alert 1"](assets/images/aws/monitoring-alert_dlm-mail-1.png "CloudWatch DLM Mail alert 1")
!["CloudWatch DLM Mail alert 2"](assets/images/aws/monitoring-alert_dlm-mail-2.png "CloudWatch DLM Mail alert 2")

#### DLM - Shares

!["CloudWatch DLM Shares alert 1"](assets/images/aws/monitoring-alert_dlm-shares-1.png "CloudWatch DLM Shares alert 1")
!["CloudWatch DLM Shares alert 2"](assets/images/aws/monitoring-alert_dlm-shares-2.png "CloudWatch DLM Shares alert 2")

#### EC2 - System

!["CloudWatch EC2 System alert 1"](assets/images/aws/monitoring-alert_ec2-system-1.png "CloudWatch EC2 System alert 1")
!["CloudWatch EC2 System alert 2"](assets/images/aws/monitoring-alert_ec2-system-2.png "CloudWatch EC2 System alert 2")

#### EC2 - Instance

!["CloudWatch EC2 Instance alert 1"](assets/images/aws/monitoring-alert_ec2-instance-1.png "CloudWatch EC2 Instance alert 1")
!["CloudWatch EC2 Instance alert 2"](assets/images/aws/monitoring-alert_ec2-instance-2.png "CloudWatch EC2 Instance alert 2")

#### Zentyal log

!["CloudWatch Log alert 1"](assets/images/aws/monitoring-alert_log-1.png "CloudWatch Log alert 1")
!["CloudWatch Log alert 2"](assets/images/aws/monitoring-alert_log-2.png "CloudWatch Log alert 2")
