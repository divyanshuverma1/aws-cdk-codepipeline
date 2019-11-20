# cdk-pipeline

## Overview

AWS CDK is a software development framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation. 

When working with AWS CDK, many customers want to know how to check in the code into a code versioning tool, and automate the provisioning of the infrastructure without having to synthesize the code into CloudFormation. This article explains how to do this via AWS CodePipeline and AWS CodeBuild.

## AWS CodePipeline

First create an AWS pipeline such as the below. It consists of 2 stages. The first stage takes the CDK code checked in into AWS CodeCommit. The second stage uses AWS CodeBuild to run the CDK using the CDK cli.

![img1]

[img1]:https://github.com/tohwsw/aws-cdk-codepipeline/blob/master/img/cdkpipeline.png

## AWS CodeDeploy

After creating the CDK project, check in the project to AWS CodeCommit. After which, create a buildspec.yml file such as the below. AWS CodeBuild will make use of this buildspec file. In the install phase, it installs the nodejs runtime. In the pre_build phase, it installs the CDK tool and required libraries. During the build phase, it compiles the CDK typescript into javascript. After which it deploys the CDK stack to the required account.

** buildspec.yml **
```
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 10
  pre_build:
    commands:
      - echo Starting Build...
      - node --version
      - npm install -g aws-cdk@1.12.0
      - npm install -g typescript 
      - npm install @aws-cdk/aws-iam@1.12.0
      - cdk --version
      - mkdir ~/.aws
      - aws sts get-caller-identity
  build:
    commands:
      - echo Build started on `date`
      - echo Compiling the CDK typescript...
      - npm run build
  post_build:
    commands:
      - cdk deploy CdkiamStack --require-approval never
```