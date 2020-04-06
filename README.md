# Introduction to Azure AD Security Concepts

These are some utilities for learning Azure Active Directory concepts.

## Defnitions

**Application Registration** - An registration entry for an application that will be used by other (or the same) tenant. Has an Application ID that will be shared with any Enterprise Applications.

**Service Principal** - This is an identity (like a user) for a program or application. There are different varieties of Service Principals:

  * **Enterprise Application** - The instantiation of an Application Registration inside of a tenant. Shares the Application Registration's Application ID. Get's its own Object ID.
	
  * **User-Assigned Managed Identity** - A service principal created separately from an application that can be used to access Key Vault, databases, etc.
	
  * **System-Assigned Managed Identity** - A service principal created by the system and applied to an application that can be used to access Key Vault, databases, etc.

> NOTE: When you [register an Azure AD application](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals#application-registration) in the Azure portal, two objects are created in your Azure AD tenant:
> * Application object
> * Service Principal object

## Getting Started

```bash

# Clone this repository
git clone https://github.com/cwiederspan/aad-security-concepts.git

# Login to the Azure CLI
az login

```

## Create an App Registration

```bash

APP_NAME='cdw-mycustomapp-20200404-appreg'

APP_ID=$(az ad app create --display-name $APP_NAME --app-roles @app-roles-manifest.json --query appId -o tsv)

# Check the results
az ad app list --display-name $APP_NAME
echo $APP_ID

```

## Create a Service Principal for the Application

```bash

SP_ID=$(az ad sp create --id $APP_ID --query objectId --output tsv)

SP_PWD=$(az ad sp credential reset --name $APP_NAME --query password --output tsv)


# SP_PWD=$(az ad sp create-for-rbac --id $APP_ID --name $SP_NAME --skip-assignment --query password --output tsv)
# SP_ID=$(az ad sp show --id http://$APP_NAME --query objectId --output tsv)

# TODO: Set this application as a Reader (or some role) on the Subscription
# az role assignment create --assignee fed1c9aa-aafc-4aa3-b8ab-3906d20a2162 --role a_role

az login --service-principal --username $APP_ID --password $SP_PWD --tenant 72f988bf-86f1-41af-91ab-2d7cd011db47

```

## Assign a User to the App Role in the Service Principal

```bash

MY_ID=$(az ad signed-in-user show --query objectId --output tsv)
TENANT_ID=$(az account show --query homeTenantId --output tsv)

az rest --method post --uri https://graph.windows.net/$TENANT_ID/users/$MY_ID/appRoleAssignments?api-version=1.6 --body '{"id":"0146f07f-b8f8-4b12-8080-b032c9935a5d", "principalId":"$MY_ID", "resourceId":"$SP_ID" }' --headers "Content-Type=application/json"

# Example of above that can be executed from the SP's Azure CLI session

az rest --method post --uri https://graph.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/users/f446ab5a-fe3c-4db2-b7b6-912a84610b29/appRoleAssignments?api-version=1.6 --body '{"id":"0146f07f-b8f8-4b12-8080-b032c9935a5d", "principalId":"f446ab5a-fe3c-4db2-b7b6-912a84610b29", "resourceId":"d2158994-a357-4833-804c-5517620edb81" }' --headers "Content-Type=application/json"

```
