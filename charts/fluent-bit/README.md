# add log analytics connection data asa k8s secrets
# using az cli
kubectl create secret generic fluentbit-azure-secrets -n orc8r \
  --from-literal=WorkspaceId=$(az monitor log-analytics workspace show -g $LogAppRG -n $LogAppName --query customerId -o tsv) \
  --from-literal=SharedKey=$(az monitor log-analytics workspace get-shared-keys -g $LogAppRG -n $LogAppName --query primarySharedKey -o tsv)

# using strings
kubectl create secret generic fluentbit-azure-secrets -n orc8r \
  --from-literal=WorkspaceId=348f9c8b-a34b-45d9-ad64-f8ecea7d4cc5 \
  --from-literal=SharedKey=25H391QdmRLyTCW+vNsRyrgdvG+GQHHAv5fzQNpBbFsOkb+n/j/3G8Rhe0ZEEeXWm6tc8A7m7CkllcR80eCejQ==

# verify created secrets
kubectl get secret fluentbit-azure-secrets -n orc8r -o jsonpath='{.data}'