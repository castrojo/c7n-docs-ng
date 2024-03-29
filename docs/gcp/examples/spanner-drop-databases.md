Spanner - Drop Databases
========================

This policy drops (deletes) databases that have [dev]{.title-ref} in the
name and then notifies about the action taken via an email.

To configure Cloud Pub/Sub messaging please take a look at the
`gcp_genericgcpactions`{.interpreted-text role="ref"} page.

``` {.yaml}
policies:
  - name: gcp-spanner-instance-databases-delete-and-notify
    resource: gcp.spanner-database-instance
    filters:
      - type: value
        key: name
        op: contains
        value: dev
    actions:
      - type: delete
      - type: notify
        subject: The following databases were dropped
        to:
          - email@address
        transport:
            type: pubsub
            topic: projects/cloud-custodian/topics/demo-notifications
```
