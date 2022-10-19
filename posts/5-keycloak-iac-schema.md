---
title: Deploy Keycloak Clients and Roles with Code
series: Integrating Authentication Within Microservices
published: false
description: Designing an infrastructure as code solution for keycloak client deployments.
tags: 'security, microservices, keycloak'
cover_image: ../assets/images/App-Lock.jpeg
---

We last discussed how to structure our Keycloak role organization within a microservice deployment context. This discussion provides the foundation for what we will discuss today: leveraging [Infrastructure as Code (IAC)](https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac) to deploy Keycloak clients and roles easily. The goal for this project is to easily add additional microservice integrations to Keycloak so that they can be set up with minimal configuration. The tool will come as a lightweight CLI that interacts with a Keycloak server deployment to perform the client and role creation. THe tool will also provide smart defaults so new users can take advantage of this service without a full understanding of each setting.

## Proposed IaC Schema

Here is a link to the schema

Here is a table with each configuration object and description:

## Client Settings

| setting                            | description                                                                                                                                                  | type   | required | default  |
|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|----------|----------|
| client-name                        | Specifies display name of the client. For example 'My Client'.                                                                                               | String | false    |          |
| client-id                          | Specifies ID referenced in URI and tokens. For example 'my-client'.                                                                                          | String | true     |          |
| client-type                        | Specifies whether to use Open ID Connect or SAML for the client. Value must be 'oidc', or 'saml'                                                             | String | true     |          |
| client-oidc-client-authentication  | Defines the type of the OIDC client. When it's 'true', the OIDC type is set to confidential access type. When it's 'false', it is set to public access type. | Bool   | false    | false    |
| client-oidc-authorization          | Enable/Disable fine-grained authorization support for a client. Specify when client-oidc-client-authentication is 'true'                                     | Bool   | false    |          |
| client-oidc-authentication-flow    | The selected OIDC Authentication Flow type.                                                                                                                  | List   | false    |          |
| client-access-root-url             | Root URL appended to relative URLs.                                                                                                                          | String | false    |          |
| client-access-home-url             | Default URL to use when the auth server needs to redirect or link back to the client.                                                                        | String | false    |          |
| client-access-redirect-uris        | Valid URI pattern a browser can redirect to after a successful login. Wildcards and relative paths may be used.                                              | List   | false    |          |
| client-access-logout-redirect-uris | Valid URI pattern a browser can redirect to after a successful logout. A value of '+' will use the list of redirect uris.                                    | List   | false    |          |
| client-access-web-origins          | Allowed CORS origins. To permit all origins of Redirect URIs, add '+'.                                                                                       | List   | false    |          |
| client-access-admin-url            | URL to the admin interface of the client. Set this if the client supports the adapter REST API.                                                              | String | false    |          |
| client-login-theme                 | Theme to use for keycloak pages.                                                                                                                             | String | false    | base     |
| client-login-consent               | If enabled, users have to consent to client access.                                                                                                          | Bool   | false    | false    |
| client-login-display-client        | Applicable only if 'consent' is defined for this client. If 'true', there will be an item on the consent screen about this client itself.                    | Bool   | false    | false    |
| client-login-consent-text          | Applicable only if 'display-client' is 'true' for this client. Contains the text which will be on the consent screen about permissions for this client.      | String | false    |          |
| client-logout-front-channel        | If 'true', logout requires a browser redirect to client. If 'false', server performs a background invocation for logout.                                     | Bool   | false    | true     |
| client-logout-front-channel-url    | URL that will cause the client to log itself out when a logout request is sent to this realm.                                                                | String | false    | base-url |
| client-logout-back-channel-url     | URL that will cause the client to log itself out when a logout request is sent to this realm. If omitted, no logout request will be sent to the client.      | String | false    |          |
| client-logout-back-channel-session | Specifying whether a sid (session ID) Claim is included in the Logout Token when the Backchannel Logout URL is used.                                         | Bool   | false    |          |
| client-logout-back-channel-revoke  | Specifying whether a "revoke_offline_access" event is included in the Logout Token when the Backchannel Logout URL is used.                                  | Bool   | false    | false    |
|                                    |                                                                                                                                                              |        |          |          |
|                                    |                                                                                                                                                              |        |          |          |
