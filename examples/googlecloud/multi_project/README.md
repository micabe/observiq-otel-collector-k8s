# Google Cloud Multi Project Logging

This example can be used to ship logs from any Kubernetes cluster to multiple Google Cloud projects. The Open Telemetry Operator (and Cert Manager) are
not required.

Each unique namespace will be matched with a unique Google project.

## Usage

Update the configuration to reflect your environment.

**Authentication**

Deploy Google credentials:
```bash
kubectl create secret generic gcp-credentials \
    --from-file=credentials.json
```

The service account email should have `logs writer` role for all projects that you wish to send logs to.

**Set Cluster Name**

Update the cluster name environment variable. This is a "friendly" name that
can be searched on in the Cloud Logging interface.

```yaml
- name: K8S_CLUSTER
  value: eks
```

You can filter on the cluster name with this Cloud Logging query:

```
resource.labels.cluster_name="eks"
```

**Update Cluster Location (Optional)**

Google requires that a region and availability zone resource be set. This is part of the
[Monitored Resource Types](https://cloud.google.com/logging/docs/api/v2/resource-list) mapping.

Feel free to modify the following values, but understand that they must line up with Google supported values. A bad configuration
will result in logs coming in as "generic_node" resource type, instead of "k8s_container".

```yaml
resource/mapping:
  attributes:
    - key: cloud.region
      value: us-east1
      action: insert
    - key: cloud.availability_zone
      value: us-east1-b
      action: insert
    - key: cloud.platform
      value: gcp_kubernetes_engine
      action: insert
```

**Update Routing and Exporters**

The example configuration has a `routing` processor and a set of Google exporters.

Routing Processor:
```yaml
routing:
  attribute_source: resource
  from_attribute: k8s.namespace.name
  default_exporters:
  - logging
  table:
  - value: a
    exporters:
      - googlecloud/multi-project-logging-a
  - value: b
    exporters:
      - googlecloud/multi-project-logging-b
  - value: c
    exporters:
      - googlecloud/multi-project-logging-c
```

Matching Google exporters:
```yaml
googlecloud/multi-project-logging-a:
  project: multi-project-logging-a

googlecloud/multi-project-logging-b:
  project: multi-project-logging-b

googlecloud/multi-project-logging-c:
  project: multi-project-logging-c
```

1. Update the Google exporters to match your environment. You should have a single Google exporter per project you wish to send logs to.
2. Update the routing processor to map to the Google cloud projects. In this example, we are mapping based on namespace name.

**Deploy**

Deploy by applying daemonset.yaml with kubectl or your prefered tool.
