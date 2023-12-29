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



### 👉 Apply the IAM role to a Lambda function

You have created an IAM role that has an attached IAM policy that grants both read and write access to DynamoDB and write access to CloudWatch Logs. The next step is to apply the IAM role to a Lambda function.

To apply the IAM role to a Lambda function:

Select "Author from scratch". Use name `LambdaCRUDOverHTTPS` , select Python 3.7 as Runtime. Under Permissions, select "Use an existing role", and select `lambda-apigateway-dynamodb-role` that we created, from the drop down


1. Navigate to the Lambda console and choose Create function.


![Create-function-Lambda](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/1e96bdd3-3f39-4bfc-b4c6-2d95fa0072b3)














## 💰 Cost

All services used are eligible for the AWS Free Tier. However, charges will incur at some point so it's recommended that you shut down resources after completing this tutorial.





























