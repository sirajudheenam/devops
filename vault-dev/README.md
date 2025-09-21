# Vault Development Environment

```bash
export VAULT_DEV_ROOT_TOKEN_ID=dev-only-token
export VAULT_ADDR="http://127.0.0.1:8200"
mkdir -p ~/vault-data
minikube status
minikube stop
# If you don't have anything running on minikube
minikube delete
# To store data in macOS Disk
minikube start --mount --mount-string="${HOME}/vault-data:/data"
kubectl --context minikube apply -f k8s/vault-storage.yaml
kubectl --context minikube apply -f k8s/vault-config.yaml
kubectl --context minikube apply -f k8s/vault-deployment.yaml
minikube service vault
kubectl --context minikube delete -f k8s
kubectl --context minikube logs deploy/vault
kubectl --context minikube exec -it deploy/vault -- vault operator init
kubectl --context minikube exec -it deploy/vault -- vault operator unseal
kubectl --context minikube exec -it deploy/vault -- /bin/sh
kubectl --context minikube rollout history deploy/vault
kubectl --context minikube rollout restart deploy/vault
kubectl --context minikube logs deploy/vault
kubectl --context minikube exec -it deploy/vault  -- /bin/sh
# on different window - do port forwarding from your k8s cluster to localhost to test
kubectl --context minikube port-forward deploy/vault 8200:8200
kubectl logs deploy/vault
env | grep VAULT
vault status

minikube status
kubectl cluster-info
kubectl apply -f k8s-dev-host-storage/
vault login
# PASTE ROOT TOKEN

```

## Further Reading

[Official Doc](https://developer.hashicorp.com/vault/docs)
[GitHub](https://github.com/hashicorp/vault)
[Container Images](https://hub.docker.com/r/hashicorp/vault)
[GoLang Example](https://github.com/hashicorp/hello-vault-go)