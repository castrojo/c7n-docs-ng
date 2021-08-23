Developer Guide {#gcp_contribute}
===============

The c7n developer install includes c7n\_gcp. A shortcut for creating a
virtual env for development is available in the makefile:

``` {.bash}
make install
source bin/activate
```

This creates a virtual env in your enlistment and installs all packages
as editable.

Instead, you can do [pip install -r
tools/c7n\_gcp/requirements.txt]{.title-ref} to install dependencies.

Adding New GCP Resources
========================

Create New GCP Resource
-----------------------

Most resources extend the QueryResourceManager class. Each class
definition will use the \@resources.register(\'\<resource\_name\>\')
decorator to register that class as a Custodian resource substituting
\<resource\_name\> with the new resource name. The name specified in the
decorator is how the resource will be referenced within policies.

Each resource also contains an internal class called
[resource\_type]{.title-ref}, which contains metadata about the resource
definition, and has the following attributes:

-   

    `service` is required field, part of the request to GCP resource,

    :   The name of GCP service.

-   

    `component` is required field, part of the request to GCP resource,

    :   The name of GCP resource,

-   

    `version` is required field, part of the request to GCP resource,

    :   It is the [version]{.title-ref} of used resource API,

-   

    `enum_spec` is a required field,

    :   It has a tuple of ([enum\_operation]{.title-ref},
        [list\_operation]{.title-ref}, [extra\_args]{.title-ref}).

        -   \`enum\_operation\`: the name of the GCP resource method
            used to retrieve the list of resources,
        -   \`list\_operation\`: the JMESPath of the field name which
            contains the resource list in the JSON response body,
        -   \`extra\_args\`: can be used to set up additional params for
            a request to GCP.

-   

    `id` is required field,

    :   It\'s a field name of the response field that have to be used as
        resource identifier. The [id]{.title-ref} value is used for
        filtering.

-   

    `scope` is optional field, default is None,

    :   The scope of the Custodian resource. There are available 2
        options: [project]{.title-ref} or [None]{.title-ref}. If the
        [scope]{.title-ref} has a value [project]{.title-ref} the
        GOOGLE\_CLOUD\_PROJECT variable will be used for building
        request to GCP resource. If the scope is [None]{.title-ref} the
        request to GCP is built ignoring the GOOGLE\_CLOUD\_PROJECT
        variable.

-   

    `parent_spec` is an optional field that allows to build additional requests to parent resources, default is None.

    :   The field is used when the request to GCP resource should be
        created with extra parameters that can be loaded from parent
        resources. The resource should extend ChildResourceManager
        instead of QueryResourceManager and use ChildTypeInfo instead of
        TypeInfo to use the field. The [parent\_spec]{.title-ref} has
        following fields: [resource]{.title-ref},
        [child\_enum\_params]{.title-ref},
        [parent\_get\_params]{.title-ref},
        [use\_child\_query]{.title-ref}.

        -   The field [resource]{.title-ref} has value of the
            [resource\_name]{.title-ref} from
            \@resources.register(\'\<resource\_name\>\') that is used
            for the target parent resource.
        -   The field [child\_enum\_params]{.title-ref} is an array of
            tuples each of which maps parent instance field (first tuple
            element) to child\'s list argument (second tuple element).
            The mappings are used for building [list]{.title-ref}
            requests to parent resources. It works by the next scenario.
            First of all it loads a list of instances from parent
            resource. Further it loads instances for original resources
            using GCP resource field values from the loaded parent
            resources. It uses mappings for GCP resource fields from
            [child\_enum\_params]{.title-ref}. The first field in a
            tuple is a field from parent resource, the second one is the
            mapped original resource field name.
        -   The field [parent\_get\_params]{.title-ref} is an array of
            tuples each of which maps child instance field (first tuple
            element) to parent\'s [resource\_info]{.title-ref} field.
            The [resource\_info]{.title-ref} object has fields like
            Stackdriver log has. There are 2 options for the fields set:
            either [resource.labels]{.title-ref} and
            [protoPayload.resourceName]{.title-ref} or a log of the full
            event. The mappings are used for building [get]{.title-ref}
            requests to parent resources.
        -   The field [use\_child\_query]{.title-ref} controls whether
            the [query]{.title-ref} block of the current resource should
            be copied to its parent.

An example that uses [parent\_spec]{.title-ref} is available below.

``` {.python}
# the class extends ChildResourceManager
@resources.register('bq-table')
class BigQueryTable(ChildResourceManager):

    # the class extends ChildTypeInfo
    class resource_type(ChildTypeInfo):
        service = 'bigquery'  # the name of the GCP service
        version = 'v2'        # the version of the GCP service
        component = 'tables'  # the component of the GCP service
            # The `list` method in the resource. https://cloud.google.com/bigquery/docs/reference/rest/v2/tables/list
            # It requires 2 request params: `projectId` and `datasetId`. Since the resource has a parent the params will
            # be extracted from parent's instance
        enum_spec = ('list', 'tables[]', None)
        scope_key = 'projectId'
        id = 'id'
        parent_spec = {
            # the name of Custodian parent resource.
            'resource': 'bq-dataset',
            'child_enum_params': [
                # Extract the `datasetReference.datasetId` field from a parent instance and use its value
                # as the `datasetId` argument for child's `list` method (see `resource_type.enum_spec`)
                ('datasetReference.datasetId', 'datasetId'),
            ],
            'parent_get_params': [
                # Extract the `tableReference.projectId` field from a child instance and use its value
                # as the `projectId` event's field for parent's `get` method (see `resource_type.get`)
                ('tableReference.projectId', 'projectId'),
                # Similar to above
                ('tableReference.datasetId', 'datasetId'),
            ]
        }

        @staticmethod
        def get(client, event):
            return client.execute_query('get', {
                'projectId': event['project_id'],
                'datasetId': event['dataset_id'],
                'tableId': event['resourceName'].rsplit('/', 1)[-1]
            })
```

Most resources have get methods that are created based on the
corresponding [get]{.title-ref} method of the actual GCP resource. As a
rule the Custodian [get]{.title-ref} method has
[resource\_info]{.title-ref} param. The param has fields that can be
found in Stackdriver logs in [protoPayload.resourceName]{.title-ref} and
[resource]{.title-ref} fields. Examples of the Stackdriver logs are
available in tools/c7n\_gcp/tests/data/events folder.

There is an example of the resource below.

``` {.python}
from c7n_gcp.provider import resources
from c7n_gcp.query import QueryResourceManager, TypeInfo


@resources.register('loadbalancer-address')
class LoadBalancingAddress(QueryResourceManager):

    class resource_type(TypeInfo):
        service = 'compute'
        component = 'addresses'
        version = 'v1'
        enum_spec = ('aggregatedList', 'items.*.addresses[]', None)
        scope = 'project'
        id = 'name'

    @staticmethod
    def get(client, resource_info):
        return client.execute_command('get', {
            'project': resource_info['project_id'],
            'region': resource_info['location'],
            'address': resource_info[
                'resourceName'].rsplit('/', 1)[-1]})
```

Load New GCP Resource
---------------------

If you created a new module for a GCP service (i.e. this was the first
resource implemented for this service in Custodian), then import the new
service module in entry.py:

`entry.py`.

``` {.python}
import c7n_gcp.resources.<name of a file with created resources>
```

Each resource has to have test cases. There are implemented test cases
for resources list methods and get methods.

Testing
=======

c7n\_gcp follows the same guideance for test authoring as the
`core c7n code
base<Creating Tests>`{.interpreted-text role="ref"}. Examples with
relative directories (for example, `tests/foo/bar`) should be applied to
`tools/c7n_gcp` and not the root of the cloud-custodian repository.

In order to execute live tests you will need to set two additional
environment variables. These can be either set in your IDE or exported
on the command line. The first is `GOOGLE_CLOUD_PROJECT` - this should
point to a valid Google Cloud Project you have access to. The second,
`GOOGLE_APPLICATION_CREDENTIALS` should be the corresponding credentials
json with access to the aforementioned project.

For example:

``` {.bash}
export GOOGLE_CLOUD_PROJECT=cloud-custodian
export GOOGLE_APPLICATION_CREDENTIALS=data/credentials.json
pytest tools/c7n_gcp/tests -n auto
```

Updating Existing Tests
-----------------------

Many of tests in this project were created prior to the current
functional testing practices. To convert an existing test see
`how to convert existing tests<Converting Tests>`{.interpreted-text
role="ref"}.
