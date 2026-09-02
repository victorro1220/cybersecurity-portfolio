# Microsoft Entra ID – Cloud IAM & Identity Security Lab

## Overview

This project documents the implementation and validation of a cloud-based Identity and Access Management environment using Microsoft Entra ID.

The lab was designed to move beyond basic user administration and demonstrate practical IAM concepts including:

- Cloud identity administration
- Security groups
- Role-Based Access Control (RBAC)
- Least Privilege
- Administrative role separation
- Multi-Factor Authentication
- Sign-in log visibility
- Enterprise Applications
- App Registrations
- Service Principals
- OAuth 2.0
- Microsoft Graph API permissions
- Delegated permissions
- Application permissions
- Administrative consent
- Non-interactive application authentication
- Client Credentials flow
- Identity security troubleshooting

The environment simulates multiple business and administrative roles and validates both permitted and restricted actions.

---

## Lab Environment

### Platform

- Microsoft Entra ID
- Microsoft Entra Admin Center
- Microsoft Graph
- PowerShell
- OAuth 2.0

### Identity Model

The lab uses:

- Cloud-native Microsoft Entra identities
- Microsoft-provided `onmicrosoft.com` domain
- Security groups
- Built-in administrative roles
- Enterprise Applications
- Service Principals
- Microsoft Graph permissions

---

## Lab Architecture

```text
Microsoft Entra ID
        │
        ├── Cloud Users
        │
        ├── Security Groups
        │
        ├── Administrative Roles
        │
        ├── Sign-in Logs
        │
        ├── Enterprise Applications
        │
        ├── App Registrations
        │
        ├── Service Principals
        │
        └── Microsoft Graph
```

The lab was structured to simulate different departments and administrative responsibilities while enforcing separation of duties.

---

## Cloud User Accounts

The following test identities were created:

| User | Function |
|---|---|
| John Smith | Standard Employee |
| Anna Finance | Finance |
| Mike IT | IT |
| SOC Analyst | Security Operations |
| Lab Helpdesk | Help Desk |
| Lab IAM Admin | Identity Administration |

All lab identities were created as Microsoft Entra cloud users.

---

## Security Groups

Security groups were created to organize identities according to business function.

Groups:

- `GG_Lab_Employees`
- `GG_Lab_Finance`
- `GG_Lab_IT`
- `GG_Lab_SOC`
- `GG_Lab_Helpdesk`
- `GG_Lab_IAM`

Membership design:

```text
John Smith
└── GG_Lab_Employees

Anna Finance
├── GG_Lab_Employees
└── GG_Lab_Finance

Mike IT
├── GG_Lab_Employees
└── GG_Lab_IT

SOC Analyst
├── GG_Lab_Employees
└── GG_Lab_SOC

Lab Helpdesk
├── GG_Lab_Employees
└── GG_Lab_Helpdesk

Lab IAM Admin
├── GG_Lab_Employees
└── GG_Lab_IAM
```

This group structure provides a foundation for Role-Based Access Control and future identity lifecycle exercises.

---

## Role-Based Access Control

Administrative roles were assigned according to operational responsibilities.

| Identity | Microsoft Entra Role |
|---|---|
| Lab Helpdesk | Helpdesk Administrator |
| SOC Analyst | Security Reader |
| Lab IAM Admin | User Administrator |

Standard users intentionally received no administrative roles.

This design follows the principle of Least Privilege.

Each administrative role was validated through both permitted and restricted actions.

---

## Helpdesk Administrator Validation

The identity `Lab Helpdesk` was assigned the `Helpdesk Administrator` role.

The account was tested in a separate authenticated session.

### Allowed Action – Password Reset

The Helpdesk Administrator successfully reset the password of `John Smith`.

This validated that the role had sufficient permissions to perform basic user support operations.

```text
Helpdesk Administrator
        ↓
Standard User
        ↓
Password Reset
        ↓
Allowed
```

### Restricted Action – Global Administrator Assignment

The Helpdesk Administrator accessed:

`Roles and administrators → Global Administrator`

The account could view the role but could not add new assignments.

The `Add assignments` option was unavailable.

Validated access model:

```text
Helpdesk Administrator

✓ Reset standard user passwords
✓ Perform basic support operations

✗ Assign Global Administrator
✗ Manage privileged administrative roles
```

This demonstrated that Helpdesk permissions were sufficient for support functions without granting unnecessary privileged administration access.

---

## Security Reader Validation

The identity `SOC Analyst` was assigned the `Security Reader` role.

This account was tested to verify read-only access to security information.

### Allowed Action – Sign-In Logs

The SOC Analyst successfully accessed Microsoft Entra sign-in logs.

Recent authentication events could be reviewed from the Entra Admin Center.

The account could review information including:

- User
- Application
- Authentication activity
- Sign-in timestamps

### Restricted Action – Password Reset

The SOC Analyst attempted to reset the password of `John Smith`.

Microsoft Entra rejected the operation because the account did not have sufficient administrative privileges.

Validated access model:

```text
Security Reader

✓ Review sign-in logs
✓ Investigate identity activity
✓ Read security-related information

✗ Reset user passwords
✗ Modify user identities
```

This demonstrated separation between security visibility and identity administration.

---

## User Administrator Validation

The identity `Lab IAM Admin` was assigned the `User Administrator` role.

The role was tested against standard user management operations.

### Allowed Actions

The IAM Administrator was able to:

- View users
- Edit user properties
- Manage standard users
- Initiate password resets

### Administrative Scope

The account did not receive Global Administrator privileges.

Validated access model:

```text
User Administrator

✓ Manage standard users
✓ Edit user properties
✓ Reset user passwords

✗ Global Administrator privileges
```

This demonstrated how administrative responsibilities can be delegated without granting full control of the tenant.

---

## RBAC and Least Privilege Validation

The resulting administrative model was:

```text
                    Microsoft Entra ID
                           │
             ┌─────────────┼─────────────┐
             │             │             │
        Helpdesk          SOC        IAM Admin
             │             │             │
      User support     Security      User
       operations      visibility  administration
             │             │             │
             └──── Least Privilege ──────┘
```

Each role was tested against its intended responsibilities.

This provided practical experience with:

- RBAC
- Separation of duties
- Administrative delegation
- Least Privilege
- Privilege boundary testing

---

## Multi-Factor Authentication

During authentication testing, administrative lab identities were required to register an additional authentication method.

Multi-Factor Authentication was configured during the sign-in process.

```text
Username + Password
        ↓
Additional Authentication Factor
        ↓
Microsoft Entra Authentication
```

This demonstrated an additional identity security layer beyond passwords.

---

## Application Registration

An application was registered in Microsoft Entra:

`CyberLab-IAM-App`

Configuration:

- Single tenant
- Microsoft identity platform
- No initial redirect URI

The App Registration created an application object containing:

- Application (Client) ID
- Directory (Tenant) ID
- Object ID
- API permission configuration
- Authentication configuration
- Credential configuration

---

## App Registration vs Enterprise Application

An important part of this lab was understanding the distinction between an App Registration and an Enterprise Application.

### App Registration

The App Registration represents the definition of the application.

It includes:

- Application / Client ID
- API permissions
- Redirect URIs
- Certificates
- Client secrets
- Token configuration
- Application roles

```text
App Registration
       ↓
Application Object
       ↓
Defines the application
```

### Enterprise Application

Microsoft Entra also created an Enterprise Application for `CyberLab-IAM-App`.

This object represents the application's Service Principal inside the tenant.

```text
App Registration
       │
       │ defines
       ↓
Application Object

       +

Enterprise Application
       │
       │ represents the app
       │ inside the tenant
       ↓
Service Principal
```

The App Registration and Enterprise Application shared the same Application ID but had different Object IDs.

This distinction is fundamental to Microsoft Entra application identity management.

---

## Enterprise Application Assignment

The Enterprise Application was configured with:

`Assignment required? → Yes`

A test identity was explicitly assigned:

`Anna Finance`

Another user, `Mike IT`, was intentionally left unassigned.

```text
Enterprise Application
        │
        ├── Anna Finance
        │      ↓
        │   Assigned
        │
        └── Mike IT
               ↓
           Not Assigned
```

This demonstrated explicit user-based application access control.

---

## Licensing Limitation Identified

During application assignment testing, Microsoft Entra displayed:

`Groups are not available for assignment due to your Active Directory plan level.`

The current tenant plan did not support assigning security groups directly to the Enterprise Application.

Individual user assignment remained available.

Rather than abandoning the exercise, the access model was adapted to use direct user assignment.

This demonstrated an important real-world IAM consideration:

`Security architecture can be affected by licensing capabilities.`

---

## Microsoft Graph API Permissions

The App Registration initially contained the Microsoft Graph permission:

`User.Read`

Configuration:

```text
Permission: User.Read
Type: Delegated
Admin Consent Required: No
```

This permission allows an application to access the signed-in user's profile.

---

## Delegated Permissions

Delegated permissions operate in the context of an authenticated user.

```text
User
   ↓
Signs in
   ↓
Application
   ↓
Delegated Permission
   ↓
Microsoft Graph
```

The application acts on behalf of the signed-in user.

The initial permission `User.Read` was configured as `Delegated`.

---

## Application Permissions

The lab then introduced:

`User.Read.All`

as an Application permission.

Unlike Delegated permissions, Application permissions do not require an interactive user session.

```text
Application
     ↓
Authenticates as itself
     ↓
Application Permission
     ↓
Microsoft Graph
```

The configured permission was:

```text
User.Read.All
Type: Application
Access: Read all users' full profiles
Admin Consent Required: Yes
```

---

## Delegated vs Application Permissions

The lab validated the following distinction:

| Permission | Type | Access Context |
|---|---|---|
| User.Read | Delegated | Signed-in user's profile |
| User.Read.All | Application | All directory users |

Application permissions can allow background services or automation systems to access Microsoft Graph without a signed-in user.

---

## Admin Consent

After `User.Read.All` was added as an Application permission, its initial state was:

```text
User.Read.All
Type: Application
Status: Not Granted
Admin Consent Required: Yes
```

The permission was configured but was not yet usable by the application.

A Global Administrator reviewed and granted Admin Consent for the tenant.

After approval:

```text
User.Read.All
Type: Application
Status: Granted
```

This demonstrated an important distinction:

```text
Permission Configured
        ≠
Permission Granted
```

For privileged Application permissions, administrator consent is required before the Service Principal can use the permission.

---

## Service Principal Credentials

A Client Secret was created for:

`CyberLab-IAM-App`

The credential was used only for controlled authentication testing.

The Client Secret value is intentionally excluded from this repository.

The application authentication components were:

```text
Tenant ID
Client ID
Client Secret
```

These values allowed the Service Principal to authenticate as an application identity.

---

## Credential Security

During testing, the difference between:

- Secret ID
- Secret Value

was validated.

Only the Client Secret Value can be used as the application credential.

Sensitive information is not stored in this repository.

Excluded values include:

- Client Secret Values
- OAuth access tokens
- Temporary passwords
- MFA secrets
- Authentication credentials

---

## OAuth 2.0 Client Credentials Grant

The application was authenticated using OAuth 2.0 Client Credentials.

This flow is designed for application-to-application authentication without a signed-in user.

```text
CyberLab-IAM-App
        │
        ├── Tenant ID
        ├── Client ID
        └── Client Secret
                │
                ↓
      Microsoft Entra ID
         Token Endpoint
                │
                ↓
        OAuth Access Token
```

No interactive user authentication was required.

---

## PowerShell Authentication

PowerShell was used to request an OAuth access token.

The conceptual request was:

```text
POST
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
```

Parameters:

```text
client_id
client_secret
scope=https://graph.microsoft.com/.default
grant_type=client_credentials
```

After successful authentication:

```text
TOKEN OK
```

confirmed that Microsoft Entra issued an access token to the application.

---

## Client Credentials Troubleshooting

The first authentication attempt failed because the application credential was invalid.

The Tenant ID and Client ID were validated successfully.

The problem was isolated to the Client Secret.

The distinction between:

`Secret ID`

and:

`Secret Value`

was reviewed.

After using the correct Client Secret Value, authentication succeeded.

This provided practical experience troubleshooting OAuth application authentication.

---

## Microsoft Graph Query

The application access token was then used to query Microsoft Graph.

Endpoint:

```text
GET https://graph.microsoft.com/v1.0/users
```

Selected attributes:

```text
displayName
userPrincipalName
```

PowerShell successfully returned directory users.

The results included:

- Administrative accounts
- Anna Finance
- Lab Helpdesk
- Lab IAM Admin
- John Smith
- Mike IT
- SOC Analyst

This validated that the Service Principal could access Microsoft Graph using its Application permission.

---

## Complete Application Authentication Flow

The validated identity flow was:

```text
CyberLab-IAM-App
        ↓
Client ID + Client Secret
        ↓
OAuth 2.0 Client Credentials
        ↓
Microsoft Entra ID
        ↓
Application Access Token
        ↓
Microsoft Graph
        ↓
GET /users
        ↓
Directory Users Returned
```

This operation occurred without a signed-in user.

---

## Security Significance

Application permissions can be highly privileged because they allow Service Principals to operate independently of human users.

The lab demonstrated that `User.Read.All` allowed the application to query directory users after admin consent was granted.

This highlights the importance of:

- Least Privilege
- Application permission review
- Admin consent governance
- Service Principal monitoring
- Secret management
- Credential rotation
- Application owner review
- API permission auditing

An application with excessive permissions can create significant security risk even when no human administrator is actively signed in.

---

## Identity Security Concepts Demonstrated

| IAM / Security Control | Status |
|---|---|
| Cloud identities | Implemented |
| Security groups | Implemented |
| Group membership | Implemented |
| RBAC | Implemented |
| Least Privilege | Validated |
| Separation of duties | Validated |
| Helpdesk delegation | Validated |
| Security Reader access | Validated |
| User Administrator access | Validated |
| MFA registration | Implemented |
| Sign-in log visibility | Validated |
| App Registration | Implemented |
| Enterprise Application | Implemented |
| Service Principal | Implemented |
| Application assignment | Implemented |
| Delegated Graph permissions | Implemented |
| Application Graph permissions | Implemented |
| Admin consent | Implemented |
| Client Secret | Implemented |
| OAuth 2.0 Client Credentials | Validated |
| Microsoft Graph API access | Validated |
| Application identity | Validated |

---

## Troubleshooting Performed

The lab included troubleshooting and investigation of:

- Microsoft Entra role naming
- English vs translated portal interfaces
- Administrative role assignment
- Helpdesk permission boundaries
- Security Reader restrictions
- User Administrator scope
- Sign-in identifiers vs Sign-in logs
- Enterprise Application licensing limitations
- Direct user assignment vs group assignment
- App Registration vs Enterprise Application
- Application Object vs Service Principal
- Delegated vs Application permissions
- Admin consent requirements
- OAuth Client Credentials
- Invalid Client Secret errors
- Client Secret Value vs Secret ID
- Microsoft Graph application authentication
- PowerShell REST API interaction

---

## Skills Demonstrated

### Microsoft Entra ID

- User administration
- Cloud identities
- Security groups
- Group membership
- Administrative roles
- RBAC
- Least Privilege
- Sign-in logs
- MFA
- App Registrations
- Enterprise Applications
- Service Principals

### Identity and Access Management

- Role-Based Access Control
- Separation of duties
- Administrative delegation
- User access validation
- Privilege boundary testing
- Application access control
- Identity lifecycle foundations
- Least Privilege architecture

### Application Identity

- App Registrations
- Application Objects
- Enterprise Applications
- Service Principals
- Application assignment
- Client IDs
- Tenant IDs
- Client secrets
- Non-interactive authentication

### OAuth 2.0

- Client Credentials Grant
- Token endpoints
- OAuth access tokens
- Application authentication
- `.default` scope
- Application-to-application authentication

### Microsoft Graph

- Microsoft Graph REST API
- Delegated permissions
- Application permissions
- `User.Read`
- `User.Read.All`
- Admin consent
- Directory user queries
- API access validation

### Identity Security

- Least Privilege
- Privilege separation
- Service Principal risk
- API permission governance
- Administrative consent review
- Credential security
- Application permission analysis
- Authentication troubleshooting

### PowerShell

- PowerShell variables
- REST API requests
- OAuth token requests
- HTTP authorization headers
- Microsoft Graph queries
- API response processing

---

## Security Practices

Sensitive data is intentionally excluded from the repository.

The following values must never be committed:

```text
Client Secrets
OAuth Access Tokens
Temporary Passwords
MFA Information
User Credentials
Authentication Tokens
```

Client Secrets used for testing should be rotated, expired, or deleted when they are no longer required.

---

## Current Lab Status

- [x] Microsoft Entra tenant prepared
- [x] Test users created
- [x] Security groups created
- [x] Group memberships configured
- [x] Helpdesk Administrator assigned
- [x] Security Reader assigned
- [x] User Administrator assigned
- [x] Helpdesk permissions validated
- [x] Security Reader permissions validated
- [x] User Administrator permissions validated
- [x] Least Privilege tested
- [x] MFA registration tested
- [x] Sign-in logs reviewed
- [x] App Registration created
- [x] Enterprise Application created
- [x] Service Principal identified
- [x] App Registration vs Service Principal analyzed
- [x] Enterprise Application assignment required
- [x] User assignment configured
- [x] Licensing restriction identified
- [x] Delegated permission reviewed
- [x] Application permission configured
- [x] Admin consent granted
- [x] Client Secret created
- [x] OAuth authentication tested
- [x] OAuth troubleshooting performed
- [x] Application access token obtained
- [x] Microsoft Graph queried
- [x] Directory users returned through Graph
- [x] Non-interactive application identity validated

---

## Planned Advanced Phase

The next phase of this project will move from IAM configuration into deeper Identity Security engineering and investigation.

Planned exercises include:

- Service Principal security review
- API permission reduction
- Least Privilege remediation
- Client Secret rotation
- Secret expiration analysis
- Credential lifecycle management
- App Roles
- Application authorization
- Microsoft Entra Audit Logs
- Service Principal sign-in investigation
- Admin consent investigation
- Excessive API permission detection
- Conditional Access
- Privileged account protection
- Break-glass account design
- Joiner-Mover-Leaver lifecycle
- Access reviews
- Identity governance
- Privileged identity investigation
- IAM incident response
- Identity attack-path analysis

The objective of the next phase is to simulate tasks closer to real-world:

- IAM Analyst
- Identity Security Analyst
- Microsoft Entra Administrator
- Cloud Security Analyst

---

## Conclusion

This lab began with basic cloud identity administration and progressively moved into application identity and OAuth security.

The environment currently validates the following chain:

```text
Cloud Users
     ↓
Security Groups
     ↓
RBAC
     ↓
Least Privilege
     ↓
Administrative Role Validation
     ↓
Enterprise Applications
     ↓
App Registrations
     ↓
Service Principals
     ↓
Microsoft Graph Permissions
     ↓
Admin Consent
     ↓
OAuth 2.0 Client Credentials
     ↓
Application Access Token
     ↓
Microsoft Graph
     ↓
Directory Data
```

The project demonstrates that identity security extends beyond user accounts and passwords.

Applications and Service Principals are identities as well, and their permissions, credentials, ownership, and consent must be managed using the same Least Privilege principles applied to human users.

The next phase will focus on advanced Identity Security scenarios including privilege reduction, Service Principal auditing, Conditional Access, identity governance, credential lifecycle management, and IAM incident investigation.
