---
layout: default
---

## AWS Monitoring

**CloudTrail vs CloudWatch vs XRay**

CloudTrail - Used to audit API calls made by users and services, and is useful to detect unauthorized calls or the root cause of changes.

CloudWatch - Used to view metrics and logs, and to create alarms and events.

X-Ray - Must more granular than CloudWatch and is based around Trace Analysis and Central Service Map Visualization

#### CloudWatch

cloudWatch is basically a metrics repository that collects and tracks metrics from AWS resources and applications. Metrics are variables that can be measures, such as CPU utilization and disk reads and writes. CloudWatch provides system-wide visibility into resource utilization, app performance and operational health.

**Concepts**

*Namespaces* - A container for metrics, where the metrics within a namespace are isolated from one another so they are not mistakenly aggregated into the same statistic. There is no default namespace, and a namespace must be specified for each data point published to CloudWatch.

*Metrics* - The fundamental concept in CloudWatch that represents a time-ordered set of data points. For example, the CPU usage of an EC2 instance is one metric provided by EC2, and the data points can come from any app or business activity from which data is collected. Each data point in a metric has a time stamp and an optional unit of measure. 

Metrics exist only in the Region they were created in, and cannot be deleted. They expire after 15 months if no new data is published to them. 

Some resources allow *detailed monitoring*. For example, EC2 instances have metrics every 5 minutes, but can enable *detailed monitoring* (for a cost) to allow metrics every 1 minute.

Each metric is either *Standard resolution* or *High resolution*. Standard resolution means data has one minute of granularity, and high resolution means data has one second granularity. Metrics are standard by default. Every call to *PutMetricData* incurs a cost, so high resolution will lead to higher charges.

If throttle errors occur (sending too much data to CloudWatch), use exponential backoff.

*Dimensions* - A name/value pair that is part of the identity of a metric. Essentially an attribute of a metric, such as instance id, environment, etc. Up to 10 dimensions can be assigned to a metric. 

**Alarms** - Used to trigger notifications for any metric such as Auto Scaling, EC2 Actions, SNS notifications, etc. Alarms watch a single metric over a period of time and performs one or more actions based on the value of the metric relative to a threshold over time. The state must have changed and been maintained for a specified number of periods. When creating an alarm, choose a monitoring period greater than or equal to the metrics resolution; e.g. if standard resolution (5 mins), then choose an alarm period of 5 mins.

Alarm States:
  OK - The metric or expression is within the defined threshold
  ALARM - Metric is outside the defined threshold
  INSUFFICIENT_DATA - Alarm has just started, metric is not available, or not enough data is available to determine the alarm state

Evaulating an Alarm - Three settings for an alarm:
  * Period - Length of time to evaulate the metric to create each individual data point for an alarm, expressed in seconds.
  * Evaluation Periods - Number of the most recent periods (data points) to evaluate when determining alarm state.
  * Datapoints to Alarm - Number of data points witin the evaluation period that must be above the threshold to cause the alarm to enter ALARM state.

  Example: For an EC2 CPU Utilization metric, the period is set to 5 minutes (standard resolution), the evaluation period is 3 (3 periods = 15 minutes, so the metric will be evaluated over a 15 minute timespan), and the datapoints to alarm is 3. Therefore, within a 15 miniute timespan, 3 datapoints must be over the threshold to send the alarm to ALARM state. 

  If the Evaluation period is set to 5 (5 periods = 25 minute timespan) and the datapoints to alarm is 3, then within a 25 minute timespan, only 3 datapoints must be over threshold to send the alarm to ALARM state.

**CloudWatch Logs** - Applications can send logs to CloudWatch using the SDK. Logs can be collected from Beanstalk, ECS containers, Lambda functions, VPC Flow logs, EC2 machines, Route53, etc. Logs can be send to S3 for archival or streamed to ElasticSearch.

  * Logs are kept indefinitely by default, but can be adjusted for each log group to retain from years to days.
  * Insights can be used to interactivly query log data
  * Logs can be tailed using the AWS CLI.
  * IAM permissions must be properly set in order to send logs to CloudWatch.
  * Logs are encrypted at the Group level using KMS

CloudWatch Logs concepts:
  Events - A record of some activity recorded by the app or resource being monitored
  Streams - A sequence of log events that share the same source, an app or resource.
  Groups - Groups of log streams that share the same retention, monitoring and access control settings (an application). Each stream belongs to one group, and each group can have many streams.

*CloudWatch Log Agent for EC2* - EC2 instances do not log to CloudWatch by default. In order to do so, a CloudWatch Agent is needed on the EC2 machine, and IAM permissions need to be set on the EC2 machine. On-premise machine can be set up as well.

There are two CloudWatch Agents:
  * CloudWatch Logs Agent - The older version of the agent which doesnt have the granularity of the new version
  * CloudWatch Unified Agent - The newer version which has much more granular metrics, an can have centralized configuration using SSM Paramter Store. Metrics include CPU, Disk, RAM, Netstat, Processes, and Swap space.

*Metric Filters* - Log data coming from CloudWatch Logs can be searched and filtered by creating a metric filter. They define the terms and patterns to look for in log data, and turn log data into numerical metrics that can be graphed or set an alarm on.

Metric filters do not retroactivly filter data, however. They only publish datapoints created after the filter was created.

Metric filters are created from log streams; open the desired log stream, click Create Metric Filter. Then the metric filter can be viewed within the Metric page of CloudWatch, after the filter picks up the specified entry from the log. Alarms can then be created for the metric filter.

**CloudWatch Events** - Delivers near real-time stream of system events that describe changes in AWS services, such as CodePipeline failures, CodeCommit pushes and pull requests, etc. Events respond to these changes and sends messages to targets such as Lambda functions, SNS topics, Kinesis, ECS task, etc.

Events can also be scheduled to run at certain times (minutes, hours or days), or using a cron expression.

*Rules* match incoming events and routes them to targets. A single rule can route to multiple targets that are processed in parallel, and are not processed in any particular order.

*Event buses* receive events from sources and match them to rules. 

JSON documents are created to give information about the change.

**EventBridge** - The next generation of CloudWatch Events, a serverless event bus that adds capabilities to connect with third party services (SaaS partners) and your own applications. It allows to build event driven architectures, loosely coupled and distributed.

EventBridge uses the same CloudWatc Event API, so all existing CloudWatch Events rules and events are within EventBridge.

The *default event bus* is generated by CloudWatch Events.

A *partner event bus* can be created to receive events from SaaS services or applications (DataDog, Zendesk, etc.)

*Custom event buses* can be created to receive events from your own applications.

Event buses can be accessed by other AWS accounts.

EventBridge can analyze events in a bus and infer the schema; how the data is structured. The *Schema Registry* allows to generate code for an application that will know in advance how data is structured in the event bus. Custom schemas can be created or uploaded.

#### X-Ray

AWS X-Ray is a service that collects data about requests that your application serves, and provides tools you can use to view, filter, and gain insights into that data to identify issues and opportunities for optimization. Traced requests can display detailed information about the calls the app makes to downstream AWS resources, microservices, databases and HTTP web APIs. 

X-Ray leverages tracing, where each component dealing with a request adds its own trace. Tracing is made of segments and sub-segments, and annotations can be added to traces to provide extra information.

Security for X-Ray contains IAM authorization (to send API calls to X-Ray) and KMS for encryption at rest.

X-Ray is compatible with Lambda, Beanstalk, ECS, ELB, API Gateway, EC2 instances, on premise instances, etc.

X-Ray advantages:
  * Troubleshoot performance
  * Understand dependencies in a microservice architecture
  * Pinpoit service issues
  * Review request behavior
  * Find errors & exceptions

The X-Ray SKD provides:
  * Interceptors - Can be added to code to trace incoming HTTP requests
  * Client handlers - Instrument AWS SDK clients the app uses to call other AWS services
  * An HTTP client to use instrument calls to other internal and external HTTP web services. 
  * Support for instrumenting calls to SQL databases

To enable X-Ray for an application, the code needs to be modified to import the AWS X-Ray SDK:
  1) Application code (Java, Python, Go, Node.js, .NET) needs to import AWS X-Ray SDK
    * SDK will then capture calls to AWS Services, HTTP requests, Database calls, and Queue calls (SQS)
  2) For certain services (EC2 or on-premise), the X-Ray daemon needs to be installed. For services that already have the daemon installed (Lambda, Beanstalk, etc.), the daemon needs to be enabled.
    * The daemon works as a low level UDP packet interceptor
    * Each app must have proper IAM permissions to write data to X-Ray

To enable X-Ray in Beanstalk, use ebextensions file or enable X-Ray from the console.

To the X-Ray daemon in ECS, there are a few ways:
  1) For a normal ECS cluster using EC2, stand up an X-Ray daemon container on each instance
  2) Using EC2, can also run the daemon as a *side car* with each app container
  3) If using Fargate, the daemon must be ran as a side car. This can be defined in the task definition config, mapping port 2000/UDP of the side car container and setting the 'AWS_XRAY_DAEMON_ADDRESS' environment variable in the app container to 'xray-daemon:2000'

**X-Ray Concepts**

AWS X-Ray receives data from services as *segments*. X-Ray then groups segments that have a common request into *traces*. X-Ray processes the traces to generate a *service graph* that provides a visual representation of your application.

*Traces* track the path of a request thorugh an app, collecting all segments generated by a single request (GET or POST). 

A *segment* provides the resources name, details about the request and details about the work done. Details about the work done can be broken down into *subsegments*, which provide more granular timing information and details about the downstream calls.

*Sampling* decreases amount of data sent to X-Ray, reducing cost. The X-Ray SDK applies a sampling algorithm to determine which requests get traced. By default, the SDK records the first request each second, and five percent of any additional requests. One request per second is the *reservoir*, a fixed number of matching requests to instrument per second before applying the *fixed rate* of five percent.
  * Example; if resevoir size is 50 and fixed rate is 10%, then if 100 requests per second match the rule, the total sampled is 55 requests per second (first 50, then 10% of the other 50)

*Instrumentation* - Measuring the products performance, diagnosing errors and writing trace information. Use the X-Ray SDK to instrument an application, modifying the app code to customize the data (annotations) the SDK sends to X-Ray, using interceptors, filters, handlers, middleware, etc.

*Annotations* - Key-value pairs that are indexed and used for filtering. Annotations can be added to any segment or subsegment, and are aggregated at the trace level. They can be used to record data that can group traces in the console.

*Metadata* - Key-value pairs with values of any type (objects and lists), but are not indexed or used for filtering. Used to store data in the trace that doesn't need to be used for searching.

**API Overview**

X-Ray Write API; permissions needed for the daemon
```
"Effect": "Allow",
"Action": [
    "xray:PutTraceSegments",                  # Uploads segment docs to X-Ray
    "xray:PutTelemetryRecords",               # Used by daemon to upload telemetry (segments received, segments rejected, etc.)
    "xray:GetSamplingRules",                  # Retrieve sampling rules to know what and when to send
    "xray:GetSamplingTargets",                # advanced
    "xray:GetSamplingStatisticSummaries"      # advanced
],
"Resource": [
    "*"
]

```

X-Ray Read APIs:
  * GetServiceGraph - Get the main graph
  * BatchGetTraces - Retrieve a list of traces specified by ID. 
  * GetTraceSummaries - Retrieve IDs and annotations for traces available for a specified time frame using an optional filter
  * GetTraceGraph - Retrieve a service graph for one or more specific trace IDs


#### CloudTrail

Provides governance, compliance and audit for an AWS account, and is enabled by default.

Allows to get a history of events & API calls made within an account by console, SDK, CLI, AWS Services, etc. 

Can put logs from CloudTrail into CloudWatch logs.

If a resource is deleted, you can find out who did it by looking at CloudTrail