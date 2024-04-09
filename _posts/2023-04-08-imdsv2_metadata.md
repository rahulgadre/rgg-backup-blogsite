---
title: Retrieving the billing code for an IMDSv2 enabled RHEL EC2 instance 
date: 2024-04-08 05:41:47 +07:00
# modified: 2020-07-11 16:49:47 +07:00
tags: [aws]
description: Steps to retrieve the billing code for an IMDSv2 enabled RHEL EC2 instance 
#image: "/apa-itu-shell/shell_evolution.png"
---

As IMDSv2 is becoming prevalent, the commands to retrieve instance metadata need to be updated as IMDSv2 uses session-oriented requests. The command that is used to check the billing product on an IMDSv1 enabled RHEL instance returns no output on an IMDSv2 enabled RHEL EC2 instance. This article covers the commands which can be used to retrieve the billing code for an IMDSv2 enabled RHEL EC2 instance.

When Red Hat connects to the RHUI repositories that AWS uses, Red Hat uses the billing product to check for a valid subscription. For IMDSv1, the following command can be used to check the billing product on a RHEL instance.

#### Getting the billing code on an IMDSv1 enabled RHEL EC2 instance:

`$ curl http://169.254.169.254/latest/dynamic/instance-identity/document      2>/dev/null | grep billingProducts
`

As the aforementioned command returns no output on an IMDSv2 enabled RHEL EC2 instance, the following commands can be used to retrieve the billing code on an IMDSv2 enabled RHEL EC2 instance.

#### Getting the billing code on an IMDSv2 enabled RHEL EC2 instance:

As IMDSv2 uses session-oriented requests, a session token that defines the session duration needs to be generated using the following command:

```
$ TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
```
***Note: A token with a validity of 21600 seconds or 6 hours is created using the aforementioned command.***

The same token can be used for subsequent requests. However, a new token is required for future requests once the specified duration expires.

Once the token is created, the following command can be used to retrieve the billing product code of an IMDSv2 enabled RHEL instance:

`$ curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/dynamic/instance-identity/document      2>/dev/null | grep billingProducts`
