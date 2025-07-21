# Deploy Argocd on kubernetes cluster using helm chart

## Deploy Argocd on kubernetes cluster
```
•	helm repo list
•	helm repo add argo https://argoproj.github.io/argo-helm
•	helm repo list
•	helm repo update
•	kubectl create ns argocd
•	kubectl get ns
•	helm install argocd argo/argo-cd --namespace argocd
•	kubectl get all -n argocd
•	kubectl edit svc argocd-server -n argocd
•	kubectl get all -n argocd
•	kubectl get nodes -o wide
•	kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

```


# Delete Argocd
```
helm uninstall argocd --namespace argocd
```
