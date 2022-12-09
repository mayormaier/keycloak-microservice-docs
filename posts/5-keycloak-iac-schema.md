---
title: Deploy Keycloak Clients and Roles with Code
series: Integrating Authentication Within Microservices
published: true
description: Designing an infrastructure as code solution for keycloak client deployments.
tags: 'security, microservices, keycloak'
cover_image: ../assets/images/App-Lock.jpeg
id: 1278966
date: '2022-12-09T23:27:27Z'
---

We last discussed how to structure our Keycloak role organization within a microservice deployment context. This discussion provides the foundation for what we will discuss today: leveraging [Infrastructure as Code (IAC)](https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac) to deploy Keycloak clients and roles easily. The goal for this project is to easily add additional microservice integrations to Keycloak so that they can be set up with minimal configuration. The tool will come as a lightweight CLI that interacts with a Keycloak server deployment to perform the client and role creation. THe tool will also provide smart defaults so new users can take advantage of this service without a full understanding of each setting.

## Proposed IaC Schema

Here is a [link to the JSON schema](https://github.com/elmsln/kraxen/blob/main/keycloak-deploy.schema.json)
Here is a [link to the schema documentation](https://github.com/mayormaier/keycloak-microservice-docs/blob/main/resources/keycloak-schema-docs.md)

The proposed schema for this tool breaks down the Keycloak client configuration into more manageable and modular segments to ease template creation. For example, in this simple example, the hierarchical nature of the template enables users to specify their client features more easily.

## Usage

In order to run the kraxen CLI, you must have NPM installed.
Install the CLI tool, aka Kraxen, by cloning the kraxen repository and installing it globally:

```bash
git clone https://github.com/elmsln/kraxen && cd kraxen
npm install -g .
```

Firstly, a configuration file for kraxen (`.kraxen`) must be created in either a user's home directory, or the current working directory. The configuration file must specify three things: `KEYCLOAK_HOST`, `KEYCLOAK_ADMIN_USERNAME`, and `KEYCLOAK_ADMIN_PASSWORD`:

```
KEYCLOAK_HOST=https://keycloak-host.org
KEYCLOAK_ADMIN_USERNAME=user
KEYCLOAK_ADMIN_PASSWORD=password
```

These values can also be set via environment variables.

Next, the configuration file for the client must be created. These configuration files are YAML files that define attributes of a Keycloak client that kraxen will create. Kraxen is configured to set smart defaults for a decoupled application that implements OIDC authentication. The only required values are the `clientId`, `protocol`, `realm` settings. Most of the other settings are set as smart defaults and can be overridden via the YAML configuration file. [See an example of a template here.](https://github.com/elmsln/kraxen/blob/main/testclient_kc.yaml)

Lastly, we can run the kraxen CLI and watch our template create!

```
kraxen -t template-file.yaml -c kraxen-config
```

The `-c` flag can be omitted if the `.kraxen` configuration is present in the user's home directory.

## Final Notes

This initial executable is just a simple proof-of-concept model to demonstrate how Keycloak APIs can be leveraged to programmatically create clients and roles for use within an application. In the future, the CLI will scaffold out the template based on prompts that a user enters. It will also include sub-commands such as `kraxen init` to begin CLI setup, `kraxen create` to create a new set of Keycloak clients and roles, and `kraxen sync` to update the client configuration to match the current template. I hope to use the great example of elmsln's wcfactory tool to build a more robust CLI that can integrate even more features.
