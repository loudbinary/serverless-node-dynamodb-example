# Tutorial

Video : [https://youtu.be/1lYNuR2LwMw](https://youtu.be/1lYNuR2LwMw)

Prerequisites:
* AWS account [http://aws.amazon.com/](http://aws.amazon.com/)
* nodejs + npm [https://nodejs.org/en/download/](https://nodejs.org/en/download/)
* 

Optional:
* Atom editor [https://atom.io/](https://atom.io/)

##### Step 1: Create an IAM admin user

Create an [IAM user](https://console.aws.amazon.com/iam) and download or copy the credentials (Access Key ID and Secret Access Key). Click on the newly created IAM user, click on the "Permissions" tab and on the "Attach Policy" button. Select "Administrator Access".

##### Step 2: Install the serverless framework and create a new project

* Install the serverless framework.
* Create a new serverless project and go through the steps of the command line wizard

Open your command line (shell, terminal) and type the following commands:

```
npm i serverless -g
sls help
sls project create
```

##### Step 3: Create a component and function

After you have created the project, `cd` into the project directory (e.g., `cd serverless-v1gaqr`) and type the following commands:

```
sls component create blog
sls function create blog/post
```

This will create a nodejs component with name `blog` and a function with name `post`.

##### Step 4: Install component dependencies

In this project, we want to access dynamodb and also need a way to generate uuids. Therefore type in your commandline:

```
cd blog
npm i --save dynamodb node-uuid
```

##### Step 5: Implement the lambda function

Open `blog/post/handler.js` and enter the following code:

```javascript
'use strict';

var doc = require('dynamodb-doc');
var dynamo = new doc.DynamoDB();

// Require Serverless ENV vars
var ServerlessHelpers = require('serverless-helpers-js').loadEnv();

// Require Logic
var lib = require('../lib');

// Lambda Handler
module.exports.handler = function(event, context) {
  console.log('Received event:',JSON.stringify(event,null,2));
  console.log('Context:',JSON.stringify(context,null,2));

  var operation = event.operation;
  if(event.tableName) {
    event.payload.TableName = event.tableName;
  }

  switch(operation) {
    case 'create':
      var uuid = require('node-uuid');
      event.payload.Item.postId = uuid.v1();
      dynamo.putItem(event.payload,context.succeed({"postId":event.payload.Item.postId}));
      break;
    case 'read':
      dynamo.getItem(event.payload,context.done);
      break;
    case 'update':
      dynamo.updateItem(event.payload, context.done);
      break;
    case 'delete':
      dynamo.deleteItem(event.payload, context.done);
      break;
    case 'list':
        dynamo.scan(event.payload, context.done);
        break;
    default:
        context.fail(new Error('Unrecognized operation "' + operation + '"'));
  }

};
```

##### Step 6: Define AWS API Gateway endpoints

We define API endpoints by
* Creating a template file `s-templates.json` that contains request and response mapping templates
* Completing the `s-function.json` file which contains some lambda function configurations and API endpoint configurations

Create `blog/post/s-templates.json` and enter the following code:

```json
{
  "apiRequestPostCreateTemplate": {
    "application/json" : {
      "operation": "create",
      "tableName": "blog-posts",
      "payload": {
        "Item": {
          "content": "$input.json('$')"
        }
      }
    }
  },
  "apiRequestPostReadTemplate": {
    "application/json" : {
      "operation": "read",
      "tableName": "blog-posts",
      "payload": {
        "Key": {
          "postId": "$input.params('postId')"
        }
      }
    }
  },
  "apiRequestPostUpdateTemplate": {
    "application/json" : {
      "operation": "update",
      "tableName": "blog-posts",
      "payload": {
        "Key": {
          "postId": "$input.params('postId')"
        },
        "UpdateExpression": "set content = :c",
        "ExpressionAttributeValues": {
          ":c": "$input.json('$')"
        }
      }
    }
  },
  "apiRequestPostDeleteTemplate": {
    "application/json" : {
      "operation": "delete",
      "tableName": "blog-posts",
      "payload": {
        "Key": {
          "postId": "$input.params('postId')"
        },
        "ConditionExpression": "postId = :p",
        "ExpressionAttributeValues": {
          ":p": "$input.params('postId')"
        }
      }
    }
  },
  "apiRequestPostListTemplate": {
    "application/json": {
      "operation": "list",
      "tableName": "blog-posts",
      "payload": {}
    }
  },
  "apiResponsePostCreateTemplate": "{\"postId\":\"$input.path('$').postId\"}",
  "apiResponsePostReadTemplate": "{\"post\": {\"postId\":\"$input.path('$').Item.postId\", \"content\":{\"user\":\"$input.path('$').Item.content.user\",\"message\":\"$input.path('$').Item.content.message\"}}}",
  "apiResponsePostListTemplate": "{\"posts\" : [#foreach($post in $input.path('$').Items){\"postId\" : \"$post.postId\",\"content\" : { \"user\":\"$post.content.user\",\"message\":\"$post.content.message\" }}#if($foreach.hasNext),#end #end ] }"
}
```

Open `blog/post/s-function.json` and enter the following code:

```json
{
  "name": "post",
  "customName": false,
  "customRole": false,
  "handler": "post/handler.handler",
  "timeout": 6,
  "memorySize": 1024,
  "custom": {
    "excludePatterns": [],
    "envVars": []
  },
  "endpoints": [
    {
      "path": "post",
      "method": "GET",
      "type": "AWS",
      "authorizationType": "none",
      "apiKeyRequired": false,
      "requestParameters": {
        "integration.request.querystring.postId": "method.request.querystring.postId"
      },
      "requestTemplates": "$${apiRequestPostReadTemplate}",
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {},
          "responseTemplates": {
            "application/json": "$${apiResponsePostReadTemplate}"
          }
        }
      }
    },
    {
      "path": "post",
      "method": "POST",
      "type": "AWS",
      "authorizationType": "none",
      "apiKeyRequired": false,
      "requestParameters": {},
      "requestTemplates": "$${apiRequestPostCreateTemplate}",
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {},
          "responseTemplates":  {
            "application/json": "$${apiResponsePostCreateTemplate}"
          }
        }
      }
    },
    {
      "path": "post",
      "method": "PUT",
      "type": "AWS",
      "authorizationType": "none",
      "apiKeyRequired": false,
      "requestParameters": {
        "integration.request.querystring.postId": "method.request.querystring.postId"
      },
      "requestTemplates": "$${apiRequestPostUpdateTemplate}",
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {},
          "responseTemplates":  {
            "application/json": ""
          }
        }
      }
    },
    {
      "path": "post",
      "method": "DELETE",
      "type": "AWS",
      "authorizationType": "none",
      "apiKeyRequired": false,
      "requestParameters": {
        "integration.request.querystring.postId": "method.request.querystring.postId"
      },
      "requestTemplates": "$${apiRequestPostDeleteTemplate}",
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {},
          "responseTemplates":  {
            "application/json": ""
          }
        }
      }
    },
    {
      "path": "posts",
      "method": "GET",
      "type": "AWS",
      "authorizationType": "none",
      "apiKeyRequired": false,
      "requestParameters": {},
      "requestTemplates": "$${apiRequestPostListTemplate}",
      "responses": {
        "400": {
          "statusCode": "400"
        },
        "default": {
          "statusCode": "200",
          "responseParameters": {},
          "responseModels": {},
          "responseTemplates": {
            "application/json": "$${apiResponsePostListTemplate}"
          }
        }
      }
    }
  ],
  "events": []
}
```
