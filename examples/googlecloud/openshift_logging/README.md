**Create Project**

```bash
oc create -f project.yaml
```

**Create Google Cloud Credential Secret**

Create a Google service account and grant it `logs writer` role. Download an application json key and name it `credentials.json` (The file name is important, as that is what will be referenced by the daemonset volume mount).

```bash
oc -n observiq create secret generic gcp-credentials \
    --from-file=credentials.json
```

**Apply Setup**

Create the Security Context Constraints and RBAC rules.

```bash
oc apply -f setup.yaml
```

**Create Configuration**

Two configuration options are supported for reading container logs. Choose the option that matches
your cluster's configured log driver. OpenShift should use `file` by default.

- File (docker `json-file` and containerd `text-file`)
- Journald

```bash
oc apply -f config/file/config.yaml
# OR
oc apply -f config/journald/config.yaml
```

**Deploy Collector Daemonset**

1. Open the file `observiq-otel-collector.yaml` with your editor of choice

2. Modify the cluster name by finding and replacing `minishift`. This is a friendly name that will allow you to search between multiple clusters.
```yaml
- name: K8S_CLUSTER
  value: minishift
```

3. Deploy the collector.
```bash
oc apply -f observiq-otel-collector.yaml
```
