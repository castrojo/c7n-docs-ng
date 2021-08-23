SQL - Find all SQL Databases with Premium SKU {#azure_examples_sqldatabasewithpremiumsku}
=============================================

Find all SQL databases with Premium SKU.

``` {.yaml}
policies:
  - name: sqldatabase-with-premium-sku
    resource: azure.sqldatabase
    filters:
      - type: value
        key: sku.tier
        op: eq
        value: Premium
```
