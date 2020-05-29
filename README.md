* kapp deploy
```
NETWORK=ptn0
kustomize build overlays/${NETWORK} > overlays/${NETWORK}/output.yaml
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml
```
* kapp redeploy
```
NETWORK=fftn
NODES=2
TYPE=passive
# flush genesis, topology and db
for nodenum in $(seq 0 $(( NODES - 1 )))
do
  kubectl iexec -n cardano-${NETWORK} ${TYPE}-node-${nodenum} -- bash -c 'rm /opt/cardano/cnode/files/genesis.json; rm /opt/cardano/cnode/files/topology.json'
done
kubectl scale statefulset -n cardano-${NETWORK} --replicas=0 passive-node
for nodenum in $(seq 0 $(( NODES - 1 )))
do
  kubectl delete pvc -n cardano-${NETWORK} ${TYPE}-node-db-${TYPE}-node-${nodenum}
done
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${NETWORK}/output.yaml --filter-kind StatefulSet
```
* Enter container (in e.g. to modify and test config)
```
NETWORK=ptn0
kubectl iexec -n cardano-${NETWORK} passive-node bash
```
* Change files
```
kubectl cp -n cardano-ptn0 passive-node-8b84b999f-8tnjv:/opt/cardano/cnode/files/configuration.yaml configuration.yaml
# edit file
kubectl cp -n cardano-fftn configuration.yaml passive-node-6b766564f6-6kc92:/opt/cardano/cnode/files/configuration.yaml
# restart pod:
kubectl delete pod -n cardano-fftn passive-node-6b766564f6-6kc92
```
* Live cardano-node logs:
```
logcli query --no-labels -o raw -qt '{namespace=cardano-ptn0}' | sed 's|.*stdout F ||'
kubectl tail -n cardano-ptn0 -l cardano_node_type=passive
```
