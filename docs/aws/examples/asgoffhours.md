ASG - Offhours Support {#asgoffhours}
======================

The following example policy will stop all ASGs with the
`custodian_downtime` tag at 10pm daily and start them back up at 10am
daily, leaving them off during weekends.

``` {.yaml}
policies:
  - name: offhour-stop-22
    resource: asg
    comments: |
      Daily stoppage at 10pm
    filters:
      - type: offhour
        tag: custodian_downtime
        offhour: 22
    actions:
      - suspend

  - name: onhour-start-10
    resource: asg
    comments: |
      Daily start at 10am
    filters:
      - type: onhour
        tag: custodian_downtime
        onhour: 10
    actions:
      - resume
```

For detailed information on offhours/onhours support and configuration,
see `offhours`{.interpreted-text role="ref"}.
