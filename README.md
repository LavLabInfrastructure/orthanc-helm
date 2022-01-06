# Korthweb - Orthanc deployment on Kubernetes
Korthweb is an open-source project to deploy [Orthanc](https://www.orthanc-server.com/) on Kubernetes platform. Orthanc is an open-source application to ingest, store, display and distribute medical images. Korthweb provides a number of deployment approaches. Korthweb is a sister project of [Orthweb](https://github.com/digihunch/orthweb), an deployment automation project for Orthanc on AWS EC2. 

## Kubernetes Cluster

We need a Kubernetes cluster. If you do not have a cluster, refer to the instruction in the *[cluster](https://github.com/digihunch/korthweb/tree/main/cluster)* directory to build a Kubernetes cluster first.


## Deployment Approaches
This project explores the following deployment approaches.
| Approach | Tools | Description |  |
|--|--|--|--|
| Manual | kubectl, helm, Istioctl | Use YAML manifests and external Helm charts to install Istio, PostgreSQL and Orthanc. Use this approach for troubleshooting and learning. For automation, go with the GitOps approach. | 
| GitOps | kubectl, helm, flux | The files help FluxCD install Istio, PostgreSQL, and Orthanc. Istio provides mTLS between services, and Ingress for north-south traffic. This approach is to automate the steps in the Manual approach. |
| Helm Chart | kubectl, helm  | This repo provides a Helm chart named orthanc is created to install PostgreSQL (installing external charts) and Orthanc. The Chart also configures TLS between Orthanc and PostgreSQL.  |

Each approach has its own sub-directory with instruction in their respective sub-directory.
* [Manual](https://github.com/digihunch/korthweb/tree/main/manual)
* [GitOps](https://github.com/digihunch/korthweb/tree/main/gitops)
* [Helm Chart](https://github.com/digihunch/korthweb/tree/main/helm)

The following tools are used:
* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl): connect to API server to manage the Kubernetes cluster. With multiple clusters, you need to [switch context](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).
* [helm](https://helm.sh/docs/intro/install/): helm is package manager for Kubernetes. It is used in all three approaches to install third party charts such as PostgreSQL
* [istioctl](https://helm.sh/docs/intro/install/): istioctl is an alternative to helm to install istio manually.
* [flux](https://fluxcd.io/docs/): FluxCD is a GitOps tool to keep target Kubernetes cluster in sync with the source of configuration in the GitOps directory.

## Architectural Considerations

### Helm Chart
The purpose of this repo is to provide a Helm Chart to deploy Orthanc on Kubernetes with a single command, including the creation of self-signed certificates. The Helm Chart is defined in the *[orthanc](https://github.com/digihunch/korthweb/tree/main/orthanc)* directory and is customizable with parameters. The rest of this instruction is based on automatic deployment.
The directory *[manual](https://github.com/digihunch/korthweb/tree/main/manual)* is also kept in this repository to help userstand the deployment and guide development of Helm Chart.
*Note*: currently, the automatic deployment doesn't support Istio because of some limitation with Helm. See [readme](https://github.com/digihunch/korthweb/blob/main/istio/README.md).

### Istio Service Mesh
Applications running as [microservices](https://www.digihunch.com/2021/11/from-microservice-to-service-mesh/) requires many common features for observability (e.g. tracing), security (e.g. mTLS), traffic management (e.g. ingress and egress, traffic splitting), and resiliency (circuit breaking, retry/timeout). Service Mesh commodifies these features into a layer between Kubernetes platform and the workload. Istio is a popular choice for Service Mesh. 





