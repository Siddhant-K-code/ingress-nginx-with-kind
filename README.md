# Running Kubernetes with Devcontainer (`ingress-nginx` with `kind`)

## Setup

### Create cluster

```sh
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

### Introduce `ingress-nginx`

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

_Wait until ingress-nginx is ready..._

```sh
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=180s
```

### Create sample content

```sh
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/usage.yaml
```

## Demo

Verify that `ingress-nginx` is running

```sh
curl localhost/foo/hostname
```

> Output: `foo-app`

```sh
curl localhost/bar/hostname
```

> Output: `bar-app`
