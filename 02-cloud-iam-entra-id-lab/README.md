# Microsoft Entra ID – Cloud IAM & Identity Security Lab

## Overview

This project documents the implementation, validation, and security review of a cloud-based Identity and Access Management environment using Microsoft Entra ID.

The lab was designed to move beyond basic user administration and demonstrate practical IAM and Identity Security concepts including:

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
- Service Principal security review
- Application permission remediation
- OAuth token claim analysis
- Microsoft Entra Audit Logs
- Identity security troubleshooting

The environment simulates multiple business and administrative roles and validates both permitted and restricted actions.

The project later evolved into a Service Principal security assessment that followed a complete workflow:

```text
Discovery
    ↓
Risk Assessment
    ↓
Remediation
    ↓
Technical Validation
    ↓
Audit Trail Review
```

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
        ├── MFA
        │
        ├── Sign-in Logs
        │
        ├── Audit Logs
        │
        ├── Enterprise Applications
        │
        ├── App Registrations
        │
        ├── Service Principals
        │
        ├── OAuth 2.0
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

Each administrative role was validated through both:

- permitted actions
- restricted actions

---

## Helpdesk Administrator Validation

The identity:

`Lab Helpdesk`

was assigned:

`Helpdesk Administrator`

The account was tested in a separate authenticated session.

### Allowed Action – Password Reset

The Helpdesk Administrator successfully reset the password of:

`John Smith`

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

The:

`Add assignments`

option was unavailable.

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

The identity:

`SOC Analyst`

was assigned:

`Security Reader`

This account was tested to verify read-only access to security information.

### Allowed Action – Sign-In Logs

The SOC Analyst successfully accessed:

`Users → Sign-in logs`

Recent Microsoft Entra authentication events were visible.

The account could review information including:

- User
- Application
- Authentication activity
- Sign-in timestamps

### Restricted Action – Password Reset

The SOC Analyst attempted to reset the password of:

`John Smith`

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

This demonstrated separation between:

`Security visibility`

and:

`Identity administration`

---

## User Administrator Validation

The identity:

`Lab IAM Admin`

was assigned:

`User Administrator`

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

An important part of this lab was understanding the distinction between:

- App Registration
- Enterprise Application

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

Microsoft Entra also created an:

`Enterprise Application`

for:

`CyberLab-IAM-App`

This object represents the application's:

`Service Principal`

inside the tenant.

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

Another user:

`Mike IT`

was intentionally left unassigned.

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

The application acts:

`On behalf of the signed-in user`

The initial permission:

`User.Read`

was configured as:

`Delegated`

---

## Application Permissions

The lab then introduced:

`User.Read.All`

as a Microsoft Graph:

`Application permission`

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

After:

`User.Read.All`

was added as an Application permission, its initial state was:

```text
User.Read.All
Type: Application
Status: Not Granted
Admin Consent Required: Yes
```

The permission was configured but was not yet usable by the application.

A Global Administrator reviewed and granted:

`Admin Consent`

for the tenant.

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

Only the:

`Client Secret Value`

can be used as the application credential.

Sensitive information is not stored in this repository.

Excluded values include:

- Client Secret Values
- OAuth access tokens
- Temporary passwords
- MFA secrets
- Authentication credentials

---

## OAuth 2.0 Client Credentials Grant

The application was authenticated using:

`OAuth 2.0 Client Credentials`

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

The authentication process was investigated.

The Tenant ID and Client ID were validated successfully.

The problem was isolated to:

`Client Secret`

The distinction between:

`Secret ID`

and:

`Secret Value`

was reviewed.

After using the correct Client Secret Value, authentication succeeded.

This provided practical experience troubleshooting OAuth application authentication.

---

## Microsoft Graph Query

The application access token was then used to query:

`Microsoft Graph`

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

# Service Principal Security Review and Remediation

A security review was performed against the:

`CyberLab-IAM-App`

Service Principal to simulate a real-world Identity Security assessment.

The objective was to identify governance and privilege risks, validate active application usage, remediate excessive access, and confirm the resulting controls using Microsoft Entra logs and Microsoft Graph.

---

## Initial Findings

The review identified several security findings:

- The Service Principal had no assigned owner.
- The application had the `User.Read.All` Microsoft Graph Application permission.
- Administrative consent had previously been granted.
- A valid Client Secret was active.
- The Service Principal showed recent successful sign-in activity.
- The application successfully accessed Microsoft Graph using non-interactive authentication.

This represented a realistic governance and Least Privilege concern.

---

## Service Principal Activity Investigation

Microsoft Entra:

`Service principal sign-ins`

confirmed recent successful activity for:

`CyberLab-IAM-App`

Observed information included:

- Status: `Success`
- Resource: `Microsoft Graph`
- Service Principal ID
- Application ID
- Request ID
- Correlation ID
- Application sign-in timestamp

The activity confirmed that the Service Principal was actively being used rather than simply existing as an unused application identity.

The event also showed that the target resource was:

`Microsoft Graph`

This correlated with the earlier OAuth Client Credentials testing.

---

## Service Principal Sign-In Analysis

The detailed sign-in event included:

- Application: `CyberLab-IAM-App`
- Resource: `Microsoft Graph`
- Status: `Success`
- Application ID matching the lab application
- Request ID available for investigation
- Correlation ID available for cross-log analysis
- Continuous Access Evaluation information
- Non-agent authentication context

This demonstrated how Entra sign-in telemetry can be used during Service Principal investigations.

---

## Finding 1 – Missing Application Ownership

The Enterprise Application initially displayed:

`No application owners found`

An application without a defined owner creates a governance risk because no individual is explicitly responsible for:

- Permission reviews
- Credential rotation
- Application lifecycle
- Security incident response
- Access reviews
- Application retirement

### Remediation

An administrator was assigned as the Service Principal owner.

This established accountability for future application management.

```text
Before
------
Service Principal
      ↓
No Owner


After
-----
Service Principal
      ↓
Assigned Owner
      ↓
Clear Accountability
```

---

## Finding 2 – Excessive Application Permission

The Service Principal had been granted:

`User.Read.All`

as a Microsoft Graph:

`Application permission`

This allowed the application to read all users' profiles without requiring an interactive user session.

The permission had already been validated using OAuth 2.0 Client Credentials.

```text
CyberLab-IAM-App
        ↓
Client Credentials
        ↓
Application Token
        ↓
User.Read.All
        ↓
Microsoft Graph
        ↓
GET /users
        ↓
SUCCESS
```

Because the lab application did not require permanent directory-wide read access, the permission was identified as excessive.

---

## Least Privilege Remediation

The:

`User.Read.All`

Application permission was removed.

The remaining Microsoft Graph permission was:

`User.Read`

which is:

`Delegated`

This distinction is important because Delegated permissions require an authenticated user and cannot provide the same application-only access through the Client Credentials flow.

After remediation:

```text
Microsoft Graph Permissions

User.Read
Type: Delegated

User.Read.All
Removed
```

---

## New OAuth Token Validation

A new OAuth 2.0 Client Credentials token was requested after the permission change.

A new token was required because previously issued tokens could still contain claims representing permissions that existed before the remediation.

The new token was successfully issued.

Token length validation confirmed that a valid JWT access token had been received.

---

## OAuth Token Claim Analysis

The new token payload was decoded locally using PowerShell.

Relevant claims were reviewed.

Observed:

```text
aud
https://graph.microsoft.com

appid
CyberLab-IAM-App Client ID

roles
empty

scp
empty
```

The `roles` claim no longer contained:

`User.Read.All`

This confirmed that the new application token no longer carried the removed Application permission.

The:

`scp`

claim was also empty.

This is expected for:

`Client Credentials`

because Delegated permissions are associated with user-delegated scopes, while application-only authentication uses Application roles.

---

## Microsoft Graph Access Validation After Remediation

The same Microsoft Graph request was executed using the new access token:

```text
GET https://graph.microsoft.com/v1.0/users
```

Result:

```text
HTTP 403 Forbidden
```

The Service Principal could no longer enumerate directory users.

This technically validated the Least Privilege remediation.

---

## Before vs After

```text
BEFORE

CyberLab-IAM-App
        ↓
User.Read.All
        ↓
Application Permission
        ↓
Client Credentials Token
        ↓
GET /users
        ↓
SUCCESS


AFTER

CyberLab-IAM-App
        ↓
User.Read.All Removed
        ↓
New Client Credentials Token
        ↓
No Application Roles
        ↓
GET /users
        ↓
403 Forbidden
```

This demonstrated that the security control directly reduced the application's effective access.

---

## Audit Log Investigation

Microsoft Entra Audit Logs were reviewed to confirm that the remediation actions were recorded.

Relevant events included:

- `Add owner to service principal`
- `Remove app role assignment from service principal`

Additional historical application events were also visible, including:

- `Consent to application`
- `Add app role assignment`
- `Add delegated permission`
- `Update service principal`

This provided visibility into the application's administrative lifecycle.

---

## Permission Removal Audit Event

The audit event for the permission removal showed:

```text
Activity Type:
Remove app role assignment from service principal

Category:
ApplicationManagement

Status:
success
```

The event also contained:

- Date and time
- Correlation ID
- Initiating actor
- Tenant information
- Session information

This demonstrated that the Least Privilege remediation was traceable through Microsoft Entra Audit Logs.

---

## Owner Assignment Audit Event

The audit event for application ownership showed:

```text
Activity Type:
Add owner to service principal

Category:
ApplicationManagement

Status:
success
```

The event also recorded:

- Date and time
- Correlation ID
- Initiating actor
- Session information
- Tenant context

This confirmed that governance changes to Service Principals are auditable.

---

## Complete Identity Security Investigation Workflow

The final Service Principal investigation followed this sequence:

```text
Service Principal Review
        ↓
Broad Permission Identified
        ↓
Missing Owner Identified
        ↓
Service Principal Activity Confirmed
        ↓
Security Risk Assessed
        ↓
Owner Assigned
        ↓
User.Read.All Removed
        ↓
New OAuth Token Requested
        ↓
Token Claims Reviewed
        ↓
Application Roles Removed
        ↓
Microsoft Graph Access Retested
        ↓
403 Forbidden
        ↓
Audit Logs Reviewed
        ↓
Remediation Confirmed
```

This represents a complete Identity Security workflow:

`Finding → Risk Assessment → Remediation → Validation → Audit Trail`

---

## Security Outcome

The Service Principal security posture was improved by:

- Assigning an accountable owner
- Removing unnecessary Application permissions
- Reducing Microsoft Graph access
- Validating privilege reduction through OAuth token claims
- Confirming access denial after remediation
- Reviewing Service Principal sign-ins
- Reviewing audit logs for change traceability

The exercise demonstrated that Microsoft Entra Service Principals must be treated as identities with their own:

- privileges
- credentials
- activity
- ownership
- lifecycle
- security risk

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
| Service Principal activity review | Validated |
| Service Principal ownership review | Validated |
| Excessive permission finding | Validated |
| Permission remediation | Validated |
| OAuth token claim analysis | Validated |
| Post-remediation access test | Validated |
| Entra Audit Logs | Validated |
| Audit trail correlation | Validated |

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
- Lost PowerShell variables
- Invalid token endpoint construction
- Empty access token variables
- OAuth token regeneration
- JWT payload decoding
- Application roles vs delegated scopes
- Microsoft Graph authorization failures
- Service Principal sign-in investigation
- Entra Audit Log investigation
- Permission removal validation

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
- Audit logs
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
- Application ownership
- IAM governance

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
- Service Principal sign-ins
- Application ownership

### OAuth 2.0

- Client Credentials Grant
- Token endpoints
- OAuth access tokens
- Application authentication
- `.default` scope
- Application-to-application authentication
- JWT token analysis
- Token claims
- Application roles

### Microsoft Graph

- Microsoft Graph REST API
- Delegated permissions
- Application permissions
- `User.Read`
- `User.Read.All`
- Admin consent
- Directory user queries
- API access validation
- Post-remediation authorization testing

### Identity Security

- Least Privilege
- Privilege separation
- Service Principal risk
- API permission governance
- Administrative consent review
- Credential security
- Application permission analysis
- Service Principal monitoring
- Missing-owner detection
- Privilege remediation
- Identity investigation
- Audit trail validation
- Security control validation

### PowerShell

- PowerShell variables
- REST API requests
- OAuth token requests
- HTTP authorization headers
- Microsoft Graph queries
- API response processing
- JWT parsing
- Base64URL decoding
- Error handling
- HTTP status validation

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

Client Secrets used for testing should be:

- rotated
- expired
- or deleted

when they are no longer required.

Secrets exposed during testing should be considered compromised and replaced.

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
- [x] Service Principal security review completed
- [x] Service Principal sign-in activity investigated
- [x] Successful Microsoft Graph service principal access confirmed
- [x] Missing Service Principal owner identified
- [x] Service Principal owner assigned
- [x] Excessive `User.Read.All` permission identified
- [x] `User.Read.All` removed
- [x] New OAuth token issued after remediation
- [x] OAuth token claims inspected
- [x] Application roles confirmed removed
- [x] Delegated scope behavior reviewed
- [x] Microsoft Graph access retested
- [x] HTTP 403 validated after privilege reduction
- [x] Entra Audit Logs reviewed
- [x] Permission removal audit event validated
- [x] Owner assignment audit event validated
- [x] Complete remediation audit trail documented

---

## Final Security Review Summary

The advanced phase of this lab simulated a real-world Service Principal security review.

### Findings

1. Missing Service Principal owner
2. Broad Microsoft Graph Application permission
3. Active Client Secret
4. Successful non-interactive Microsoft Graph access
5. Active Service Principal sign-in history

### Remediation

1. Assigned accountable ownership
2. Removed unnecessary `User.Read.All`
3. Requested a new OAuth token
4. Inspected token claims
5. Confirmed removal of Application roles
6. Retested Microsoft Graph access
7. Confirmed HTTP 403
8. Validated changes through Entra Audit Logs

### Result

```text
Initial State
     ↓
Broad Application Access
     ↓
Security Review
     ↓
Least Privilege Remediation
     ↓
Access Reduction
     ↓
Technical Validation
     ↓
Audit Confirmation
```

The final environment demonstrated not only IAM configuration, but also security assessment and remediation of application identities.

---

## Conclusion

This lab began with basic cloud identity administration and progressively moved into application identity, OAuth security, Service Principal investigation, and Least Privilege remediation.

The environment validated the following progression:

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
     ↓
Service Principal Review
     ↓
Risk Identification
     ↓
Permission Remediation
     ↓
Token Claim Validation
     ↓
403 Forbidden
     ↓
Audit Trail Review
```

The project demonstrates that identity security extends beyond human users and passwords.

Applications and Service Principals are identities as well, and their:

- permissions
- credentials
- ownership
- activity
- consent
- lifecycle

must be managed using the same Least Privilege and governance principles applied to human identities.

The lab concluded with a complete Identity Security workflow:

```text
Finding
   ↓
Risk Assessment
   ↓
Remediation
   ↓
Validation
   ↓
Audit Trail
```

This project provided hands-on experience relevant to:

- IAM Analyst
- Identity Security Analyst
- Microsoft Entra Administrator
- Cloud Security Analyst
- Identity Governance
- Application Identity Security
