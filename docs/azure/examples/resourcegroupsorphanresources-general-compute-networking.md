Resource Groups - Delete or report on orphan resources (NICs, Disks, Public IPs) {#azure_orphanresources}
================================================================================

::: {#azure_orphanresources-disk}
Deletes all disks that are not being managed by a VM:
:::

``` {.yaml}
policies:
  - name: orphaned-disk
    resource: azure.disk
    filters:
      - type: value
        key: managedBy
        value: null
    actions:
      - type: delete
```

::: {#azure_orphanresources-nic}
Gets all Network Interfaces that are not attached to any VMs:
:::

``` {.yaml}
policies:
  - name: orphaned-nic
    resource: azure.networkinterface
    filters:
      - type: value
        key: properties.virtualMachine
        value: null
```

::: {#azure_orphanresources-publicip}
Queues an email with Public IPs that are not attached to any Network
Interfaces. See [c7n\_mailer
readme.md](https://github.com/cloud-custodian/cloud-custodian/blob/master/tools/c7n_mailer/README.md#using-on-azure)
for more information on how to send an email.
:::

``` {.yaml}
policies:
  - name: orphaned-ip
    resource: azure.publicip
    filters:
      - type: value
        key: properties.ipConfiguration
        value: null
    actions:
      - type: notify
        template: default
        subject: Orphaned Public IP resource
        to:
          - someone@somewhere.com
        transport:
          type: asq
          queue: https://storagename.queue.core.windows.net/queuename
```
