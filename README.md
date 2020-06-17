* kapp deploy
```
NETWORK=ptn0
kustomize build overlays/${NETWORK} > overlays/${NETWORK}/output.yaml
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml
```
* kapp redeploy
```
kustomize build overlays/${NETWORK} > overlays/${NETWORK}/output.yaml
NETWORK=ptn0
NODES=2
TYPE=leader
kubectl delete statefulset -n cardano-${NETWORK} ${TYPE}-node
kubectl delete secret -n cardano-ptn0 pool-keys
kubectl delete pvc -n cardano-${NETWORK} ${TYPE}-node-db-${TYPE}-node-0
TYPE=passive
kubectl delete statefulset -n cardano-${NETWORK} ${TYPE}-node
for nodenum in $(seq 0 $(( NODES - 1 )))
do
  kubectl delete pvc -n cardano-${NETWORK} ${TYPE}-node-db-${TYPE}-node-${nodenum}
done
kapp deploy -yc -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml --filter-kind ConfigMap
kapp deploy -yc -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml --filter-kind StatefulSet --filter-name passive-node
# register pool
# create pool keys secret
# kubectl create secret -n cardano-ptn0 generic pool-keys --from-file=hot.skey --from-file=vrf.skey --from-file=op.cert
```
* `kubectl logs --tail 100000 -n cardano-ptn0 leader-node-0 | tail -n +10 | jq -r .data.kind | sort | uniq | while read line; do jq -r .panels[].title /tmp/cardano-stakepools\ overview-1591323309975.json | sort | grep $line || echo "not - $line"; done`
