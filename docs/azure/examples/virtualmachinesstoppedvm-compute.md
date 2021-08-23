Virtual Machines - Find Stopped Virtual Machines
================================================

Filter to select all virtual machines that are not running:

``` {.yaml}
policies:
  - name: stopped-vm
    resource: azure.vm
    filters:
     - type: instance-view
       key: statuses[].code
       op: not-in
       value_type: swap
       value: "PowerState/running"
```
