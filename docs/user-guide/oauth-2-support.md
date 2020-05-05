---
layout: docwithnav
title: OAuth 2.0 Support
description: OAuth 2.0 Support

---

* TOC
{:toc}

## Overview

ThingsBoard allows you to provide Single Sign On functionality for your customers and automatically create tenant, customers or subcustomers using external access management platforms, that supports OAuth 2.0 protocol. As an example of these platforms: [Google](https://developers.google.com/identity/protocols/oauth2/openid-connect), [Okta](https://www.okta.com/), [Auth0](https://auth0.com/) etc.   


## OAuth 2.0 authentication flow

ThingsBoard supports the Authorization Code grant type to exchange an authorization code for an access token. Once user returns to the ThingsBoard client via redirect URL, the platform will get the authorization code from the URL and use it to request an access token from external user management platform.
Using [basic](/docs/user-guide/oauth-2-support/#basic-mapper) or [custom](/docs/user-guide/oauth-2-support/#custom-mapper) mapper access token will be converted from external system into ThingsBoard access token and then regular ThingsBoard authorization flow will happen.

## Scenario description

In this sample we are going to use [Google](https://developers.google.com/identity/protocols/oauth2/openid-connect) for authentication. 
Once user will grant access to his Google Account details (email is a required detail), we are going to create a separate Tenant for this user, using user's email as Tenant Name. 

As a second step we are going to add a new external provider for authentication - [Auth0](https://auth0.com/).
In this case a single Tenant is going to be created for the entire organization, but for every user new Customer is going to be created.  

If [basic mapper](/docs/user-guide/oauth-2-support/#basic-mapper) functionality will not fit your business needs, with the help of Custom mapper you are able to add an implementation that fits your specific goals.

### Login with Google

To use Google’s OAuth 2.0 authentication platform for Login, you must set up a project in the Google API Console to obtain OAuth 2.0 credentials.

Please follow the instructions on the [OpenID Connect](https://developers.google.com/identity/protocols/oauth2/openid-connect) page.
After completing the instructions above, you should have a new OAuth Client with credentials consisting of a Client ID and a Client Secret.

![image](/images/user-guide/oauth-2-support/credentials-list.png)

![image](/images/user-guide/oauth-2-support/oauth-2-credentials.png)

Please add to Authorized redirect URIs section default ThingsBoard redirect URI that we are going to use in this sample:

```
http://localhost:8080/login/oauth2/code/
```

![image](/images/user-guide/oauth-2-support/oauth-2-google-redirect-uri.png)

#### Configuration of ThingsBoard

You will need to modify OAuth 2 parameters in [thingsboard.yml](/docs/user-guide/install/config/#thingsboardyml)

```bash
# Security parameters
security:
  ...
  oauth2:
      # Enable/disable OAuth 2 login functionality
      # For details please refer to https://thingsboard.io/docs/user-guide/oauth-2-support/
      enabled: "${SECURITY_OAUTH2_ENABLED:false}"
      # Redirect URL where access code from external user management system will be processed
      loginProcessingUrl: "${SECURITY_OAUTH2_LOGIN_PROCESSING_URL:/login/oauth2/code/}"
      # List of SSO clients
      clients:
        default:
          # Label that going to be show on login button - 'Login with {loginButtonLabel}'
          loginButtonLabel: "${SECURITY_OAUTH2_DEFAULT_LOGIN_BUTTON_LABEL:Default}"
          # Icon that going to be show on login button. Material design icon ID (https://material.angularjs.org/latest/api/directive/mdIcon)
          loginButtonIcon: "${SECURITY_OAUTH2_DEFAULT_LOGIN_BUTTON_ICON:}"
          clientName: "${SECURITY_OAUTH2_DEFAULT_CLIENT_NAME:ClientName}"
          clientId: "${SECURITY_OAUTH2_DEFAULT_CLIENT_ID:}"
          clientSecret: "${SECURITY_OAUTH2_DEFAULT_CLIENT_SECRET:}"
          accessTokenUri: "${SECURITY_OAUTH2_DEFAULT_ACCESS_TOKEN_URI:}"
          authorizationUri: "${SECURITY_OAUTH2_DEFAULT_AUTHORIZATION_URI:}"
          scope: "${SECURITY_OAUTH2_DEFAULT_SCOPE:}"
          # Redirect URL that must be in sync with 'security.oauth2.loginProcessingUrl', but domain name added
          redirectUriTemplate: "${SECURITY_OAUTH2_DEFAULT_REDIRECT_URI_TEMPLATE:http://localhost:8080/login/oauth2/code/}"
          jwkSetUri: "${SECURITY_OAUTH2_DEFAULT_JWK_SET_URI:}"
          # 'authorization_code', 'implicit', 'refresh_token' or 'client_credentials'
          authorizationGrantType: "${SECURITY_OAUTH2_DEFAULT_AUTHORIZATION_GRANT_TYPE:authorization_code}"
          clientAuthenticationMethod: "${SECURITY_OAUTH2_DEFAULT_CLIENT_AUTHENTICATION_METHOD:post}" # basic or post
          userInfoUri: "${SECURITY_OAUTH2_DEFAULT_USER_INFO_URI:}"
          userNameAttributeName: "${SECURITY_OAUTH2_DEFAULT_USER_NAME_ATTRIBUTE_NAME:email}"
          mapperConfig:
            # Mapper type of converter from external user into internal - 'basic' or 'custom'
            type: "${SECURITY_OAUTH2_DEFAULT_MAPPER_TYPE:basic}"
            basic:
              # Allows to create user if it not exists
              allowUserCreation: "${SECURITY_OAUTH2_DEFAULT_MAPPER_BASIC_ALLOW_USER_CREATION:true}"
              # Key from attributes of external user object to use as email
              emailAttributeKey: "${SECURITY_OAUTH2_DEFAULT_MAPPER_BASIC_EMAIL_ATTRIBUTE_KEY:email}"
              firstNameAttributeKey: "${SECURITY_OAUTH2_DEFAULT_MAPPER_BASIC_FIRST_NAME_ATTRIBUTE_KEY:}"
              lastNameAttributeKey: "${SECURITY_OAUTH2_DEFAULT_MAPPER_BASIC_LAST_NAME_ATTRIBUTE_KEY:}"
              # Strategy for generating Tenant from external user object - 'domain', 'email' or 'custom'
              # 'domain' - name of the Tenant will be extracted as domain from the email of the user
              # 'email' - name of the Tenant will email of the user
              # 'custom' - please configure 'tenantNamePattern' for custom mapping
              tenantNameStrategy: "${SECURITY_OAUTH2_DEFAULT_MAPPER_BASIC_TENANT_NAME_STRATEGY:domain}"
              # %{attribute_key} as placeholder for attribute value of attributes of external user object
              tenantNamePattern: "${SECURITY_OAUTH2_DEFAULT_MAPPER_BASIC_TENANT_NAME_PATTERN:}"
              # If this field is not empty, user will be created as a user under defined Customer
              # %{attribute_key} as placeholder for attribute value of attributes of external user object
              customerNamePattern: "${SECURITY_OAUTH2_DEFAULT_MAPPER_BASIC_CUSTOMER_NAME_PATTERN:}"
            custom:
              url: "${SECURITY_OAUTH2_DEFAULT_MAPPER_CUSTOM_URL:}"
              username: "${SECURITY_OAUTH2_DEFAULT_MAPPER_CUSTOM_USERNAME:}"
              password: "${SECURITY_OAUTH2_DEFAULT_MAPPER_CUSTOM_PASSWORD:}"
```

Please modify this section with information (**clientId** and **clientSecret**) from your Google API console.

Use this [link](https://developers.google.com/identity/protocols/oauth2/openid-connect#discovery) to see the list of up-to-date URLs like **accessTokenUri**, **authorizationUri** etc. 

Here is the list of modified parameters:

```
# Security parameters
security:
  ...
  oauth2:
    enabled: true
    loginProcessingUrl: /login/oauth2/code/
    clients:
      default:
        loginButtonLabel: Google
        loginButtonIcon: mdi:google
        clientName: TB Google test
        clientId: 8978905XXXXXXXXXXXXXXXXXX53je4d.apps.googleusercontent.com
        clientSecret: Ss4uXXXXXXXXXXAfFN
        accessTokenUri: https://oauth2.googleapis.com/token
        authorizationUri: https://accounts.google.com/o/oauth2/auth
        scope: openid,email,profile
        redirectUriTemplate: http://localhost:8080/login/oauth2/code/
        jwkSetUri: https://www.googleapis.com/oauth2/v3/certs
        authorizationGrantType: authorization_code
        clientAuthenticationMethod: post
        userInfoUri: https://openidconnect.googleapis.com/v1/userinfo
        userNameAttributeName: email
        mapperConfig:
          type: basic
          basic:
            allowUserCreation: true
            emailAttributeKey: email
            firstNameAttributeKey: 
            lastNameAttributeKey: 
            tenantNameStrategy: email
            tenantNamePattern:
            customerNamePattern:
          custom:
            url:
            username:
            password:
```

If we navigate to Login screen, we will see additional Login option with Google:

![image](/images/user-guide/oauth-2-support/login-with-google.png)

Once we click it and select on of our Google Account, we are going to be logged into ThingsBoard with our Google's email as Tenant Administrator email:

![image](/images/user-guide/oauth-2-support/google-email.png)

If we logged as System Administrator, you will see that Tenant name is our Google email's, according to basic mapper:

![image](/images/user-guide/oauth-2-support/tenant-title-as-email.png)

### Login with Auth0

Now let's add one more SSO provider to our list - [Auth0](https://auth0.com/).
This time we are going to create Customers for our users inside a single domain tenant.

To use Auth0 authentication platform for Login, let's create new application of 'Regular Web App' type following this [link](https://auth0.com/docs/quickstarts/).

![image](/images/user-guide/oauth-2-support/auth0-regular-app.png)

From the list of technology please select 'Java Spring Security':

![image](/images/user-guide/oauth-2-support/auth0-spring-security.png)

Once your application will be created you can navigate to application details to obtain **clientId** and **clientSecret**:

![image](/images/user-guide/oauth-2-support/auth0-app-details.png)

As well please update you allowed Callback URLs:

```
http://localhost:8080/login/oauth2/code/
```

![image](/images/user-guide/oauth-2-support/auth0-allowed-redirect.png)

In the advanced details section you'll be able to find all the required URLs (endpoints) for OAuth 2.0 configuration:

![image](/images/user-guide/oauth-2-support/auth0-advanced-endpoints.png)

#### Configuration of ThingsBoard

Now it's time to update [thingsboard.yml](/docs/user-guide/install/config/#thingsboardyml) with the additional SSO provider. 
This snippet contains both providers that are used in our sample:

```
# Security parameters
security:
  ...
  oauth2:
    enabled: true
    loginProcessingUrl: /login/oauth2/code/
    clients:
      google:
        loginButtonLabel: Google
        loginButtonIcon: mdi:google
        clientName: TB Google test
        clientId: XXXXXXXXXXXXXXXXXX
        clientSecret: YYYYYYYYYYYYYYYYY
        accessTokenUri: https://oauth2.googleapis.com/token
        authorizationUri: https://accounts.google.com/o/oauth2/auth
        scope: openid,email,profile
        redirectUriTemplate: http://localhost:8080/login/oauth2/code/
        jwkSetUri: https://www.googleapis.com/oauth2/v3/certs
        authorizationGrantType: authorization_code
        clientAuthenticationMethod: post
        userInfoUri: https://openidconnect.googleapis.com/v1/userinfo
        userNameAttributeName: email
        mapperConfig:
          type: basic
          basic:
            allowUserCreation: true
            emailAttributeKey: email
            firstNameAttributeKey: 
            lastNameAttributeKey: 
            tenantNameStrategy: email
            tenantNamePattern:
            customerNamePattern:
          custom:
            url:
            username:
            password:
      auth0:
        loginButtonLabel: Auth0
        loginButtonIcon: mdi:shield-account
        clientName: My App
        clientId: XXXXXXXXXXXXXXXXXXX
        clientSecret: ZZZZZZZZZZZZZZZZZ
        accessTokenUri: https://dev-jwnt7l67.auth0.com/oauth/token
        authorizationUri: https://dev-jwnt7l67.auth0.com/authorize
        scope: openid,email,profile
        redirectUriTemplate: http://localhost:8080/login/oauth2/code/
        jwkSetUri: https://dev-jwnt7l67.auth0.com/.well-known/jwks.json
        authorizationGrantType: authorization_code
        clientAuthenticationMethod: post
        userInfoUri: https://dev-jwnt7l67.auth0.com/userinfo
        userNameAttributeName: email
        mapperConfig:
          type: basic
          basic:
            allowUserCreation: true
            emailAttributeKey: email
            firstNameAttributeKey: 
            lastNameAttributeKey: 
            tenantNameStrategy: domain
            tenantNamePattern:
            customerNamePattern: %{email}
          custom:
            url:
            username:
            password:
```

If we navigate to Login screen, we will see two possible Login with options - Google and Auth0:

![image](/images/user-guide/oauth-2-support/login-with-google-and-auth0.png)

Once we click it and select our Auth0 Account, we are going to be logged into ThingsBoard with our email's as Customer User:

![image](/images/user-guide/oauth-2-support/customer-email.png)

If we logged as System Administrator, you will see that Tenant name is our Auth0 email domain name, according to basic mapper:

![image](/images/user-guide/oauth-2-support/tenant-title-as-domain.png)

We have completed our sample and now your users not required to create accounts inside ThingsBoard - they can use already exist SSO providers for this.

## Mapping of external user into ThingBoard internal user structure

Mapping of the external user object into ThingBoard user can be achieved in two ways - using Basic and Custom mappers. 
Main functionality of the mapper is to map key-value attributes from the external user information object into expected structure of the ThingsBoard user:

```
public class OAuth2User {
    private String tenantName;
    private TenantId tenantId;
    private String customerName;
    private CustomerId customerId;
    private String email;
    private String firstName;
    private String lastName;
}
```

### Basic mapper

### Custom mapper

## OAuth 2.0 configuration parameters list description

## Next steps