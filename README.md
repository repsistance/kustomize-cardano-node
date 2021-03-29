# Deployment

## Prerequisites

- docker
- k8s
- kustomize
- kapp

* testnet deploy
```
NETWORK=iohk-tn
NAMESPACE=cardano-${NETWORK}
OVERLAY=local-${NETWORK}
kubectl get ns ${NAMESPACE} || kubectl create ns ${NAMESPACE}
kustomize build overlays/${OVERLAY} > overlays/${OVERLAY}/output.yaml
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${OVERLAY}/output.yaml
```
* mainnet deploy
```
NETWORK=iohk-mn
NAMESPACE=cardano-${NETWORK}
OVERLAY=local-${NETWORK}
kubectl get ns ${NAMESPACE} || kubectl create ns ${NAMESPACE}
kustomize build overlays/${OVERLAY} > overlays/${OVERLAY}/output.yaml
kapp deploy -c -a cardano-${NETWORK} -n cardano-${NETWORK} -f overlays/${OVERLAY}/output.yaml
```
* Upload pool keys (generated, ie, with [cntools])
```
kubectl create secret -n ${NAMESPACE} generic pool-keys \
  --from-file=hot.skey \
  --from-file=vrf.skey \
  --from-file=op.cert
```


## Setup local k3d

```
CARDANO_NODE_PORT=31337
CLUSTER_NAME=cardano-iohk-testnet

k3d cluster create \
  -p "${CARDANO_NODE_PORT}:${CARDANO_NODE_PORT}@server[0]" \
  ${CLUSTER_NAME}
kubectl config use-context k3d-${CLUSTER_NAME}
kubectl get pods -A
```

