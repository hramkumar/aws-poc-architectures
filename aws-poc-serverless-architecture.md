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

```{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
		"dynamodb:GetItem",
		"dynamodb:DeleteItem",
                "dynamodb:DescribeTable"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ],
	{
            "Sid": "",
            "Resource": "*",
            "Action": [
		"logs:CreateLogGroup",
		"logs:CreateLogStream",
		"logs:PutLogEvents"
            ],
            "Effect": "Allow"
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

1. In the AWS Management Console search box, enter Lambda and from the list, choose Lambda.

2. Choose Create function and configure the following settings:
* Function option: Author from scratch
* Function name: POC-Lambda-1
* Runtime: Python 3.9
* Change default execution role: Use an existing role
* Existing role: `Lambda-DynamoDB-Role`

3. Choose Create function.

### Adding and deploying the function code

1. On the POC-Lambda-1 page, in the Code tab, replace the default Lambda function code with the following code:

```
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```

2. Choose Deploy.

The Lambda code passes arguments to a function call. As a result, when a trigger invokes a function, Lambda runs the code that you specify.

When you use Lambda, you are responsible only for your code. Lambda manages the memory, CPU, network, and other resources to run your code.

### Testing the POC-Lambda-1 Lambda function

1. In the Test tab, create a new event that has the following settings:
* Event name: `POC-Lambda-Test-1`
* Template-Optional: `DynamoDB`
The SQS template appears in the Event JSON field.

3. Save your changes and choose Test.

After the Lambda function runs successfully, the “Execution result: succeeded” message appears in the notification banner in the Test section.

### Create API in API Gateway

1. In the AWS Management Console, search for and open `API Gateway`.

2. On the REST API card with a public authentication, choose Build and configure the following settings:
* Choose the protocol: REST
* Create new API: New API
* API name: POC-API
* Endpoint Type: Regional

3. Choose Create API.

4. On the Actions menu, choose `Create Resource`.

5. Input `DynamoDBManager` in the Resource Name, Resource Path will get populated. Click "Create Resource"

6. Select `DynamoDBManager` and on the Actions menu, choose `Create Method`.

7. Open the method menu by choosing the down arrow, and choose POST. Save your changes by choosing the check mark.

8. In the POST - Setup pane, configure the following settings:
* Integration type: Lambda Function
* AWS Region: us-east-1

9. The integration will come up automatically with "Lambda Function" option selected. Select `POC-Lambda-Test-1` function that we created earlier.

### Deploy the API

1. Click "Actions", select "Deploy API"

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"

We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen

### Running our AWS POC Serverless solution

1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:

```
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```

2. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select `POC-Lambda-Test-1` table, select "Items" tab, and the newly inserted item should be displayed.

### Cleaning up

In this task, we will delete the AWS resources that you created for this exercise.

1. Delete the DynamoDB table.
  * Open the DynamoDB console.
  * In the navigation pane, choose Tables.
  * Select the orders table.
  * Choose Delete and confirm your actions.

2. Delete the Lambda functions.
  * Open the Lambda console.
  * Select the Lambda function that you created in this exercise: POC-Lambda-1
  * Choose Actions, Delete.  
  * Confirm your actions and close the dialog box.
  
3. Delete the API that you created.
* Open the API Gateway console.
* Select POC-API.
* Choose Actions, Delete.
* Confirm your actions.  
