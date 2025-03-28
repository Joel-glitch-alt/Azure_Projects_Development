Streamline and automate the process of onboarding a new employee into Azure AD and assigning necessary Azure resources.

**********************ANSWER**************************************

Perfect! If you want to do employee onboarding using the Azure Portal (Console) without scripting, here is the step-by-step process:

✅ Step 1: Create the User
Go to Azure Portal → https://portal.azure.com

Search for Azure Active Directory (or Microsoft Entra ID)

Click Users → New user

Fill in:

User Name (UPN)

Name

Password (auto-generated or custom)

Click Create

✅ Step 2: Assign Group Membership
Still inside Azure Active Directory → Users → Click the created user

Click Groups → + Add Membership

Select one or more groups (e.g., Developers, Finance, etc.)

Click Select

✅ Step 3: Assign Licenses
In the same user panel, click Licenses → + Assignments

Select the appropriate license (e.g., Microsoft 365 E3, Azure AD Premium, etc.)

Click Save

✅ Step 4: Assign Azure Resource Roles (RBAC)
Go to Subscriptions or Resource Groups or specific resource

Click Access control (IAM)

Click + Add → Add role assignment

Select the role (e.g., Reader, Contributor)

Search for the user you just created

Click Save

✅ Step 5: Enforce Password Reset & MFA
Go back to Azure AD → Users → select the user

Under Authentication methods, click Require password reset on next login

Optionally, go to Azure Active Directory → Security → Conditional Access

Create a policy to enforce MFA for all new users

✅ Optional Step 6: Send a Welcome Email (Manual)
This part isn't built into Azure, but you can send a manual welcome email with:

Username

Temporary Password

Link to log in: https://portal.office.com

Instructions to reset password

