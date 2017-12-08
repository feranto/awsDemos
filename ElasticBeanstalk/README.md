# Using AWS Elastic Beanstalk to deploy a NodeJS API

In this tutorial we will show you how to leverage AWS Elastic Beanstalk to deploy and run a NodeJS/Express/SwaggerUI API.

## Basic requirements ##

*   Git [git](https://git-scm.com/download/win)
*   A bash terminal (could be Git Bash if on Windows)
*  [Visual Studio Code](https://code.visualstudio.com/download)
*  [NodeJS](https://nodejs.org/en/download/)
*  [Python](https://www.python.org/downloads/)

## Donwloading and Running the NodeJS API Locally ##

1.  The NodeJS/Express/SwaggerUI api code we will be deploying can be located in this repo [https://github.com/feranto/NodeJSExpressSwaggerUIBackend](https://github.com/feranto/NodeJSExpressSwaggerUIBackend). (All the details of how to recreate this API and file structure can be located in the repo README).


![alt text][repo]

[repo]: images/1.png  "Code repo"


2.  We clone the repo locally ``` git clone https://github.com/feranto/NodeJSExpressSwaggerUIBackend.git ```
3.  We open the cloned repo with VSCode ``` code NodeJSExpressSwaggerUIBackend```

![alt text][vscode]

[vscode]: images/3.PNG  "Visual Studio Code"

3.  We run ``` npm install ``` to download the node modules
![alt text][npminstall]

[npminstall]: images/4.png  ""
4.  We run ``` node server.js ``` to start the API in a local server
5.  Now we can access the api locally by opening the [localhost url](http://localhost:8000/docs) ``` http://localhost:8000/docs ```

![alt text][5]

[5]: images/5.png  ""

![alt text][6]

[6]: images/6.png  ""
6.  Good, now that we have our API running locally, now we prepare to deploy it to AWS Elastic Beanstalk.

## Installing and configuring EB-cli ##
1.  First we have to install EB-cli with the [following steps](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) 
2.  After installing successfully EB-cli we must configure an access-key to be able to manager our AWS resources via EB Cli. You can use the [following steps](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey).


## Configuring EB-Cli ##

1.  First we hace to set our access keys in EB-cli, to do this we run the command ``` eb init```. This will ask us for a region, we choose option 1. Then it will ask for our access key:

![alt text][7]

[7]: images/7.png  ""

We should enter the access key and secret we got form AWS Management Console.

2.  We define a name for the application, by default it takes the name of the nodejs app.

![alt text][8]

[8]: images/8.png  ""

3.  It will detect the platform of our code by default:

![alt text][9]

[9]: images/9.png  ""

4.  It will also enable the CodeCommit feature, creating an ssh pair of keys to upload the git changes safely to Elastic Beanstalk.

![alt text][12]

[12]: images/12.png  ""


5.  Now we execute the command ``` eb create``` , it will ask for a name of the elastic beanstalk environment, a dns name to access it publicly, a [type of load balancer](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.elb.html), it will create a [service role](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) and finally it will upload our code and run it. 


        eb list
        eb status
        eb health
        eb events
        eb deploy http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-getting-started.html
            we modify, comit, push and then eb deploy
        eb parameters http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-create.html
        eb terminate
            http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_nodejs_express.html?carousel=true
