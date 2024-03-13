
# Orthanc-Helm

<a href="https://www.orthanc-server.com/"><img style="float" align="right" src="docs/assets/images/orthanc_logo.png"></a>

[![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?logo=kubernetes&logoColor=white)](https://www.kubernetes.io)
[![Helm](https://img.shields.io/badge/helm-%230f1689.svg?logo=helm&logoColor=white)](https://helm.sh/)

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

This Helm chart deploys Orthanc, an open-source DICOM server, along with an optional authentication service, in a Kubernetes cluster. Derived from [digihunch](https://github.com/digihunch)'s [Korthweb](https://github.com/digihunch/korthweb).

## Prerequisites

- Kubernetes cluster
- Helm 3

## Installation (LavLab Repo)

1. Add the Helm repository (if not already added):

```bash
helm repo add lavlabinfra https://lavlabinfrastructure.github.io/charts/
helm repo update
```

2. Install the chart:
```bash
helm install orthanc orthanc/orthanc --values values.yaml
```

Replace `values.yaml` with your custom values file.

## Installation (From Source)
1. Clone the Helm chart repository:

   ```bash
   git clone https://github.com/lavlabinfrastructure/charts.git
   cd charts
   ```
2. Install the chart:
```bash
helm install orthanc ./orthanc --values values.yaml
```

## Configuration 

The following table lists the configurable parameters of the Orthanc chart and their default values.

| Parameter                           | Description                               | Default                |
| ----------------------------------- | ----------------------------------------- | ---------------------- |
| `replicaCount`                      | Number of Orthanc replicas                | `1`                    |
| `image.repository`                  | Orthanc image repository                  | `orthancteam/orthanc`  |
| `image.pullPolicy`                  | Orthanc image pull policy                 | `Always`               |
| `image.tag`                         | Orthanc image tag                         | `24.3.3`               |
| `serviceAccount.create`             | Create a service account                  | `true`                 |
| `serviceAccount.name`               | Service account name                      | `"orth-service"`       |
| `service.type`                      | Kubernetes Service type                   | `ClusterIP`            |
| `service.httpPort`                  | Orthanc HTTP port                         | `8042`                 |
| `service.dicomPort`                 | Orthanc DICOM port                        | `4242`                 |
| `resources`                         | CPU/Memory resource requests/limits       | `{}`                   |
| `autoscaling.enabled`               | Enable autoscaling                        | `false`                |
| `autoscaling.minReplicas`           | Minimum number of replicas                | `1`                    |
| `autoscaling.maxReplicas`           | Maximum number of replicas                | `5`                    |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization percentage   | `85`                   |
| `autoscaling.targetMemoryUtilizationPercentage` | Target memory utilization percentage | `85`              |
| `ingress.enabled`                   | Enable ingress controller                 | `true`                 |
| `ingress.hosts[0].host`             | Hostname to your Orthanc instance         | `orthanc.example.com`  |
| `ingress.hosts[0].paths[0].path`    | Path within the URL structure             | `/`                    |
| `ingress.tls`                       | Ingress TLS configuration                 | `[]`                   |
| `env`                               | Environment variables for Orthanc         | `[]`                   |
| `orthancJson`                       | Orthanc configuration in JSON format      | `{}`                   |
| `auth_service.enabled`              | Enable the authentication service         | `false`                |
| `auth_service.image.repository`     | Authentication service image repository   | `orthancteam/orthanc-auth-service` |
| `auth_service.image.tag`            | Authentication service image tag          | `24.2.0`               |
| `auth_service.env`                  | Environment variables for the authentication service | `[]`     |
| `auth_service.resources`            | CPU/Memory resource requests/limits for the authentication service | `{}` |
| `auth_service.permissionsJson`      | Permissions configuration for the authentication service | `{...}`  |

For more detailed information about each parameter, refer to the Orthanc documentation and the `values.yaml` file.

## Auth Service Configuration
This chart does not handle interconfigurations, meaning enabling auth_service will not make your orthanc instance start using it! You are expected to manually configure this in your orthancJson. Please reference the [orthanc-auth-service documentation](https://orthanc.uclouvain.be/book/plugins/authorization.html) as well as their example deployments in their [repository](https://github.com/orthanc-team/orthanc-auth-service/tree/main).

Here's an example orthanc-auth-service configuration that uses an external keycloak instance that we assume is properly configured with an orthanc client:

```yaml
auth_service:
  enabled: true
  env:
    - name: ENABLE_KEYCLOAK
      value: 'true'
    - name: KEYCLOAK_URI
      value: https://keycloak.example/realms/your-realm
    - name: KEYCLOAK_CLIENT_SECRET
      value: asdfsecretkeystring
    - name: PUBLIC_ORTHANC_ROOT
      value: https://your.orthanc.url.local/
    - name: PUBLIC_LANDING_ROOT
      value: https://your.orthanc.url.local/ui/app/token-landing.html
    - name: USERS
      value: |
        {
          "share-user": "change-me"
        }
  # optionally define rbac, default is the admin, doctor, and external roles as defined in the auth service's example keycloak deployment
  permissionsJson: |-
    {
      ...
    }
orthancJson: |

  {  
    "Name": "Orthanc",
    "RemoteAccessAllowed": true,
    "SslEnabled": false,
    "AuthenticationEnabled": false,    
    "Authorization": {
      "WebServiceRootUrl": "http://localhost:8000/",
      "WebServiceUsername": "share-user",
      "WebServicePassword": "supersecretpassword",
      "StandardConfigurations": [
        "orthanc-explorer-2"
      ],
      "TokenHttpHeaders": [ "api-key" ],
      "CheckedLevel": "studies"
    },
    "RegisteredUsers": {
      "admin": "orthanc"
    },
    "DicomTlsEnabled": false,
    "DicomTlsRemoteCertificateRequired": false,
    "PostgreSQL": {
      "EnableIndex": true,
      "EnableStorage": true,
      "Host": "${DB_ADDR}",
      "Port": ${DB_PORT},
      "Database": "${DB_DBNAME}",
      "Username": "${DB_USERNAME}",
      "Password": "${DB_PASSWORD}",
      "EnableSsl": true,
      "Lock": false
    },
    "DicomWeb": {
      "Enable": true,                
      "Root": "/dicom-web/",         
      "EnableWado": true,            
      "WadoRoot": "/wado",           
      "Ssl": false,                   
      "QidoCaseSensitive": true,     
      "Host": "",                    
      "StudiesMetadata": "Full",     
      "SeriesMetadata": "Full",      
      "EnableMetadataCache": true,    
      "MetadataWorkerThreadsCount": 4,     
      "PublicRoot": "/dicom-web/"    
    },
    "OHIF": {
      "DataSource": "dicom-web"
    },
    "OrthancExplorer2": {
      "Enable": true,
      "IsDefaultOrthancUI": true,
      "Tokens": {
        "InstantLinksValidity": 3600,
        "ShareType": "ohif-viewer-publication"
      },
      "Keycloak": {
        "Enable": true,
        "Url": "https://keycloak.example",
        "Realm": "your-realm",
        "ClientId": "orthanc"
      }
    },
    "Plugins": [
      "/usr/share/orthanc/plugins-available/libOrthancPostgreSQLIndex.so",
      "/usr/share/orthanc/plugins-available/libOrthancPostgreSQLStorage.so",
      "/usr/share/orthanc/plugins-available/libOrthancOHIF.so",
      "/usr/share/orthanc/plugins-available/libOrthancDicomWeb.so",
      "/usr/share/orthanc/plugins-available/libOrthancAuthorization.so",
      "/usr/share/orthanc/plugins-available/libOrthancExplorer2.so"
    ]
  }
```
Properly configuring an OAuth client in Keycloak is beyond the scope of this tutorial.
### Please note that orthancJson.Authorization.WebServiceRootUrl should always be http://localhost:8000 as this links it to the authservice container in the orthanc pod.

## Generic Plugin Configuration
In our deployment we have configured postgres, object storage, python scripts, and all our other orthanc plugins using a combination of orthanc.json configs, volume mounts, and environmental variables. See [the list of plugins](https://orthanc.uclouvain.be/book/plugins.html) for more information. 

Here's an example configuration that is a simplified version of our deployment. It uses postgres indexing with s3 object storage backing, as well as the Explorer2 UI and OHIF viewer.
```yaml
env:
  - name: DB_ADDR
    value: orthanc-db-rw.pacs-system.svc
  - name: DB_DBNAME
    valueFrom:
      secretKeyRef:
        key: dbname
        name: orthanc-db-app
  - name: DB_PORT
    valueFrom:
      secretKeyRef:
        key: port
        name: orthanc-db-app
  - name: DB_USERNAME
    valueFrom:
      secretKeyRef:
        key: username
        name: orthanc-db-app
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        key: password
        name: orthanc-db-app
  - name: AWS_ACCESS_KEY_ID
    valueFrom:
      secretKeyRef:
        key: AWS_ACCESS_KEY_ID
        name: orthanc-bucket
  - name: AWS_SECRET_ACCESS_KEY
    valueFrom:
      secretKeyRef:
        key: AWS_SECRET_ACCESS_KEY
        name: orthanc-bucket
  - name: BUCKET_HOST
    valueFrom:
      configMapKeyRef:
        key: BUCKET_HOST
        name: orthanc-bucket
  - name: BUCKET_NAME
    valueFrom:
      configMapKeyRef:
        key: BUCKET_NAME
        name: orthanc-bucket
  - name: BUCKET_PORT
    valueFrom:
      configMapKeyRef:
        key: BUCKET_PORT
        name: orthanc-bucket
  - name: BUCKET_REGION
    valueFrom:
      configMapKeyRef:
        key: BUCKET_REGION
        name: orthanc-bucket
extraVolumeMounts:
  - mountPath: /var/s3-keys
    name: s3-keys
    readOnly: true
extraVolumes:
  - name: s3-keys
    secret:
      secretName: orthanc-aes-encryption-key
orthancJson: |
  {  
    "Name": "Orthanc",
    "RemoteAccessAllowed": true,
    "SslEnabled": false,
    "AuthenticationEnabled": true,
    "RegisteredUsers": {
      "admin": "orthanc"
    },
    "DicomTlsEnabled": false,
    "DicomTlsRemoteCertificateRequired": false,
    "PostgreSQL": {
      "EnableIndex": true,
      "EnableStorage": true,
      "Host": "${DB_ADDR}",
      "Port": ${DB_PORT},
      "Database": "${DB_DBNAME}",
      "Username": "${DB_USERNAME}",
      "Password": "${DB_PASSWORD}",
      "EnableSsl": true,
      "Lock": false
    },
    "AwsS3Storage": {
      "BucketName": "${BUCKET_NAME}",
      "Region": "${BUCKET_REGION}",
      "Endpoint": "http://${BUCKET_HOST}:${BUCKET_PORT}/",
      "AccessKey": "${AWS_ACCESS_KEY_ID}",
      "SecretKey": "${AWS_SECRET_ACCESS_KEY}",
      "VirtualAddressing": false,
      "StorageEncryption": {
        "Enable": true,
        "MasterKey": [1, "/var/s3-keys/aes1.key"], // key id - path to the base64 encoded key
        "PreviousMasterKeys": [],
        "MaxConcurrentInputSize": 1024   // size in MB
      }
    },
    "DicomWeb": {
      "Enable": true,                
      "Root": "/dicom-web/",         
      "EnableWado": true,            
      "WadoRoot": "/wado",           
      "Ssl": false,                   
      "QidoCaseSensitive": true,     
      "Host": "",                    
      "StudiesMetadata": "Full",     
      "SeriesMetadata": "Full",      
      "EnableMetadataCache": true,    
      "MetadataWorkerThreadsCount": 4,     
      "PublicRoot": "/dicom-web/"    
    },
    "OHIF": {
      "DataSource": "dicom-web"
    },
    "OrthancExplorer2": {
      "Enable": true,
      "IsDefaultOrthancUI": true
    },
    "Plugins": [
      "/usr/share/orthanc/plugins-available/libOrthancPostgreSQLIndex.so",
      "/usr/share/orthanc/plugins-available/libOrthancAwsS3Storage.so",
      "/usr/share/orthanc/plugins-available/libOrthancOHIF.so",
      "/usr/share/orthanc/plugins-available/libOrthancDicomWeb.so",
      "/usr/share/orthanc/plugins-available/libOrthancExplorer2.so"
    ]
  }
```

## Orthanc Documentation

For more information about Orthanc and its features, visit the [Orthanc documentation](https://book.orthanc-server.com/).

## Contributing

If you have any suggestions or contributions to make to this Helm chart, feel free to create an issue or submit a pull request on the GitHub repository.


