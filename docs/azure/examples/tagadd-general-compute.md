Tags - Add tag to Virtual Machines
==================================

Add the tag [TagName]{.title-ref} with value [TagValue]{.title-ref} to
all VMs in the subscription

``` {.yaml}
policies:
  - name: tag-add
    description: |
      Adds a tag to all virtual machines
    resource: azure.vm
    actions:
      - type: tag
        tag: TagName
        value: TagValue
```
