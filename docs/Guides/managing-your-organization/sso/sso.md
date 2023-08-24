---
layout: default
title: Set up SSO authentication
description: Learn about setting up SSO authentication for Firebolt. 
nav_order: 4
parent: Manage your organization
grand_parent: Guides
has_children: true
has_toc: true
---

# Set up SSO authentication

This topic describes how to configure Firebolt for Single Sign On (SSO) using SAML 2.0. Before you configure SSO on the Firebolt side, you must have already [configured your identity provider to work with Firebolt](configuring-idp-for-sso.md).

Firebolt supports the following identity providers (IDPs):
- Okta
- OneLogin
- Salesforce
- PingFederate (Ping Identity)
- Custom

{: .note}
Managing SSO settings requires the org_admin role.

## Configure SSO

SSO can be configured in two ways - using SQL or the UI.  To configure SSO using SQL, use the [`ALTER ORGANIZATION`](../../../sql_reference/commands/access-control/alter-organization.md) statement. For example:

```
ALTER ORGANIZATION SET SSO = ‘{
  “signOnUrl”: “https://abc.okta.com/app/okta_firebolt_app_id/sso/saml”,
  “signOutUrl”: “https://myapp.exampleco.com/saml/logout”, 
  “issuer”: “Okta”,
  “provider”: “Okta”,
  “label”: “Okta”,
  “fieldMapping”: “mapping”,
  “certificate”: “XXXXXXXXXXXXXXXX”,
}’;
```

For examples, see [Configure your identity provider](configuring-idp-for-sso.md)

To set up SSO via the UI:
1. Click **Configure** to open the configure space, then choose **SSO**.

2. Enter the following information:
        <br>* **Sign-on URL:** The sign-on URL, provided by the SAML identity provider, to which Firebolt sends the SAML requests. The URL is IdP-specific and is determined by the identity provider during configuration. For example, using Okta as an identity provider, the URL might look like: `https://okta_account_name.okta.com/app/okta_firebolt_app_id/sso/saml`
        <br>* **Sign-out URL (optional):** The sign-out URL, provided by the application owner, to be used when the user signs out of the application. 
        <br>* **Issuer (optional):** A unique value generated by the SAML identity provider specifying the issuer value.
        <br>* **Provider:** The provider's name - for example: “Okta”. Possible values are: Okta, Onelogin, Salesforce, PingFederate, or Custom. Use the "Custom" label if you are using a SAML 2.0-compliant service or application as your IdP. Once the provider has been selected, it can't be changed, but you can delete the SSO configuration ([see below](#delete-sso)) to then set up using a different provider. 
        <br>* **Label (optional):** The label to use for the SSO login button. If not provided, the Provider field value is used. 
        <br>* **Certificate:** The certificate to verify the communication between the identity provider and Firebolt. The certificate needs to be in PEM or CER format, and can be uploaded from your computer by choosing **Import certificate** or entered in the text box. 
        <br>* **Field mapping (optional):** Mapping to your identity provider's first and last name in key-value pairs. If additional fields are required, choose **Add another key-value pair**. Mapping is required for Firebolt to fill in the login’s given and last names the first time the user logs in using SSO. If this field remains empty when a login that represents the user is being created (read more in the [log in using SSO](#log-in-using-sso) section), the login's first and last name fields will contain “NA”. Those fields can be updated later by running the [`ALTER LOGIN`](../../../sql_reference/commands/access-control/alter-login.md) command. 

      Here’s an example of how to set up field mapping:
        
        {
            "given_name": "name",
            "family_name": "surname"
        }


      where the "given_name" (first name) is mapped to the "name" field from the IDP, and the "family_name" (last name) is mapped from the "surname" field.
3. Choose **Update changes**.

SSO is now configured for your organization!

## Edit SSO settings

SSO settings can be edited in two ways - using SQL or the UI.  To edit SSO settings using SQL, use the [`ALTER ORGANIZATION`](../../../sql_reference/commands/access-control/alter-organization.md) statement. For example:

```
ALTER ORGANIZATION SET SSO = ‘{
  “signOnUrl”: “https://abc.okta.com/app/okta_firebolt_app_id/sso/saml”,
  “signOutUrl”: “https://myapp.exampleco.com/saml/logout”, 
  “issuer”: “issuer”,
  “provider”: “Okta”, 
  “label”: “Okta”,
  “fieldMapping”: “mapping”,
  “certificate”: “XXXXXXXXXXXXXXXX”,
}’;
```

To edit SSO settings via the UI:
1. Click **Configure** to open the configure space, then choose **SSO**.

2. Edit the desired properties.

3. Choose **Update changes**.

## Delete SSO

To disable the ability to log in using SSO, SSO settings can be deleted in two ways - using SQL or the UI.  To edit SSO settings using SQL, use the following command:

```ALTER ORGANIZATION SET SSO = DEFAULT;```

To edit SSO settings via the UI:
1. Click **Configure** to open the configure space, then choose **SSO**.

2. Choose **Clear SSO configuration**.

3. Choose **Update changes**.


Once SSO configuration is deleted:
- Logins created via SSO will remain in your organization, but won't be able to log in to Firebolt unless enabled to log in using a password. Use the [`ALTER LOGIN`](../../../sql_reference/commands/access-control/alter-login.md) statement to configure this. 
- All logins with `is_sso_provisioned=true` will be updated to `sso_provisioned=false`.


## Log in using SSO

1. Go to [go.firebolt.io/login](go.firebolt.io/login).
2. Enter the name of your organization and choose **Continue to log in**.
If you can’t remember the name of the organization, next to “Don’t know your organization name?” and choose **Find out**.
3. Enter the email address you use to log in to Firebolt and choose **Send link**.
4. Check your inbox for an email with the link to log in directly to your organization. You can bookmark this link so that you can use it for future reference.
5. Choose **Log in with <IDP>**.
You will be redirected to the IDP to authenticate. Once authenticated, you will be redirected back to Firebolt.

Upon login, if a login with the specified email address does not exist, Firebolt will create a new login with the email address, first, and last name as specified in the SAML assertation received from the IDP. 
When the login is created, it will be able to authenticate via SSO only - the IS_PASSWORD_ENABLED property will be set to False.

If the login exists (and field mapping is not empty), but the first or last name is different than the one specified in the IDP, Firebolt will update the properties with the new values. Otherwise,  those fields remain the same and the login will authenticate normally.