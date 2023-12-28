# ☁️ AWS Project - Serverless AWS Microservices Lab - REST API

## 📄 Introduction

A hands-on step-by-step lab of building a serverless microservices architecture right in the AWS console. We will create a working REST API to trigger our lambda function to perform CRUD operations in our database.

## 📝 Lab Overview

As shown in the architecture design below, we have serverless API microservice architecture, if we read it from left to right you'll notice that we have a client and that client is going to evoke a ref API through our API Gateway the API Gateway will then trigger a Lambda function and in our Lambda function, we're going to embed it with code python code and that code will allow the Lambda function to perform CRUD operations (Create Read Update and Delete), then Lambda will have the ability to perform CRUD operations into DynamoDB.




## 📐 Architecture Design

![Microservice API Gateway](https://github.com/julien-muke/AWS-Serverless-Microservices-API-Architecture/assets/110755734/93cd73bd-e126-44de-9cfa-104bc8539395)



## ➡️ Step 1 - Create DynamoDB table

Create the DynamoDB table that the Lambda function uses.

To create a DynamoDB table

1. Open the DynamoDB console.
2. Choose Create table.
3. Create a table with the following settings.
     * Table name – apigateway-lambda-crud
     * Primary key – id (string)
4. Choose Create.





## 💰 Cost

All services used are eligible for the AWS Free Tier. However, charges will incur at some point so it's recommended that you shut down resources after completing this tutorial.





























