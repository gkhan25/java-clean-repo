# Building CI/CD pipeline using open source SCA, SAST and DAST tools

## intro to DevSecOps
DevOps is a combination of cultural philosophies, practices, and tools that combine software development with information technology operations. These combined practices enable companies to deliver new application features and improved services to customers at a higher velocity. DevSecOps takes this a step further, integrating security into DevOps. With DevSecOps, you can deliver secure and compliant application changes rapidly while running operations consistently with automation.

## Services and tools

In this section, we discuss the various AWS services and third-party tools used in this solution.

### For CI/CD, we use the following AWS services:

    Amazon Simple Notification Service – Amazon SNS is a fully managed messaging service for both application-to-application (A2A) and application-to-person (A2P) communication.
    Amazon Simple Storage Service – Amazon S3 is storage for the internet. You can use Amazon S3 to store and retrieve any amount of data at any time, from anywhere on the web.
    Jenkins – Jenkins is an open source continuous integration/continuous delivery and deployment (CI/CD) automation DevOps Tool.

### Continuous testing tools

 Following scanning tools that are integrated in the pipeline ,  You can use the static code review tool Amazon CodeGuru for static analysis, but at the time of this writing, it’s not yet available in GovCloud and currently supports Java and Python (available in preview).

    Amazon CodeGuru – Amazon CodeGuru is a developer tool that provides intelligent recommendations to improve code quality and identify an application’s most expensive lines of code
    OWASP Dependency-Check – A Software Composition Analysis (SCA) tool that attempts to detect publicly disclosed vulnerabilities contained within a project’s dependencies.
    SonarQube (SAST) – Catches bugs and vulnerabilities in your app, with thousands of automated Static Code Analysis rules.
    OWASP Zap (DAST) – Helps you automatically find security vulnerabilities in your web applications while you’re developing and testing your applications.


## Prerequisites

Before getting started, make sure you have the following prerequisites:

- Elastic Beanstalk environments with an application deployed. In this post, we use Java Tomcat, but you can use any other application.
- A CodeCommit/Github/Bitbucket repo with your application code.
- The provided sonar-project.properties file, Zap.py(to run DAST Scan) file uploaded to the root of the application code repository.
- A SonarQube URL and generated API token for code scanning.
- An OWASP ZAP URL and generated API key for dynamic web scanning.
- An application web URL to run the DAST testing.
- An email address to receive notifications.

## Operations services

The following are AWS operations services:

- AWS CloudFormation – Gives you an easy way to model a collection of related AWS and third-party resources, provision them quickly and consistently, and manage them throughout their lifecycles, by treating infrastructure as code.
- AWS Elastic Beanstalk – An easy-to-use service for deploying and scaling web applications and services developed with Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker on familiar servers such as Apache, Nginx, Passenger, and IIS. This post uses Elastic Beanstalk to deploy LAMP stack with WordPress and Amazon Aurora MySQL. Although we use Elastic Beanstalk for this post, you could configure the pipeline to deploy to various other environments on AWS or elsewhere as needed.

The main steps are as follows:
- A cloudformation template will provision an EC2 machine and install OWASP ZAP and SonarQube on it.
- For this particular example we need a Java Tomcat base elastic beanstalk application Runing on aws 
- A Jenkins Server provissioned with javan11 all default plugins installed
- Optional: Connect your github repo to Codeguru

## Workflow

- When a user commits the code to a github repository, a Github webhook will triggers the jenkins Pipeline and start the build process.
- For this Example we have github repo attached with Codeguru So Code Commit to Github will trigger the CodeGuru Scanning and Jenkins Pipeline with wait till it's completed during the first stage.
- During next two stage Jenkins scans the code with an SCA tool (OWASP Dependency-Check) and SAST tool SonarQube and generates Detailed Reports and pushing those reports to AWS S3. 
- For Detailed Sonarqube report you can go Sonarqube server console.
- During SCA and SAST scan we are fetching report Summary and checking if scan finish up with Error then fail Jenkins Pipeline and send an Alert to relevent persons using AWS SNS.
- If there are no vulnerabilities, we are building the Jar file for Java application and deploy the Jar file to the Elastic Beanstalk environment.
- After the deployment succeeds, In the last stage of this pipeline we run DAST scanning with the OWASP ZAP tool.
    