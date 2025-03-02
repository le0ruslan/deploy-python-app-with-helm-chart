# Deploy Golang App with Helm chart

## Project Description
This repository implements the deployment of the application using Kind and Helm.

## Installing kind

To install kind, follow the official installation guide from kind's 
[website](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)

1. Download the kind binary (for AMD64 / x86_64 architecture)

```
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-amd64

```

2. Make the file executable

```
chmod +x ./kind
```

3. Move kind to /usr/local/bin/kind

```
sudo mv ./kind /usr/local/bin/kind
```

4. Check kind version

```
kind --version
```

5. Install the [cloud provider](https://github.com/kubernetes-sigs/cloud-provider-kind/tree/main) for kind (required for Ingress functionality)

```
go install sigs.k8s.io/cloud-provider-kind@latest
```


## Deploying the Application

1. Create the cluster:

```
sudo kind create cluster --config .deploy/k8s/kind/kind-cluster.yml
```

2. Create the Namespace:
```
sudo kubectl apply -f .deploy/k8s/manifests/namespace.yaml3.
```

3. Apply the Secret:
```
sudo kubectl apply -f .deploy/k8s/helm/postgresql/secret-postgres.yaml
```

4. Install the Bitnami PostgreSQL chart:
```
sudo helm install bitnami-postgres-01 oci://registry-1.docker.io/bitnamicharts/postgresql \
  --namespace workshop2025 -f .deploy/k8s/helm/postgresql/values.yaml
```
5. Deploy the Backend chart:
```
sudo helm upgrade --install backend .deploy/k8s/helm/backend \
  --namespace workshop2025 -f .deploy/k8s/helm/backend/values.yaml
```
6. Install the Ingress Controller:
```bash    
    sudo kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

7. Deploy the Frontend chart:
```
sudo helm upgrade --install frontend .deploy/k8s/helm/frontend \
  --namespace workshop2025 -f .deploy/k8s/helm/frontend/values.yaml
```


## Verifying the Application


* Navigate to the cloud provider kind directory, which was installed in step 5 of the "Installing kind" section, and run it:

```
./cloud-provider-kind 
```

* In another terminal, retrieve the external IP address assigned by the Ingress:

```
sudo kubectl get svc frontend-service -n workshop2025 -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

* Open the application in a browser by navigating to:

```
http://{external-ip}/
```
