# Event Driven Architecture With Event Bridge

## Let's setup the services required for this project

## Launch the AWS CloudFormation template

We can run this project inside your AWS account. To follow along with this project we need to make sure that we have setup the required numbers of services and provisioned them.

You can select your preferred region and cost may vary according to the region you choose.For this project i have used "Asia Pacific (Singapore)
ap-southeast-1".

The link to above region is mentioned below :

[Link to Singapore Region](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks/quickcreate?stackName=aws-event-driven-architectures-workshop&templateURL=https://aws-event-driven-architecture-workshop-assets.s3.amazonaws.com/master-v2.yaml)

We need to follow the following steps to configure our prerequisites services/resources:

1. Enter a stack name 

2. Check the boxes in the Capabilities section

3. Click Create stack

![cf1](https://user-images.githubusercontent.com/98637502/216836665-802a75f7-0ffd-4299-8bfc-f827bbd504df.jpg)

![cf2](https://user-images.githubusercontent.com/98637502/216836671-ddd4cb99-b517-4352-a7dc-9f2c6baaf18c.jpg)

Now we need to configure our Event Generator this process is called as Event Generation.

Event Generation: An event is generated by an application or service within the AWS infrastructure. This could be a user action within a web application, a change in the state of a resource in AWS, or a message in an Amazon Simple Queue Service (SQS) queue.

# Event Generator setup

## Configure the Event Generator

Once the stack creation has been completed navigate to the stack Outputs tab. Here you will find all the values necessary to configure the Event Generator to work with your account.

1. EventGeneratorConfigurationUrl - this url will pre-populate the Event Generator with all the settings you need.

2. CognitoPassword - your password for logging into the Event Generator

3. CognitoUsername - your user name (defaults to "user")

![EventGeneratorURL1](https://user-images.githubusercontent.com/98637502/216836806-23ea66c2-9b5e-4da6-ad3a-5b4bdc820189.jpg)

4. Right-click the url in the EventGeneratorConfigurationUrl output, opening a new browser window.

5. This will open the Event Generator website and pre-populate the modal dialog box with values for a Cognito User Pool and Cognito Identity Pool provisioned in your AWS account. Click Configure Cognito User Pool to view the sign-in page.

![EventGenerator2](https://user-images.githubusercontent.com/98637502/216836856-9a9c0c31-4483-47aa-885f-95e45be81979.jpg)

![CognitoUserConfig](https://user-images.githubusercontent.com/98637502/216836875-d0f5c7c2-d93c-42f8-8aa3-63bb2e812452.jpg)

6. On the Sign In page add your the CognitoUsername and CognitoPassword from the CloudFormation Stack Output page.

7. Click the Sign In button.

![ConfigCognito](https://user-images.githubusercontent.com/98637502/216836898-e092dad0-895c-4bca-868c-d47ab9fc8f64.jpg)

8. After successful sign in, you verify that the Cognito user has been configured and that your account id has been pre-populated in the AWS Account ID field.

![EventGenerator3](https://user-images.githubusercontent.com/98637502/216836936-988bc2a1-0743-49aa-b533-51e27e7e45e5.jpg)

Till now we have successfully configured our Event Generator.Now, our real project begins from here.

# Event-Driven with Event Bridge.

## High level structure 

![image](https://user-images.githubusercontent.com/98637502/216837050-34557a98-b9b9-4a82-892a-4afa46b18308.png)

let us know bit about the Event-Bridge

Amazon EventBridge is a serverless event bus service that makes it easy to connect your applications with data from a variety of sources. EventBridge delivers a stream of real-time data from your own applications, Software-as-a-Service (SaaS) applications, and AWS services and routes that data to targets such as AWS Lambda. You can set up routing rules to determine where to send your data to build application architectures that react in real time to all of your data sources.

The key concept that are mandatory in understanding the Event Bridges are :

- Events - An event indicates a change in an environment. As you can see in high level design the Events can be SaaS Apps, Microservices and any of your custom app.

- Rules - A rule matches incoming events and routes them to targets for processing. A single rule can route to multiple targets, all of which are processed in parallel. Rules aren't processed in a particular order. This enables different parts of an organization to look for and process the events that are of interest to them. A rule can customize the JSON sent to the target, by passing only certain parts or by overwriting it with a constant.

- Targets - A target processes events. Targets can include Amazon EC2 instances, Lambda functions, Kinesis streams, Amazon ECS tasks, Step Functions state machines, Amazon SNS topics, Amazon SQS queues, and built-in targets. A target receives events in JSON format.

- Event buses - An event bus receives events. When you create a rule, you associate it with a specific event bus, and the rule is matched only to events received by that event bus. Your account has one default event bus, which receives events from AWS services. You can create custom event buses to receive events from your custom applications. You can also create partner event buses to receive events from SaaS partner applications.

We will use Event Bridge to create an event bus, route events to different targets like SNS, API Gateway and Step Function in our case,and use scheduling expressions to create recurring events.

Let's create our First Event Bridge and Target

# Event Bridge and Target

![image](https://user-images.githubusercontent.com/98637502/216837391-e1c6a821-35a4-45e3-bca2-9f822973534c.png)

you will create a custom EventBridge event bus, Orders, and an EventBridge rule, OrderDevRule, which matches all events sent to the Orders event bus and sends the events to a CloudWatch Logs log group, /aws/events/orders. See the diagram above:

The technique of logging all events to CloudWatch Logs is useful when implementing EventBridge rules.

## Create a Custom Event Bus

1. Open The [AWS Management Console for the Event Bridge](https://ap-southeast-1.console.aws.amazon.com/events/home?region=ap-southeast-1#/).

2. On the EventBridge homepage, under Events, select Event buses from the left navigation.

![EB1](https://user-images.githubusercontent.com/98637502/216837555-8aa9ca08-50e4-484e-9116-9c9a7e851d9e.jpg)

3. Click Create event bus.

![EB2](https://user-images.githubusercontent.com/98637502/216837572-015e38b1-482d-421c-abd1-93c0406e55c1.jpg)

4. Name the event bus Orders.

5. Leave Event archive and Schema discovery disabled, Resource-based policy blank.

6. Click Create.

![EB3](https://user-images.githubusercontent.com/98637502/216837598-7f75ddd7-72f0-4b4f-b79f-56a484a05a7a.jpg)

## Set up Amazon CloudWatch target

1. From the left-hand menu, select Rules.

2. From the Event bus dropdown, select the Orders event bus

3. Click Create rule

![EB4](https://user-images.githubusercontent.com/98637502/216837646-5995b947-a238-4e15-8fa0-629e178f0116.jpg)

4. Define rule detail:

 - Add OrdersDevRule as the Name of the rule
 
 - Add Catchall rule for development purposes for Description
 
 - Select Rule with an event pattern for the Rule type
 
![EB5](https://user-images.githubusercontent.com/98637502/216837691-6557e5b4-3b34-4c2d-88ef-3c6095896a29.jpg)

5. In the next step, Build event pattern

![EB6](https://user-images.githubusercontent.com/98637502/216837720-c939f5a4-1a77-46f6-952a-22d19c20f898.jpg)

Under Event pattern, further down the screen, enter the following pattern to catch all events from com.aws.orders:

```bash
{
   "source": ["com.aws.orders"]
}
```

Select next.

6. Select your rule target:

 - From the Target dropdown, select CloudWatch log group
 
 - Name your log group /aws/events/orders
 
![EB7](https://user-images.githubusercontent.com/98637502/216837799-d6f9d08e-ab51-48f0-9150-acbd458b1099.jpg)

7. Skip through the configure tags section, review your rule configuration and click Create.

## Test your Dev rule

1. Select the EventBridge tab in the Event Generator

2. Make sure that the Event Generator is populated with the following :
  
  - AWS Region should be to the region in which you are running the workshop
  - Event Bus selected to Orders
  - Source should be com.aws.orders
  - In the Detail Type add Order Notification
  - JSON payload for the Detail Template should be:
  
  ```bash
  {
   "category": "lab-supplies",
   "value": 415,
   "location": "eu-west"
}
```

![EB8](https://user-images.githubusercontent.com/98637502/216837904-7e6f8e9f-e301-4652-a414-4a94c33d788b.jpg)

3. Click Publish.

4. Open the [AWS Management Console for Cloud Watch](https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1#).

5. Choose Log groups in the left navigation and select the /aws/events/orders log group.

![EB9](https://user-images.githubusercontent.com/98637502/216837988-38fc7f9b-399c-49da-a0e6-d3c46e92e95e.jpg)

6. Select the Log stream.

![EB10](https://user-images.githubusercontent.com/98637502/216838000-ae6ff28b-9e19-47c4-bd7b-ab542673a869.jpg)

7. Toggle the log event to verify that you received the event.

![EB11](https://user-images.githubusercontent.com/98637502/216838034-af73b6fd-e901-46d2-abbd-9406859d4e1f.jpg)

## Review event structure

In the following sections, you will use event data to implement EventBridge custom rules to route events. Due to the OrdersDevRule that you created in this section, all events to the Orders event bus will be sent to CloudWatch Logs, which you can use to view sample data in order to implement and troubleshoot rules matching logic.

For your reference:

```bash
{
    "version": "0",
    "id": "c04cc8c1-283c-425e-8cf6-878bbc67a628",
    "detail-type": "Order Notification",
    "source": "com.aws.orders",
    "account": "111111111111",
    "time": "2020-02-20T23:10:29Z",
    "region": "us-west-2",
    "resources": [],
    "detail": {
        "category": "lab-supplies",
        "value": 415,
        "location": "eu-west"
    }
}
```

## Working with EventBridge rules

![image](https://user-images.githubusercontent.com/98637502/216838150-4cd4d407-4ba9-4a5d-a88a-81272d42b2ab.png)

Rules match incoming events and routes them to targets for processing. A single rule can route to multiple targets, all of which are processed in parallel. Rules aren't processed in a particular order. A rule can customize the JSON sent to the target, by passing only certain parts or by overwriting it with a constant. EventBridge supports 28+ AWS service targets!

In this project, you will walk through the steps to create an Orders event bus rule to match an event with a com.aws.orders source and to send the event to an Amazon API Gateway endpoint, invoke a AWS Step Function, and send events to an Amazon Simple Notification Service (Amazon SNS) topic.

## API Destination End Point

### Identify the API URL

Open the [AWS Management Console for Cloud formation](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks). You can find the API URL for this challenge in the Outputs of the CloudFormation Stack with a name containing ApiUrl.

![APIURL](https://user-images.githubusercontent.com/98637502/216838326-47fc8bb6-a6db-4ec0-8f2e-0fc106ec3120.jpg)

### Configure the EventBridge API Destination with basic auth security

1. Open the [AWS Management Console for Event Bridge](https://ap-southeast-1.console.aws.amazon.com/events/home?region=ap-southeast-1#/).

2. On the EventBridge homepage, select API destinations from the left navigation.

3. On the API destinations page, select Create API destination.

![CreateAPIDestination](https://user-images.githubusercontent.com/98637502/216838445-ec21ef7f-bad3-4a0a-b288-cce73205c949.jpg)

4. On the Create API destination page

 - Enter api-destination as the Name
 - Enter the API URL identified in Step 1 as the API destination endpoint
 - Select POST as the HTTP method
 - Select Create a new connection for the Connection
 - Enter basic-auth-connection as the Connection name
 - Select Basic (Username/Password) as the Authorization type
 - Enter your userName
 - Enter your Password

![CreateAPIDestinationConfig](https://user-images.githubusercontent.com/98637502/216838556-5859a75f-ba65-4656-9c3c-ed30aeac0f7d.jpg)

5. Click Create

### Configure an EventBridge rule to target the EventBridge API Destination

1. From the left-hand menu, select Rules.

2. From the Event bus dropdown, select the Orders event bus.

3. Click Create rule

4. On the Define rule detail page
 - Enter OrdersEventsRule as the Name of the rule
 - Enter Send com.aws.orders source events to API Destination for Description

5. Under Build event pattern
 - Choose Other for your Event source
 - Copy and paste the following into the Event pattern, and select Next to specify your target:

```bash
{
    "source": [
        "com.aws.orders"
    ]
}
```

![EBruletargetAPI1](https://user-images.githubusercontent.com/98637502/216838763-681d106b-e9b2-440a-94b0-7c1c68bde5b6.jpg)

6. Select your rule target:

 - Select EventBridge API destination as the target type.
 - Select api-destination from the list of existing API destinations

![EBruletargetAPI2](https://user-images.githubusercontent.com/98637502/216838780-8b077eff-07b2-44d4-84a2-4ed0ce3321bd.jpg)

7. Click Next and finish walking through the rest of the walk-through to create the rule.

### Send test Orders event

Using the Event Generator, send the following Order Notification events from the source com.aws.orders:

```bash
{ "category": "lab-supplies", "value": 415, "location": "us-east" }
```

### Verify API Destination

If the event sent to the Orders event bus matches the pattern in your rule, then the event will be sent to an API Gateway REST API endpoint.

1. Open the [AWS Management Console for Cloud Watch Log Group](https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1#logsV2:log-groups).

2. Select the Log group with an API-Gateway-Execution-Logs prefix.

![VerifyApiDestination](https://user-images.githubusercontent.com/98637502/216838959-d7727130-0a79-4bf5-b8ce-31208a98199f.jpg)

3. Select the Log stream.

![SelectAPILogStream](https://user-images.githubusercontent.com/98637502/216838978-93ed0f2c-b764-47c9-8a8b-ddf05e0e98e6.jpg)

4. Toggle the log event to verify the basic authorization was successful

![APILogEvent](https://user-images.githubusercontent.com/98637502/216839001-4772d4a1-9ea9-4c16-aff0-f5cd300dbdb0.jpg)

And hereby you have successfully created your first custom event(API Gateway).

## Step Function End Point

###  Implement an EventBridge rule to target Step Functions

Use the EventBridge Console to:

1. Add a rule to the Orders event bus with the name "EUOrdersRule"
2. Define an event pattern to match events with a detail location in "eu-west or eu-east".

```bash
{
  "source": [
    "com.aws.orders"
  ],
  "detail": {
    "location": [
      "eu-west",
      "eu-east"
    ]
  }
}

```

4. Target the OrderProcessing Step Functions state machine

You can refer this simple sample event to create your event pattern:
```bash
{
    "version": "0",
    "id": "6e6b1f6d-48f8-5dff-c2d2-a6f22c2e0086",
    "detail-type": "Order Notification",
    "source": "com.aws.orders",
    "account": "111111111111",
    "time": "2020-02-23T15:35:41Z",
    "region": "us-east-1",
    "resources": [],
    "detail": {
        "category": "office-supplies",
        "value": 300,
        "location": "eu-west"
    }
}
```

make sure to enclose every value in any array or an object format or else it'll generate an "Event Patern is not valid " error.
For example :
```bash
"version" : ["0"] 
```

Also, make sure to write the target service ARN in resource and fill the account and region with your AWS account number and region you choose for the project.

![StepFunction](https://user-images.githubusercontent.com/98637502/216886226-c58045a1-4ed7-4fd0-993f-2b13d9476c93.jpg)

###  Send test EU Orders events

To verify our event is reached to it's desired destination (Step Function) we'll generate an event using Event Generator and send the following "Order Notification" events from the source "com.aws.orders":

```bash
{ "category": "office-supplies", "value": 300, "location": "eu-west" }

{ "category": "tech-supplies", "value": 3000, "location": "eu-east" }
```

### Verify Step Functions workflow execution

If the event sent to the Orders event bus matches the pattern in your rule, then the event will be sent to the OrderProcessing Step Functions state machine for execution.

1. Open the [AWS Management Console for Step Function](https://ap-southeast-1.console.aws.amazon.com/states/home?region=ap-southeast-1).

2. On the Step Functions homepage, open the left hand navigation and select State machines.

![State Machine](https://user-images.githubusercontent.com/98637502/216887127-41a9591c-a805-4eb7-b3bd-c6700b16ef17.jpg)

3. Enter OrderProcessing in the Search for state machines box and verify the state machine execution has succeeded.

![Step Function](https://user-images.githubusercontent.com/98637502/216929763-989b16e8-e202-4bc2-bb9b-6f813dbcb504.jpg)

Note :

If you are not seeing any executions in your OrderProcessing state machine after successfully publishing an event, it is possible that there is an issue with the event or with the configuration of the state machine. Here are a few things to check:

1. Verify that the event was published to the correct EventBridge bus and with the correct event pattern. Make sure that the event pattern you used to create the target for your state machine matches the event that was published.

2. Check the CloudWatch logs for the state machine to see if there were any errors or exceptions that prevented the state machine from being triggered by the event.

3. Check the state machine definition to verify that it is correctly configured to listen to events from EventBridge. Ensure that the state machine is set up as a target for the EventBridge rule that you created.

4. Verify that the IAM role associated with the state machine has the necessary permissions to access EventBridge and start executions.

5. Check the event details in the EventBridge console to see if there were any errors or issues with the event.

By checking these factors, you should be able to identify and resolve the issue with the executions not appearing in your OrderProcessing state machine.

And upto this point we have successfully completed a Step Function event task.

And now we have to use the similar approach for SNS Topic,let's take a deep dive.

## SNS Topic Challenge

Our primary goal is to process only orders from US locations (us-west or us-east) that are lab-supplies using a Amazon SNS target (Orders). Similar to the previous use case, but using SNS.

let's go for it!!

## Implement an EventBridge rule to target SNS

Use the EventBridge Console to:

1. Add a rule to the Orders event bus with the name "USLabSupplyRule"

2. With an event pattern to match events with a detail location in us-west or us-east, and a detail category with lab-supplies.

```bash
{
  "source": [
    "com.aws.orders"
  ],
  "detail": {
    "location": [
      "us-west",
      "us-east"
    ],
    "category": "lab-supplies"
  }
}
```

3. Target the Orders SNS topic

![Rule for SNS Topic](https://user-images.githubusercontent.com/98637502/216888304-226a6524-1949-4cb9-95a8-0be4d671c880.jpg)

Here, is the sample event to refer :

```bash
{
    "version": "0",
    "id": "6e6b1f6d-48f8-5dff-c2d2-a6f22c2e0086",
    "detail-type": "Order Notification",
    "source": "com.aws.orders",
    "account": "111111111111",
    "time": "2020-02-23T15:35:41Z",
    "region": "us-east-1",
    "resources": [],
    "detail": {
        "category": "lab-supplies",
        "value": 300,
        "location": "us-east"
    }
}
```

### Send test US Orders events

Note : One of the following events should match the event rule pattern and one should not. Use CloudWatch Logs to verify events that were successfully sent to EventBridge but were not delivered to the target.

To verify our event is reached to it's desired destination (SNS Topic) we'll generate an event using Event Generator and send the following "Order Notification" events from the source "com.aws.orders":

```bash

{ "category": "lab-supplies", "value": 415, "location": "us-east" }

{ "category": "office-supplies", "value": 1050, "location": "us-west", "signature": [ "John Doe" ] }

```

###  Verify SNS topic

If the event sent to the Orders event bus matches the pattern in your rule, then the event will be sent to the Orders SQS Queue (via Orders SNS Topic).

1. Open the [AWS Management Console for SQS](https://ap-southeast-1.console.aws.amazon.com/sqs/v2/home?region=ap-southeast-1#/queues) in a new tab.

2. On the SQS homepage, select the Orders queue.

![Orders Queue](https://user-images.githubusercontent.com/98637502/216888814-27f38bfe-6880-4fa8-af62-addeacc016f6.jpg)

3. Select the Send and receive messages button.

![SNS2](https://user-images.githubusercontent.com/98637502/216888964-5ee1f2e0-1c1b-40b9-bacc-5e2ad5e9cd95.jpg)

4. Select Poll for Messages and verify the first message was delivered and the second was not.

![SQS1](https://user-images.githubusercontent.com/98637502/216931836-5217d60e-264b-4b5b-8acd-1a36c2f04a52.jpg)

![SQS2](https://user-images.githubusercontent.com/98637502/216931885-573a243e-8a17-4a20-92ff-fe6b35d9d2d5.jpg)

5. To clean up, select the event, select the Delete button, and select the Delete button again on the Delete Messages confirmation dialog.

![Delete1](https://user-images.githubusercontent.com/98637502/216932783-39efc1b5-786e-4d8c-9d3f-46e4dfa1b8eb.jpg)

![Delete2](https://user-images.githubusercontent.com/98637502/216932847-ccb704b5-6798-460f-b977-a4add1cafe2f.jpg)

Note :

If you are not seeing your messages in SQS Queue then their can be few reasons that you should check :

1. Check if the rule is correctly associated with the SNS topic and the SQS queue. Confirm that the ARN of the SNS topic in the rule is correct and that the SQS queue is correctly specified as the target of the rule.

2. Verify that the event pattern in the rule matches the events generated. The events must match the event pattern for the rule to trigger and send messages to the SQS queue.

3. Ensure that the SNS topic has sufficient permissions to publish messages to the SQS queue. You can check this in the SNS topic's access policy.

4. Confirm that the SQS queue has sufficient permissions to receive messages from the SNS topic. You can check this in the SQS queue's access policy.

5. Make sure that the SQS queue is not configured to delay delivery of messages. If it is, messages will not be immediately available in the queue.

6. Check if there are any errors in the CloudWatch logs. This could indicate a problem with the rule or the target.

Congratulations, We have successfully completed our SNS Challenge.

We, can also schedule expressions for rule.

## Scheduling expressions for rules

You can create rules that self-trigger on an automated schedule in EventBridge using cron or rate expressions. All scheduled events use UTC time zone and the minimum precision for schedules is 1 minute.

EventBridge supports cron expressions and rate expressions. Rate expressions are simpler to define but don't offer the fine-grained schedule control that cron expressions support. For example, with a cron expression, you can define a rule that triggers at a specified time on a certain day of each week or month. In contrast, rate expressions trigger a rule at a regular rate, such as once every hour or once every day.

Let's give it a try!

### Create a scheduled Orders Reconciliation rule

1. Open the [AWS Management Console for Event Bridge](https://ap-southeast-1.console.aws.amazon.com/events/home?region=ap-southeast-1#/) in a new tab.

2. On the EventBridge homepage, select Rules from the left-hand menu.

3. Select the default event bus and click the Create rule button.

![Scheduled Orders](https://user-images.githubusercontent.com/98637502/216892139-a15c66c8-497a-4a2a-80e7-3a4a0bb2e976.jpg)

Note : You can only use a scheduled expression for the default event bus.

4. On the Create rule page:

 - Add OrdersReconciliation as the Name of the rule
 - Add Runs reconciliation routine on orders every minute Monday to Friday for Description
 - Select Schedule for Rule type
 - As we are still using the previous scheduler, you should click on the Continue to create rule button.

![NewRuleForSchedule](https://user-images.githubusercontent.com/98637502/216892304-c415af5c-51b6-4fe8-a42e-1d9808633926.jpg)

5. In Schedule Pattern select the left option, which represnts a cron expression.

6. Add * * ? * MON-FRI * in the Cron expression.

![DefineSchedule](https://user-images.githubusercontent.com/98637502/216892422-f023875a-9fee-46a8-8700-f2669457d289.jpg)

This will schedule a message to be delivered every minute, Monday through Friday.

7. Configure your target to be a CloudWatch log group

 - Name the log group /aws/events/orders_reconciliation

![SelectTarget](https://user-images.githubusercontent.com/98637502/216892678-ac0a0d12-46a5-494a-9e59-c152c4dbf30d.jpg)

8. Click Next and finish walking through the rest of the walk-through to create the rule.

### Verify scheduled message delivery

1. Open the [AWS Management Console for Cloud Watch Log Group](https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1#logsV2:log-groups).

2. Choose Log groups in the left navigation and select the /aws/events/orders_reconciliation log group.

3. After a few minutes your streams for /aws/events/orders_reconciliation should look similar to this:

![CloudWatchForSchedule](https://user-images.githubusercontent.com/98637502/216893513-760dce78-d163-45fc-a4d1-cef47929bf17.jpg)

And thereby we hace completed our Event Driven Architecture using Amazon Event Bridge!!!

Make sure that your terminate each of the service/resources to be safe from the unexpected surprise bill at the end of the month ???? !!




















