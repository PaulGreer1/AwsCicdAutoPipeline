![Request_response_sequence_diagram](https://github.com/PaulGreer1/WebsiteLamp/blob/main/UKAPPCODER_002.png)

## AwsCicdAutoPipeline
Automated AWS pipeline which compiles, packages and deploys software artefacts.
### Project: AwsCicdAutoPipeline 2022 - 2023

#### Overview

Our software development and delivery system includes an automated pipeline which compiles, packages and deploys software artefacts, and is central to the continual integration and continuous deployment (CICD) of our Java server.

An AWS CodePipeline service provides the stages, transitions and executions for a 'source-compile-package-deploy' sequence. From a Git push in the local development environment, to the final remote deployment by the AWS CloudFormation service, I use AWS microservices to manage and automate the procedure. I have ensured that changes in the CodeCommit repository are detected from source push, not from CodeBuild polling.

![Request_response_sequence_diagram](https://github.com/PaulGreer1/AwsCicdAutoPipeline/blob/main/AwsCicdAutoPipelineInSdds.png)

#### 5. Monitoring the pipeline

CloudWatch monitors state changes in executions, actions, etc. of CodePipeline components. We can receive email notifications by setting up event rules, which is useful when more than one developer is working on the system, and access logs, error messages, etc..

#### 6. Building the deployment package with CodeBuild

In the install phase of the CodeBuild's buildspec.yml file, the Java Corretto 17 runtime is specified. In the build phase, buildspec.yml provides the commands to be executed in order to compile the source, produce a deployment package and upload it to an S3 bucket. The 'mvn package' and 'sam package' commands are used to achieve this.

The input artefacts to CodeBuild are the source outputs from the pipeline's CodeCommit stage. This includes the Java source code which was pushed from the local development and testing environment. This initial push to the CodeCommit repository from the local Git repository via SSH triggers the CodePipeline.

The output artefact from CodeBuild is a .zip archive which CodeBuild uploads to an S3 bucket. CodeBuild gets the location for this upload from the buildspec.yml file, which also provides the target location for the deployment package. The buildspec.yml file also has other sections that we can use to tell Maven to run other commands before and after the build.

#### 7. CloudFormation as the orchestrator

CloudFormation orchestrates our stack of resources to provide easier management of the remote components of our CICD system. Also, we may eventually need the same stack to be portable across regions, and this is a feature of the CloudFormation template. The template can also be developed and maintained with local Git and remote CodeCommit in the same way as source code.

#### 8. Deploying the Lambda package

A service role was created for the CloudFormation component, and it was granted the sts::AssumeRole permission so it could switch roles in order to perform various actions. So, CloudFormation may assume this role which has an attached policy containing permissions that allow CloudFormation to perform the actions necessary to deploy the artefact. This deployment artefact (an output of the CodeBuild stage in the pipeline) is in the form of a .zip archive. The deployment actions which CloudFormation can perform include the download of the artefact from the S3 bucket, the creation of the Lambda function and the creation of a role for the Lambda function defined in the CloudFormation template file.

At this stage we can illustrate how useful the pipeline's compilation output messages can be. For example, during the first couple of deployment executions, an error message was returned stating that a certain role could not be assumed or was invalid. Further investigation revealed that I had mistakenly defined CodeDeploy as the 'Deploy Provider' when setting up the pipeline. I should have specified CloudFormation as the 'Deploy Provider'. So, the error was fixed within a few minutes and the Lambda function was deployed. 

#### 9. Configuring microservices

An understanding of the nature of the various configuration files was necessary. Microservices are apps, or systems, i.e. units of input-process-output. They take inputs, do processing, then produce outputs. Configuration and many initialisations, declarations, definitions, etc. are done by the developer just as with any other program or application. The main difference is that this is done within files separate from the source code files.

Much configuration can be done using declarative language. These static files don't just contain declarations, definitions, configuration, etc., they also contain functionality, Boolean conditions, regular expressions, etc.. The formatting and syntax are straightforward. There are resource templates (JSON or YAML), the most common of which is the CloudFormation template which is a container for the templates of other resources. There are also policies (JSON or YAML), events (JSON), etc.. They also have formal documentation, or reference, which is similar in principle to documentation for packages, classes, methods, attributes, etc., which tell us about their properties, intrinsic functions, etc., and how to use them. Others include the buildspec.yml file, the AppSpec and the policy files for granting permissions to various entities such as users, roles, etc..

So, the config files are just another way of configuring services. They do many of the same things as the web console UI, the AWS CLI and the AWS SDK, (although these interfaces do have varying sets of functions and tools). To write the config files, we need to know what needs to be set up for any particular project. This information is available from the Amazon and AWS guides and references (docs) which enable us to write configuration files from scratch if necessary, but often we can find ready-made files which we can use as a basis for our own purposes.

***
