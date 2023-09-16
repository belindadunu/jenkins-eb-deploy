# Jenkins Pipeline for Deploying to Elastic Beanstalk

## Overview

In this project, we will create a Jenkins pipeline that builds, tests, and packages our application code, and then automatically deploys it to AWS Elastic Beanstalk. 

We will use an EC2 instance as the virtual machine to run Jenkins to achieve this. We'll install Jenkins and its dependencies to execute the pipeline. After the code builds successfully, the pipeline will automatically deploy the application to Elastic Beanstalk. 

**Please note: Many of the steps below have guided instructions on how to re-create the environment setup below. Check out my previous projects in my [repositories](https://github.com/belindadunu?tab=repositories) for more.**

### Requirements

The key elements we'll need are:

- AWS Account
- Elastic Beanstalk
- Jenkins 
- GitHub Repository
- EC2 Instance

## Steps 

### Set up a Virtual Environment

First, we'll set up the virtual environment to run Jenkins:

1. Create an AWS account if you don't have one.

2. Use the AWS console to launch an EC2 instance.

3. SSH into the EC2 instance to configure it.

### Install Jenkins

Next, we'll install Jenkins on the EC2 instance:

1. Install prerequisites like Java, Python, etc.

2. Complete initial Jenkins setup and install plugins.

3. Use the below commands to install python3.10-venv, python-pip, and unzip on Ubuntu:

#### Install python3.10-venv
- sudo apt update
- sudo apt install python3.10-venv

#### Install python-pip
sudo apt install python3-pip

#### Install unzip 
sudo apt install unzip

### Create Jenkins Pipeline

We can now create our CI/CD pipeline in Jenkins:

1. Create a new Multibranch Pipeline job in Jenkins UI.

2. You'll see an option called branch sources. Choose GitHub and enter your GitHub link and credentials.

<img width="1440" alt="Screen Shot 2023-09-15 at 9 26 21 PM" src="https://github.com/belindadunu/jenkins-eb-deploy/assets/139175163/356bcc08-3193-4cb1-a9e7-3844474204bc">

4. Configure to connect to your GitHub repository. 

5. Run build, test, deploy (later on) stages.

<img width="1440" alt="Screen Shot 2023-09-15 at 9 48 32 PM" src="https://github.com/belindadunu/jenkins-eb-deploy/assets/139175163/3413ff69-1714-43cf-b055-a738a625a2ee">

### Integrate AWS CLI

To enable automated Elastic Beanstalk deployments:

1. Install AWS CLI on Jenkins server.

2. Configure AWS credentials in Jenkins.

3. You'll need access keys in order to be able to run [`aws configure`](https://scribehow.com/shared/How_to_Install_AWS_CLI__1MnhqmpcRxupkx_F-EcreQ) and install [`AWS EB CLI`](https://scribehow.com/shared/How_to_install_AWS_EB_CLI__J6eBRB9FQl2fGenfUVemlA).

4. Once you `cd` into your jenkins workspace, you should see your multibranch pipeline name. CD into your multibranch pipeline name.

<img width="665" alt="Screen Shot 2023-09-15 at 10 12 05 PM" src="https://github.com/belindadunu/jenkins-eb-deploy/assets/139175163/2fdc1af0-11e7-4893-a3c1-33e1ac0dc310">
<img width="665" alt="Screen Shot 2023-09-15 at 10 13 44 PM" src="https://github.com/belindadunu/jenkins-eb-deploy/assets/139175163/a56ba0c4-1a66-4852-9b8b-859ba54f159a">

5. Once you've CD into your multibranch pipeline name, run `eb init` and select the following:

- `us-east-1`

- `press enter`

- `Python version 3.9`

- `"n" for code commit`

- `"n" for ssh`

6. Next run `eb create`and then enter the below answers:

- `press enter`

- `press enter`

- `press enter`

- `"n" for spot fleet`

7. Once EB finishes, you can see the application URL on the third to last line (Application available at URL)!

<img width="1142" alt="Screen Shot 2023-09-15 at 10 29 35 PM" src="https://github.com/belindadunu/jenkins-eb-deploy/assets/139175163/4b1e3ff6-7d44-4645-9349-2cfc5760ded2">

Once this step is completed, head back to your GitHub repo. Update the Jenkins file to include:
`    stage('Deploy') {
      steps {
        sh '/var/lib/jenkins/.local/bin/eb deploy'
      }
    }
  }
}`
Then, run your Jenkins build again. If it succeeds, you can move on to the next step.

### Configure Webhook

A webhook allows GitHub to notify Jenkins whenever code is pushed to the repository. This triggers an automatic pipeline run to build and deploy the changes.

1. In GitHub, create a new webhook under repository settings.
2. Click `Webhooks`
3. Add `Click "Webhooks` 
4. Set the Payload URL to the Jenkins hook address.
5. Click `Recent Deliveries`
6. Click on the `Hash` to see the following 200 response code.

<img width="1414" alt="Screen Shot 2023-09-16 at 12 10 20 AM" src="https://github.com/belindadunu/jenkins-eb-deploy/assets/139175163/ddb70cee-35bd-46a6-aaaa-4c55b5bcbd2e">

### Make a Code Change

I updated the `page_not_found.html` file and pushed the change to GitHub. This triggered a Jenkins build, which deployed the change automatically to Elastic Beanstalk.
The webhook enabled CI/CD by running the pipeline on code changes.

<img width="1414" alt="Screen Shot 2023-09-16 at 12 39 19 AM" src="https://github.com/belindadunu/jenkins-eb-deploy/assets/139175163/8031b642-6e53-4634-98d9-a4a06718d365">

## Issues

### Jenkins Installation 

- Initially got "package jenkins has no installation candidate" errors
  - This was fixed by properly configuring the Jenkins apt repository

### Elastic Beanstalk Permissions

- Encountered "Role assumption" errors during Elastic Beanstalk deployment
  - Troubleshot and reconfigured the IAM role to grant correct access

### Missing EB CLI 

- The first pipeline build failed because I forgot to include EB CLI commands
  - Added `eb deploy` commands to Jenkinsfile to deploy to Elastic Beanstalk

## Optimization

Some ways to improve this deployment:

- Use a Docker container to encapsulate the Jenkins environment
- Implement infrastructure-as-code with Terraform to provision AWS resources
- Set up monitoring and alerts for the Elastic Beanstalk environment

## System Diagram

[Deployment3.drawio.pdf](https://github.com/belindadunu/jenkins-eb-deploy/files/12640763/Deployment3.drawio.pdf)

## Conclusion

This project implemented a full CI/CD pipeline using Jenkins, GitHub, and Elastic Beanstalk. Code changes trigger automatic builds, and the application is deployed to AWS on each successful build. Overall, the pipeline streamlines the delivery of this application!
