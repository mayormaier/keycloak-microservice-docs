---
title: Introduction to User Authentication for Microservices 
published: false
description: How to integrate token based authentication into micro-frontends. We will explore Keycloak, SAML, OpenIDConnect, and make sense of the best ways to design these integrations.
tags: 'security, microservices, keycloak'
cover_image: ./assets/images/iam.jpeg
canonical_url: null
---

Do you have a collection of microservices that you've been painstakingly integrating into an authentication service one by one? Maybe you're running these services in a federated user environment, or you need a way to control authentication from a single point. Regardless of your situation, it is important to consider how to design an authentication system that is robust, extensible, and meets business requirements.

Today, we'll be looking at a few standards to integrate authentication into microservices through a centralized identity and access management system. We'll be reviewing [Security Assertion Markup Language]() (SAML) [OpenIDConnect](), and [Oauth2](). We'll also begin to discuss how to integrate these services into applications at a high level. This article will begin a series that journeys through the process of integrating authentication at the system, backend, and frontend level. Our application model will follow btopro's microservice design process, with the goal of adding authentication to the [threaded-discussion web component]().

Without further ado, lets get started!

## Authentication standards

## Centralized Identity Providers and SSO

## High-Level Authentication Flow