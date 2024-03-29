Developer Guide {#azure_contribute}
===============

The c7n developer install includes c7n\_azure. A shortcut for creating a
virtual env for development is available in the makefile:

``` {.bash}
$ make install
$ source bin/activate
```

This creates a virtual env in your enlistment and installs all packages
as editable.

You can also simply do [pip install -r requirements-dev.txt]{.title-ref}
to install all dev/test dependencies.

Adding New Azure Resources
--------------------------

### Install Azure Dependencies

Custodian interfaces with ARM resources using Azure\'s SDKs. Install the
resources SDK in `setup.py`.

``` {.python}
install_requires=["azure-mgmt-authorization",
                  ...
                  "azure-mgmt-containerregistry",
                  ...
```

### Create New Azure Resource

Create your new Azure Resource.

-   `service`: The Azure SDK dependency added in step 1.

-   `client`: Client class name of the Azure Resource SDK of the
    resource you added.

-   

    `enum_spec`: Is a tuple of (enum\_operation, list\_operation, extra\_args). The resource SDK client will have a list of operations this resource has.

    :   Place the name of the property as the enum\_operation. Next, put
        [list]{.title-ref} as the operations.

-   `default_report_fields`: Fields that are used for running
    `custodian report`.

``` {.python}
from c7n_azure.provider import resources
from c7n_azure.resources.arm import ArmResourceManager


@resources.register('containerregistry')
class ContainerRegistry(ArmResourceManager):

    class resource_type(ArmResourceManager.resource_type):
        service = 'azure.mgmt.containerregistry'
        client = 'ContainerRegistryManagementClient'
        enum_spec = ('registries', 'list', None)
        default_report_fields = (
            'name',
            'location',
            'resourceGroup',
            'sku.name
        )
```

### Load New Azure Resource

Once the required dependencies are installed and created the new Azure
Resource, custodian will load all registered resources. Import the
resource in `entry.py`.

``` {.python}
import c7n_azure.resources.container_registry
```

Testing
-------

Tests for c7n\_azure run automatically with other Custodian tests. See
`Testing for Developers <developer-tests>`{.interpreted-text role="ref"}
for information on how to run Tox.

If you\'d like to run tests at the command line or in your IDE then
reference [tox.ini]{.title-ref} to see the required environment
variables and command lines for running [pytest]{.title-ref}.

### Test framework

c7n\_azure uses [VCR.py]{.title-ref} for tests. This framework is used
for tests in the official Azure Python SDK.

VCRpy documentation can be found here: [VCR.py
documentation](https://vcrpy.readthedocs.io/en/latest/).

### ARM templates

To ensure VCR cassettes can be easily re-recorded, there are ARM
templates to deploy Azure tests infrastructure.

These templates will allow you to provision real Azure resources
appropriate for recreating the VCR cassettes used by the unit tests.
They will let you run the unit tests against real resources.

ARM templates and helper scripts can be found in
[tools/c7n\_azure/tests/templates]{.title-ref} folder.

There are two scripts [provision.sh]{.title-ref} and
[cleanup.sh]{.title-ref} to provision and delete resources.

These scripts will provision or delete all ARM templates ([.json
files]{.title-ref}) in this directory using resource groups named after
the template files ([test\_\<filename\>]{.title-ref}).

This scripts use Azure CLI, so you need to [az login]{.title-ref} and
[az account set -s \'subscription name\']{.title-ref} first.

You can optionally pass a list of file names without extension to the
scripts to act only on those templates:

``` {.bash}
provision.sh vm storage
cleanup.sh storage
```

or do everything

``` {.bash}
provision.sh
```

If test method requires real infrastructure, please decorate this method
with the ARM template file name to ensure this test can automatically
create required infrastructure if needed.

``` {.python}
@arm_template('template.json')
def test_template(self):
```

### Cassettes

[AzureVCRBaseTest]{.title-ref} attempts to automatically obscure keys
and other secrets in cassettes and replace subscription ids, but it is
required to verify cassettes don\'t contain any sensitive information
before submitting.

For long standing operations cassette can be modified to reduce test
execution time (in case recorded cassette contains some responses with
Retry-After headers or Azure SDK waits until resource is provisioned).

### Running tests

You can use [tox]{.title-ref} to run all tests or instead you can use
[pytest]{.title-ref} and run only Azure tests (or only specific set of
tests). Running recorded tests still requires some authentication, it is
possible to use fake data for authorization token and subscription id.

``` {.bash}
export AZURE_ACCESS_TOKEN=fake_token
export AZURE_SUBSCRIPTION_ID=ea42f556-5106-4743-99b0-c129bfa71a47
pytest tools/c7n_azure/tests
```
