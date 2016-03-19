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
