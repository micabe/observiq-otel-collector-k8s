# Apache HTTPD Metrics and Logs with Google Cloud Logging

This example can be used to ship Apache HTTPD metrics and logs from any Kubernetes cluster to Google Cloud.

## Usage

Update the configuration to reflect your environment.

**Authentication**

If running **outside of GCP**, deploy Google credentials:
```bash
kubectl create secret generic gcp-credentials \
    --from-file=credentials.json
```

If running within GCP with the correct instance scopes enabled, comment the authentication section in `environments/googlecloud/agent_gateway.yaml`.

**Set Cluster Name**

Update the cluster name environment variables. Because cluster name cannot be detected, it must be set
manually. Edit `logs.yaml` and `metrics.yaml`. Cluster name is required for metrics but not logs, however, it is still useful
for logging as it will allowing you to query for Apache logs on a per cluster basis.

```yaml
- name: OTEL_RESOURCE_ATTRIBUTES_CLUSTER_NAME
  value: apache
```

You can filter on the cluster name with this Cloud Logging query:

```
resource.labels.cluster_name="apache"
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
