# Serverless Functions
This repository records my journey is using serverless functions (which refers to AWS Lambda, Azure Functions, and Google Cloud Functions collectively).

### What are serverless functions?
It is one way to implement cloud computing. Cloud computing is running code on other peoples' computers. (AWS, Azure, Google)

![Comparsion of cloud services](https://www.cloudhealthtech.com/sites/default/files/images/Aws%20vs%20Azure.png)
 
## Levels of abstraction in cloud computing 
Down the list there the offerings have fewer restrictions but more expensive and less scalable.
- **Serverless computing** (serverless functions). There are significant limitations, and it is by design. You can only use one supported language per function. You are only billed per invocation of the function.
- **Platform as a Service**. You can now deploy a collection of functions, which may be different languages. You have access to temporary storage. The platform can be automatically scaled with Kubernetes if you can pack your code into a container. You are billed per uptime of the platform.
- **Virtual servers**. You are essentially having a remote computer to yourself, which you can do anything to it. You are billed as long as the computer is online, even if it is idle.

In this repository, I will experiment with serverless functions, and record how did I make the functions work. I will record down the tutorials and forum replies (i.e., StackOverflow) that helped me, and provide a working snippet of code. The beginner end goal is **a collection of functions that can call each other**, and some of these functions **can be invoked externally**.

The three major cloud providers offer serverless functions with different terminologies and specifications. 

![Comparsion of serverless functions](https://www.simform.com/wp-content/uploads/2018/09/Dia1-5-1.png)


# How to set up for different cloud providers
The respective folders will include a working set of function code.

For each platform, I will briefly describe its merits. Then I will explore how to implement the following list of steps.
- **Getting the first individually working function**. Well, you need to start somewhere.
- **Logging**. We need logging to ensure our function is working as intended during the development stage. During production, we might want to trace requests and errors.
- **Installing packages**. (Who doesn't use packages?) Installation may not be straightforward. We also need to know, and if possible change, of the package installed, and where is it noted.
- **Calling external API**. The function may need to communicate with external API - for example, to put information on Google Sheets. With such an extent of restrictions, we cannot assume that this is a given.
- **Allowing functions to call one another**. It is important to modularise your code for readability and reuseablility. 
- **Asynchronous functions**. JavaScript "is single threaded and has a synchronous execution model. Single threaded means that one command is being executed at a time. Synchronous means one at a time i.e. one line of code is being executed at a time in order the code appears."
This issue happens when you need to get a response from a POST request for the next command. 
In typical implementation (I think this example can be improved) like ```axios = require('axios'); var result = axios.post('/user', {firstName: 'Fred',lastName: 'Flintstone'} console.log(result);``` will return undefined, because the logging function does not wait for the response. 
One remedy is to use callback, but that adds complexity to the code due to nesting. Another way is to use the `await` keyword which forces the command (?) to be completed before proceeding to the next line. The change here that `await` could only be used inside an `async` function. 
- **Exposing the functions externally**. In an internship project, these functions are to serve chatbots. The chatbots need to call these functions. One way is to expose these functions through a public URL. Special bindings may be used depending on your specific cloud provider, like in my case the bot-core is hosted on Azure.
- **Code organisation**. The structure of your code is dictated by your cloud provider with such stringent requirements. We nevertheless want to understand code structure so we can more efficiently convert a typical nodejs app into cloud functions. Moreover, each function may have different levels of privileges and exposure, and this needs to be documented, as well. 
- **Repository synchronisation**. For sustainable code development, we need to be able to synchronise with a repository, which allows easy access for the developer to understand when the code is edited. 
- **Middleware structure implementation**. Non-technical people require specific instructions to deploy code with custom fulfillment format. They need to read and edit easy code structure at the which looks like this:
```
// handles conditional response with redis
app.post('/api/redis-cache/nextIntentAfterLimit', 
  (req, res, next) => {console.log('/api/redis-cache/nextIntentAfterLimit is called...'); next()},
  monika.checkTokenToLimit,
  monika.buildResponseMessages,
  monika.activateNextIntent,
  (req, res) => res.send(req.body))
```
It is a list of functions, at the start it merely logs, and then it passes through middleware functions and at the end the response is sent. 
https://hackernoon.com/middleware-the-core-of-node-js-apps-ab01fee39200 (most useful)
https://stackoverflow.com/questions/7337572/what-does-middleware-and-app-use-actually-mean-in-expressjs
https://expressjs.com/en/guide/using-middleware.html 
https://expressjs.com/en/guide/using-middleware.html 
https://medium.com/@selvaganesh93/how-node-js-middleware-works-d8e02a936113

## AWS
AWS is the most commonly used cloud provider and is compliant with corporate settings. 

### Getting the first individually working function
The UI is pretty intuitive to guide me to save and test an individual working function. lambda_1:
```
exports.handler = function(event, context) {
  console.log('Lambda B Received event:', JSON.stringify(event, null, 2));
  context.succeed('Hello ' + event.name);
};
```

### Logging
`console.log()` works for testing.

### Installing packages
It seems like you need to upload your .zip file with the node_modules.
https://stackoverflow.com/questions/34437900/how-to-load-npm-modules-in-aws-lambda

### Allowing functions to call one another
How to let one function call another: https://stackoverflow.com/questions/35754766/nodejs-invoke-an-aws-lambda-function-from-within-another-lambda-function

Changes made: https://stackoverflow.com/a/35795883 need to add (or replace with) `AWSLambdaRole` and `AWSLambdaBasicExecutionRole` as per https://stackoverflow.com/a/50656623. 
The regions need to be consistent with your resource, as well.

lambda_2 (which calls lambda_1 above):
```
var AWS = require('aws-sdk');
AWS.config.region = 'us-east-1';   # make this shows your region
var lambda = new AWS.Lambda();

exports.handler = function(event, context) {
  var params = {
    FunctionName: 'lambda_1', // the lambda function we are going to invoke
    InvocationType: 'RequestResponse',
    LogType: 'Tail',
    Payload: '{ "name" : "Mike" }'
  };

  lambda.invoke(params, function(err, data) {
    if (err) {
      context.fail(err);
    } else {
      context.succeed('lambda_1 said '+ data.Payload);
    }
  })
};
```

### Calling external API
TODO

### Asynchronous functions
https://aws.amazon.com/blogs/compute/node-js-8-10-runtime-now-available-in-aws-lambda/

This is a working example
```
exports.handler = async (event, context) => {
  console.log('Lambda B Received event:', JSON.stringify(event, null, 2));
  return event.name
};
```

### Exposing the functions externally
https://stackoverflow.com/questions/43779871/call-aws-lambda-function-without-using-api-gateway-and-ec2-instance
Seems quite difficult. 

TODO

### Code organisation
All of the lambda functions that you can access should appear on https://console.aws.amazon.com/lambda/home?your-region-here#/functions. 
Each function will be assigned a certain role. Each role will need to be assigned a few policies, and then the functions can call each other.  

### Middleware structure implementation
TODO

Good luck achieving this with serverless functions.

### Future work
Understand what AWS Step Functions does.

## Azure
Along with AWS, Azure is compliant with corporate settings. Azure's UI is not the best.

### Getting the first individually working function
When I wanted to create a nodejs function, I was asked to read these tutorials in this order
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-node

A working sample of the individually running function is found here:
https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function (HTTPS trigger)
Choose Windows as the underlying OS (?) because otherwise, you cannot edit the function code in the browser.

This is the sample working code they have provided:
```
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.query.name || (req.body && req.body.name)) {
        context.res = {
            // status: 200, /* Defaults to 200 */
            body: "Hello " + (req.query.name || req.body.name)
        };
    }
    else {
        context.res = {
            status: 400,
            body: "Please pass a name on the query string or in the request body"
        };
    }
};
```

### Logging
Working snippet of code that logs.      
```
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    // You can call and await an async method here
    return {
        body: "Hello, world!"
    };
}
```

### Installation of packages
It seems that as long as I include `axios = require('axios')` the package will automatically install. Which version of the package - probably it will resolve itself since the node version is already specified.

### Allowing functions to call one another
Functions with "Anonymous" "Authorization level" can be called from anywhere, whereas calling "Function" and "Admin" level functions have keys.
> Authorization level controls whether the function requires an API key and which key to use; Function uses a function key; Admin uses your master key. The function and master keys are found in the 'keys' management panel on the portal, when your function is selected. For user-based authentication, go to Function App Settings.

For now, "Anonymous" "Authorization level" functions should be sufficient. However, I think there should be better way functions can call each other without resorting to URLs (which are slow?).

### Calling external API
TODO

### Exposing the functions externally
This can be done by exposing "Anonymous" "Authorization level" level function.

TODO: Allow the functions to be called by each other, and called externally.

Making http call from function: https://stackoverflow.com/questions/52546638/how-to-make-https-call-from-azure-function
https://stackoverflow.com/questions/50823783/in-azure-functions-using-javascript-how-to-call-send-request-to-a-endpoint

https://medium.com/@burkeholland/solving-the-azure-functions-challenge-with-javascript-65205b0c4920 - functions does not call each other :/


### Code organisation
Functions are organised by function app.
Each function app must be authored in the same language.

### Repository synchronisation
TODO

### Future work
TODO 

## GCP
(Google Cloud Platform) While GCP is noted to be developer-friendly, Google is still inexperienced in securing corporate compliance.

### Getting the first individually working function
https://cloud.google.com/functions/docs/writing/
https://cloud.google.com/functions/docs/deploying/filesystem
https://github.com/GoogleCloudPlatform/nodejs-docs-samples/tree/master/functions/helloworld

You need to install GCP CLI on your local computer first (i.e. running `gcloud` on your machine should work). Log in to your account as well with `gcloud auth login`.

Create a folder and create `index.js` with the following code:
```
const escapeHtml = require('escape-html');

/**
 * HTTP Cloud Function.
 *
 * @param {Object} req Cloud Function request context.
 *                     More info: https://expressjs.com/en/api.html#req
 * @param {Object} res Cloud Function response context.
 *                     More info: https://expressjs.com/en/api.html#res
 */
exports.helloHttp = (req, res) => {
  res.send(`Hello ${escapeHtml(req.query.name || req.body.name || 'World')}!`);
};

exports.helloHttp2 = (req, res) => {
  res.send(`Halo ${escapeHtml(req.query.name || req.body.name || 'World')}!`);
};
```

Run the following functions. These deploys the function helloHttp from `index.js`.
`gcloud functions deploy helloHttp --runtime nodejs10 --trigger-http`

And soon this should be the reply
```
Deploying function (may take a while - up to 2 minutes)...done.
availableMemoryMb: 256
entryPoint: helloHttp
httpsTrigger:
  url: https://<some-subdomain>.cloudfunctions.net/helloHttp
labels:
  deployment-tool: cli-gcloud
name: projects/...<and so on>
```
Calling the URL should run the logic properly:
https://us-central1-ubtech-216703.cloudfunctions.net/helloHttp
https://us-central1-ubtech-216703.cloudfunctions.net/helloHttp?name=lee
This is dope. So easy :)

You can include more than one function in index.js
`gcloud functions deploy helloHttp2 --runtime nodejs10 --trigger-http`
```
A single index.js file at the root of your project that exports one or more functions:
.
└── index.js
```

### Logging
https://cloud.google.com/functions/docs/monitoring/logging 
You run `gcloud functions logs read FUNCTION_NAME` to print the logs of the function.

### Installing packages
https://cloud.google.com/functions/docs/writing/specifying-dependencies-nodejs
```
For example, the following configurations of source code are valid:
An app.js file that exports one or more functions, with a package.json file that contains "main": "app.js":
.
├── app.js
└── package.json
```

### Calling external API

### Allowing functions to call one another
In the middleware format, the middleware functions (which take in request and return response in a similar format) serve many endpoint functions. The functions called by the middleware functions also serve other middleware functions. 

```
An index.js file that imports code from a foo.js file and then exports one or more functions:
.
├── index.js
└── foo.js
```
In https://cloud.google.com/functions/docs/concepts/exec:
> Every deployed function is isolated from all other functions—even those deployed from the same source file. In particular, they don’t share memory, global variables, file systems, or other state.

### Asynchronous functions
https://cloud.google.com/functions/docs/concepts/nodejs-8-runtime
```
const fetch = require('node-fetch');

/**
 * Background Cloud Function demonstrates use of async/await.
 *
 * @param {object} data The event payload.
 */
exports.helloAsync = async data => {
  const result = await fetch('https://www.example.com');
  return result;
};
```

### Exposing the functions externally
As you see, https://us-central1-ubtech-216703.cloudfunctions.net/helloHttp?name=some_name is public.

### Code organisation
Each function needs to be housed in a folder, with either of the above formats. 
The endpoint of functions under same project and region are under the same subdomain.

### Repository synchronisation
Guide is here https://cloud.google.com/functions/docs/deploying/repo
This is the command that I have ran:
```gcloud functions deploy dialogflowEngine --source https://source.developers.google.com/projects/ubtech-216703/repos/github_tonghuikang_chatbot-sheets-fulfilment/moveable-aliases/gcloudfunc/paths// --runtime nodejs10 --trigger-http```
This will be the endpoint: https://us-central1-ubtech-216703.cloudfunctions.net/dialogflowEngine
The repository needs to be on Google Cloud Source Repository. You can sync the repository with Github. However I think you need to run the command to update the function, unless you implement some CI/CD processes. (For example for k8s, once a commit is pushed, building can begin automatically and will deploy to a container when done. I have yet to try this, however. https://cloud.google.com/cloud-build/)
(Instead of asking the some client to run some long script to deploy on Kubernetes, we could have asked them to fork my repo, synchronise with Google Cloud Source Repository, and run the one line command. However, usage of Google Repository still require a billing account.)

### Middleware structure implementation
"Using middleware to handle HTTP requests" - from https://cloud.google.com/functions/docs/concepts/nodejs-8-runtime
It seems like GCF serve as the middleware function with its request-response convention. The middleware endpoint code as shown in the first section is a list of middleware functions. In GCF, I am expected to export one function, not a list of functions. 
It may be possible to define a wrapper (?) that takes codes a list of functions, and the wrapper will carry out the request-response-next structure. It still remains to be seen whether can we fulfilment to chained-request-and-break-when-response style (explaination needed).

TODO: Later.


# Conclusion
To elaborate: The extent of vendor lock-in and best practices to ensure portability. 

Developers and decision makers are concerned the extent of lock-in. It is true that it is more difficult to migrate serverless function from one cloud provider to another, compared to solutions which provide more control (e.g. Docker). 

However, the lock-in happens when you imitately use triggers. For example, Google Cloud offers to trigger a cloud function when an entry is added to the cloud bucket. The migration of the cloud function is now less possible, you need to migrate the bucket and modify the function to work equivalently on AWS for example, or write a server that process this process which is troublesome.
