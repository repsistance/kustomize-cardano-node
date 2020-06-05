* kapp deploy
```
NETWORK=ptn0
kustomize build overlays/${NETWORK} > overlays/${NETWORK}/output.yaml
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml
```
* kapp redeploy
```
NETWORK=ptn0
NODES=2
TYPE=passive
kubectl delete statefulset -n cardano-${NETWORK} ${TYPE}-node
for nodenum in $(seq 0 $(( NODES - 1 )))
do
  kubectl delete pvc -n cardano-${NETWORK} ${TYPE}-node-db-${TYPE}-node-${nodenum}
done
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml --filter-kind StatefulSet
```
* `kubectl logs --tail 100000 -n cardano-ptn0 leader-node-0 | tail -n +10 | jq -r .data.kind | sort | uniq | while read line; do jq -r .panels[].title /tmp/cardano-stakepools\ overview-1591323309975.json | sort | grep $line || echo "not - $line"; done`
