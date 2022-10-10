---
title: Considerations for Creating Keycloak Roles
series: Integrating Authentication Within Microservices
published: true
description: How I plan to organize roles in my Keycloak instance
tags: 'security, microservices, keycloak'
cover_image: ../assets/images/App-Lock.jpeg
id: 1213687
---

Now that we have Keycloak up and running, we are ready to start configuring it for our microservices. A large part of keycloak is the concept of *roles*

What are roles exactly? Keycloak defines roles as the following:

> *"Roles are type or category of user. Applications often assign access and permissions to specific roles rather than individual users as dealing with users can be too fine grained and hard to manage.*

> *"A user role mapping defines a mapping between a role and a user. A user can be associated with zero or more roles. This role mapping information can be encapsulated into tokens and assertions so that applications can decide access permissions on various resources they manage."*

One important thing to get started with Keycloak is to understand that it does **NOT** link roles to *application specific* permissions. Instead, permissions must be assigned within the application (or microservice). With that being said, we should consider how to organize groups of users based on the level of permissions that they require, and also decide any global roles that have implications across all applications (e.g., Admin, default role, etc.)

There are some additional constructs that Keycloak provides to make role management easier. For example:

**Groups** manage groups of users. Attributes can be defined for a group. *You can map roles to a group* as well. Users that become members of a group inherit the attributes and role mappings that group defines.

A **composite role** is a role that can be associated with other roles. For example a `superuser` composite role could be associated with the `sales-admin` and `order-entry-admin` roles. If a user is mapped to the `superuser` role they also inherit the `sales-admin` and `order-entry-admin` roles.

**Client scopes** define protocol mappers (configuration to pass specific user attributes) and role scope mappings for that client. 

With **client roles**, clients can define roles that are specific to them. This is basically a role namespace dedicated to the client.

## Proposed Role Structure

Typically, Keycloak role structures mimic organizational structure, where each role is related to a specific job title. This makes sense in cases which job titles and roles do not change frequently. In our case, we must consider the educational environment, where there are 3-5 job titles: Administrator, Professor, Student, etc. Clearly, we need to have more granular control.

### Global Roles

Global Roles apply to all applications within the realm. These are more applicable to Administrative permissions and developers.

**Examples:**
**Site Admin:** global.admin (i.e., access to global application configuration)
**Site Developer:** global.dev (i.e., access to test and development resources)

Additionally, there should be global roles for authenticated users to differentiate them from unauthenticated users, even if the user does not match the required roles to perform a specific action.

### Application Specific Roles

Application Specific Roles broadly limit the types of actions that users may take in a specific application. For example, these roles could be used to grant user access to create new content within the application or adjust global settings. General Application Specific Roles are not granular and are not meant to control access to content.

Additionally, Keycloak client roles define smaller sets of roles to pass to each client. Role scopes will ensure that permission grants are efficient and roles which are not required are not passed to the client.

Examples:
Application User: <application>.user (i.e., create new application resources and manage their own)
Application Admin: <application>.admin (i.e., access to application specific configuration)

### Site Specific Roles

Application Specific roles are a subset of Application Specific Roles. They grant user access to individual sites within an application, and make it possible to provide different permissions for different microservices. For example, A user could have read access to site content, but write access to create comments on that site.

Examples:
Site Admin: <application>.<site-uid>.admin (i.e., manage site settings and content)
Service User: <application>.<site-uid>.user (i.e., view/edit/update created service content)

## How to Deal With Roles

Now that we have a role structure outlined, we must consider the following:

1. How roles should be created
2. How roles should be assigned to users
3. How permissions should delegated to users

Questions 1 and 2 will be answered in later parts of the series, as we plan to automate role creation and assignment. The Keycloak REST API makes it easy for administrators to automate the creation roles as new applications are registered. Additionally, the creation of Keycloak clients (or application registrations within Keycloak) should be automated within the deployment lifecycle.

## References

[Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/)
