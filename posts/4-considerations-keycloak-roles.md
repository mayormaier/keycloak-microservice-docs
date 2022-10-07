---
title: Considerations for Creating Keycloak Roles
series: Integrating Authentication Within Microservices
published: false
description: How I plan to organize roles in my Keycloak instance
tags: 'security, microservices, keycloak'
cover_image: ../assets/images/App-Lock.jpeg
id: 1213687
---

Now that we have Keycloak up and running, we are ready to start configuring it for our microservices. A large part of keycloak is the concept of *roles*

What are roles exactly? Keycloak defines roles as the following:

*"A type or category of user. Applications often assign access and permissions to specific roles rather than individual users as dealing with users can be too fine grained and hard to manage.*

*"A user role mapping defines a mapping between a role and a user. A user can be associated with zero or more roles. This role mapping information can be encapsulated into tokens and assertions so that applications can decide access permissions on various resources they manage."*

One important thing to get started with Keycloak is to understand that it does **NOT** link roles to *application specific* permissions. Instead, permissions must be assigned within the application (or microservice). With that being said, we should consider how to organize groups of users based on the level of permissions that they require, and also decide any global roles that have implications across all applications (e.g., Admin, default role, etc.)

There are some additional constructs that Keycloak provides to make role management easier. For example:

**Groups** manage groups of users. Attributes can be defined for a group. *You can map roles to a group* as well. Users that become members of a group inherit the attributes and role mappings that group defines.

A **composite role** is a role that can be associated with other roles. For example a `superuser` composite role could be associated with the `sales-admin` and `order-entry-admin` roles. If a user is mapped to the `superuser` role they also inherit the `sales-admin` and `order-entry-admin` roles.

**Client scopes** define protocol mappers (configuration to pass specific user attributes) and role scope mappings for that client. 

With **client roles**, clients can define roles that are specific to them. This is basically a role namespace dedicated to the client.

## References

[Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/)
