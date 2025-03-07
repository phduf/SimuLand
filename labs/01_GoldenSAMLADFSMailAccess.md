# Azure AD Hybrid Identity: Stealing the AD FS Token Signing Certificate to Forge SAML Tokens and Access Mail Data

## Overview

Microsoft’s identity solutions span on-premises and cloud-based capabilities. These solutions create a common user identity for authentication and authorization to all resources, regardless of location. We call this hybrid identity and one of the authentication methods available is federation with Active Directory Services (AD FS).
In this step-by-step guide, we simulate an adversary stealing the AD FS token signing certificate, from an “on-prem” AD FS server, in order to sign SAML token, impersonate a privileged user and eventually collect mail data in a tenant via the Microsoft Graph API. This lab also focuses on showing the detection capabilities of Microsoft Defender security products and Azure Sentinel. Therefore, each simulation step is mapped to its respective alert and detection queries when possible.

## Deploy Environment
The first step is to deploy a testing environment and the following document walks you through the preparation and deployment of the infrastructure and services required to run the simulation plan. 

[Deploy Environment Steps](../2_deploy/aadHybridIdentityADFS/README.md)

## Simulate and Detect Adversary
This simulation starts with a compromised “on-prem” AD FS Server where a threat actor managed to obtain the credentials of the AD FS service account.

### Credential Access

**Export ADFS Token Signing Certificate - (Unsecured Credentials: Private Keys - T1552.004)**

Connect to the AD FS server (ADFS01) via the [Azure Bastion service](../2_deploy/helper_docs/configureAADConnectADFS) as the AD FS service account and simulate a threat actor exporting the AD FS token signing certificate by retrieving the AD FS DKM master key and using it to decrypt the certificate stored in the AD FS database configuration. This configuration will be obtained locally from the AD FS Windows Internal Database (WID).

[Local Export ADFS Token Signing Certificate Steps](../3_simulate_detect/credential-access/localExportADFSTokenSigningCertificate.md)

**Forge SAML Token - (Forge Web Credentials: SAML Tokens - T1606.002)**

Next, simulate a threat actor signing a new SAML token to impersonate a privileged user. Remember that a threat actor could sign new SAML tokens outside of the compromised organization with the stolen AD FS token signing certificate.

[Forge SAML Token Steps](../3_simulate_detect/credential-access/signSAMLToken.md)

### Persistence
Furthermore, with the new SAMl token, request an access token to the Microsoft Graph API to grant delegated permissions to an Azure AD application to access mail data on behalf of the impersonated user. Use the same Microsoft Graph access token tp add credentials (password) to application.

**Request Access Token with SAML Token - (Valid Accounts: Cloud Accounts – T1078.004)**

Follow the next steps to simulate a threat actor getting an access token for Microsoft Graph using the Azure Active Directory PowerShell application as client app. That application has permissions to perform the next two steps in this simulation exercise.

[Request Access Token with SAML Token Steps](../3_simulate_detect/persistence/getAccessTokenSAMLBearerAssertionFlow.md)

**Grant OAuth permissions to Application - (Account Manipulation: Exchange Email Delegate Permissions - T1098.002)**

Next, use the new Microsoft Graph access token to simulate a threat actor granting delegated permissions to an Azure AD application. Usually, a threat actor would prefer to use an existing application that contains the desired permissions. Grant `Mail.ReadWrite` permissions.

[Grant OAuth Permissions to Application Steps](../3_simulate_detect/persistence/grantDelegatedPermissionsToApplication.md)

**Add Credentials to OAuth Application - (Account Manipulation: Additional Cloud Credentials – T1098.001)**

Once permissions have been granted, we can add credentials to the application to then use it to request anoter Microsoft Graph access token to use the `Mail.ReadWrite` permissions.

[Add credentials to OAuth Application Steps](../3_simulate_detect/persistence/addCredentialsToApplication.md)

**Request Access Token with SAML Token - (Valid Accounts: Cloud Accounts – T1078.004)**

Time to simulate a threat actor using the credentials added to the Azure AD application and getting a Microsoft Graph access token to read mail from the impersonated user.

[Request Access Token with SAML Token Steps](../3_simulate_detect/persistence/getAccessTokenSAMLBearerAssertionFlow.md)

### Collection

**Access account mailbox via Graph API**

Finally, access the mail box of the impersonated user and read some messages.

[Access account mailbox via Graph API steps](../3_simulate_detect/collection/mailAccessDelegatedPermissions.md)

