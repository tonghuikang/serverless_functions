# Serverless Functions
This repository records my journey is using serverless functions (which refers to AWS Lambda, Azure Functions, and Google Cloud Functions collectively).

# What are serverless functions?
It is one way to implement cloud computing. Cloud computing is running code on other peoples' computers. (AWS, Azure, Google)

![Comparsion of cloud services](https://www.cloudhealthtech.com/sites/default/files/images/Aws%20vs%20Azure.png)
 
## Levels of abstraction in cloud computing 
Down the list there the offerings have fewer restrictions but greater cost of commitment.
- **Serverless computing** (serverless functions). There are great limitations, and it is by design. You can only use one supported language per function. You are only billed per invocation of the function.
- **Platform as a Service**. You can now deploy a collection of functions, which may be different languages. You have access to temporary storage. The platform can be automatically scaled with Kubernetes if you can pack your code into a container. You are billed per uptime of the platform.
- **Virtual servers**. You are essentially having a remote computer to yourself, which you can do anything to it. You are billed as long as the computer is online, even if it is idle.

In this repository, I will experiment with serverless functions, and record how did I make the functions work. I will record down the tutorials and forum replies (i.e., StackOverflow) that helped me, and provide a working snippet of code. The beginner end goal is **a collection of functions that can call each other**, and some of these functions **can be invoked externally**.

The three major cloud providers offer serverless functions with different terminologies and specifications. 

![Comparsion of serverless functions](https://www.simform.com/wp-content/uploads/2018/09/Dia1-5-1.png)


# How to set up for different cloud providers
The respective folders will include a working set of function code.

## AWS
The UI is pretty intuitive to guide me to save and test an individual working function. 

How to let one function call another: https://stackoverflow.com/questions/35754766/nodejs-invoke-an-aws-lambda-function-from-within-another-lambda-function

Changes made: https://stackoverflow.com/a/35795883 need to add (or replace with) `AWSLambdaRole` and `AWSLambdaBasicExecutionRole` as per https://stackoverflow.com/a/50656623. 
The regions need to be consistent with your resource, as well.

TODO: Allow the function to be called externally.

## Azure
When I wanted to create a nodejs function, I was asked to read these tutorials in this order
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node

A working sample of the individually running function is found here:
https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function (HTTPS trigger)

TODO: Allow the function to be called by each other, and called externally.

## GCP 

TODO: Later.
