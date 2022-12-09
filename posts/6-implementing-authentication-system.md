---
title: Integrating Keycloak Identity Management in the Frontend and Backend
series: Integrating Authentication Within Microservices
published: true
description: Designing a full stack system that implements Keycloak authentication to enable federated access.
tags: 'security, microservices, keycloak'
cover_image: ../assets/images/playbook.png
---

Now that our authentication system is in place, client is created, and frontend assets are in motion, we will explore how authentication can be implemented into a micro-frontend component. The intended use case would be that this micro frontend could operate alone, or inside of a larger content system with other components that require authentication.

For the purposes of this example, let's consider a "discussion" microservice with a frontend web component and backend API.

## Frontend Component

For the micro-frontend use case and other decoupled applications (i.e., applications with a distinctly separate frontend and backend service that communicate via API calls), a "public" Keycloak client is required. Public clients are used with applications that expose the frontend logic of an application and make it impossible to store Keycloak secrets securely. The alternative, a "confidential" client utilizes a secret key to create a trust relationship between the application and the Keycloak server. These types of clients are applicable for monolithic or server-rendered applications. Public clients enable applications to conduct authentication flows with Keycloak to return a signed token that contains information about the user. Security controls are implemented via origin policies and well defined redirect URIs. By controlling the endpoints that Keycloak is able to redirect users to, we are able to prevent external or rogue services from hijacking our Keycloak client and receiving tokens on behalf of our users.

In order for the frontend component to execute the authentication flow, there must be support within the front end of the application to prompt the user to log in before application content can be displayed. This portion of the front end should be modular, so that it follows a singleton pattern on a page. For example, if the discussion component was placed on a page by itself, the component should prompt the user to log in. However, if the discussion component was placed in the context of a larger application that requires authentication, the individual components within that application should defer to the wider application context without handling authentication individually.

## Authentication Flow

Based on the requirement outlined above, the Open ID Connect (OIDC) [Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow) should be implemented. This flow is implemented by the [keycloak-js frontend adapter by default.](https://www.keycloak.org/docs/latest/securing_apps/index.html#_javascript_implicit_flow)

1. First, the user will visit the page containing the web component. *At this point, the micro-frontend is "disabled". A login button will be displayed to prompt the user to log in to the application.*
2. When the user clicks on the login button, a second window will appear, directing the user to the Keycloak sign-in page. With this request, the application includes a redirect URL that defines where the user will be redirected after the authentication process.
3. The user selects an identity provider and is redirected to their log-in page.
4. The user authenticates with their identity provider of choice. Upon a successful authentication, the user is redirected from the identity provider back to Keycloak.
5. The user is then redirected from Keycloak back to the application using the redirect URI specified during the original authentication request. The response includes an "authorization code" that is used to obtain the Access, Identity, and Refresh tokens from Keycloak.
6. The application requests the Access, Identity, and Refresh tokens from Keycloak.
7. Keycloak validates that the requesting application is able to request the tokens.
8. Keycloak returns the tokens to the application. *At this point, the application is "enabled" and can make API requests to resources that require authentication.*
9. The application makes requests that load the initial data from the API. The Identity token is included in this API call.
10. The API verifies that the token is valid and that the user has the appropriate permissions to execute the request. The application then issues a response to the application.

A benefit to using the keycloak-js library is the abstraction of the token exchanges from steps 5-8 in the flow above. The keycloak-js adapter also builds the requests and URLs that are requires to begin the authentication flow. This enables developers to simply focus on the visual and system requirements for their application without focusing on the cumbersome request flow and security requirements.

## Backend Implementation

A few functions must be defined on the backend API in order to enable it to participate in the flow defined above. Firstly, it should verify the validity of the token provided by the user. The Open ID Connect specification defines that JSON Web Tokens (JWTs) will be provided as the Identity, Access, and Refresh tokens in this case, thus developers can use this specification to validate user requests without interfacing directly with the keycloak server.

Firstly, the backend API should check if the identity token is valid. A valid token is one that is signed properly and is not expired. The [jsonwebtoken package](https://www.npmjs.com/package/jsonwebtoken) contains functions that can be called to verify that a token is valid. In order to execute the `jwt.verify` function, the public key for the Keycloak realm should be defined. This public key is required to verify the token signature.

In cases where keycloak roles are utilized, the role claim within the token should be evaluated against the list of applicable roles for the given API. Since the token has been declared cryptographically valid at this point, the role claim can be trusted and evaluated independent of the Keycloak server. This requires the API to be aware of the applicable keycloak roles for each resource thus centralizing the authorization decision. A benefit of this is that other identity management tools with role definition capabilities can be used in place of keycloak as a systems needs change. This reduces the amount of time required to process each API request.

In cases where processing related to the requesting user is required, the API should verify that the user record exists and create it if it does not. Keycloak returns the `sub` claim as a unique identifier for the user, [following the OIDC specification.](https://auth0.com/docs/get-started/apis/scopes/openid-connect-scopes#standard-claims) An example of this requirement for the discussion application would be if a list of users is maintained with a unique id. Tracking the user's ID would allow them to edit or delete their posts without granting the user admin permissions.
