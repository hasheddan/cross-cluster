# Cross-Cluster

Using Crossplane to deploy an application across regions.

## Building

1. Build and push `cross-cluster` Configuration

```
kubectl crossplane build configuration
kubectl crossplane push configuration hasheddan/cross-cluster:v0.0.1
```

## Deploying

1. Install Crossplane

```
kubectl create namespace crossplane-system

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```

2. Install `cross-cluster` Configuration

```
kubectl crossplane install configuration hasheddan/cross-cluster:v0.0.1
```

3. Setup GCP Provider Credentials

```
kubectl create secret generic gcp-creds -n crossplane-system --from-file=key=./gcp-creds.json
PROJECT_ID=crossplane-playground envsubst < ./gcp-provider-config.yaml | kubectl apply -f -
```

4. Create east and west clusters

```
kubectl apply -f examples/cluster-east.yaml
kubectl apply -f examples/cluster-west.yaml
```

5. Create east and west apps

```
kubectl apply -f examples/app-east.yaml
kubectl apply -f examples/app-west.yaml
```

7. Get kubeconfigs

```
kubectl get secret kubeconfig-east -n crossplane-system -o=jsonpath={.data.kubeconfig} | base64 --decode > east.kube
kubectl get secret kubeconfig-west -n crossplane-system -o=jsonpath={.data.kubeconfig} | base64 --decode > west.kube
```

8. View apps

```
kubectl port-forward new-doc-7cb5656cfd-gdh2x 8081:5000 --kubeconfig=east.kube
kubectl port-forward new-doc-549ccd5d78-vbthd 8082:5000 --kubeconfig=west.kube
```
