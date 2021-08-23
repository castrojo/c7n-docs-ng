Authentication & Access {#azure_authentication}
=======================

Custodian supports four distinct authentication types, including Azure
CLI integration, Service Principal, MSI, and raw tokens.

Azure CLI
---------

If none of the following environment variables are set, Custodian will
attempt to pull credentials and the default subscription from Azure CLI.
This requires that you have already run `az login` and selected your
subscription in Azure CLI.

Service Principal
-----------------

A Service Principal can be provided by setting the following environment
variables.

``` {.bash}
AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID
AZURE_CLIENT_ID
AZURE_CLIENT_SECRET
```

You can create a Service Principal with Azure CLI or via the Azure
Portal. The Service Principal requires the Contributor role to be
assigned.

### Azure Portal

Follow instructions to create and assign required permissions to the new
Service Principal: [How to create Service
Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal)

### Azure CLI

``` {.bash}
# select correct subscription
az account set -s "my subscription name"

# create Service Principal
az ad sp create-for-rbac --name <name> --password <password>
```

This will yield something like:

``` {.javascript}
{
  "appId": appid,
  "displayName": name,
  "name": name,
  "password": password,
  "tenant": guid
}
```

Map the Service Principal to environment variables for Custodian:

``` {.bash}
AZURE_TENANT_ID=tenant
AZURE_SUBSCRIPTION_ID=subscriptionId
AZURE_CLIENT_ID=appId
AZURE_CLIENT_SECRET=password
```

The Service Principal already has Contributor role.

### c7n-org

If you\'re using a Service Principal across subscriptions with
[c7n-org]{.title-ref}, you\'ll need to grant it access to each of the
subscriptions.

To grant access, see:
[instructions](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#assign-the-application-to-a-role)

Access Token
------------

Passing access tokens directly is useful for integration or fake test
authentication.

For fake test authentication, environment variables should be
configured:

``` {.bash}
AZURE_ACCESS_TOKEN=fake_token
AZURE_SUBSCRIPTION_ID=ea42f556-5106-4743-99b0-c129bfa71a47
```

You will also find this configuration in tox.ini.

Managed Service Identity
------------------------

For more information about MSI, see [Azure
Documentation](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview).

If [AZURE\_USE\_MSI]{.title-ref} is set to any value, Custodian will
attempt to use MSI.

If [AZURE\_CLIENT\_ID]{.title-ref} is not set, Custodian will use the
System Identity.

If [AZURE\_CLIENT\_ID]{.title-ref} is set, Custodian will use the User
Identity which matches the client id.

You must set [AZURE\_SUBSCRIPTION\_ID]{.title-ref}.

``` {.bash}
AZURE_USE_MSI=1
AZURE_SUBSCRIPTION_ID=subscriptionId
AZURE_CLIENT_ID=clientId
```

Azure Key Vault Integration
---------------------------

If you run Custodian inside Azure VM, AKS, ACI or Azure Functions, you
can leverage Azure Key Vault to store Service Principal credentials. You
can store the json-formatted authentication file as a Key Vault secret.
It will be used to extend your local authentication options. Best
practice is to configure **Subscription ID** and **Tenant ID** locally
and store the Service Principal credentials in Azure Key Vault.

Cloud Custodian leverages Managed Service Identity or User Assigned
Identity to access Key Vault and retrieve the extended configuration.

The following example shows the secret stored in a Key Vault:

``` {.json}
{"client_id": "<your-sp-id>", "client_secret": "<your-sp-password>"}
```

To enable Key Vault integration support, populate
`AZURE_KEYVAULT_SECRET` environment variable with Secret Identifier. If
you use User Assigned Identity, provide `AZURE_KEYVAULT_CLIENT_ID` with
your UAI Service Principal ID.

``` {.bash}
AZURE_KEYVAULT_SECRET=https://<vault_name>.vault.azure.net/secrets/<secret_name>/<optional_secret_version>
AZURE_KEYVAULT_CLIENT_ID=<UAI_SP_ID>
```

You can find more information on how to create Azure Key Vault Secrets
in this quickstart:
[link](https://docs.microsoft.com/en-us/azure/key-vault/quick-create-portal).

Azure Storage access
--------------------

If your Service Principal will be writing logs to storage or leveraging
queues for mailer you should assign Storage roles, either at the
subscription level or resource group/storage account level.

Note: you cannot leverage Azure Storage functionality if you use Access
Token for authentication.

-   [Blob Data Contributor]{.title-ref}
-   [Queue Data Contributor]{.title-ref}

More details about Azure Storage access rights: [Azure
Documents](https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac)

Azure Cloud Offerings
---------------------

Cloud Custodian supports four different Cloud offerings: 1. Default
Azure Public Cloud ([AzureCloud]{.title-ref}) 2. Azure China Cloud
([AzureChinaCloud]{.title-ref}) 3. Azure German Cloud
([AzureGermanCloud]{.title-ref}) 4. Azure US Government
([AzureUSGovernment]{.title-ref})

To target these Clouds, pass the cloud name in the `--region` flag

``` {.bash}
custodian run -s . --region=AzureChinaCloud my-policy.yml
```

**Note**: All Custodian features may not be available in every Cloud.
