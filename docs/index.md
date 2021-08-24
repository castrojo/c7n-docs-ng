## Welcome to the Cloud Custodian Documentation!

Cloud Custodian unifies the dozens of tools and scripts organizations use for managing their public cloud accounts into one, easy to use open source
tool.
Hundreds of organizations trust Custodian to manage their cloud environments by
ensuring compliance to security policies, tag policies, garbage collection of
unused resources, and cost management. 
Open Source since 2015 and with hundreds of diverse contributors, c7n (as we call it) is on the forefront of Governance as Code. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/plhcFnhymHA" title="Introduction to Cloud Custodian" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

It uses a stateless rules engine for policy definition and enforcement, with
metrics, structured outputs and detailed reporting for clouds infrastructure.
It integrates tightly with serverless runtimes to provide real time
remediation/response with low operational overhead. Cloud Custodian can be bound to serverless event streams across multiple
cloud providers that maps to security, operations, and governance use
cases.
Custodian adheres to a compliance as code principle, so you can
validate, dry-run, and review changes to your policies.

Cloud Custodian policies are expressed in YAML and include the
following:

-   The type of resource to run the policy against
-   [Filters](filters/) to narrow down the set of resources
-   [Actions](actions/) to take on the filtered set of resources

Get started with Cloud Custodian!

=== "Linux and OSX"

    To install Cloud Custodian :

        python3 -m venv custodian
        source custodian/bin/activate
        pip install c7n       # This includes AWS support

    To install Cloud Custodian for Azure, you will also need to run:

        pip install c7n_azure # Install Azure package

    To install Cloud Custodian for GCP, you will also need to run:

        pip install c7n_gcp   # Install GCP Package


=== "Windows" 
    
    To install Cloud Custodian run:

        python3 -m venv custodian
        ./custodian/bin/activate
        pip install c7n    # This includes AWS support

    To install Cloud Custodian for Azure, you will also need to run:

        pip install c7n_azure

    To install Cloud Custodian for GCP, you will also need to run:

        pip install c7n_gcp

=== "Docker"

    To install via docker, run:

        docker pull cloudcustodian/c7n

    You'll need to export cloud provider credentials to the container when
    executing. One example, if you\'re using environment variables for
    provider credentials:

        docker run -it \
          -v $(pwd)/output:/home/custodian/output \
          -v $(pwd)/policy.yml:/home/custodian/policy.yml \
          --env-file <(env | grep "^AWS\|^AZURE\|^GOOGLE") \
             cloudcustodian/c7n run -v -s /home/custodian/output /home/custodian/policy.yml

### Now you're ready to [explore Cloud Custodian](quickstart/#explore-cloud-custodian)