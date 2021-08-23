Logging, Metrics and Output {#azure_monitoring}
===========================

Writing Custodian Logs to Azure App Insights
--------------------------------------------

Cloud Custodian can upload its logs to Application Insights. Each
policy's log output contains the **policy name**, **subscription id**
and **execution id properties**. These logs will be found under the
**trace** source in Application Insights.

Usage example using an instrumentation key:

> ``` {.sh}
> custodian run -s <output_directory> -l azure://<instrumentation_key_guid> policy.yml
> ```

Usage example using a resource name:

> ``` {.sh}
> custodian run -s <output_directory> -l azure://<resource_group_name>/<app_insights_name> policy.yml
> ```

Writing Custodian Metrics to Azure App Insights
-----------------------------------------------

By default, Cloud Custodian will upload the following metrics in all
modes:

-   **ResourceCount** - the number of resources that matched the set of
    filters
-   **ActionTime** - the time to execute the actions.

In **pull** and **azure-periodic** mode, Cloud Custodian will also
publish the following metric:

-   **ResourceTime** - the time to query for and filter the resources,

Additionally, some custom filters and actions may generate their own
metrics. These metrics will be found under the **customMetrics** source
in Application Insights.

Usage example using an instrumentation key:

> ``` {.sh}
> custodian run -s <output_directory> -m azure://<instrumentation_key_guid> policy.yml
> ```

Usage example using a resource name:

> ``` {.sh}
> custodian run -s <output_directory> -m azure://<resource_group_name>/<app_insights_name> policy.yml
> ```

In **azure-periodic** and **azure-event-grid** modes, you can configure
metrics under the **execution-options**. As above, you can provide an
instrumentation key or a resource name.

> ``` {.yaml}
> policies:
>   - name: periodic-mode-logging-metrics
>     resource: azure.storage
>     mode:
>       type: azure-periodic
>       schedule: '0 0 * * * *'
>       provision-options:
>         servicePlan:
>           name: cloud-custodian
>           location: eastus
>           resourceGroupName: cloud-custodian
>       execution-options:
>         metrics: azure://<instrumentation_key_guid>
> ```

Writing Custodian Output to Azure Blob Storage {#azure_bloboutput}
----------------------------------------------

You can pass the URL of a blob storage container as the output path to
Custodian. You must change the URL prefix from https to azure.

By default, Custodian will add the policy name and date as the prefix to
the blob.

> ``` {.sh}
> custodian run -s azure://mystorage.blob.core.windows.net/logs mypolicy.yml
> ```

In addition, you can use [pyformat]{.title-ref} syntax to format the
output prefix. This example is the same structure as the default one.

> ``` {.sh}
> custodian run -s azure://mystorage.blob.core.windows.net/logs/{policy_name}/{now:%Y/%m/%d/%H/} mypolicy.yml
> ```

Use [{account\_id}]{.title-ref} for Subscription ID.

Authentication to Storage
-------------------------

The account working with storage will require [Storage Blob Data
Contributor]{.title-ref} on either the storage account or a higher
scope.
