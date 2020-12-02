# BackingServices Helm chart

A backing service is a feature from Pega Infinity platform which can be deployed as an independent service module. For example `Search and Reporting Service` or `SRS` is a backing service which can replace the embedded search feature from Pega Infinity Platform and can be provisioned independently and configured as an external service providing search capabilities with one or more Pega Infinity environments.  

The backingservices chart supports deployment options for `Search and Reporting Service` (abbreviated as SRS). A backing service may be configured with one or more Pega deployments. 
These backing services should be deployed in their own namespace and may be shared across multiple Pega Infinity environments.

**Example:**

**_Single backing service shared across all pega environments:_**
backingservice 'Search and Reporting Service' provisioned in its own namespace, and the service endpoint configured across dev, staging and production pega environments. The service provides isolation of data in a shared setup.

**_Multiple backing service deployments:_**
You can deploy more than one instance of backing service deployments, in case you want to host a separate deployment of 'Search and Reporting Service' for non-prod and production pega infinity environments. You need to configure the appropriate service endpoint with the pega infinity deployment values.

### Search and Reporting Service

The Search and Reporting Service provides next gen search capabilities to the Pega Infinity platform 8.6 and above versions. 

This service replaces the legacy search module from the platform with an independently deployable and scalable service along with the built-in capabilities to support more than one Pega environments with its data isolation features. 
The service deployment provisions runtime service pods along with a dependency on a backing technology ElasticSearch service for storage and retrieval of data. 

#### SRS Version compatibility matrix
Pega Infinity version   | SRS version   | ElasticSearch version     | Description
---                     | ---           | ---                       | ---
< 8.6                   | NA            | NA                        | SRS service can be used with Pega Infinity 8.6 and above
\>= 8.6                 | \> 1.6.0      | 7.1.x                     | Pega Infinity 8.6 and above version may use SRS Image tag version 1.6.0 and above. Current SRS versions are certified to work with Elasticsearch version 7.1.x.


#### SRS service runtime configuration:

The values.yaml provides configuration options to define the deployment resources along with option to either provision ElasticSearch cluster automatically for data storage, or you can choose to configure an existing managed elasticsearch cluster to use as a datastore with the SRS service runtime. 

If an externally managed elasticsearch cluster is being used, make sure the service is accessible to the k8s cluster where SRS is deployed.

You may enable the component of ([Elasticsearch](https://github.com/helm/charts/tree/master/stable/elasticsearch/values.yaml) in the backingservices by configuring the 'srs.elasticsearch' section in values.yaml file to deploy ElasticSearch cluster automatically. For more configuration options available for each of the components, see their Helm Charts.

#### configuration settings:
Configuration                       | Usage
---                                 | ---
`enabled`                           | Enable the Search and Reporting Service deployment as a backing service.
`deploymentName`                    | The name of your SRS cluster.  Resources created will be prefixed with this string. This is also the service name for SRS.
`srsRuntime`                        | This section defines the SRS service specific resource configuration options like image, replica count, cpu and memory resource settings etc.
`elasticsearch`                     | Define the elasticsearch cluster configurations using this section. The chart from ([Elasticsearch](https://github.com/helm/charts/tree/master/stable/elasticsearch/values.yaml) is used for provisioning the cluster.
`elasticsearch.provisionCluster`    | This setting when enabled will provision Elasticsearch cluster automatically with SRS runtime. Disable this to use an existing external ElasticSearch cluster with the SRS runtime.

Example:

```yaml
srs:
  enabled: true
  deploymentName: "YOUR_SRS_DEPLOYMENT_NAME"
  srsRuntime:
    #srs-service values
    replicaCount: 2
    srsImage:
        name: platform-services/fnx-search
        version: 1.6.1
        pullPolicy: IfNotPresent
    resources:
        limits:
            cpu: 1300m
            memory: "2Gi"
        requests:
            cpu: 650m
            memory: "2Gi"
    serviceType: "ClusterIP"
    env:
        #AuthEnabled may be set to true when there is an authentication mechanism in place between SRS and Pega Infinity.
        AuthEnabled: false
        PublicKeyURL:
  
  elasticsearch:
    # elasticsearch.provisionCluster true will provision an internal elasticsearch cluster with specified configuration
    provisionCluster: true
    # set the external Elasticsearch cluster URL and port details below when using an externally managed elasticsearch
    # make sure the endpoint is accessible from the kubernetes cluster pods.
#    domain: managed-elasticsearch.acme.io
#    port: 443
#    protocol: https
