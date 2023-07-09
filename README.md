# ArgoCD Example Apps

This repository contains example applications for demoing ArgoCD functionality. Feel free
to register this repository to your ArgoCD instance, or fork this repo and push your own commits
to explore ArgoCD and GitOps!

| Application | Description |
|-------------|-------------|
| [guestbook](guestbook/) | A hello word guestbook app as plain YAML |
| [ksonnet-guestbook](ksonnet-guestbook/) | The guestbook app as a ksonnet app |
| [helm-guestbook](helm-guestbook/) | The guestbook app as a Helm chart |
| [jsonnet-guestbook](jsonnet-guestbook/) | The guestbook app as a raw jsonnet |
| [jsonnet-guestbook-tla](jsonnet-guestbook-tla/) | The guestbook app as a raw jsonnet with support for top level arguments |
| [kustomize-guestbook](kustomize-guestbook/) | The guestbook app as a Kustomize 2 app |
| [pre-post-sync](pre-post-sync/) | Demonstrates Argo CD PreSync and PostSync hooks |
| [sync-waves](sync-waves/) | Demonstrates Argo CD sync waves with hooks |
| [helm-dependency](helm-dependency/) | Demonstrates how to customize an OTS (off-the-shelf) helm chart from an upstream repo |
| [sock-shop](sock-shop/) | A microservices demo app (https://microservices-demo.github.io) |
| [plugins](plugins/) | Apps which demonstrate config management plugins usage |
| [blue-green](blue-green/) | Demonstrates how to implement blue-green deployment using [Argo Rollouts](https://github.com/argoproj/argo-rollouts)
| [apps](apps/) | An app composed of other apps |

# ArgoCD Notes

ArgoCD - Minikube

minikube start --driver=docker

kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.5.8/manifests/install.yaml

## Verify and wait for everything to be up and running
 watch -n .2 "kubectl get pods -n argocd"

## Once all pods a running, verify all other resources are good
kubectl get all -n argocd

## Ensure we can reach argocd from outside cluster
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address 0.0.0.0
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --reload

## Get initial password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

## Login via host 0.0.0.0:8080
https://192.168.32.130:8080/
username: admin
password: secret from previous command

## Go update the password
https://192.168.32.130:8080/user-info?changePassword=true

## Update DNS of cluster if you have problems with Argo resolving public addresses (Github)
kubectl edit configmap coredns -n kube-system
	forward . 8.8.8.8 8.8.4.4
	#forward . /etc/resolv.conf {
	#   max_concurrent 1000
	#}
	
argocd login 192.168.32.130:8080 --insecure --username admin

## Create second minikube cluster
minikube start -p cluster2

## switch to cluster-2
minikube profile cluster2

## Add remote cluster to argocd
argocd cluster add cluster-2

## Delete all Argo ApplicationSets
for i in $(kubectl get applicationsets.argoproj.io -n argocd -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'); do echo $i; kubectl delete applicationsets.argoproj.io -n argocd $i; done

## Delete all Argo Applications
for i in $(kubectl get applications.argoproj.io -n argocd -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'); do echo $i; kubectl delete applications.argoproj.io -n argocd $i; done

