# A guide on deploying a AWS Lambda function using Claudia.js

This repository is a starting point to using [Claudia](https://claudiajs.com/) as a deployment utility for AWS Lambda.

### What is Claudia?
Claudia makes it easy to deploy Node.js projects to AWS Lambda and API Gateway. It automates away all those error-prone deployment and configuration tasks.

Simply:
- Claudia helps easily deploy functions to AWS Lambda
- Claudia API Builder simplifies web routing

### What is AWS Lambda?

AWS Lambda is a serverless platform on Amazon Web Services. AWS Lambda allows you to build services as independent units where you only pay for the time it takes to run the operations in the Lambda function.

AWS Lambda allows you to focus on your business logic and not worry about:
- Security and networking
-  VPC instances
-  EC2 instances
-  Security groups
-  Load balancing

### What does Claudia solve?
Claudia is a convenience deployment utility where you create and upload an AWS Lambda function via the command line in addition to configuring an API Gateway for load balancing using the claudia-api-builder package.


### Step 0: Store your AWS credentials locally
Cluadia requires your AWS credentials to be stored on your local filesystem. 

- Visit [Amazon's AWS configuration guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html) to create and store your credentials on your local drive.

### Install all the packages

Install Claudia to your global path

    npm install -g claudia
    
 Install AWS cli

    pip install awscli

(Optional) Install jq for JSON pretty printing
    
    brew install jq

### Sample function
This sample application takes advantage of the [ticketmaster npm module](https://www.npmjs.com/package/ticketmaster).

- Create a [ticketmaster developer account](https://developer.ticketmaster.com/) to obtain an API key.

Create a new folder, initialize npm, and install the npm module for your microservice:

    mkdir concerts
    npm init
    touch index.js
    npm install claudia-api-builder ticketmaster

Open the concerts folder in your [favorite code editor](https://code.visualstudio.com/).

Paste the following into `index.js`

    'use strict'
    
    const ticketmaster = require('ticketmaster');
    const API = require('claudia-api-builder');
    const api = new API();
    
    api.get('/concerts/{artist}', (request) => {
        return new Promise((resolve, reject) => {
            ticketmaster('YOUR_TICKETMASTER_API_KEY_HERE').discovery.v2.event.all({
                    keyword: request.pathParams.artist
                })
                .then(results => {
                    let tours = [];
                    results.items.forEach(tour => {
                        tours.push({
                            name: tour.name,
                            timezone: tour.dates.timezone,
                            url: tour.url
                        });
                    });
                    resolve(tours);
                });
        });
    });
    
    module.exports = api;


- Replace `YOUR_TICKETMASTER_API_KEY_HERE` with your API Key found in your [ticketmaster developer account](https://developer.ticketmaster.com/).
- Save
### Deploying to AWS
Type the following in a command line:

    claudia create --region us-west-2 --api-module index

Where `--region` is the [AWS region](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html) to deploy to and `--api-module` is the name of the starting point file of your Lambda function.

### Retrieve events from your newly deployed AWS service
Let's test our newly deployed AWS Lambda function by piping our output through jq

    curl https://{{YOUR_LAMBDA_URL}}/concerts/creed | jq

Output:

     [ {
        "name": "Creed Bratton From the Office",
        "timezone": "America/Edmonton",
        "url": "https://www.ticketweb.ca/event/creed-bratton-from-the-office-union-hall-tickets/8610875?REFERRAL_ID=tmfeed"
      },
      {
        "name": "Creed Bratton (From NBC's the Office)",
        "timezone": "America/New_York",
        "url": "https://www.ticketweb.com/event/creed-bratton-from-nbcs-the-middle-east--downstairs-tickets/8619495?REFERRAL_ID=tmfeed"
      },
      {
        "name": "An Evening of Music & Comedy with Creed Bratton",
        "timezone": "America/New_York",
        "url": "https://www.ticketweb.com/event/an-evening-of-music-dingbatz-tickets/8628535?REFERRAL_ID=tmfeed"
      }
    ]

### Summary

Claudia makes it easy to deploy Node.js projects to AWS Lambda and API Gateway. You can learn more by visiting [claudiajs.com](https://claudiajs.com/) or [Claudia's GitHub repository](https://github.com/claudiajs/claudia).
