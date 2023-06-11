# RBAC with Minikube

## Where Minikube store CA
```
ls -ltr ~/.minikube
```

in Kubernetes cluster, it stores in control-plane node
```
ls -ltr /etc/kubernetes/pki
```

## Create User Certificate
### Create private key
```
openssl genrsa -out <username>.key 2048
```
### Create certificate signing request (CSR) by using private key
```
openssl req -new -key <username>.key -out <username>.csr -subj "/CN=<username>/O=<namespace>"
```
### Create certificate by using Minikube CA
```
openssl x509 -req -in <username>.csr -CA /path/to/minikube/ca.crt -CAkey /path/to/minikube/ca.key -CAcreateserial -out <username>.crt -days 365
```

## Create Kube Config
```
export KUBECONFIG=~/.kube/new-config

# can find Kube Server URL in ~/.kube/config --> cat ~/.kube/config

kubectl config set-cluster dev-cluster --server=<Kube Server URL> \
--certificate-authority=ca.crt \
--embed-certs=true
```

## Login Minikube with user
```
kubectl config set-credentials <username> --client-certificate=<username>.crt --client-key=<username>.key --embed-certs=true

kubectl config set-context dev --cluster=dev-cluster --namespace=<namespace> --user=<username>

kubectl config use-context dev
```

## Give User Access
```
# On new Terminal (Open new terminal & run it)

kubectl create ns <namespace>

kubectl -n <namespace> apply -f .\role.yaml
kubectl -n <namespace> apply -f .\rolebinding.yaml
```

## Create Service Accounts
```
kubectl -n shopping apply -f serviceaccount.yaml

kubectl -n shopping apply -f pod.yaml

kubectl -n shopping apply -f serviceaccount-role.yaml

kubectl -n shopping apply -f serviceaccount-rolebinding.yaml
```

## TEST Service account
```
kubectl -n <namespace> exec -it <container_name> -- bash

token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

curl --cacert /path/to/minikube/ca.crt --header "Authorization: Bearer $token" https://kubernetes.defult.svc/api/v1/namespaces/<namespace>/pods/
```
