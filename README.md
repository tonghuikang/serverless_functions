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

### Characterisitics
- Organisation of functions. All of the lambda functions that you can access should appear on https://console.aws.amazon.com/lambda/home?your-region-here#/functions. Each function will be assigned certain role, each role will need to be assigned certain policy (as above), and then the functions can call each other.  

TODO: Allow the function to be called externally. Understand what AWS Step Functions does.

## Azure
When I wanted to create a nodejs function, I was asked to read these tutorials in this order
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node

A working sample of the individually running function is found here:
https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function (HTTPS trigger)

TODO: Allow the functions to be called by each other, and called externally.

Making http call from function: https://stackoverflow.com/questions/52546638/how-to-make-https-call-from-azure-function
https://stackoverflow.com/questions/50823783/in-azure-functions-using-javascript-how-to-call-send-request-to-a-endpoint

https://medium.com/@burkeholland/solving-the-azure-functions-challenge-with-javascript-65205b0c4920 - functions does not call each other :/

Authorization level - What is the difference? 
> Authorization level controls whether the function requires an API key and which key to use; Function uses a function key; Admin uses your master key. The function and master keys are found in the 'keys' management panel on the portal, when your function is selected. For user-based authentication, go to Function App Settings.

### Characterisitics
- Organisation of functions. Functions are organised by function app, and each function app must be authored in the same language.
- 


## GCP 

TODO: Later.

### Characterisitics
- Organisation of functions
- 

# Conclusion
To elaborate: The extent of vendor lock-in, and best practices to ensure portability. 
