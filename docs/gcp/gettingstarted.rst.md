Getting Started (Beta) {#gcp_gettingstarted}
======================

The GCP provider (Beta) is an optional package which can be installed to
enable writing policies which interact with GCP related resources.

Install GCP Plugin {#gcp_install-cc}
------------------

First, ensure you have
`installed the base Cloud Custodian application <install-cc>`{.interpreted-text
role="ref"}. Cloud Custodian is a Python application that supports
Python 2 and 3 on Linux and Windows. We recommend using Python 3.6 or
higher.

Once the base install is complete, you are now ready to install the GCP
provider package using one of the following options:

### Option 1: Install released packages to local Python Environment

``` {.bash}
pip install c7n
pip install c7n_gcp
```

### Option 2: Install latest from the repository

``` {.bash}
git clone https://github.com/cloud-custodian/cloud-custodian.git
pip install -e ./cloud-custodian
pip install -e ./cloud-custodian/tools/c7n_gcp
```

Connect Your Authentication Credentials {#gcp_authenticate}
---------------------------------------

In order for Custodian to be able to interact with your GCP resources,
you will need to configure your GCP authentication credentials on your
system in a way in which the application is able to retrieve them.

Choose from one of the following methods to configure your credentials,
depending on your use case. In either option, after the configuration is
complete, Custodian will implicitly pick up your credentials when it
runs.

### GCP CLI

If you are a general user accessing a single account, then you can use
the GCP CLI to configure your credentials.

First, [install](https://cloud.google.com/sdk/install) `gcloud` (the GCP
Command Line Interface).

Then run the following command, substituting your username:

``` {.bash}
gcloud auth application-default login
```

Executing the command will open a browser window with prompts to finish
configuring your credentials. For more information on this command,
[view its
documentation](https://cloud.google.com/sdk/gcloud/reference/auth/login).

### Environment Variables

If you are planning to run Custodian using a service account, then
configure your credentials using environment variables.

Follow the steps outlined in the [GCP documentation to configure
credentials for service
accounts.](https://cloud.google.com/docs/authentication/getting-started)

Write Your First Policy {#gcp_write-policy}
-----------------------

A policy is the primary way that Custodian is configured to manage cloud
resources. It is a YAML file that follows a predetermined schema to
describe what you want Custodian to do.

There are three main components to a policy:

-   Resource: the type of resource to run the policy against
-   Filters: criteria to produce a specific subset of resources
-   Actions: directives to take on the filtered set of resources

In the example below, we will write a policy that filters for compute
engine resources, and then stops each resource.

Filename: `custodian.yml`

``` {.yaml}
policies:
  - name: my-first-policy
    description: |
      Stops all compute instances that are named "test"
    resource: gcp.instance
    filters:
      - type: value
        key: name
        value: test
    actions:
      - type: stop
```

Run Your Policy {#gcp_run-policy}
---------------

First, ensure you have
`configured one of the supported authentication mechanisms <gcp_authenticate>`{.interpreted-text
role="ref"}.

Next, run the following command to execute the policy with Custodian:

``` {.bash}
GOOGLE_CLOUD_PROJECT="project-id" custodian run --output-dir=. custodian.yml
```

If successful, you should see output similar to the following on the
command line:

    2016-12-20 08:35:06,133: custodian.policy:INFO Running policy my-first-policy resource: gcp.instance
    2016-12-20 08:35:07,514: custodian.policy:INFO policy: my-first-policy resource: gcp.instance has count:3 time:1.38
    2016-12-20 08:35:08,188: custodian.policy:INFO policy: my-first-policy action: stop: 3 execution_time: 0.67

You should also find a new `my-first-policy` directory with a log and
other files (subsequent runs will append to the log by default, rather
than overwriting it).

See `filters`{.interpreted-text role="ref"} for more information on the
features of the Value filter used in this sample.
