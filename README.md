## Amazon CloudWatch Cross Account Cross Region Dashboards - AWS Cloud Formation Sample

Sample AWS Cloud Formation scripts to generate an Amazon CloudWatch cross-account, cross-region dashboard.


## Setting up your first cross-account, cross-region dashboard

Before attempting to create a cross-account, cross-region dashboard you must ensure that your dashboard user has the required permissions to read metrics and logs in the remote account. Follow the instructions below to create the prerequisite permissions. 

Once the permissions are setup the recommended way to create your cross-account, cross-region dashboard is via Cloud Formation. It is possible to add either metric or log widgets to a cross-account, cross-region dashboard. 

### 1.	Setting up permissions

Amazon CloudWatch cross-account, cross-region dashboards uses IAM roles to create read-only visibility between accounts. Your console user in one account will assume a role in a remote account to read metric and log data to display in the dashboard. Let’s walk through a specific scenario. 

Scenario: You want to create a dashboard in account A that contains a metric or log widget referencing metrics or logs in account B. There are 2 steps required to setup the permissions. 

### Step 1: AssumeRole Permission in Local Account

Allow the user in account A to assume a role in account B. You must associate this permission with the user that is viewing the dashboard:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::*:role/*"
        }
    ]
}
```
### Step 2: Create Cross-Account Role in Remote Account

Create a role in account B that grants read-only access to metrics, alarms and logs to account A. See file [cross-account-role.json](cross-account-role.json).

## 2.	Creating a cross-account, cross region dashboard

Once the require IAM permissions are in place you can create your cross-account, cross-region dashboard. The recommended approach is to create the dashboard using Cloud Formation. 

Two new fields are available as part of the JSON widget definitions:
*	accountId – the source account for the metric data
*	role – the name of the role to assume in the source account. This can be the role name or the full ARN of the role. 

## Creating Dashboard in Cloud Formation

The recommended approach is to create cross-account cross-region dashboards using Cloud Formation. You will find an example in [cross-account-monitoring.json](cross-account-monitoring.json). 

You will see the JSON dashboard definition in the DashboardBody parameter. You can copy and paste the JSON from the console by selecting Actions -> View/edit source from your dashboard. Once copied you need to escape quotes with the backslash before pastinginto the Cloud Formation template. 

### Editing Metric Widgets in Console

Pick one of the dashboards that you want to make cross-account cross-region and edit the JSON via Actions -> View/edit source. Editing the JSON for a graph is also available in the Metrics tab.

In the JSON, each metric is an array with strings: the first is the namespace, the second is the metric name, eventually followed by dimension names and values. Add the accountId and role parameters into the properties section:

For example:
```json
"widgets": [
   ...
   “properties”: {
   	"metrics": [
      	[ "SomeNamespace", "SomeMetricName" ]
   	],
   	"region": "<region>",
   	"accountId": "<account_id>",
   	"role": "<roleName or ARN>",
   	...
   }
   ...
]
```

Full example: 
```json
{
    "widgets": [
         {
            "type": "metric",
            "x": 0,
            "y": 0,
            "width": 12,
            "height": 6,
            "properties": {
                "metrics": [
                    [ "AWS/EC2", "CPUUtilization", "InstanceId", "i-07cde5bf63de1c48a" ],
                    [ "...", "i-0b34c5009f0271523" ],
                    [ "...", "i-0bbc7e5a6b96627f7" ]
                ],
                "accountId": 123456789,
                "role": "CloudWatchCrossAccount",
                "period": 300,
                "stat": "Average",
                "region": "us-east-1",
                "title": "Compute - Shared Acct - CPUUtilization",
                "view": "timeSeries",
                "stacked": false
            }
        }   
    ]	
}
```

### Log Insights Widget

Add accountId and role entries to properties of widget. Full example:
```json
{
    "widgets": [
        {
            "type": "log",
            "x": 0,
            "y": 6,
            "width": 24,
            "height": 6,
            "properties": {
                "query": "SOURCE '/aws/lambda/PutMetricDataSingleValue' | fields @timestamp, @message\n| sort @timestamp desc\n| limit 20",
                "region": "us-east-1",
                "title": "API Requests - Shared Acct",
                "accountId": 12345678,
                "role": "CloudWatchCrossAccount"
            }
        }
    ]
}
```

### Alarm Widget

Add accountId and role entries to properties of widget. Full example:
```json
{
    "widgets": [
        {
            "type": "metric",
            "x": 6,
            "y": 2,
            "width": 6,
            "height": 6,
            "properties": {
                "title": "XA - Average CPUUtilization",
                "annotations": {
                    "alarms": [
                        "arn:aws:cloudwatch:us-east-1:12345678:alarm:XA - Average CPUUtilization"
                    ]
                },
                "accountId": 12345678,
                "role": "CloudWatchCrossAccount",
                "view": "timeSeries",
                "stacked": false
              }
        }
    ]
}
```

## Troubleshooting

If you haven't setup cross-account access correctly your widgets will display a generic error. We are working on making this more specific to help teams debug setup issues, but in the meantime here are the main gotchas:
*	You must be logged into the master account as an IAM user - root login sessions are not able to assume IAM roles.
*	The master account IAM user must have permission to assume roles -
```json
{ "Effect": "Allow", "Action": "sts:AssumeRole", "Resource": "arn:aws:iam::*::role/*" }
```
*	The role in each remote account must have enough permissions to display the data for the particular widget.
*	If you are not seeing any results in a Log Insights widget, ensure that the time period of the dashboard matches what you expect to see your log data. The default period on the dashboard is 3hrs. 

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
