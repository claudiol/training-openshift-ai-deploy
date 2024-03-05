# OpenShift AI Training Deployment

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)


## Start Here

This is a WIP repository.  Basically we are using the Validated Patterns framework to deploy the resources needed for the 
OpenShift AI training that was done on March 4 - 6.

## Rationale

The goal for this pattern is to:

* Use a GitOps approach to manage the OpenShift AI training deployments across both public and private clouds.

The Validated Patterns framework also allows:
* Enable cross-cluster governance and application lifecycle management.
* Securely manage secrets across the deployment.

This basically deploys the resources found in the following repo: https://github.com/openshift-ai-examples/openshift-ai-examples/tree/main/openshift-ai-install

To deploy this do the following:

* Create an empty OpenShift environment
** You can use demo.redhat.com to deploy an OpenShift environment.
* Clone the repository
`git clone https://github.com/claudiol/training-openshift-ai-deploy`
or 
`git clone git@github.com:claudiol/training-openshift-ai-deploy.git`
* To deploy just run the following:
`./pattern.sh make install`

The above uses a container that includes all the tools (helm, make etc) that are needed to deploy the pattern.  You can find the container 
definition in the following Validated Patterns repository: https://github.com/validatedpatterns/utility-container

* The Validated Patterns framework describes all the resources that will be deployed to the OpenShift cluster in the `values-hub.yaml` file.

`values-hub.yaml` details

The clusterGroup section defines whether this is a hub or an edge site.
clusterGroup:
  name: hub
  isHubCluster: true

**Understanding Namespace Creation using the Validated Patterns Framework**

In the realm of Kubernetes and containerized environments, managing namespaces efficiently is pivotal. It ensures proper organization, security, and resource isolation within a cluster. With the Validated Patterns framework, creating namespaces becomes not just systematic but also highly customizable.

In this blog we will talk about the different ways to create namespaces by describing them in the Validated Patterns values files. We provide examples of the different options we support and the reasoning behind them.

**Describing Namespaces in the Validated Patterns values files**

Namespaces in Kubernetes offer a way to divide cluster resources between multiple users, teams, or projects. They act as virtual clusters within a physical cluster, enabling isolation and segmentation.

The Validated Patterns framework provides a structured approach to creating namespaces, allowing for a range of configurations to meet specific requirements. In addition, the Validated Patterns framework, by default, creates an Operator Group, defined by the OperatorGroup resource, which provides multi-tenant configuration to OLM-installed Operators.

**The namespaces: Configuration structure**

The Validated Patterns values files have several required sections that fall under the clusterGroup: top level section. The structure to describe namespaces starts with the declaration of the namespaces: section in the values-*.yaml files.

clusterGroup:
  name: example
  isHubCluster: true
  sharedValueFiles:
  - /values/{{ .Values.global.clusterPlatform }}.yaml
  - /values/{{ .Values.global.clusterVersion }}.yaml

  namespaces:
     …
The Validated Patterns framework, defined by the clusterGroup helm chart in the common github repository, accepts two formats for the namespaces: section:

* A list object
* A map object

If a namespace is described as a list item the Validated Patterns framework will use that list element and create a Namespace resource, using that name, as well as an OperatorGroup resource. By default the Validated Patterns framework will add the namespace to the spec.targetNamespaces for the OperatorGroup resource.

You can use both a list, and a map object, to describe a namespace. Here’s an example of using a list to describe a namespace.

  namespaces:
    - open-cluster-management
    - vault
    - golang-external-secrets
    - config-demo
    - hello-world

To define labels and annotations you would describe the namespace as a map object, adding the labels and annotations needed, using the following structure in the values.yaml file:

  namespaces:
  - open-cluster-management:
      labels:
        openshift.io/node-selector: ""
        kubernetes.io/os: linux
      annotations:
        openshift.io/cluster-monitoring: "true"
        owner: "namespace owner"

For more information on using the Validated Patterns to create namespaces please see our blog at: 
The subscriptions section lists the operators that are needed by the validated pattern:
https://validatedpatterns.io/blog/2023-12-15-understanding-namespaces/

**Describing Operators to be deployed on OpenShift**

You can also describe the subscriptions, or operators, that you need to be deployed on the OpenShift cluster.  Here's an example on how to describe subscriptions in the `values-hub.yaml` file:

````
  subscriptions:                                                                                       
  - name: advanced-cluster-management                                                                  
    namespace: open-cluster-management                                                                 
    channel: release-2.3                                                                               
    csv: advanced-cluster-management.v2.3.2                                                            
                                                                                                       
  - name: seldon-operator                                                                              
    namespace: manuela-ml-workspace                                                                    
    source: community-operators                                                                        
    csv: seldon-operator.v1.7.0                                                                        
                                                                                                       
  - name: opendatahub-operator                                                                         
    source: community-operators                                                                        
    csv: opendatahub-operator.v1.1.0                                                                   
                                                                                                       
  - name: openshift-pipelines-operator-rh                                                              
    csv: redhat-openshift-pipelines.v1.5.1
    
  - name: amq-streams                                                                                  
    namespace: manuela-tst-all                                                                         
    channel: amq-streams-1.7.x                                                                         
    csv: amqstreams.v1.7.1                                                                             
                                                                                                       
  - name: red-hat-camel-k                                                                              
    namespace: manuela-data-lake-central-s3-store                                                      
    channel: 1.4.x                                                                                     
    csv: red-hat-camel-k-operator.v1.4.0                                                               
                                                                                                       
  - name: red-hat-camel-k                                                                              
    namespace: manuela-tst-all                                                                         
    channel: 1.4.x                                                                                     
    csv: red-hat-camel-k-operator.v1.4.0   
````


Notice that the operator entry can include the target namespace where it should be installed and also the csv, or ClusterServiceVersion, of the operator. If you choose to use the CSV attribute you that will tell the operator that that is the starting version you want to use.  The operator may upgrade to a higher version depending on the operator install process. 

The next two sections are related to the ArgoCD environment. We describe projects and applications in the values file that will be deployed into the ArgoCD application instance.

Projects provide a logical grouping of applications, which is useful when Argo CD is used by multiple teams. An ArgoCD Application represents a deployed application instance in an environment.

````
  projects:
  - datacenter

  applications:
  - name: acm
    namespace: open-cluster-management
    project: datacenter
    path: common/acm

  - name: odh
    namespace: opendatahub
    project: datacenter
    path: charts/datacenter/opendatahub

````

For more information on Validated Patterns go to: https://validatedpatterns.io.
