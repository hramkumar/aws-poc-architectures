# aws-poc-serverless-architecture

This exercise provides with instructions for how to build a proof of concept for a serverless solution in the AWS Cloud.

Suppose you have a customer that needs a serverless web backend hosted on AWS. The customer sells something and they need an architecture that can easily scale in and out as demand changes. The customer also wants to ensure that the application has decoupled application components. In other words a microservices architecture.

The following architectural diagram shows the flow for the serverless solution that will be built.

![aws-poc-serverless-architecture drawio](https://user-images.githubusercontent.com/8630317/208302822-1e7fccfb-3769-4d12-a9d2-18bb8b6a87bf.png)


### Creating necessary Policies and Roles

1. Sign in to the AWS Management Console.

2. In the search box, enter IAM.

3. From the results list, choose IAM.

4. In the navigation pane, choose Policies.

5. Choose Create policy.

The Create policy page appears. You can create and edit a policy in the visual editor or use JSON. In this exercise, we provide JSON scripts to create policies. In total, you must create four policies.

6. In the JSON tab, paste the following code:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Action": [
                "dynamodb:PutItem",
		"dynamodb:GetItem",
		"dynamodb:DeleteItem",
                "dynamodb:DescribeTable"
            ],
            "Effect": "Allow",
            "Resource": "*"
        },
	    {
            "Sid": "VisualEditor1",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Effect": "Allow",
            "Resource": "*"       
	    }
	]
}
```
7. Choose Next: Tags and then choose Next: Review.

8. For the policy name, enter `Lambda-Write-DynamoDB`.

9. Choose Create policy.

### Creating IAM roles and attaching policies to the roles

1. In the navigation pane of the IAM dashboard, choose Roles.

2. Choose Create role and in the Select trusted entity page, configure the following settings:
* Trusted entity type: AWS service
* Common use cases: Lambda
* Choose Next.

3. On the Add permissions page, select `Lambda-Write-DynamoDB` and Choose Next

4. For Role name, enter `Lambda-DynamoDB-Role`.

5. Choose Create role.

### Creating a DynamoDB table

1. In the search box of the AWS Management Console, enter `DynamoDB`.

2. From the list, choose the DynamoDB service.

3. On the Get started card, choose Create table and configure the following settings:
* Table: `name`
* Partition key: `id`
* Data type: Keep String

4. Keep the remaining settings at their default values, and choose Create table.

### Creating a Lambda function

1. In the AWS Management Console search box, enter Lambda and from the list, choose `Lambda`.

2. Choose Create function and configure the following settings:
* Function option: Author from scratch
* Function name: `POC-Lambda-1`
* Runtime: Python 3.9
* Change default execution role: Use an existing role
* Existing role: `Lambda-DynamoDB-Role`

3. Choose Create function.

### Adding and deploying the function code

1. On the POC-Lambda-1 page, in the Code tab, replace the default Lambda function code with the following code:

Note: The source code is from https://github.com/saha-rajdeep

```
from __future__ import print_function

import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
```

2. Choose Deploy.

The Lambda code passes arguments to a function call. As a result, when a trigger invokes a function, Lambda runs the code that you specify.

When you use Lambda, you are responsible only for your code. Lambda manages the memory, CPU, network, and other resources to run your code.

### Testing the POC-Lambda-1 Lambda function

1. In the Test button, click on the drop down arrow and select `Configure test event`
2. And choose the following settings:
* Event name: `POC-Lambda-Test-1`
* Template-Optional: `dynamodb-update`

3. In the Event JSON field, paste the below code and click on Save.

```
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```

4. Now, click on Test.

After the Lambda function runs successfully, the “Execution result: succeeded” message appears in the notification banner in the Test section.

### Copy the ARN of the Lambda Function
Navigate to Lambda > Functions > POC-Lambda-1 if you are on some other page.
1. On the POC-Lambda-1 page, expand "Function overview" and copy the Function ARN.

It will look something like this:
```
arn:aws:lambda:us-east-1:xxxxxxxxxxxx:function:POC-Lambda-1
```

### Create API in API Gateway

1. In the AWS Management Console, search for and open `API Gateway`.

2. On the REST API card with a public authentication, choose Build and configure the following settings:
* Choose an API type: `REST API` and click on "Build"
Note: If you get any pop-up "Create your first API" click on X (close)
* Choose the protocol: `REST`
* Create new API: `New API`
* API name: `POC-API`
* Endpoint Type: Regional

3. Choose Create API.

4. On the Actions menu, choose `Create Resource`.
5. Input `DynamoDBManager` in the Resource Name, Resource Path will get populated. Click "Create Resource"
6. Select `DynamoDBManager` and on the Actions menu, choose `Create Method`.
7. Open the method menu by choosing the down arrow, and choose `POST`. Save your changes by choosing the check mark.
8. In the POST - Setup pane, configure the following settings:
* Integration type: Lambda Function
* AWS Region: us-east-1
9. The integration will come up automatically with "Lambda Function" option selected. Copy and Paste the Function ARN for `POC-Lambda-Test-1` that we created earlier.
10. On the pop-up "Add Permission to Lambda Function" clock on OK.

### Testing the API Gateway with DynamoDB

1. Select the POST Method and click on the TEST icon.
2. Paste the below code in `Request Body` and click on TEST.

```
{
    "operation": "create",
    "tableName": "name",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}
```
3. The test should be successful and the response should be something like this:

```
Request: /dynamodbmanager
Status: 200
Latency: 299 ms
Response Body
```

### Deploy the API

1. Click "Actions", select "Deploy API"
2. Now it is going to ask you about a stage.
3. Select `[New Stage]` for "Deployment stage".
4. Give `Staging` as "Stage name". Click "Deploy".
5. Copy the "Invoke URL" from screen and paste it somewhere.

```
https://11qe1w1rt1.execute-api.us-east-1.amazonaws.com/Staging/dynamodbmanager
```

To Test with our API endpoint, we need the endpoint url.

We're all set to run our solution!

### Running our AWS POC Serverless solution

1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:

```
{
    "operation": "create",
    "tableName": "name",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "name": "Rob"
        }
    }
}
```

2. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select `name` table, select "Items" tab, and the newly inserted item should be displayed.

### Cleaning up

In this task, we will delete the AWS resources that you created for this exercise.

1. Delete the DynamoDB table.
* Open the DynamoDB console.
* In the navigation pane, choose Tables.
* Select the `name` table.
* Choose Delete and confirm your actions.

2. Delete the Lambda functions.
* Open the Lambda console.
* Select the Lambda function that we created `POC-Lambda-1`
* Choose Actions, Delete.  
* Confirm your actions and close the dialog box.
  
3. Delete the API that you created.
* Open the API Gateway console.
* Select `POC-API`.
* Choose Actions, Delete.
* Confirm your actions.  

I hope you enjoyed and learnt something. Keep learning!
