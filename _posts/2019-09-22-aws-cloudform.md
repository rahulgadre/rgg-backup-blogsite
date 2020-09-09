---
title: AWS CloudFormation
date: 2019-09-22 9:58:47 +07:00
# modified: 2020-07-11 16:49:47 +07:00
tags: [aws]
description: Getting started with CloudFormation
---

The official definition of AWS Cloudformation can be found in AWS documentation. 
However, in simple terms, AWS Cloudformation is an AWS service that lets to deploy AWS resources by using a code. 
It can also be recognized as an infrastructure as code service. The code to deploy AWS resources is written in either JSON or YAML format. 
A stack is then created and deployed which will then execute the code and deploy AWS resources as per the code.

The following YAML code shows the deployment of 2 EC2 instances using AWS Cloudformation. 
Several other variations can be tested by tweaking some of the properties shown in the code below.

``` yml

AWSTemplateFormatVersion: '2010-09-09'
Description: "Deploying an EC2 Instance using CloudFormation"
Resources:
  EC2Instance1:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: "us-west-2a"
      ImageId: "ami-061392db613a6357b"
      SecurityGroups: 
        - name-of-SG
      Tags: 
        -
          Key: Name
          Value: EC2_Cform_3
      InstanceType: "t2.micro"
      KeyName: "aws-keyname"
  EC2Instance2:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: "us-west-2c"
      ImageId: "ami-061392db613a6357b"
      SecurityGroups: 
        - name-of-SG
      Tags: 
        -
          Key: Name
          Value: EC2_Cform_4
      InstanceType: "t2.micro"
      KeyName: "aws-keyname"


```
Note: There’s no extra charge to use AWS CloudFormation as you only pay for the resources deployed.
