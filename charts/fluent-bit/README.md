# Fluent Bit Helm chart

[Fluent Bit](https://fluentbit.io) is a fast and lightweight log processor and forwarder or Linux, OSX and BSD family operating systems.

## Installation

To add the `fluent` helm repo, run:

```sh
helm repo add fluent https://fluent.github.io/helm-charts
```

To install a release named `fluent-bit`, run:

```sh
helm install fluent-bit fluent/fluent-bit
```

## Chart values

```sh
helm show values fluent/fluent-bit
```

## Using Lua scripts
Fluent Bit allows us to build filter to modify the incoming records using custom [Lua scripts.](https://docs.fluentbit.io/manual/pipeline/filters/lua)

### How to use Lua scripts with this Chart

First, you should add your Lua scripts to `luaScripts` in values.yaml, for example:

```yaml
luaScripts:
  filter_example.lua: |
    function filter_name(tag, timestamp, record)
        -- put your lua code here.
    end
```

After that, the Lua scripts will be ready to be used as filters. So next step is to add your Fluent bit [filter](https://docs.fluentbit.io/manual/concepts/data-pipeline/filter) to `config.filters` in values.yaml, for example:

```yaml
config:
  filters: |
    [FILTER]
        Name    lua
        Match   <your-tag>
        script  /fluent-bit/scripts/filter_example.lua
        call    filter_name
```
Under the hood, the chart will:
- Create a configmap using `luaScripts`.
- Add a volumeMounts for each Lua scripts using the path `/fluent-bit/scripts/<script>`.
- Add the Lua script's configmap as volume to the pod.

### Note
Remember to set the `script` attribute in the filter using `/fluent-bit/scripts/`, otherwise the file will not be found by fluent bit.


# add log analytics connection data asa k8s secrets
# using az cli
kubectl create secret generic fluentbit-azure-secrets -n orc8r \
  --from-literal=WorkspaceId=$(az monitor log-analytics workspace show -g $LogAppRG -n $LogAppName --query customerId -o tsv) \
  --from-literal=SharedKey=$(az monitor log-analytics workspace get-shared-keys -g $LogAppRG -n $LogAppName --query primarySharedKey -o tsv)

# using strings
kubectl create secret generic fluentbit-azure-secrets -n kube-system \
  --from-literal=WorkspaceId=348f9c8b-a34b-45d9-ad64-f8ecea7d4cc5 \
  --from-literal=SharedKey=25H391QdmRLyTCW+vNsRyrgdvG+GQHHAv5fzQNpBbFsOkb+n/j/3G8Rhe0ZEEeXWm6tc8A7m7CkllcR80eCejQ==

# verify created secrets
kubectl get secret fluentbit-azure-secrets -n orc8r -o jsonpath='{.data}'

env.name                            WorkspaceId
env.name[0].valueFrom.secretKeyRef.name     fluentbit-azure-secrets
env.name[0].valueFrom.secretKeyRef.key      WorkspaceId

env.name[1].name                            SharedKey
env.name[1].valueFrom.secretKeyRef.name     fluentbit-azure-secrets
env.name[1].valueFrom.secretKeyRef.key      SharedKey

env:
- name: WorkspaceId
    valueFrom:
    secretKeyRef:
        name: fluentbit-azure-secrets
        key: WorkspaceId

- name: SharedKey
    valueFrom:
    secretKeyRef:
        name: fluentbit-azure-secrets
        key: SharedKey


# adjust kernel settings
sudo sysctl -w fs.inotify.max_queued_events=50000
sudo sysctl -w fs.inotify.max_user_instances=16383
sudo sysctl -w  fs.inotify.max_user_watches=50000


config:
    outputs: |
        [OUTPUT]
            Name azure
            Match kube.*
            Customer_ID     ${WorkspaceId}
            Shared_Key      ${SharedKey}
            Log_Type        ${LogName}

        [OUTPUT]
            Name azure
            Match host.*
            Customer_ID     ${WorkspaceId}
            Shared_Key      ${SharedKey}
            Log_Type        ${LogName}
            Retry_Limit False