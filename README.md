# ☁️ AWS Project - Serverless AWS Microservices Lab - REST API

## 📄 Introduction

A hands-on step-by-step lab of building a serverless microservices architecture right in the AWS console. We will create a working REST API to trigger our lambda function to perform CRUD operations in our database.

## 📝 Lab Overview

As shown in the architecture design below, we have serverless API microservice architecture, if we read it from left to right you'll notice that we have a client and that client is going to evoke a ref API through our API Gateway the API Gateway will then trigger a Lambda function and in our Lambda function, we're going to embed it with code python code and that code will allow the Lambda function to perform CRUD operations (Create Read Update and Delete), then Lambda will have the ability to perform CRUD operations into DynamoDB.




## 📐 Architecture Design

![Microservice API Gateway](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/93cd73bd-e126-44de-9cfa-104bc8539395)




The client will perform the API call by using the "POST" HTTP method. Our client must provide a request payload to perform an API call. The request payload identifies the DynamoDB operation (CRUD) the client wants to perform with the necessary data. 

*Assuming **apigateway-lambda-crud** is the table name of our DynamoDB table.*

An example request payload for a CREATE opertion shows as follows:

```json
{
    "operation": "create",
    "tableName": "apigateway-lambda-crud",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Sam"
        }
    }
}
```

An example request payload for a DELETE operation shows as follows:

```json
{
    "operation": "delete",
    "tableName": "apigateway-lambda-crud",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Sam"
        }
    }
}
```

An example request payload for a READ operation shows as follows:

```json
{
    "operation": "read",
    "tableName": "apigateway-lambda-crud",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```




## ➡️ Step 1 - Create DynamoDB table

Create the DynamoDB table that the Lambda function uses.

To create a DynamoDB table

1. Open the DynamoDB console.
2. Choose Create table.
3. Create a table with the following settings.
     * Table name – apigateway-lambda-crud
     * Primary key – id (string)
4. Choose Create.


![Create-table-Amazon-DynamoDB](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/54655498-dce3-48db-b14a-867687d4b815)


## ➡️ Step 2 - Create Lambda IAM Role

Create the execution role that gives your function permission to access AWS resources.
Before you can apply your Lambda Policy to a Lambda function, you have to create the policy in your own account and then apply it to an IAM role.

To create an IAM policy:

1. Navigate to the IAM console and choose Policies in the navigation pane. Choose Create policy.



![Screenshot 2023-12-28 at 17 17 32](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/1d415ade-44d1-4efd-a8eb-216d823171e4)



2. Because I have already written the policy in JSON, you don’t need to use the Visual Editor, so you can choose the JSON tab and paste the content of the JSON policy document shown earlier in this post (remember to replace the placeholder account ID with your own account ID). Choose Review policy.



```bash
{
"Version": "2012-10-17",
"Statement": [
    {
        "Sid": "VisualEditor0",
        "Effect": "Allow",
        "Action": [
            "logs:CreateLogStream",
            "dynamodb:PutItem",
            "dynamodb:DeleteItem",
            "dynamodb:GetItem",
            "dynamodb:Scan",
            "dynamodb:Query",
            "dynamodb:UpdateItem",
            "logs:PutLogEvents",
            "logs:CreateLogGroup"
        ],
        "Resource": "*"
    }
]
}

```




![Create-policy-IAM-Global](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/1a7dfdf2-1715-46fe-b326-41b3c14f26c3)




3. Name the policy `lambda-apigateway-dynamodb-policy` and give it a description that will help you remember the policy’s purpose. You also can view a summary of the policy’s permissions. Choose Create policy.



![Create-policy-IAM-Global (2)](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/4c249817-c3ae-4c9d-8a1e-5c42baa4598c)


You have created the IAM policy that you will apply to the Lambda function.


### 👉 Attach the IAM policy to an IAM role


To apply `lambda-apigateway-dynamodb-policy` to a Lambda function, you first have to attach the policy to an IAM role.

To create a new role and attach MyLambdaPolicy to the role:

1. Navigate to the IAM console and choose Roles in the navigation pane. Choose Create role.



![Screenshot 2023-12-28 at 17 51 57](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/9367ae57-6a38-4efc-b93f-cc1b368ca434)



2. Choose AWS service and then choose Lambda. Choose Next: Permissions.


![Create-role-IAM-Global](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/e63d2fc0-0788-4ed7-bab2-a2c764386684)



3. On the Attach permissions policies page, type `lambda-apigateway-dynamodb-policy` in the Search box. Choose `lambda-apigateway-dynamodb-policy` from the list of returned search results, and then choose Next: Review.



![Screenshot 2023-12-28 at 18 46 19](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/15f4bbb7-a9b2-4fbb-a004-f9b8c7cd10f7)




4. On the Review page, type `lambda-apigateway-dynamodb-role` in the Role name box and an appropriate description, and then choose Create role.



![Create-role-IAM-Global (1)](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/53e22a64-4d5a-4d2d-9f4e-d3a73db5c099)



You have attached the policy you created earlier to a new IAM role, which in turn can be used by a Lambda function.



## ➡️ Step 3 - Apply the IAM role to a Lambda function

You have created an IAM role that has an attached IAM policy that grants both read and write access to DynamoDB and write access to CloudWatch Logs. The next step is to apply the IAM role to a Lambda function.

To apply the IAM role to a Lambda function:

Select "Author from scratch". Use name `LambdaCRUDOverHTTPS` , select Python 3.7 as Runtime. Under Permissions, select "Use an existing role", and select `lambda-apigateway-dynamodb-role` that we created, from the drop down


1. Navigate to the Lambda console and choose Create function.


![Create-function-Lambda](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/1e96bdd3-3f39-4bfc-b4c6-2d95fa0072b3)



2. Replace the existing sample code with the following code snippet and click "Deploy"



```bash
     import boto3
import json

print('Loading function')

def lambda_handler(event,context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    
    print("Received event: " + json.dumps(event, indent=1))
     
    operation = event['operation']
    

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
    #CRUD operations shown below:
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'echo': lambda x: x
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
        
```



![Screenshot 2023-12-28 at 19 01 22](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/07402df1-4c91-40ed-9102-0b1c60faf3e1)




## ➡️ Step 4 - Test Lambda Function


Let's test our newly created function. We will test our "echo" operation AND our "create" operation. "echo" is an operation that should output whatever the request/input/event submitted. "create" is an operation that will create a real record into the DynamoDB table (apigateway-lambda-crud) we created in the first step.



🔵 "echo" Operation TEST

1. Click the arrow on the "Test" button and click "Configure test events".


![Screenshot 2023-12-29 at 10 13 58](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/acce94dc-2bd5-44b8-909d-6920c2d6847b)



2. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Create" to save.


```bash
{
  "operation": "echo",
  "payload": {
    "testkey1": "testvalue1",
    "testkey2": "testvalue2"
  }
}
```


![Screenshot 2023-12-29 at 11 37 22](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/bf1e377a-1cc4-4e87-b4da-59cdb2253503)



3. Click "Test", and it will execute the test event. You should see the output in the console


![Screenshot 2023-12-29 at 11 37 51](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/b1112030-f688-4db9-bcb5-70474eec6c5c)



🔵 "create" Operation TEST


1. Click the arrow on the "Test" button and click "Configure test events"


![Screenshot 2023-12-29 at 11 38 33](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/b347782f-c7ba-4660-b8fb-1325b8fb1bd3)



2. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it will create a new record into our DynamoDB table. Click "Create" to save.


```bash
{
  "operation": "create",
  "tableName": "apigateway-lambda-crud",
  "payload": {
    "Item": {
      "id": "ABC",
      "number": 5,
      "name": "Bob",
      "age": 42
    }
  }
}
```


![Screenshot 2023-12-29 at 11 40 24](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/099a8874-952f-4fc9-966e-c6e25fbe24f4)



3. Click "Test", and it will execute the test event. You should see the output in the console


![Screenshot 2023-12-29 at 11 54 33](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/dceec698-947e-4796-81e8-86daf6de8051)




🔵 "read" Operation TEST

1. Click the arrow on the "Test" button and click "Configure test events"

2. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Create" to save.
3. Click "Test", and it will execute the test event. You should see the output in the console.


```bash
{
  "operation": "read",
  "tableName": "apigateway-lambda-crud",
  "payload": {
    "Key":{"id":"ABC"}
    
  }
}
```


![Screenshot 2023-12-29 at 11 57 27](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/e1d21876-3cc3-4881-a52a-27dd5b12d7f4)



![Screenshot 2023-12-29 at 11 58 03](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/c2ee8238-cb57-445a-b39a-696db40c4930)


 Go back to your DynamoDB tables, click "Explore Items", if you check the `apigateway-lambda-crud` it will execute the test event successfully. You should see the output in the console.


 ![Screenshot 2023-12-29 at 12 53 35](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/0618a5f6-ba70-4e23-a53d-295fa7c65fa8)



## ➡️ Step 5 - Create API


To create the API

1. Go to API Gateway console
2. Scroll down to REST API and click "Build"


![Screenshot 2023-12-29 at 12 08 05](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/505ab2a2-4d03-44e8-9c41-479c685e288c)



3. Make sure to select "New API" and Give the API name as `DynamoOperations`, keep everything as is, click "Create API"



![Screenshot 2023-12-29 at 12 10 28](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/97dd2c9b-c87f-4f6b-9768-9ff0c806c251)



4. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next.

Click slash `/`, then click "Create Resource"

![Screenshot 2023-12-29 at 12 11 29](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/0fd84d6e-3798-4662-a8c1-b9b5b994d807)



5. Input `DynamoOperations` in the Resource Name, Resource Path `/` will get populated. Click "Create Resource"



![Screenshot 2023-12-29 at 12 12 17](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/a4bc062a-a02c-466f-89fe-95c19503aa37)



6. Let's create a `POST` Method for our API. With the `/DynamoOperations` resource selected, Click "Create Method".


![Screenshot 2023-12-29 at 12 13 06](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/60af99f8-e5f8-4437-9976-16f904b70302)



7. Select `POST` from drop down,

The integration will come up automatically with "Lambda Function" option selected. Select `LambdaCRUDOverHTTPS` function that we created earlier. As you start typing the name, your function name will show up. Select and click "Save". Click "Create method"


![Screenshot 2023-12-29 at 12 15 27](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/061d0dc3-6b20-4538-8c3e-ce35a68b7e59)



Our API-Lambda integration is successfully created.



## ➡️ Step 6 - Deploy the API


In this step, you deploy the API that you created to a stage called Prod.

1. Click `POST`, select "Deploy API"



![Screenshot 2023-12-29 at 12 16 50](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/55d60170-d797-4031-9fec-2acc9348a9bf)



2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give `Prod` as "Stage name". Click "Deploy"



![Screenshot 2023-12-29 at 12 17 30](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/221faab5-26ee-419f-bd6c-e40c21634143)



3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen (we are going to need it in Step 7)



![Screenshot 2023-12-29 at 12 47 27](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/1d445af6-85f7-40aa-bd38-326d94cefc71)



## ➡️ Step 7 - Running our solution

1. Go to Postman.com and go to your workspace. (If you've never used Postman then you will have to sign up, no worries it is free and you can use the browser interface)

2. Click the "New" button then select HTTP Request as shown in the screenshot below. 


![Screenshot 2023-12-29 at 12 24 54](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/12d743a9-7c4b-46b3-bd4b-f4f731319393)



3. Switch `GET` to `POST`. Next, copy and paste the Invoke URL that we copied from our API Gateway in the previous step. Lastly, select "Body" then the `raw` option. All shown in the screenshot below.

4. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:

```bash
{
    "operation": "create",
    "tableName": "apigateway-lambda-crud",
    "payload": {
        "Item": {
            "id": "ABCD",
            "number": 879
        }
    }
}
```


![Screenshot 2023-12-29 at 12 27 34](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/3ebbc876-32bc-48e7-8980-71970233eadc)




5. Click "Send". API should execute and return "HTTPStatusCode" 200. 


![Screenshot 2023-12-29 at 12 48 18](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/fe8c51b1-35ec-40a7-9bea-dc82b1ddfba1)



6. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select `apigateway-lambda-crud` table, select "Explore items" tab, and the newly inserted item should be displayed.



![Screenshot 2023-12-29 at 12 53 35](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/aabeb512-02f2-4bd9-9e29-f0388fd3f234)



## 💰 Cost

All services used are eligible for the AWS Free Tier. However, charges will incur at some point so it's recommended that you shut down resources after completing this tutorial.





























