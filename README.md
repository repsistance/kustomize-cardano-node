* kapp deploy
```
NETWORK=ptn0
kustomize build overlays/${NETWORK} > overlays/${NETWORK}/output.yaml
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml
```
* kapp redeploy
```
NETWORK=ptn0
kustomize build overlays/${NETWORK} > overlays/${NETWORK}/output.yaml
LEADER_NODES=2
PASSIVE_NODES=2
TYPE=passive
kubectl scale statefulset -n cardano-${NETWORK} --replicas=0 ${TYPE}-node
for nodenum in $(seq 0 $(( PASSIVE_NODES - 1 )))
do
  kubectl delete pvc -n cardano-${NETWORK} ${TYPE}-node-db-${TYPE}-node-${nodenum}
done
kubectl scale statefulset -n cardano-${NETWORK} --replicas=${PASSIVE_NODES} ${TYPE}-node
TYPE=leader
kubectl scale statefulset -n cardano-${NETWORK} --replicas=0 ${TYPE}-node
for nodenum in $(seq 0 $(( LEADER_NODES - 1 )))
do
  kubectl delete pvc -n cardano-${NETWORK} ${TYPE}-node-db-${TYPE}-node-${nodenum}
done
kubectl scale statefulset -n cardano-${NETWORK} --replicas=${LEADER_NODES} ${TYPE}-node
kapp deploy -yc -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml --filter-kind ConfigMap
kapp deploy -yc -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml --filter-kind StatefulSet --filter-name passive-node
# register pool
# create pool keys secret
# kubectl create secret -n cardano-ptn0 generic pool-keys --from-file=hot.skey --from-file=vrf.skey --from-file=op.cert
```
* `kubectl logs --tail 100000 -n cardano-ptn0 leader-node-0 | tail -n +10 | jq -r .data.kind | sort | uniq | while read line; do jq -r .panels[].title /tmp/cardano-stakepools\ overview-1591323309975.json | sort | grep $line || echo "not - $line"; done`
