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

**Set the Apache Pod Name**

Modify `logs.yaml` and ensure the include file path is correct. By default, it is set to:

```yaml
receivers:
  filelog/apache:
    include:
      - /var/log/containers/httpd*.log
```

This means all container log files that start with `httpd` will be read. If you Apache pod's have a different name, be
sure to update it here.

**Deploy**

Deploy Apache HTTP with an observIQ collector as a side car. This configuration is an example

```bash
kubectl apply -f metrics.yaml
```

Deploy the logging daemonset

```bash
kubectl apply -f logs.yaml
```

You can view your logs with the following query
```
resource.labels.cluster_name="apache"
resource.type="k8s_container"
```

Metrics will show up under "Kubernetes Pod" in the metrics explorer.
