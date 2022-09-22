---
title: Architecting a Keycloak Deployment in AWS
published: true
description: 'How to deploy Keycloak in the cloud.'
tags: 'security, aws, keycloak'
cover_image: ../assets/images/multi-cloud-IAM.jpeg
---
Now that we've defined our use case for Keycloak, let's dive in to how we will actually deploy it. Keycloak is designed to support a variety of environments including Docker, Kubernetes, and OpenShift. For our use case, we will be utilizing Amazon Web Services Elastic Container Service (ECS). ECS is a contianer orchestration playform that enables users to deploy their continers to the cloud with minimal overhead. If you've been *baffled* by Kubernetes, I assure you that ECS will be a much better experience.

## Architecture Overview

First, let's take a look at the various services at play in this deployment:

![Architecture Diagram](../assets/images/)

### VPC Overview

**Virtual Private Clouds (VPCs)** are logically isolated sections of the AWS cloud that act are assigned to one customer. Think of a VPC like a virtual data center in the cloud. Each VPC is divided into **subnets**, each with their own **route table** and set of **security groups**. Subnets are blocks of IP address space within the larger VPC block. Route tables determine how each subnet handles traffic, and security groups act as a host based firewall, but security groups can be applied to multiple hosts. For more information about the structure of VPCs, check out [this article by BMC's Joe Hertvik](https://www.bmc.com/blogs/aws-vpc-virtual-private-cloud/).

### How is Traffic Routed?

Firstly, traffic destined for the Keycloak server will be sent to the resolved DNS CNAME that points to the **Application Load Balancer.**

Next, after entering the Amazon Web Services network, the traffic traverses to the **Internet Gateway** Internet Gateways are common constructs of the VPC, and they are required to route traffic from the internet to VPC based hosts and services.

After entering the VPC, requests are routed to the first public subnet where the **Application Load Balancer** is located. Elastic Load Balancers (the umbrella term for all load balancers) distribute traffic across a group of hosts to maintain high availability and scalability. Application Load balancers enable specific route based forwarding through a set of rules. In our use case, we have a single rule that simply forwards traffic to the Keycloak instance.

Additionally, the **Application Load Balancer** performs TLS termination for our Keyloak deployment, and enables users to connect securely to the Keycloak server. A TLS Certificate is defined in **Amazon Certificate Manager**, which is referenced by the Application Load Balancer. As requests arrive at the load balancer, the certificate is returned to begin the TLS encryption process.

Next, traffic is routed into the private subnet where our application resources live. We opted to utilize **ECS on EC2** for this deployment, as it affords us the additional control required to meet compliance requirements. In this deployment strategy, containers are scheduled to run on a defined EC2 instance that is managed by the user. Even though this responsibility is on the user, container configuration and management is still defined by the ECS service. One requirement of Application Load Balancers is the definition of multiple subnets across multiple availability zones to schedule containers in. In the event that a container exits or fails a health check, ECS will deploy an instance in another availability zone, and start the container there. Thus, high availability is built into the deployment by default.

Lastly, the deployment reaches the Keycloak container, running the latest [Keycloak Docker image](https://quay.io/repository/keycloak/keycloak).

ECS deployments are broken into **Clusters**, which can have many **services** that contain a set of **tasks**. Clusters typically contain all of the services required for a given application. For example, an application with multiple backend services would be grouped into the same cluster. A **Task Definition** is a document that outlines the configuration for an ECS Task. Tasks are essentially containers that are running instances of the task definition. Task definitions can define a variety of configuration items such as the container image URI, environment variables, port mappings, and desired container counts.

Because Keycloak requires a database to function, we've created an EC2 instance to run the database instance. We've chosen MariaDB, but many other options are available [per the Keycloak documentation](https://www.keycloak.org/server/db). In the future we may upgrade this to a true RDS instance, but for now, EC2 suits our needs.
