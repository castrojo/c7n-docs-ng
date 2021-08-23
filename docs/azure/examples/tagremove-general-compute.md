Tags - Remove tag From Virtual Machines
=======================================

Remove the tags [TagName]{.title-ref} and [TagName2]{.title-ref} from
all VMs in the subscription

``` {.yaml}
policies:
    - name: tag-remove
      description: |
        Removes tags from all virtual machines
      resource: azure.vm
      actions:
       - type: untag
         tags: ['TagName', 'TagName2']
```
