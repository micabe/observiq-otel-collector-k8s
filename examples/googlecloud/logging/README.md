# Google Cloud Logging

This example can be used to ship logs from any Kubernetes cluster to Google Cloud. The Open Telemetry Operator (and Cert Manager) are
not required.

## Usage

Update the configuration to reflect your environment.

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

