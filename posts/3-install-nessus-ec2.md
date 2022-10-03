---
title: Installing Nessus Agents on AWS EC2
series: Integrating Authentication Within Microservices
published: true
description: 'How to improve security in the cloud with Nessus vulnerability scanning on AWS EC2'
tags: 'security, aws, cloud'
---
Public cloud services like AWS make it easy to create scalable resources that expand and contract with load. While this solves many operational IT problems, it fails to address them securely.

The threat landscape is constantly evolving, so IT teams must be ahead of the curve when it comes to patching. [Nessus](https://www.tenable.com/products/nessus) is one vulnerability scanning and remediation tool that many organizations utilize to monitor the security of their assets. Nessus is deployed via an agent that runs on each asset. It runs automated vulnerability scans and reports the results to a centralized cloud platform which can be accessed by security analysts.

Nessus is a great solution for security on premises- but it also can protect cloud resources. As part of my centralized microservice strategy, I implemented Keycloak Authentication as a cloud instance. To comply with organizational policy, all servers utilizing the company's domain name must have the Nessus agent installed. This posed a challenge as I originally planned to deploy Keycloak with [ECS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html) - a serverless container solution. ECS also works with EC2 to expose the underlying infrastructure and enable administrators to have greater control over the host.

As with many of my deployments, I leveraged AWS CloudFormation to deploy my resources. This enables me to leverage Infrastructure as Code to maintain visibility and consistency into my infrastructure configurations. See my [CloudFormation template for this deployment here.](https://github.com/mayormaier/keycloak-microservice-docs/blob/main/resources/keycloak-auth-ec2.yaml) **NOTE:** You may experience some problems with this CloudFormation template if you do not complete the below configuration steps. Please read below before deploying the template.

ECS on EC2 requires a few additional resources to operate:

- **Auto Scaling Group:** Defines a set of EC2 instances that can dynamically grow and shrink with application demand. ECS will schedule containers to run on these instances.
- **EC2 Launch Configuration:** Defines the configuration of EC2 instances that will be launched into the autoscaling group. Some configuration options include: AMI ID, Security Groups, and Instance Type. Another configuration item is EC2 User Data, which is a bootstrap script that is run when the instance is launched.

In our use case, user data is the primary mechanism for configuring the Nessus agent on our instances. Because User Data is defined in the launch configuration, we can be sure that each instance launched in our ECS Auto Scaling Group will include the Nessus Agent.

## Configuration Steps

1. **Download the Nessus Agent Installer:** Nessus packages are available for download directly from the nessus website. In fact, Nessus requires users to download it right from their website. 
2. **Create an Amazon S3 Bucket to store the Nessus installers:** Next, create an S3 bucket to store the Nessus agent installer. This bucket should block all public access. Upload the installer to this bucket and note the filename.
3. **Create a Role that enables EC2 instance to access files from the S3 bucket:** Create a role that allows `s3:GetObject` access to the agent storage s3 bucket. Note: If using the provided CloudFormation template, the role and instance profile are configured, **BUT** they reference a [User managed policy](https://gist.github.com/mayormaier/86d05d74f0c38dcf314a0aeab7c91a84) called `S3NessusAgentAccess`. This policy should be created before deploying the template. You should be sure to replace the bucket name `nessus-agents` in the policy to reflect your bucket name.
4. **Use the following UserData to install the agent:**

```bash
#!/bin/bash
yum install -y aws-cfn-bootstrap
yum install -y unzip
echo ECS_CLUSTER=${ECSCluster} >> /etc/ecs/ecs.config
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install 
/usr/local/bin/aws s3 cp ${NessusAgentUri} .
rpm -ivh NessusAgent-*.rpm
/opt/nessus_agent/sbin/nessuscli agent link --key=${NessusAgentKey} --groups=${NessusAgentGrp} --cloud
service nessusagent stop
/opt/nessus_agent/sbin/nessusd -R
service nessusagent start
/opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource ECSAutoScalingGroup --region ${AWS::Region}
```

This user data is formatted for CloudFormation with a few required parameters:

- **NessusAgentUri:** The path of the Nessus agent installer
- **NessusAgentKey:** The tenant key for Nessus
- **NessusAgentGrp:** The name of the group to link the agent to
- **ECSCluster:** [Optional] The name of the ECS Cluster that the instance should be available to.

Another note for this UserData is that it is designed to work within ECS configured via CloudFormation. This script would also work with regular EC2 instances after brief modifications.

If not using this script with AWS ECS, omit the following line:
`echo ECS_CLUSTER=${ECSCluster} >> /etc/ecs/ecs.config`

If not using this script with AWS CloudFormation, replace all CloudFormation parameters, and omit the following lines:
`yum install -y aws-cfn-bootstrap`
`/opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource ECSAutoScalingGroup --region ${AWS::Region}`

Lastly, if you are using this script with an AMI that includes the AWS CLI (e.g., Amazon Linux 2), omit the installaiton of the CLI package:

```bash
yum install -y unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
