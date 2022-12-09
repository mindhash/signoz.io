---
id: sso-authentication
title: Single Sign-on Authentication
sidebar_label: SSO & SAML
---

Single Sign-on Settings can be configured through `Settings > Organization Settings > Authenticated Domains (section)`. You can use SSO settings to let your team log in through an Identity Providers (like Google Workspace, Okta, etc) instead of using passwords.  

## Google Workspace
Google Workspace single sign-on (SSO) provides password-free access to the invited members of your workspace to SigNoz.  

#### Who can use this feature?
- Google Workspace Owners and Org Owners
- Available in `ENTERPRISE` plans. 

#### Steps to configure Google OAuth 2.0
Google Workspace single sign-on (SSO) lets all members of your workspace sign in to SigNoz using their Google accounts. If they don’t have a account in SigNoz yet, they will have to be invited by Admin from `Settings > Organization Settings > Invite Members`.

1. Register your signoz instance with your Google org by visiting the [cloud console](https://console.cloud.google.com/apis/credentials). You must [create a developers project](https://redash.io/help/open-source/admin-guide/google-developer-account-setup) if you have not already. Then follow the Create Credentials flow.

2. Set the Authorized Redirect URL(s) to http(s)://${SIGNOZ_BASEURL}/api/v1/complete/google

3. During the setup you will obtain a client id and a client secret. Note it down as you will need them while setting up google auth in SigNoz.

4. Go to `Settings > Organization Settings > Authenticated Domains`. Click `Add a Domain`. Enter your domain name (e.g. user@your-email-domain.com). 

5. After domain is created, click on `Configure SSO`. Choose `Google Authentication` from the list. 

6. Now, enter the client id and secret you obtained in step 3. Click `Save Settings`. 

7. Click on `Enforce SSO` (next to your domain in Authenticated Domains) to enable google SSO login. When you enforce SSO, all users with user name format `<username>@your-email-domain.com`  will be forced to log in through Google. 

8. To test your setup, we recommend you to log in from a new browser window in Incognito mode. 

9. If you face issue signining in, review the query service logs. To log into SigNoz for correcting SSO settings, admins may use this special URL to use password based login: http(s)://${SIGNOZ_BASEURL}/login?password=Y


## SAML based Authentication

#### Who can use this feature?
- Available in `ENTERPRISE` plans. 

#### Steps to configure Okta based SAML auth


#### Steps to configure Microsoft Active Directory (AD) based SAML auth